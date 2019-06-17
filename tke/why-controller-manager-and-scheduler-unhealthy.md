# 为什么 controller-manager 和 scheduler 状态显示 Unhealthy

``` bash
kubectl get cs
NAME                 STATUS      MESSAGE                                                                                        ERROR
scheduler            Unhealthy   Get http://127.0.0.1:10251/healthz: dial tcp 127.0.0.1:10251: getsockopt: connection refused
controller-manager   Unhealthy   Get http://127.0.0.1:10252/healthz: dial tcp 127.0.0.1:10252: getsockopt: connection refused
etcd-0               Healthy     {"health": "true"}
```

查看组件状态发现 controller-manager 和 scheduler 状态显示 Unhealthy，但是集群正常工作，是因为TKE metacluster托管方式集群的  apiserver 与 controller-manager 和 scheduler 不在同一个节点导致的，这个不影响功能。如果发现是 Healthy 说明 apiserver 跟它们部署在同一个节点，所以这个取决于部署方式。

**更详细的原因：**

apiserver探测controller-manager 和 scheduler写死直接连的本机
``` go
func (s componentStatusStorage) serversToValidate() map[string]*componentstatus.Server {
	serversToValidate := map[string]*componentstatus.Server{
		"controller-manager": {Addr: "127.0.0.1", Port: ports.InsecureKubeControllerManagerPort, Path: "/healthz"},
		"scheduler":          {Addr: "127.0.0.1", Port: ports.InsecureSchedulerPort, Path: "/healthz"},
	}
```
源码：https://github.com/kubernetes/kubernetes/blob/v1.14.3/pkg/registry/core/rest/storage_core.go#L256

相关 issue:
- https://github.com/kubernetes/kubernetes/issues/19570
- https://github.com/kubernetes/enhancements/issues/553