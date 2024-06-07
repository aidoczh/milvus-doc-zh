---
id: gcp_layer7.md
title: 在 GCP 上为 Milvus 设置 Layer-7 负载均衡器
related_key: cluster
summary: 学习如何在 GCP 上为 Milvus 集群部署一个位于 Layer-7 负载均衡器后面的负载均衡器。
---

## 在 GCP 上为 Milvus 设置 Layer-7 负载均衡器

与 Layer-4 负载均衡器相比，Layer-7 负载均衡器提供智能负载均衡和缓存功能，是云原生服务的绝佳选择。

本指南将指导您为已经运行在 Layer-4 负载均衡器后面的 Milvus 集群设置一个 Layer-7 负载均衡器。

### 开始之前

- 您的 GCP 账户中已经存在一个项目。

  要创建一个项目，请参考[创建和管理项目](https://cloud.google.com/resource-manager/docs/creating-managing-projects)。本指南中使用的项目名称为 **milvus-testing-nonprod**。

- 您已经在本地安装了 [gcloud CLI](https://cloud.google.com/sdk/docs/quickstart#installing_the_latest_version)、[kubectl](https://kubernetes.io/docs/tasks/tools/) 和 [Helm](https://helm.sh/docs/intro/install/)，或者决定使用基于浏览器的 [Cloud Shell](https://cloud.google.com/shell)。

- 您已经使用您的 GCP 账户凭据 [初始化了 gcloud CLI](https://cloud.google.com/sdk/docs/install-sdk#initializing_the)。

- 您已经在 GCP 上 [部署了一个运行在 Layer-4 负载均衡器后面的 Milvus 集群](gcp.md)。

### 调整 Milvus 配置

本指南假设您已经在 GCP 上 [部署了一个运行在 Layer-4 负载均衡器后面的 Milvus 集群](gcp.md)。

在为这个 Milvus 集群设置 Layer-7 负载均衡器之前，运行以下命令移除 Layer-4 负载均衡器。

```bash
helm upgrade my-release milvus/milvus --set service.type=ClusterIP
```

作为 Layer-7 负载均衡器的后端服务，Milvus 必须满足 [特定的加密要求](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-http2)，以便能够理解来自负载均衡器的 HTTP/2 请求。因此，您需要按照以下步骤在 Milvus 集群上启用 TLS。

```bash
helm upgrade my-release milvus/milvus --set common.security.tlsMode=1
```

### 设置健康检查端点

为了确保服务的可用性，在 GCP 上进行 Layer-7 负载均衡需要探测后端服务的健康状况。因此，我们需要设置一个 BackendConfig 来包装健康检查端点，并通过注释将 BackendConfig 与 Milvus 服务关联起来。

以下是 BackendConfig 的设置代码。将其保存为 `backendconfig.yaml` 以备后用。

```yaml
apiVersion: cloud.google.com/v1
kind: BackendConfig
metadata:
  name: my-release-backendconfig
  namespace: default
spec:
  healthCheck:
    port: 9091
    requestPath: /healthz
    type: HTTP
```

然后运行以下命令来创建健康检查端点。

```bash
kubectl apply -f backendconfig.yaml
```

最后，更新 Milvus 服务的注释，要求稍后创建的第 7 层负载均衡器使用刚刚创建的端点执行健康检查。

```bash
kubectl annotate service my-release-milvus \
    cloud.google.com/app-protocols='{"milvus":"HTTP2"}' \
    cloud.google.com/backend-config='{"default": "my-release-backendconfig"}' \
    cloud.google.com/neg='{"ingress": true}'
```

<div class="alert note">

- 至于第一个注释，

  Milvus 是基于 gRPC 构建的，而 gRPC 基于 HTTP/2。因此，我们可以使用 HTTP/2 作为第 7 层负载均衡器和 Milvus 之间的通信协议。

- 至于第二个注释，

  Milvus 只在 gRPC 和 HTTP/1 上提供健康检查端点。我们需要设置一个 BackendConfig 来包装健康检查端点，并将其与 Milvus 服务关联起来，以便第 7 层负载均衡器探测该端点以获取 Milvus 的健康状况。

- 至于第三个注释，

  它要求在创建 Ingress 后创建网络端点组（NEG）。当 NEGs 与 GKE Ingress 一起使用时，Ingress 控制器会促使负载均衡器的所有方面的创建。这包括创建虚拟 IP 地址、转发规则、健康检查、防火墙规则等。详情请参考 [Google Cloud 文档](https://cloud.google.com/kubernetes-engine/docs/how-to/container-native-load-balancing)。

</div>

### 准备 TLS 证书

TLS 需要证书才能工作。有两种创建证书的方式，即自管理和 Google 管理。

本指南使用 **my-release.milvus.io** 作为访问我们的 Milvus 服务的域名。

#### 创建自管理证书

运行以下命令创建证书。

```bash
# 生成 tls.key。
openssl genrsa -out tls.key 2048

# 创建证书并使用前述密钥进行签名。
openssl req -new -key tls.key -out tls.csr \
    -subj "/CN=my-release.milvus.io"

openssl x509 -req -days 99999 -in tls.csr -signkey tls.key \
    -out tls.crt
```

然后在您的 GKE 集群中使用这些文件创建一个 secret 以备后用。

```bash
kubectl create secret tls my-release-milvus-tls --cert=./tls.crt --key=./tls.key
```

#### 创建 Google 管理证书

以下代码片段是一个 ManagedCertificate 设置。将其保存为 `managed-crt.yaml` 以备后用。

```yaml
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: my-release-milvus-tls
spec:
  domains:
    - my-release.milvus.io
```

通过将设置应用到您的 GKE 集群来创建一个托管证书，操作如下：

```bash
kubectl apply -f ./managed-crt.yaml
```

这可能需要一段时间。您可以通过运行以下命令来检查进度

```bash
kubectl get -f ./managed-crt.yaml -o yaml -w
```

输出应类似于以下内容：
```shell
状态:
  证书名称: mcrt-34446a53-d639-4764-8438-346d7871a76e
  证书状态: 配置中
  域名状态:
  - 域名: my-release.milvus.io
    状态: 配置中
```
一旦 **certificateStatus** 变为 **Active**，您就可以开始设置负载均衡器。

### 创建 Ingress 以生成第 7 层负载均衡器

创建一个包含以下代码片段的 YAML 文件。

- 使用自管理证书

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: my-release-milvus
    namespace: default
  spec:
    tls:
    - hosts:
        - my-release.milvus.io
      secretName: my-release-milvus-tls
    rules:
    - host: my-release.milvus.io
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: my-release-milvus
              port:
                number: 19530
  ```

- 使用 Google 管理的证书

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: my-release-milvus
    namespace: default
    annotations:
      networking.gke.io/managed-certificates: "my-release-milvus-tls"
  spec:
    rules:
    - host: my-release.milvus.io
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: my-release-milvus
              port:
                number: 19530
  ```

然后，您可以通过将文件应用于您的 GKE 集群来创建 Ingress。

```bash
kubectl apply -f ingress.yaml
```

现在，请等待 Google 设置第 7 层负载均衡器。您可以通过运行以下命令来检查进度

```bash
kubectl  -f ./config/samples/ingress.yaml get -w
```

输出应类似于以下内容：

```shell
NAME                CLASS    HOSTS                  ADDRESS   PORTS   AGE
my-release-milvus   <none>   my-release.milvus.io             80      4s
my-release-milvus   <none>   my-release.milvus.io   34.111.144.65   80, 443   41m
```

一旦 **ADDRESS** 字段中显示了一个 IP 地址，第 7 层负载均衡器就准备就绪了。上面的输出显示了端口 80 和端口 443。请记住，为了您的利益，应始终使用端口 443。

## 通过第 7 层负载均衡器验证连接

本指南使用 PyMilvus 来验证刚刚创建的第 7 层负载均衡器后面的 Milvus 服务的连接。有关详细步骤，请阅读 [此处](example_code)。

请注意，连接参数会根据您选择管理证书的方式在 [准备 TLS 证书](#prepare-tls-certificates) 中有所不同。

```python
from pymilvus import (
    connections,
    utility,
    FieldSchema,
    CollectionSchema,
    DataType,
    Collection,
)

# 对于自管理证书，您需要在用于建立连接的参数中包含证书。
connections.connect("default", host="34.111.144.65", port="443", server_pem_path="tls.crt", secure=True, server_name="my-release.milvus.io")

# 对于 Google 管理的证书，则无需这样做。
connections.connect("default", host="34.111.144.65", port="443", secure=True, server_name="my-release.milvus.io")
```

<div class="alert note">
- 在 **host** 和 **port** 中的 IP 地址和端口号应与 [创建 Ingress 以生成 Layer-7 负载均衡器](#create-an-ingress-to-generate-a-layer-7-load-balancer) 末尾列出的内容匹配。
- 如果您已设置 DNS 记录将域名映射到主机 IP 地址，请用域名替换 **host** 中的 IP 地址，并省略 **server_name**。

</div>