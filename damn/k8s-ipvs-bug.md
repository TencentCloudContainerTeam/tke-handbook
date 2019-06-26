# K8S 对 IPVS 支持的 BUG

## 使用 hostPort 导致所在节点的pod访问不到同端口号的 service

启用 ipvs 的集群，如果某 pod 使用 hostPort 监听端口，那么这个 pod 所在的节点访问跟 hostPort 同端口号的任何 Service 都会被路由到这个 hostPort 上，而不会正确路由到 Service 对应的后端 pod

- 原因：ipvs 和 iptables 都会向 netfilter 的 hook 点插入处理函数，由于 hostPort 是通过 iptables 实现的，所以当某节点的 pod 使用了 hostPort 暴露端口，就会写入 iptables 规则，让目标端口为该 hostPort 端口号的报文全部路由到该 pod；当启用 ipvs 来路由 service 时会写入 ipvs 规则，但写入的 ipvs 规则和 iptables 规则进入 netfilter 的 hook 点时，是有顺序的，netfilter 会先按顺序执行这些 hook 函数，iptables 规则的 hook 先被执行就会导致这个现象：该节点 pod 访问某个 service 的某个端口号，如果这个端口号跟 hostPort 一致，数据包就会被路由到 hostPort 对应的 pod 里

- 规避方案：用其它不跟任何 service 端口冲突的端口作为 hostPort 来绕开此问题

- Github 相关 Issue：
  - https://github.com/kubernetes/kubernetes/issues/66103
  - https://github.com/kubernetes/kubernetes/issues/60688

- 暂未验证新版是否彻底修复此 bug，感兴趣的同学可以验证下提 PR 到这里说明下

## 无法使⽤ localhost+nodeport

启用ipvs的情况下，使用localhost无法访问本机nodeport，这是一个已知bug，k8s本身也不会这样用，通常只有测试访问本机nodeport才会遇到这个问题，不影响生产环境的服务

相关issue: https://github.com/kubernetes/kubernetes/issues/67730


## 所有节点的kube-proxy不能为新的service创建路由规则
- 现象：是 kube-proxy 的 Netlink deadlock 的 bug 导致
- issue: https://github.com/kubernetes/kubernetes/issues/71071
- 1.14 版本已修复，修复的 PR: https://github.com/kubernetes/kubernetes/pull/72361