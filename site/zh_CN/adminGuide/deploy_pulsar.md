---
id: deploy_pulsar.md
title: 使用 Docker Compose 或 Helm 配置消息存储
related_key: Pulsar, storage
summary: 学习如何使用 Docker Compose 或 Helm 配置消息存储。
---

# 使用 Docker Compose 或 Helm 配置消息存储

Milvus 使用 Pulsar 或 Kafka 来管理最近更改的日志、输出流日志，并提供日志订阅。Pulsar 是默认的消息存储系统。本主题介绍如何使用 Docker Compose 或 Helm 配置消息存储。

您可以使用 [Docker Compose](https://docs.docker.com/get-started/overview/) 配置 Pulsar，也可以在 K8s 上配置 Kafka。

## 使用 Docker Compose 配置 Pulsar

### 1. 配置 Pulsar

要使用 Docker Compose 配置 Pulsar，请在 milvus/configs 路径下的 `milvus.yaml` 文件中为 `pulsar` 部分提供您的值。

```yaml
pulsar:
  address: localhost # Pulsar 的地址
  port: 6650 # Pulsar 的端口
  maxMessageSize: 5242880 # 5 * 1024 * 1024 字节，Pulsar 中每条消息的最大大小。
```

更多信息，请参阅 [与 Pulsar 相关的配置](configure_pulsar.md)。

### 2. 运行 Milvus

运行以下命令以启动使用 Pulsar 配置的 Milvus。

```shell
docker compose up
```

<div class="alert note">配置仅在 Milvus 启动后生效。更多信息，请参阅 <a href=https://milvus.io/docs/install_standalone-docker.md#Start-Milvus>启动 Milvus</a>。</div>

## 使用 Helm 配置 Pulsar

对于在 K8s 上的 Milvus 集群，您可以在启动 Milvus 的同一命令中配置 Pulsar。或者，您可以在启动 Milvus 之前在 [milvus-helm](https://github.com/milvus-io/milvus-helm) 存储库中的 /charts/milvus 路径下使用 <code>values.yml</code> 文件配置 Pulsar。

有关如何使用 Helm 配置 Milvus 的详细信息，请参阅 [使用 Helm Charts 配置 Milvus](configure-helm.md)。有关与 Pulsar 相关的配置项的详细信息，请参阅 [与 Pulsar 相关的配置](configure_pulsar.md)。

### 使用 YAML 文件

1. 在 <code>values.yaml</code> 文件中配置 <code>externalConfigFiles</code> 部分。

```yaml
extraConfigFiles:
  user.yaml: |+
    pulsar:
      address: localhost # Pulsar 的地址
      port: 6650 # Pulsar 的端口
      webport: 80 # Pulsar 的 Web 端口，如果直接连接而非通过代理，请使用 8080
      maxMessageSize: 5242880 # 5 * 1024 * 1024 字节，Pulsar 中每条消息的最大大小。
      tenant: public
      namespace: default    
```

2. 配置上述部分并保存 <code>values.yaml</code> 文件后，运行以下命令安装使用 Pulsar 配置的 Milvus。

```shell
helm install <your_release_name> milvus/milvus -f values.yaml
```

## 使用 Helm 配置 Kafka
```
对于在 K8s 上的 Milvus 集群，您可以在启动 Milvus 的同时配置 Kafka。另外，您也可以在启动 Milvus 之前，在 [milvus-helm](https://github.com/milvus-io/milvus-helm) 仓库中的 /charts/milvus 路径下使用 <code>values.yml</code> 文件来配置 Kafka。

有关如何使用 Helm 配置 Milvus 的详细信息，请参考 [使用 Helm Charts 配置 Milvus](configure-helm.md)。有关与 Pulsar 相关的配置项的详细信息，请参考 [与 Kafka 相关的配置](configure_kafka.md)。
```

### 使用 YAML 文件

1. 如果您想要将 Kafka 作为消息存储系统，请在 <code>values.yaml</code> 文件中配置 <code>externalConfigFiles</code> 部分。

```yaml
extraConfigFiles:
  user.yaml: |+
    kafka:
      brokerList:
        -  <your_kafka_address>:<your_kafka_port>
      saslUsername:
      saslPassword:
      saslMechanisms: PLAIN
      securityProtocol: SASL_SSL    
```

2. 在配置了上述部分并保存了 <code>values.yaml</code> 文件后，运行以下命令以安装使用 Kafka 配置的 Milvus。

```shell
helm install <your_release_name> milvus/milvus -f values.yaml
```

## 使用 Helm 配置 RocksMQ

Milvus 独立版使用 RocksMQ 作为默认消息存储。有关如何使用 Helm 配置 Milvus 的详细步骤，请参考 [使用 Helm Charts 配置 Milvus](configure-helm.md)。有关 RocksMQ 相关的配置项的详细信息，请参考 [RocksMQ 相关配置](configure_rocksmq.md)。

- 如果您使用 RocksMQ 启动 Milvus 并希望更改其设置，可以在以下 YAML 文件中更改设置后运行 `helm upgrade -f `。

- 如果您已经使用 Helm 安装了使用除 RocksMQ 外的其他消息存储的 Milvus 独立版，并希望将其切换回 RocksMQ，请在刷新所有集合并停止 Milvus 后，使用以下 YAML 文件运行 `helm upgrade -f `。

```yaml
extraConfigFiles:
  user.yaml: |+
    rocksmq:
      # 消息在 rocksmq 中存储的路径
      # 请在嵌入式 Milvus 中调整为：/tmp/milvus/rdb_data
      path: /var/lib/milvus/rdb_data
      lrucacheratio: 0.06 # rocksdb 缓存内存比例
      rocksmqPageSize: 67108864 # 64 MB，64 * 1024 * 1024 字节，rocksmq 中每个消息页的大小
      retentionTimeInMinutes: 4320 # 3 天，3 * 24 * 60 分钟，rocksmq 中消息的保留时间
      retentionSizeInMB: 8192 # 8 GB，8 * 1024 MB，rocksmq 中消息的保留大小
      compactionInterval: 86400 # 1 天，每天触发 rocksdb 压实以删除已删除的数据
      # 压实压缩类型，仅支持使用 0,7。
      # 0 表示不压缩，7 将使用 zstd
      # 类型的长度表示 rocksdb 级别的数量
      compressionTypes: [0, 0, 7, 7, 7]    
```

<div class="alert warning">
更改消息存储不是推荐的做法。如果您确实需要这样做，停止所有 DDL 操作，然后调用 FlushAll API 来刷新所有集合，最后在实际更改消息存储之前停止 Milvus。

</div>

## 使用 Helm 配置 NATS

NATS 是一个实验性的消息存储，可作为 RocksMQ 的替代方案。有关如何使用 Helm 配置 Milvus 的详细步骤，请参考[使用 Helm Charts 配置 Milvus](configure-helm.md)。有关 RocksMQ 相关配置项的详细信息，请参考[NATS 相关配置](configure_nats.md)。

- 如果您使用 NATS 启动了 Milvus 并希望更改其设置，可以在以下 YAML 文件中使用更改后的设置运行 `helm upgrade -f `。

- 如果您已经使用其他消息存储安装了独立的 Milvus，并希望将其更改为 NATS，请在刷新所有集合并停止 Milvus 后，使用以下 YAML 文件运行 `helm upgrade -f `。

```yaml
extraConfigFiles:
  user.yaml: |+
    mq:
      type: natsmq
    natsmq:
      # natsmq 的服务器端配置。
      server: 
        # 默认为 4222，nats 服务器监听的端口。
        port: 4222 
        # 默认为 /var/lib/milvus/nats，用于 nats 的 JetStream 存储目录。
        storeDir: /var/lib/milvus/nats 
        # (字节) 默认为 16GB，'file' 存储的最大大小。
        maxFileStore: 17179869184 
        # (字节) 默认为 8MB，消息有效负载中的最大字节数。
        maxPayload: 8388608 
        # (字节) 默认为 64MB，连接缓冲的最大字节数，适用于客户端连接。
        maxPending: 67108864 
        # (√毫秒) 默认为 4s，等待 natsmq 初始化完成的超时时间。
        initializeTimeout: 4000 
        monitor:
          # 默认为 false，如果为 true，则启用调试日志消息。
          debug: false 
          # 默认为 true，如果设置为 false，则记录不带时间戳。
          logTime: true 
          # 默认没有日志文件，日志文件路径相对于...
          logFile: 
          # (字节) 默认为 0，日志文件滚动到新文件的字节大小。
          logSizeLimit: 0 
        retention:
          # (分钟) 默认为 3 天，P 通道中任何消息的最大存储时间。
          maxAge: 4320 
          # (字节) 默认为 None，单个 P 通道可以包含的最大字节数。如果 P 通道超过此大小，则删除最旧的消息。
          maxBytes:
          # 默认为 None，单个 P 通道可以包含的最大消息数。如果 P 通道超过此限制，则删除最旧的消息。    
          maxMsgs: 
```

<div class="alert note">

**选择 RocksMQ 还是 NATS？**

RockMQ 使用 CGO 与 RocksDB 交互，并通过自身管理内存，而纯 Go 语言编写的 NATS 嵌入在 Milvus 安装中，将内存管理委托给 Go 的垃圾回收器（GC）。
在数据包小于64 kb的情况下，RocksDB在内存使用、CPU使用和响应时间方面表现优异。另一方面，如果数据包大于64 kb，NATS在具有足够内存和理想的GC调度的情况下，在响应时间方面表现出色。

目前，建议仅将NATS用于实验。

</div>

## 接下来做什么

了解如何使用Docker Compose或Helm配置Milvus的其他依赖项：
- [使用Docker Compose或Helm配置对象存储](deploy_s3.md)
- [使用Docker Compose或Helm配置元数据存储](deploy_etcd.md)