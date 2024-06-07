---
id: message_storage_operator.md
title: 使用 Milvus Operator 配置消息存储
related_key: minio, s3, 存储, etcd, pulsar
summary: 学习如何使用 Milvus Operator 配置消息存储。
---

# 使用 Milvus Operator 配置消息存储

Milvus 使用 RocksMQ、Pulsar 或 Kafka 来管理最近更改的日志、输出流日志，并提供日志订阅。本主题介绍了在使用 Milvus Operator 安装 Milvus 时如何配置消息存储依赖项。更多详情，请参阅 Milvus Operator 仓库中的 [使用 Milvus Operator 配置消息存储](https://github.com/zilliztech/milvus-operator/blob/main/docs/administration/manage-dependencies/message-storage.md)。

本主题假定您已部署了 Milvus Operator。

<div class="alert note">查看 <a href="https://milvus.io/docs/v2.2.x/install_cluster-milvusoperator.md">部署 Milvus Operator</a> 以获取更多信息。 </div>

您需要为使用 Milvus Operator 启动 Milvus 集群指定一个配置文件。

```YAML
kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_cluster_default.yaml
```

您只需要编辑 `milvus_cluster_default.yaml` 中的代码模板以配置第三方依赖项。以下各节介绍了如何分别配置对象存储、etcd 和 Pulsar。

## 开始之前
下表显示了 RocksMQ、NATS、Pulsar 和 Kafka 在 Milvus 独立模式和集群模式下的支持情况。

|                 | RocksMQ | NATS   | Pulsar | Kafka |
|:---------------:|:-------:|:------:|:------:|:-----:|
| 独立模式        |    ✔️    |    ✔️   |   ✔️    |   ✔️   |
| 集群模式        |    ✖️    |    ✖️   |   ✔️    |   ✔️   |

还有其他指定消息存储的限制：
- 仅支持为一个 Milvus 实例配置一个消息存储。但是我们仍然支持为一个实例设置多个消息存储的向后兼容性。优先级如下：
  - 独立模式：RocksMQ（默认）> Pulsar > Kafka
  - 集群模式：Pulsar（默认）> Kafka
  - 2.3 版本引入的 Nats 不参与这些向后兼容性规则。
- 在 Milvus 系统运行时无法更改消息存储。
- 仅支持 Kafka 2.x 或 3.x 版本。

## 配置 RocksMQ
RocksMQ 是 Milvus 独立模式下的默认消息存储。

<div class="alert note">

目前，您只能通过 Milvus Operator 为 Milvus 独立模式配置 RocksMQ 作为消息存储。

</div>

#### 示例

以下示例配置了一个 RocksMQ 服务。

```YAML
apiVersion: milvus.io/v1alpha1
kind: Milvus
metadata:
  name: milvus
spec:
  dependencies: {}
  components: {}
  config: {}
```

## 配置 NATS

NATS 是 NATS 的另一种消息存储选择。

#### 示例

以下示例配置了一个 NATS 服务。
```YAML
apiVersion: milvus.io/v1alpha1
kind: Milvus
metadata:
  name: milvus
spec:
  dependencies: 
    msgStreamType: 'natsmq'
    natsmq:
      # natsmq 的服务器端配置。
      server: 
        # 默认为 4222，nats 服务器监听的端口。
        port: 4222 
        # 默认为 /var/lib/milvus/nats，nats 的 JetStream 存储目录。
        storeDir: /var/lib/milvus/nats 
        # (字节) 默认为 16GB，'file' 存储的最大大小。
        maxFileStore: 17179869184 
        # (字节) 默认为 8MB，消息有效负载的最大字节数。
        maxPayload: 8388608 
        # (字节) 默认为 64MB，连接缓冲的最大字节数，适用于客户端连接。
        maxPending: 67108864 
        # (√毫秒) 默认为 4s，等待 natsmq 初始化完成的超时时间。
        initializeTimeout: 4000 
        monitor:
          # 默认为 false，如果为 true，则启用调试日志消息。
          debug: false 
          # 默认为 true，如果设置为 false，则记录不包含时间戳。
          logTime: true 
          # 默认没有日志文件，日志文件路径相对于...
          logFile: 
          # (字节) 默认为 0，无限制，日志文件达到指定大小后滚动到新文件。
          logSizeLimit: 0 
        retention:
          # (分钟) 默认为 3 天，P 通道中任何消息的最大存储时间。
          maxAge: 4320 
          # (字节) 默认为 None，单个 P 通道可以包含的最大字节数。如果 P 通道超过此大小，则删除最旧的消息。
          maxBytes:
          # 默认为 None，单个 P 通道可以包含的最大消息数。如果 P 通道超过此限制，则删除最旧的消息。    
          maxMsgs: 
  components: {}
  config: {}
```
要将消息存储从 RocksMQ 迁移到 NATS，请按照以下步骤操作：

1. 停止所有 DDL 操作。
2. 调用 FlushAll API，然后在 API 调用执行完成后停止 Milvus。
3. 将 `msgStreamType` 更改为 `natsmq`，并在 `spec.dependencies.natsmq` 中对 NATS 设置进行必要更改。
4. 再次启动 Milvus 并检查以下内容：

    - 日志中是否存在一条包含 `mqType=natsmq` 的记录。
    - 在 `spec.dependencies.natsmq.server.storeDir` 指定的目录中是否存在名为 `jetstream` 的目录。

5. （可选）备份并清理 RocksMQ 存储目录中的数据文件。

<div class="alert note">

**选择 RocksMQ 还是 NATS？**

RockMQ 使用 CGO 与 RocksDB 交互，并通过自身管理内存，而纯 Go 语言编写的 NATS 嵌入在 Milvus 安装中，将内存管理委托给 Go 的垃圾回收器（GC）。

在数据包小于 64 kb 的情况下，RocksDB 在内存使用、CPU 使用和响应时间方面表现优异。另一方面，如果数据包大于 64 kb，则在具有足够内存和理想 GC 调度的情况下，NATS 在响应时间方面表现出色。

目前建议仅在实验中使用 NATS。

</div>

## 配置 Pulsar

Pulsar 管理最近更改的日志，输出流日志，并提供日志订阅。在 Milvus 独立版和 Milvus 集群中都支持配置 Pulsar 作为消息存储。但是，使用 Milvus Operator 时，只能为 Milvus 集群配置 Pulsar 作为消息存储。在 `spec.dependencies.pulsar` 下添加必需的字段以配置 Pulsar。

`pulsar` 支持 `external` 和 `inCluster`。

### 外部 Pulsar

`external` 表示使用外部 Pulsar 服务。用于配置外部 Pulsar 服务的字段包括：

- `external`：`true` 表示 Milvus 使用外部 Pulsar 服务。
- `endpoints`：Pulsar 的端点。

#### 示例

以下示例配置了外部 Pulsar 服务。

```YAML
apiVersion: milvus.io/v1alpha1
kind: MilvusCluster
metadata:
  name: my-release
  labels:
    app: milvus
spec:
  dependencies: # Optional
    pulsar: # Optional
      # Whether (=true) to use an existed external pulsar as specified in the field endpoints or 
      # (=false) create a new pulsar inside the same kubernetes cluster for milvus.
      external: true # Optional default=false
      # The external pulsar endpoints if external=true
      endpoints:
      - 192.168.1.1:6650
  components: {}
  config: {}           
```

### 内部 Pulsar

`inCluster` 表示当 Milvus 集群启动时，在集群中会自动启动一个 Pulsar 服务。

#### 示例

以下示例配置了内部 Pulsar 服务。
```YAML
apiVersion: milvus.io/v1alpha1
kind: MilvusCluster
metadata:
  name: my-release
  labels:
    app: milvus
spec:
  dependencies:
    pulsar:
      inCluster:
        values:
          components:
            autorecovery: false
          zookeeper:
            replicaCount: 1
          bookkeeper:
            replicaCount: 1
            resoureces:
              limit:
                cpu: '4'
              memory: 8Gi
            requests:
              cpu: 200m
              memory: 512Mi
          broker:
            replicaCount: 1
            configData:
              ## Enable `autoSkipNonRecoverableData` since bookkeeper is running
              ## without persistence
              autoSkipNonRecoverableData: "true"
              managedLedgerDefaultEnsembleSize: "1"
              managedLedgerDefaultWriteQuorum: "1"
              managedLedgerDefaultAckQuorum: "1"
          proxy:
            replicaCount: 1
  components: {}
  config: {}            
```

<div class="alert note">此示例指定了 Pulsar 每个组件的副本数量、Pulsar BookKeeper 的计算资源以及其他配置。</div>

<div class="alert note">在 <a href="https://artifacthub.io/packages/helm/apache/pulsar/2.7.8?modal=values">values.yaml</a> 中找到完整的配置项，以配置内部 Pulsar 服务。根据上述示例，在 <code>pulsar.inCluster.values</code> 下添加所需的配置项。</div>

假设配置文件名为 `milvuscluster.yaml`，运行以下命令应用配置。

```Shell
kubectl apply -f milvuscluster.yaml
```

## 配置 Kafka

Pulsar 是 Milvus 集群中的默认消息存储。如果要使用 Kafka，请添加可选字段 `msgStreamType` 来配置 Kafka。

`kafka` 支持 `external` 和 `inCluster`。

### 外部 Kafka

`external` 表示使用外部 Kafka 服务。

用于配置外部 Kafka 服务的字段包括：

- `external`：`true` 值表示 Milvus 使用外部 Kafka 服务。
- `brokerList`：发送消息到的经纪人列表。

#### 示例

以下示例配置了外部 Kafka 服务。

```yaml
apiVersion: milvus.io/v1alpha1
kind: MilvusCluster
metadata:
  name: my-release
  labels:
    app: milvus
spec:
  config:
    kafka:
      # securityProtocol 支持：PLAINTEXT、SSL、SASL_PLAINTEXT、SASL_SSL 
      securityProtocol: PLAINTEXT
      # saslMechanisms 支持：PLAIN、SCRAM-SHA-256、SCRAM-SHA-512
      saslMechanisms: PLAIN
      saslUsername: ""
      saslPassword: ""
  # 省略其他字段 ...
  dependencies:
    # 省略其他字段 ...
    msgStreamType: "kafka"
    kafka:
      external: true
      brokerList: 
        - "kafkaBrokerAddr1:9092"
        - "kafkaBrokerAddr2:9092"
        # ...
```
> SASL 配置在运算符 v0.8.5 或更高版本中受支持。

### 内部 Kafka

`inCluster` 表示 Milvus 集群启动时，在集群中自动启动 Kafka 服务。

#### 示例

以下示例配置了内部 Kafka 服务。

```
apiVersion: milvus.io/v1alpha1
kind: MilvusCluster
metadata:
  name: my-release
  labels:
    app: milvus
spec: 
  dependencies:
    msgStreamType: "kafka"
    kafka:
      inCluster: 
        values: {} # 可在 https://artifacthub.io/packages/helm/bitnami/kafka 找到配置值
  components: {}
  config: {}
```

在 [这里](https://artifacthub.io/packages/helm/bitnami/kafka) 找到完整的配置项，以配置内部 Kafka 服务。根据需要在 `kafka.inCluster.values` 下添加配置项。

假设配置文件名为 `milvuscluster.yaml`，运行以下命令应用配置。

```
kubectl apply -f milvuscluster.yaml
```

## 下一步

学习如何使用 Milvus Operator 配置其他 Milvus 依赖项：
- [使用 Milvus Operator 配置对象存储](object_storage_operator.md)
- [使用 Milvus Operator 配置元数据存储](meta_storage_operator.md)