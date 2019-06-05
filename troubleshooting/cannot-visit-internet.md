# 集群内无法访问外网

首先确保 node 能上外网(node 本身有公网还是走 NAT 网关)，如果在node上都不能访问外网，则集群内也无法访问

TKE 1.10 以上的集群在 `kube-system` 下有 `ip-masq-agent` 的 Deamonset，作用是将出公网的请求的源 ip snat 成 node 的 ip，这样数据包才能出公网，确保 node 上有 `ip-masq-agent` 的 pod:
``` bash
kubectl get pod -n kube-system -o wide
```

- 如果发现完全没有，请提工单
- 如果发现有，但部分没有，检查下节点是否设置了污点(taint)，导致 Deamonset 的 pod 无法被调度上来
- 如果发现都有，还是无法访问公网，检查下 ip-masq-agent 的 配置:
  ``` bash
  kubectl -n kube-system get cm ip-masq-agent-config -o yaml
  ```
  - `nonMasqueradeCIDRs` 里的网段表示对这些网段不做snat，出公网需要snat，看是否设置了类似 `0.0.0.0/0` 的网段导致出公网的数据包也没做snat
