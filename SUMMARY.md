# Summary

- [序](README.md)

### TKE 奇淫技巧

- [利用 NAT 网关实现无公网 IP 机器访问外网](tke-skill/using-nat-gateway-visit-internet.md)
- [在 TKE 中安装 metrics-server](tke-skill/install-metrics-server-on-tke.md)
- [自定义节点的操作系统]()

### TKE 使用相关

- [挂载的nfs为何是10G](tke/why-nfs-10g.md)
- [为什么 controller-manager 和 scheduler 状态显示 Unhealthy](tke/why-controller-manager-and-scheduler-unhealthy.md)

### k8s问题排查

- [Pod 状态一直 Terminating](troubleshooting/pod-terminating-forever.md)
- [Pod 状态一直 ContainerCreating](troubleshooting/pod-containercreating-forever.md)
- [Pod 状态一直 Pending](troubleshooting/pod-pending-forever.md)
- [节点 NotReady](troubleshooting/node-notready.md)
- [Service 无法解析](troubleshooting/service-cannot-resolve.md)
- [LB 显示异常](troubleshooting/lb-abnormal.md)
- [集群内无法访问外网](troubleshooting/cannot-visit-internet.md)
- [无法访问集群外的服务](troubleshooting/cannot-visit-service-out-of-cluster.md)
- [Pod 无法被 exec 和 logs](troubleshooting/pod-cannot-exec-or-logs.md)
- [Job 无法被删除](troubleshooting/cannot-delete-job.md)
- [服务不能被访问](troubleshooting/service-cannot-be-visited.md)
- [伸缩组无法扩容]()
- [PVC 显示 Pending]()
- [CBS 云盘挂载失败]()
- [apiserver 响应慢]()

### ISTIO问题排查

- [网格内部无法访问外部](istio/cannot-visit-out-of-mesh.md)

### 躺过的坑

- [K8S 对 IPVS 支持的 BUG](damn/k8s-ipvs-bug.md)
- [开启tcp_tw_recycle内核参数在NAT环境会丢包](damn/lost-packets-once-enable-tcp-tw-recycle.md)
- [部分 DNS 查询延迟的原因与解决方案](damn/dns-lookup-delay.md)

### k8s问题定位技巧

- [分析 ExitCode 定位程序退出原因](skill/analysis-exitcode.md)
- [容器内抓包定位网络问题](skill/capture-packets-in-container.md)
- [分析docker磁盘占用](skill/analysis-docker-disk.md)
- [分析网络划分计算最大节点、service 与 pod数量](skill/analysis-cidr.md)

### k8s最佳实践

- [优雅热更新](best-practice/kubernetes-best-practice-grace-update.md)
- [处理容器数据磁盘被写满](best-practice/kubernetes-best-practice-handle-disk-full.md)
- [kubectl 高效技巧](best-practice/efficient-kubectl.md)
- [泛域名动态 Service 转发解决方案](best-practice/wildcard-domain-forward.md)
- [处理内存碎片化](best-practice/handle-memory-fragmentation.md)
- [解决长连接服务扩容失效](best-practice/scale-keepalive-service.md)
- [合理设置 request 和 limit]()
- [使用自建 DNS 解析内部域名]()