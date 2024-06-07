---
id: object_storage_operator.md
title: 使用 Milvus Operator 配置对象存储
related_key: minio, s3, storage, etcd, pulsar
summary: 学习如何使用 Milvus Operator 配置对象存储。
---

# 使用 Milvus Operator 配置对象存储

Milvus 使用 MinIO 或 S3 作为对象存储来持久化大规模文件，例如索引文件和二进制日志。本主题介绍了在使用 Milvus Operator 安装 Milvus 时如何配置对象存储依赖项。更多详情，请参考 Milvus Operator 仓库中的 [使用 Milvus Operator 配置对象存储](https://github.com/zilliztech/milvus-operator/blob/main/docs/administration/manage-dependencies/object-storage.md)。

本主题假设您已部署了 Milvus Operator。

<div class="alert note">查看 <a href="https://milvus.io/docs/v2.2.x/install_cluster-milvusoperator.md">部署 Milvus Operator</a> 获取更多信息。</div>

您需要为使用 Milvus Operator 启动 Milvus 集群指定一个配置文件。

```YAML
kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_cluster_default.yaml
```

您只需要编辑 `milvus_cluster_default.yaml` 中的代码模板以配置第三方依赖项。以下部分介绍了如何分别配置对象存储、etcd 和 Pulsar。

## 配置对象存储

Milvus 集群使用 MinIO 或 S3 作为对象存储来持久化大规模文件，例如索引文件和二进制日志。在 `spec.dependencies.storage` 下添加必需字段以配置对象存储，可能的选项为 `external` 和 `inCluster`。

### 内部对象存储

默认情况下，Milvus Operator 为 Milvus 部署了一个集群内的 MinIO。以下是一个示例配置，演示如何将此 MinIO 用作内部对象存储。

```YAML
apiVersion: milvus.io/v1beta1
kind: Milvus
metadata:
  name: my-release
  labels:
    app: milvus
spec:
  # 省略其他字段 ...
  dependencies:
    # 省略其他字段 ...
    storage:
      inCluster:
        values:
          mode: standalone
          resources:
            requests:
              memory: 100Mi
        deletionPolicy: Delete # Delete | Retain, 默认: Retain
        pvcDeletion: true # 默认: false
```

应用上述配置后，集群内的 MinIO 将以独立模式运行，内存限制最多为 100Mi。请注意：

- `deletionPolicy` 字段指定集群内 MinIO 的删除策略。默认为 `Delete`，另一个选项为 `Retain`。

  - `Delete` 表示在停止 Milvus 实例时删除集群内对象存储。
  - `Retain` 表示将集群内对象存储保留为 Milvus 实例后续启动的依赖服务。

- `pvcDeletion` 字段指定在删除集群内 MinIO 时是否删除 PVC（持久卷索取）。

`inCluster.values` 下的字段与 Milvus Helm Chart 中的字段相同，您可以在[这里](https://github.com/milvus-io/milvus-helm/blob/master/charts/minio/values.yaml)找到它们。

### 外部对象存储

在模板 YAML 文件中使用 `external` 表示使用外部对象存储服务。要使用外部对象存储，您需要正确设置 Milvus CRD 中的 `spec.dependencies.storage` 和 `spec.config.minio` 下的字段。

#### 使用亚马逊网络服务（AWS）S3 作为外部对象存储

- 配置 AWS S3 访问的 AK/SK

  通常可以通过一对访问密钥和访问密钥密码来访问 S3 存储桶。您可以创建一个 `Secret` 对象将它们存储在您的 Kubernetes 中，如下所示：

  ```YAML
  # # 将 <parameters> 更改为匹配您的环境
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-release-s3-secret
  type: Opaque
  stringData:
    accesskey: <my-access-key>
    secretkey: <my-secret-key>
  ```

  然后，您可以将 AWS S3 存储桶配置为外部对象存储：

  ```YAML
  # # 将 <parameters> 更改为匹配您的环境
  apiVersion: milvus.io/v1beta1
  kind: Milvus
  metadata:
    name: my-release
    labels:
      app: milvus
  spec:
    # 省略其他字段 ...
    config:
      minio:
        # 您的存储桶名称
        bucketName: <my-bucket>
        # 可选，配置 milvus 将使用的存储桶前缀
        rootPath: milvus/my-release
        useSSL: true
    dependencies:
      storage:
        # 启用外部对象存储
        external: true
        type: S3 # MinIO | S3
        # AWS S3 的端点
        endpoint: s3.amazonaws.com:443
        # 存储访问密钥和访问密钥密码的 Secret
        secretRef: "my-release-s3-secret"
  ```

- 通过 AssumeRole 配置 AWS S3 访问

  或者，您可以使 Milvus 使用 [AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) 访问您的 AWS S3 存储桶，这样只涉及临时凭证，而不是您的实际 AK/SK。

  如果这是您的首选，您需要在 AWS 控制台上准备一个角色并获取其 ARN，通常形式为 `arn:aws:iam::<your account id>:role/<role-name>`。

  然后，创建一个 `ServiceAccount` 对象将其存储在您的 Kubernetes 中，如下所示：

  ```YAML
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: my-release-sa
    annotations:
      eks.amazonaws.com/role-arn: <my-role-arn>
  ```

  一切准备就绪后，在模板 YAML 文件中引用上述 `ServiceAccount`，并将 `spec.config.minio.useIAM` 设置为 `true` 以启用 AssumeRole。

  ```YAML
  apiVersion: milvus.io/v1beta1
  kind: Milvus
  metadata:
    name: my-release
    labels:
      app: milvus
  spec:
    # 省略其他字段 ...
    components:
      # 使用上述 ServiceAccount
      serviceAccountName: my-release-sa
    config:
      minio:
        # 启用 AssumeRole
        useIAM: true
        # 省略其他字段 ...
    dependencies:
      存储：
        # 省略其他字段 ...
        # 注意：在这里必须使用区域端点，否则 Milvus 使用的 minio 客户端将无法连接
        endpoint: s3.<my-bucket-region>.amazonaws.com:443
        secretRef: "" # 我们不需要在这里指定密钥

#### 使用 Google Cloud Storage (GCS) 作为外部对象存储

AWS S3 对象存储并非唯一选择。您还可以使用其他公共云提供商的对象存储服务，比如 Google Cloud。

- 通过 AK/SK 配置 GCS 访问

  配置方式与使用 AWS S3 大致相似。您仍然需要在 Kubernetes 中创建一个 `Secret` 对象来存储您的凭据。
  
  ```YAML
  # # 将 <parameters> 更改为匹配您的环境
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-release-gcp-secret
  type: Opaque
  stringData:
    accesskey: <my-access-key>
    secretkey: <my-secret-key>
  ```

  然后，您只需要将 `endpoint` 更改为 `storage.googleapis.com:443`，并将 `spec.config.minio.cloudProvider` 设置为 `gcp`，如下所示：

  ```YAML
  # # 将 <parameters> 更改为匹配您的环境
  apiVersion: milvus.io/v1beta1
  kind: Milvus
  metadata:
    name: my-release
    labels:
      app: milvus
  spec:
    # 省略其他字段 ...
    config:
      minio:
        cloudProvider: gcp
    dependencies:
      storage:
        # 省略其他字段 ...
        endpoint: storage.googleapis.com:443
  ```

- 通过 AssumeRole 配置 GCS 访问

  类似于 AWS S3，如果您在使用 GKE 作为 Kubernetes 集群，也可以使用 [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity) 来使用临时凭证访问 GCS。

  `ServiceAccount` 的注释与 AWS EKS 不同。您需要指定 GCP 服务账号名称，而不是角色 ARN。

  ```YAML
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: my-release-sa
    annotations:
      iam.gke.io/gcp-service-account: <my-gcp-service-account-name>
  ```

  然后，您可以配置您的 Milvus 实例使用上述 `ServiceAccount`，并通过将 `spec.config.minio.useIAM` 设置为 `true` 来启用 AssumeRole，如下所示：

  ```YAML
  labels:
      app: milvus
  spec:
    # 省略其他字段 ...
    components:
      # 使用上述 ServiceAccount
      serviceAccountName: my-release-sa
    config:
      minio:
        cloudProvider: gcp
        # 启用 AssumeRole
        useIAM: true
        # 省略其他字段 ...  
  ```

## 接下来做什么

学习如何使用 Milvus Operator 配置其他 Milvus 依赖项：
- [使用 Milvus Operator 配置 Meta 存储](meta_storage_operator.md)
- [使用 Milvus Operator 配置消息存储](message_storage_operator.md)