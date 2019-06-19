# 部分 DNS 查询延迟的原因与解决方案

## 超时问题
客户反馈从pod中访问服务时，总是有些请求的响应时延会达到5秒。正常的响应只需要毫秒级别的时延。

## DNS 5秒延时
在pod中(通过nsenter -n tcpdump)抓包，发现是有的DNS请求没有收到响应，超时5秒后，再次发送DNS请求才成功收到响应。

在kube-dns pod抓包，发现是有DNS请求没有到达kube-dns pod， 在中途被丢弃了。

为什么是5秒？ `man resolv.conf`可以看到glibc的resolver的缺省超时时间是5s。

## 丢包原因
经过搜索发现这是一个普遍问题。
根本原因是内核conntrack模块的bug，netfilter做NAT时可能发生资源竞争导致部分报文丢弃。

Weave works的工程师[Martynas Pumputis](martynas@weave.works)对这个问题做了很详细的分析：
https://www.weave.works/blog/racy-conntrack-and-dns-lookup-timeouts

相关结论：
- 只有多个线程或进程，并发从同一个socket发送相同五元组的UDP报文时，才有一定概率会发生
- glibc, musl(alpine linux的libc库)都使用"parallel query", 就是并发发出多个查询请求，因此很容易碰到这样的冲突，造成查询请求被丢弃
- 由于ipvs也使用了conntrack, 使用kube-proxy的ipvs模式，并不能避免这个问题

## 问题的根本解决

Martynas向内核提交了两个patch来fix这个问题，不过他说如果集群中有多个DNS server的情况下，问题并没有完全解决。

其中一个patch已经在2018-7-18被合并到linux内核主线中: [netfilter: nf_conntrack: resolve clash for matching conntracks](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=ed07d9a021df6da53456663a76999189badc432a)

目前只有4.19.rc 版本包含这个patch。

## 规避办法
#### 规避方案一：使用TCP发送DNS请求
由于TCP没有这个问题，有人提出可以在容器的resolv.conf中增加`options use-vc`, 强制glibc使用TCP协议发送DNS query。下面是这个man resolv.conf中关于这个选项的说明：

```
use-vc (since glibc 2.14)
                     Sets RES_USEVC in _res.options.  This option forces the
                     use of TCP for DNS resolutions.
```
笔者使用镜像"busybox:1.29.3-glibc" (libc 2.24)  做了试验，并没有见到这样的效果，容器仍然是通过UDP发送DNS请求。

#### 规避方案二：避免相同五元组DNS请求的并发
resolv.conf还有另外两个相关的参数： 
- single-request-reopen (since glibc 2.9)
- single-request (since glibc 2.10)

man resolv.conf中解释如下：

```
single-request-reopen (since glibc 2.9)
                     Sets RES_SNGLKUPREOP in _res.options.  The resolver
                     uses the same socket for the A and AAAA requests.  Some
                     hardware mistakenly sends back only one reply.  When
                     that happens the client system will sit and wait for
                     the second reply.  Turning this option on changes this
                     behavior so that if two requests from the same port are
                     not handled correctly it will close the socket and open
                     a new one before sending the second request.
                     
single-request (since glibc 2.10)
                     Sets RES_SNGLKUP in _res.options.  By default, glibc
                     performs IPv4 and IPv6 lookups in parallel since
                     version 2.9.  Some appliance DNS servers cannot handle
                     these queries properly and make the requests time out.
                     This option disables the behavior and makes glibc
                     perform the IPv6 and IPv4 requests sequentially (at the
                     cost of some slowdown of the resolving process).
```

笔者做了试验，发现效果是这样的：
- single-request-reopen
发送A类型请求和AAAA类型请求使用不同的源端口。这样两个请求在conntrack表中不占用同一个表项，从而避免冲突。
- single-request
避免并发，改为串行发送A类型和AAAA类型请求。没有了并发，从而也避免了冲突。


要给容器的resolv.conf加上options参数，有几个办法：
##### 1) 在容器的"ENTRYPOINT"或者"CMD"脚本中，执行`/bin/echo 'options single-request-reopen' >> /etc/resolv.conf`
##### 2) 在pod的postStart hook中：
```
        lifecycle:
          postStart:
            exec:
              command:
              - /bin/sh
              - -c 
              - "/bin/echo 'options single-request-reopen' >> /etc/resolv.conf"
```
##### 3) 使用template.spec.dnsConfig (k8s v1.9 及以上才支持):
```
  template:
    spec:
      dnsConfig:
        options:
          - name: single-request-reopen
```

##### 4) 使用ConfigMap覆盖POD里面的/etc/resolv.conf
configmap:
```
apiVersion: v1
data:
  resolv.conf: |
    nameserver 1.2.3.4
    search default.svc.cluster.local svc.cluster.local cluster.local ec2.internal
    options ndots:5 single-request-reopen timeout:1
kind: ConfigMap
metadata:
  name: resolvconf
```

POD spec:
```
        volumeMounts:
        - name: resolv-conf
          mountPath: /etc/resolv.conf
          subPath: resolv.conf
...

      volumes:
      - name: resolv-conf
        configMap:
          name: resolvconf
          items:
          - key: resolv.conf
            path: resolv.conf
```

##### 5) 使用MutatingAdmissionWebhook
[MutatingAdmissionWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook-beta-in-1-9) 是1.9引入的Controller，用于对一个指定的Resource的操作之前，对这个resource进行变更。
istio的自动sidecar注入就是用这个功能来实现的。 我们也可以通过MutatingAdmissionWebhook，来自动给所有POD，注入以上3)或者4)所需要的相关内容。

---

以上方法中， 1)和2)都需要修改镜像， 3)和4)则只需要修改POD的spec， 能适用于所有镜像。不过还是有不方便的地方：
- 每个工作负载的yaml都要做修改，比较麻烦
- 对于通过helm创建的工作负载，需要修改helm charts

方法5)对集群使用者最省事，照常提交工作负载即可。不过初期需要一定的开发工作量。

#### 规避方案三：使用本地DNS缓存
容器的DNS请求都发往本地的DNS缓存服务(dnsmasq, nscd等)，不需要走DNAT，也不会发生conntrack冲突。另外还有个好处，就是避免DNS服务成为性能瓶颈。

使用本地DNS缓存有两种方式：
- 每个容器自带一个DNS缓存服务
- 每个节点运行一个DNS缓存服务，所有容器都把本节点的DNS缓存作为自己的nameserver

从资源效率的角度来考虑的话，推荐后一种方式。

##### 实施办法
条条大路通罗马，不管怎么做，最终到达上面描述的效果即可。

POD中要访问节点上的DNS缓存服务，可以使用节点的IP。 如果节点上的容器都连在一个虚拟bridge上， 也可以使用这个bridge的三层接口的IP(在TKE中，这个三层接口叫cbr0)。 要确保DNS缓存服务监听这个地址。

如何把POD的/etc/resolv.conf中的nameserver设置为节点IP呢？

一个办法，是设置 POD.spec.dnsPolicy 为 "Default"， 意思是POD里面的 /etc/resolv.conf， 使用节点上的文件。缺省使用节点上的 /etc/resolv.conf(如果kubelet通过参数--resolv-conf指定了其他文件，则使用--resolv-conf所指定的文件)。

另一个办法，是给每个节点的kubelet指定不同的--cluster-dns参数，设置为节点的IP，POD.spec.dnsPolicy仍然使用缺省值"ClusterFirst"。 kops项目甚至有个issue在讨论如何在部署集群时设置好--cluster-dns指向节点IP: https://github.com/kubernetes/kops/issues/5584


## 参考资料

- Racy conntrack and DNS lookup timeouts: https://www.weave.works/blog/racy-conntrack-and-dns-lookup-timeouts
- 5 – 15s DNS lookups on Kubernetes? :  https://blog.quentin-machu.fr/2018/06/24/5-15s-dns-lookups-on-kubernetes/
- DNS intermittent delays of 5s: https://github.com/kubernetes/kubernetes/issues/56903
- 记一次Docker/Kubernetes上无法解释的连接超时原因探寻之旅: https://mp.weixin.qq.com/s/VYBs8iqf0HsNg9WAxktzYQ