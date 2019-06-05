# 集群内无法访问外网

首先确保 node 能上外网(node 本身有公网还是走 NAT 网关)，如果在node上都不能访问外网，则集群内也无法访问

TKE 1.10 以上的集群在 `kube-system` 下有 `ip-masq-agent` 的 deamonset，作用是将出公网的请求的源 ip snat 成 node 的 ip，这样数据包才能出公网，确保 node 上有 `ip-masq-agent` 的 pod:
``` bash
kubectl get pod -n kube-system -o wide
```