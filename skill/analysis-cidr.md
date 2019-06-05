# 分析网络划分计算最大节点、service 与 pod数量

- 看 controller-manager 启动参数
``` bash
$ ps -ef | grep kube-controller-manager
/usr/bin/kube-controller-manager --cluster-cidr=10.99.0.0/19 --service-cluster-ip-range=10.99.28.0/22 ...
```
- `--cluster-cidr=10.99.0.0/19` 表示集群网络的 CIDR
- `--service-cluster-ip-range=10.99.28.0/22` 表示 Service 占用的子网(在TKE中是属于集群网络CIDR范围内的一个子网)
- TKE 默认每个节点的 CIDR 是 24 位，可以通过 `kubectl describe node` 查看 `PodCIDR` 字段来看，这里假设实际就是 24 位
- 此例中集群 Service 数量为：2^(32-22)=1024 个。公式：`2 ^ (32 - SERVICE_CIDR_MASK_SIZE)`
- 此例中集群节点最大数量：2^(24-19) - 2^(24-22) = 32 - 4 = 28 个 (Service占用4个节点子网段) 公式：`2 ^ (POD_CIDR_MASK_SIZE - CLUSTER_CIDR_MASK_SIZE) - 2 ^ (POD_CIDR_MASK_SIZE - SERVICE_CIDR_MASK_SIZE)`
- 此例中每个节点可以容纳 2^(32-24)=256 个 IP，减去网络地址、广播地址和子网为1的网桥 IP 地址(cbr0)，每个节点最多可以容纳 253 个 pod。但是节点 pod 实际最大容量还需要看 kubelet 启动参数 `--max-pods` 的值。通过 `kubectl describe node` 也能看到节点最大 pod 数 (`Capacity.pods`)。节点最大 pod 数计算公式：`2 ^ (32 - POD_CIDR_MASK_SIZE) - 3`