## RBAC细化集群的操作权限

#### 创建serviceAccount

就是创建一个ServiceAccount资源对象，同步一个样例：

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: hale
  namespace: default
```

创建之后，集群会自动给ServiceAccount绑一个secret，先记一下，后面有用。

```bash
$ kubectl get ServiceAccount hale -o jsonpath='{.secrets[0].name}' | xargs kubectl get secret -o yaml
apiVersion: v1
data:
  ca.crt: xxxxxxx
  namespace: ZGVmYXVsdA==
  token: xxxxxxx
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: hale
    kubernetes.io/service-account.uid: 99fb73b9-c273-11e9-8275-129ae4c231d4
  creationTimestamp: 2019-08-19T11:22:17Z
  name: hale-token-5crhj
  namespace: default
  resourceVersion: "89912726"
  selfLink: /api/v1/namespaces/default/secrets/hale-token-5crhj
  uid: 99fee546-c273-11e9-8275-129ae4c231d4
type: kubernetes.io/service-account-token
```

#### 配置RBAC授权

1 Role或者ClusterRole

用于配置具体的角色，一个角色代表了一组权限。根据生效的范围又分为Role（只在某个命名空间内有效）和ClusterRole（整个kubernetes集群范围内都有效）。比如授予对所有命名空间中的secret的读访问权限配置：

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  # 鉴于ClusterRole是集群范围对象，所以这里不需要定义"namespace"字段
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

2 RoleBinding或者ClusterRoleBinding

用于将角色和ServiceAccount绑定。同样，根据生效的范围又分为RoleBinding 和ClusterRoleBinding，其中，RoleBinding可以引用在同一命名空间下的Role对象，也可以引用一个ClusterRole；而ClusterRoleBinding只能引用ClusterRole对象。比如将ServiceAccount对象hale，绑一个ClusterRole权限（secret-reader）：

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: read-secret
  namespace: kube-system
subjects:
  - kind: ServiceAccount
    name: hale
    namespace: default
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

更多参考：<https://kubernetes.io/docs/reference/access-authn-authz/rbac/>

####  TKE开启公网访问或者内网访问

1. 在 “基本信息” 中，单击 “集群凭证” 中的 【显示凭证】。
2. 开启公网访问（需要配置来源网段）或者内网访问

#### 客户端配置kubeconfig

```bash
# 配置user entry
kubectl config set-credentials <user> --token=bearer_token
# 配置cluster entry
kubectl config set-cluster <cluster> --server=https://cls-xxx.ccs.tencent-cloud.com --certificate-authority=ca.crt
# 配置context entry
kubectl config set-context <context> --cluster=<cluster> --user=<user>
# 查看
kubectl config view
# 配置当前使用的context
kubectl config use-context <context>
```

备注：

Bearer_token: 来自于[创建serviceAccount](#创建serviceAccount)中secret的token字段，并base64解码。

ca.crt：来自于[创建serviceAccount](#创建serviceAccount)中secret的ca.crt字段，并base64解码（保存为文件ca.crt)。或者从集群凭证中直接拷贝“集群CA证书”。

