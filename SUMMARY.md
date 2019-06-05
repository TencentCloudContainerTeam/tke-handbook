# Summary

- [简介](README.md)

### TKE 奇淫技巧

- [利用 NAT 网关实现无公网 IP 机器访问外网](tke-skill/using-nat-gateway-visit-internet.md)
- [在 TKE 中安装 metrics-server](tke-skill/install-metrics-server-on-tke.md)

### TKE 使用相关

- [挂载的nfs为何是10G](tke/why-nfs-10g.md)

### k8s问题排查

- [Pod 状态一直 Terminating](troubleshooting/pod-terminating-forever.md)
- [Pod 状态一直 ContainerCreating](troubleshooting/pod-containercreating-forever.md)
- [节点 NotReady](troubleshooting/node-notready.md)
- [Service 无法解析](troubleshooting/service-cannot-resolve.md)
- [LB 显示异常](troubleshooting/lb-abnormal.md)
- [Job 无法被删除](troubleshooting/cannot-delete-job.md)

### k8s问题定位技巧

- [分析 ExitCode 定位程序退出原因](skill/analysis-exitcode.md)
- [容器内抓包定位网络问题](skill/capture-packets-in-container.md)
- [分析docker磁盘占用](skill/analysis-docker-disk.md)

### k8s最佳实践

- [优雅热更新](best-practice/kubernetes-best-practice-grace-update.md)
- [处理容器数据磁盘被写满](best-practice/kubernetes-best-practice-handle-disk-full.md)
- [kubectl 高效技巧](best-practice/efficient-kubectl.md)
- [泛域名动态 Service 转发解决方案](best-practice/wildcard-domain-forward.md)
- [处理内存碎片化](best-practice/handle-memory-fragmentation.md)