# 网格内部无法访问外部

## ISTIO 用法问题

- istio 默认只跟网格内部可以互通，要访问外部要用 ServiceEntry 是来注册外部服务到网格；一般用 VirtualService 表示内部服务，VirtualService 要转发请求到外部服务，destination 可以写成 ServiceEntry 里声明的 host
- ServiceEntry 的 addresses 是用来限制请求报文中的目标 ip 范围的，而不是外部服务的地址，endpoints 字段才是

## 安全组设置问题

如果是访问集群外，局域网内的服务，可能是安全组设置问题，参考：[无法访问集群外的服务](../troubleshooting/cannot-visit-service-out-of-cluster.md)

## ip-masq-agent 问题
如果是访问的公网，可能是 ip-masq-agent 的问题，参考: [集群内无法访问外网](../troubleshooting/cannot-visit-internet.md)