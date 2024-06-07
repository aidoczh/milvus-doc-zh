---
id: scale-dependencies.md
title: 规模依赖
---

# Milvus 的规模依赖

Milvus 依赖于各种组件，如 MinIO、Kafka、Pulsar 和 etcd。扩展这些组件可以增强 Milvus 对不同需求的适应性。

对于 Milvus Operator 用户，请参考 [管理 Milvus Operator 的依赖关系](manage_dependencies.md)。

## 扩展 MinIO

### 增加每个 MinIO pod 的资源

MinIO 是 Milvus 使用的对象存储系统，可以为每个 pod 增加 CPU 和内存资源。

```yaml
# new-values.yaml
minio:
  resources:
     limits:
       cpu: 2
       memory: 8Gi
```

保存文件后，使用以下命令应用更改：

```shell
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```

您还可以通过手动更改每个 MioIO 持久卷声明（PVC）的 `spec.resources.requests.storage` 值来增加 MioIO 集群的磁盘容量。请注意，您的默认存储类应允许卷扩展。

### 添加额外的 MinIO 服务器池（推荐）

建议为您的 Milvus 实例添加额外的 MioIO 服务器池。

```yaml
# new-values.yaml
minio:
  zones: 2
```

保存文件后，使用以下命令应用更改：

```shell
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```

这将向您的 MinIO 集群添加额外的服务器池，允许 Milvus 根据每个服务器池的空闲磁盘容量向 MinIO 服务器池写入数据。例如，如果一个由三个池组成的组有总共 10 TiB 的空闲空间，分布如下：

|        | 空闲空间 | 写入可能性 |
|--------|------------|------------------|
| Pool A | 3 TiB      | 30% (3/10)       |
| Pool B | 2 TiB      | 20% (2/10)       |
| Pool C | 5 TiB      | 50% (5/10)       |

<div class="alert note">

MinIO 不会自动在新的服务器池之间重新平衡对象。如果需要，您可以手动使用 `mc admin rebalance` 启动重新平衡过程。

</div>

## Kafka

### 增加每个 Kafka broker pod 的资源

通过调整每个 broker pod 的 CPU 和内存资源来增强 Kafka broker 的容量。

```yaml
# new-values.yaml
kafka:
  resources:
     limits:
        cpu: 2
        memory: 12Gi
```

保存文件后，使用以下命令应用更改：

```bash
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```

您还可以通过手动更改每个 Kafka 持久卷声明（PVC）的 `spec.resources.requests.storage` 值来增加 Kafka 集群的磁盘容量。确保您的默认存储类允许卷扩展。

## 添加额外的 Kafka broker 池（推荐）

建议为您的 Milvus 实例添加额外的 Kafka 服务器池。

```yaml
# new-values.yaml
kafka:
  replicaCount: 4
```

保存文件后，使用以下命令应用更改：
```shell
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```
这将向您的 Kafka 集群添加一个额外的 broker。

<div class="alert note">

Kafka 不会自动在所有 broker 之间重新平衡主题。如有需要，登录到每个 Kafka broker pod 后，可以使用 `bin/kafka-reassign-partitions.sh` 手动重新平衡主题/分区。

</div>

## Pulsar

Pulsar 将计算和存储分开。您可以独立增加 Pulsar broker（计算）和 Pulsar bookie（存储）的容量。

## 增加每个 Pulsar broker pod 的资源

```yaml
# new-values.yaml
pulsar:
  broker:
    resources:
       limits:
         cpu: 4
         memory: 16Gi
```

保存文件后，使用以下命令应用更改：

```shell
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```

## 增加每个 Pulsar bookie pod 的资源

```yaml
# new-values.yaml
pulsar:
  bookkeeper:
    resources:
       limits:
         cpu: 4
         memory: 16Gi
```

保存文件后，使用以下命令应用更改：

```shell
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```

您还可以通过手动更改每个 Pulsar bookie 的持久卷索赔（PVC）的 `spec.resources.requests.storage` 值来增加 Pulsar 集群的磁盘容量。请注意，您的默认存储类应允许卷扩展。

Pulsar bookie pod 有两种类型的存储：`journal` 和 `legers`。对于 `journal` 类型的存储，考虑使用 `ssd` 或 `gp3` 作为存储类。

### 添加额外的 Pulsar broker pod

```yaml
# new-values.yaml
pulsar:
  broker:
    replicaCount: 3
```

保存文件后，使用以下命令应用更改：

```shell
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```


### 添加额外的 Pulsar bookie pod（推荐）

```yaml
# new-values.yaml
pulsar:
  bookkeeper:
    replicaCount: 3
```

保存文件后，使用以下命令应用更改：

```shell
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```

## etcd

### 增加每个 etcd pod 的资源（推荐）

```yaml
# new-values.yaml
etcd:
  resources:
     limits:
       cpu: 2
       memory: 8Gi
```

保存文件后，使用以下命令应用更改：

```shell
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```

### 添加额外的 etcd pods

etcd pods 的总数应为奇数。

```yaml
# new-values.yaml
etcd:
  replicaCount: 5
```

保存文件后，使用以下命令应用更改：

```shell
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```