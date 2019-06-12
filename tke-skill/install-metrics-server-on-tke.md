# 在 TKE 中安装 metrics-server

1. 下载 metrics-server repo:
``` bash
git clone https://github.com/kubernetes-incubator/metrics-server.git
```
2. 修改 metrics-server 启动参数：`--kubelet-insecure-tls` ，防止 metrics server 访问 kubelet 采集指标时报证书问题(`x509: certificate signed by unknown authority`)， 在 `deploy/1.8+/metrics-server-deployment.yaml` 中加 `args`:
``` yaml
      containers:
      - name: metrics-server
        image: ccr.ccs.tencentyun.com/mirrors/metrics-server-amd64:v0.3.1
        args: ["--kubelet-insecure-tls"] # 这里是新增的一行
        imagePullPolicy: Always
```
3. 在项目根目录执行:
``` bash
kubectl apply -f deploy/1.8+/
```
注意是 apply 不是 create，apply 可以替换 kube-system 下的 apiservice，让 metric api 指向这个 metrics-server
4. 等待一小段时间(确保 metrics-server 采集到了 node 和 pod 的 metrics 指标数据)，通过下面的命令检验一下:
``` bash
kubectl top pod --all-namespaces
kubectl top node
```
