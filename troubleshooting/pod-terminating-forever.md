# Pod 状态一直 Terminating

查看 Pod 事件:

``` bash
$ kubectl describe pod/apigateway-6dc48bf8b6-clcwk -n cn-staging
```

### Need to kill Pod

``` bash
  Normal  Killing  39s (x735 over 15h)  kubelet, 10.179.80.31  Killing container with id docker://apigateway:Need to kill Pod
```

可能是磁盘满了，无法创建和删除 pod

处理建议是参考Kubernetes 最佳实践：[处理容器数据磁盘被写满](../best-practice/kubernetes-best-practice-handle-disk-full.md)

### DeadlineExceeded

``` bash
Warning FailedSync 3m (x408 over 1h) kubelet, 10.179.80.31 error determining status: rpc error: code = DeadlineExceeded desc = context deadline exceeded
```

怀疑是17版本dockerd的BUG。可通过 `kubectl -n cn-staging delete pod apigateway-6dc48bf8b6-clcwk --force --grace-period=0` 强制删除pod，但 `docker ps` 仍看得到这个容器

处置建议：

- 升级到docker 18. 该版本使用了新的 containerd，针对很多bug进行了修复。
- 如果出现terminating状态的话，可以提供让容器专家进行排查，不建议直接强行删除，会可能导致一些业务上问题。