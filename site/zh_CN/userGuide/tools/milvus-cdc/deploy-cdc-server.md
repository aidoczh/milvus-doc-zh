---
id: deploy-cdc-server.md
order: 2
summary: 本指南提供了部署 Milvus-CDC 服务器的逐步过程。
title: 部署 CDC 服务器
---

# 部署 CDC 服务器

本指南提供了部署 Milvus-CDC 服务器的逐步过程。

## 先决条件

在部署 Milvus-CDC 服务器之前，请确保满足以下条件：

- __Milvus 实例__：源 Milvus 和至少一个目标 Milvus 都已部署并运行。

    - 源和目标 Milvus 版本必须为 2.3.2 或更高，最好为 2.4.x。我们建议源和目标 Milvus 使用相同版本以确保兼容性。

    - 将目标 Milvus 的 `common.ttMsgEnabled` 配置设置为 `false`。

    - 配置源和目标 Milvus 的不同的元数据和消息存储设置，以避免冲突。例如，避免在多个 Milvus 实例中使用相同的 etcd 和 rootPath 配置，以及相同的 Pulsar 服务和 `chanNamePrefix`。

- __元数据存储__：准备一个 etcd 或 MySQL 数据库用于 Milvus-CDC 元数据存储。

## 步骤

### 获取 Milvus-CDC 配置文件

克隆 [Milvus-CDC 仓库](https://github.com/zilliztech/milvus-cdc)，并转到 `milvus-cdc/server/configs` 目录以访问 `cdc.yaml` 配置文件。

```bash
git clone https://github.com/zilliztech/milvus-cdc.git

cd milvus-cdc/server/configs
```

### 编辑配置文件

在 `milvus-cdc/server/configs` 目录中，修改 `cdc.yaml` 文件以自定义与 Milvus-CDC 元数据存储和源 Milvus 连接详细信息相关的配置。

- __元数据存储配置__：

    - `metaStoreConfig.storeType`：Milvus-CDC 的元数据存储类型。可能的值为 `etcd` 或 `mysql`。

    - `metaStoreConfig.etcdEndpoints`：连接到 Milvus-CDC 的 etcd 的地址。如果 `storeType` 设置为 `etcd`，则为必填项。

    - `metaStoreConfig.mysqlSourceUrl`：Milvus-CDC 服务器的 MySQL 数据库连接地址。如果 `storeType` 设置为 `mysql`，则为必填项。

    - `metaStoreConfig.rootPath`：Milvus-CDC 元数据存储的根路径。此配置支持多租户，允许多个 CDC 服务利用相同的 etcd 或 MySQL 实例，通过不同的根路径实现隔离。

    示例配置：

    ```yaml
    # cdc meta data config
    metaStoreConfig:
      # the metastore type, available value: etcd, mysql
      storeType: etcd
      # etcd address
      etcdEndpoints:
        - localhost:2379
      # mysql connection address
      # mysqlSourceUrl: root:root@tcp(127.0.0.1:3306)/milvus-cdc?charset=utf8
      # meta data prefix, if multiple cdc services use the same store service, you can set different rootPaths to achieve multi-tenancy
      rootPath: cdc
    ```

- __源 Milvus 配置__：
指定源Milvus的连接详细信息，包括etcd和消息存储，以建立Milvus-CDC服务器与源Milvus之间的连接。

- `sourceConfig.etcdAddress`：连接到源Milvus的etcd的地址。有关更多信息，请参阅[etcd相关配置](https://milvus.io/docs/configure_etcd.md#etcd-related-Configurations)。

- `sourceConfig.etcdRootPath`：源Milvus在etcd中存储数据的键的根前缀。该值可能根据Milvus实例的部署方法而有所不同：

    - __Helm__ 或 __Docker Compose__：默认为 `by-dev`。

    - __Operator__：默认为 `<release_name>`。

- `sourceConfig.pulsar`：源Milvus的Pulsar配置。如果源Milvus使用Kafka进行消息存储，请删除所有与Pulsar相关的配置。有关更多信息，请参阅[Pulsar相关配置](https://milvus.io/docs/configure_pulsar.md)。

- `sourceConfig.kafka.address`：源Milvus的Kafka地址。如果源Milvus使用Kafka进行消息存储，请取消注释此配置。有关更多信息，请参阅[Kafka相关配置](https://milvus.io/docs/configure_kafka.md)。

示例配置：

### 编译Milvus-CDC服务器

保存`cdc.yaml`文件后，转到`milvus-cdc`目录并运行以下命令之一来编译服务器：

- 对于二进制文件：

    ```bash
    make build
    ```

- 对于Docker镜像：

    ```bash
    bash build_image.sh
    ```

    对于Docker镜像，请将编译后的文件挂载到容器中的`/app/server/configs/cdc.yaml`。

### 启动服务器

- 使用二进制文件

    转到包含`milvus-cdc`二进制文件和包含`cdc.yaml`文件的`configs`目录的目录，然后启动服务器：

    ```bash
    # 目录结构
    .
    ├── milvus-cdc # 从源代码构建或从发布页面下载
    ├── configs
    │   └── cdc.yaml # cdc和源milvus的配置
    
    # 启动milvus cdc
    ./milvus-cdc server
    ```

- 使用Docker Compose：

    ```bash
    docker-compose up -d
    ```