---
id: deploy_s3.md
title: 使用 Docker Compose 或 Helm 配置对象存储
related_key: S3, storage
summary: 学习如何使用 Docker Compose 或 Helm 为 Milvus 设置 S3 存储。
---

# 使用 Docker Compose 或 Helm 配置对象存储

Milvus 默认使用 MinIO 作为对象存储，但也支持使用 [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/) 作为持久对象存储来存储日志和索引文件。本主题介绍了如何为 Milvus 配置 S3。如果您对 MinIO 满意，可以跳过此主题。

您可以使用 [Docker Compose](https://docs.docker.com/get-started/overview/) 或在 K8s 上配置 S3。

## 使用 Docker Compose 配置 S3

### 1. 配置 S3
[MinIO](https://min.io/product/overview) 兼容 S3。要使用 Docker Compose 配置 S3，请在 milvus/configs 路径下的 <code>milvus.yaml</code> 文件中为 <code>minio</code> 部分提供您的值。

```yaml
minio:
  address: <your_s3_endpoint>
  port: <your_s3_port>
  accessKeyID: <your_s3_access_key_id>
  secretAccessKey: <your_s3_secret_access_key>
  useSSL: <true/false>
  bucketName: "<your_bucket_name>"
```
更多信息请参阅 [MinIO/S3 配置](configure_minio.md)。

### 2. 优化 docker-compose.yaml
您还需要在 `docker-compose.yaml` 中为 milvus 服务删除 `MINIO_ADDRESS` 环境变量。默认情况下，milvus 将使用本地的 minio 而不是外部的 S3。

### 3. 运行 Milvus
运行以下命令启动使用 S3 配置的 Milvus。
```shell
docker compose up
```
<div class="alert note">配置仅在 Milvus 启动后生效。更多信息请参阅 <a href=https://milvus.io/docs/install_standalone-docker.md#Start-Milvus>启动 Milvus</a>。</div>

## 在 K8s 上配置 S3

对于在 K8s 上的 Milvus 集群，您可以在启动 Milvus 的同一命令中配置 S3。或者，您可以在启动 Milvus 之前在 [milvus-helm](https://github.com/milvus-io/milvus-helm) 仓库中的 /charts/milvus 路径下使用 <code>values.yml</code> 文件配置 S3。

以下表格列出了在 YAML 文件中配置 S3 的键。
| 键名             | 描述                          | 值                                 |
| --------------------- | ------------------------------------ | ------------------------------------ |
| <code>minio.enabled</code>         | 启用或禁用 MinIO。       | <code>true</code>/<code>false</code> |
| <code>externalS3.enabled</code>    | 启用或禁用 S3。          | <code>true</code>/<code>false</code> |
| <code>externalS3.host</code>       | 访问 S3 的端点。       |                                      |
| <code>externalS3.port</code>       | 访问 S3 的端口。           |                                      |
| <code>externalS3.rootPath</code>   | S3 存储的根路径。 | 默认为空字符串。          |
| <code>externalS3.accessKey</code>  | S3 的访问密钥 ID。        |                                      |
| <code>externalS3.secretKey</code>  | S3 的秘密访问密钥。    |                                      |
| <code>externalS3.bucketName</code> | S3 存储桶的名称。       |                                      |
| <code>externalS3.useSSL</code>     | 连接时是否使用 SSL | 默认值为 <code>false</code> |


### 使用 YAML 文件

1. 在 <code>values.yaml</code> 文件中配置 <code>minio</code> 部分。

```yaml
minio:
  enabled: false
```

2. 在 <code>values.yaml</code> 文件中使用您的值配置 <code>externalS3</code> 部分。

```yaml
externalS3:
  enabled: true
  host: "<your_s3_endpoint>"
  port: "<your_s3_port>"
  accessKey: "<your_s3_access_key_id>"
  secretKey: "<your_s3_secret_key>"
  useSSL: <true/false>
  bucketName: "<your_bucket_name>"
```

3. 配置上述部分并保存 <code>values.yaml</code> 文件后，运行以下命令安装使用 S3 配置的 Milvus。

```shell
helm install <your_release_name> milvus/milvus -f values.yaml
```
### 使用命令

要安装 Milvus 并配置 S3，请使用以下命令并填入您的值。

```shell
helm install <your_release_name> milvus/milvus --set cluster.enabled=true  --set minio.enabled=false --set externalS3.enabled=true --set externalS3.host=<your_s3_endpoint> --set externalS3.port=<your_s3_port> --set externalS3.accessKey=<your_s3_access_key_id> --set externalS3.secretKey=<your_s3_secret_key> --set externalS3.bucketName=<your_bucket_name>
```
## 接下来做什么

学习如何使用 Docker Compose 或 Helm 配置其他 Milvus 依赖项：
- [使用 Docker Compose 或 Helm 配置元数据存储](deploy_etcd.md)
- [使用 Docker Compose 或 Helm 配置消息存储](deploy_pulsar.md)