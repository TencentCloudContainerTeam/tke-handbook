# 服务不能被访问

## targetPort 错误
- 现象：node上直接访问pod能通，访问 service 和 nodePort 不行
- 原因：service 的 targetPort 与容器实际监听端口不一致
- 解决方法：更正 service 的 targetPort

## k8s 支持 ipvs 的 bug
- 现象： node上直接访问pod能通，访问 service 不行且集群开启了 ipvs、集群版本小于1.11、node上有pod使用了与service同端口号的hostPort
- 原因：是 k8s 对 ipvs 支持的 bug，1.11 版本修复
- 参考：
  - https://github.com/kubernetes/kubernetes/issues/66103
  - https://github.com/kubernetes/kubernetes/issues/60688
  - http://tapd.oa.com/qcloud_docker/prong/stories/view/1010140411063954339
- 解决方法：使用不与 service 端口号冲突的 hostPort 或升级集群版本

## 安全组没放通容器网段
node的安全组需要放通容器网段，因为访问 service 时，数据包可能从一个node转发到另一个node，源IP网段是容器网段，如果node安全组没放通转发就会失败