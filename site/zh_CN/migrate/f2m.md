---
id: f2m.md
title: 从 Faiss 迁移
related_key: Faiss, 迁移, 导入
summary: 学习如何将 Faiss 数据迁移到 Milvus。
---

# 从 Faiss 迁移

本指南提供了一个全面的、逐步的过程，用于从 Faiss 迁移数据到 Milvus 2.x。通过按照本指南操作，您将能够高效地转移您的数据，利用 Milvus 2.x 的高级功能和改进的性能。

## 先决条件

- **软件版本**:
    - 源 Faiss
    - 目标 Milvus: 2.x
    - 安装详情，请参阅 [安装 Faiss](https://github.com/facebookresearch/faiss/blob/main/INSTALL.md) 和 [安装 Milvus](https://milvus.io/docs/install_standalone-docker.md)。
- **所需工具**:
    - [Milvus-migration](https://github.com/zilliztech/milvus-migration) 工具。安装详情，请参阅 [安装迁移工具](milvusdm_install.md)。

## 配置迁移

将示例迁移配置文件保存为 `migration.yaml` 并根据实际情况修改配置。您可以将配置文件放在任何本地目录中。

```yaml
dumper: # 迁移作业的配置。
  worker:
    limit: 2
    workMode: faiss    # 迁移作业的操作模式。
    reader:
      bufferSize: 1024
    writer:
      bufferSize: 1024
loader:
  worker:
    limit: 2
source: # 源 Faiss 索引的配置。
  mode: local
  local:
    faissFile: ./testfiles/faiss/faiss_ivf_flat.index

target: # 目标 Milvus 集合的配置。
  create:
    collection:
      name: test1w
      shardsNums: 2
      dim: 256
      metricType: L2

  mode: remote
  remote:
    outputDir: testfiles/output/
    cloud: aws
    endpoint: 0.0.0.0:9000
    region: ap-southeast-1
    bucket: a-bucket
    ak: minioadmin
    sk: minioadmin
    useIAM: false
    useSSL: false
    checkBucket: true
  milvus2x:
    endpoint: localhost:19530
    username: xxxxx
    password: xxxxx

```

以下表格描述了示例配置文件中的参数。有关所有配置的完整列表，请参阅 [Milvus 迁移：从 Faiss 到 Milvus 2.x](https://github.com/zilliztech/milvus-migration/blob/main/README_FAISS.md#migrationyaml-reference)。

- `dumper`
  
  
    | 参数 | 描述 |
    | --- | --- |
    | `dumper.worker.limit` | Dumper 线程的并发数。 |
    | `dumper.worker.workMode` | 迁移作业的操作模式。从 Faiss 索引迁移时设置为 faiss。 |
    | `dumper.worker.reader.bufferSize` | 每批从 Faiss 中读取的缓冲区大小。单位：KB。 |
    | `dumper.worker.writer.bufferSize` | 每批写入 Milvus 的缓冲区大小。单位：KB。 |
- `loader`
  
  
    | 参数 | 描述 |
    | --- | --- |
    | `loader.worker.limit` | Loader 线程的并发数。 |
- `source`
  
  
    | 参数 | 描述 |
    | --- | --- |
    | `source.mode` | 指定源文件的读取位置。有效值：<br>- `local`：从本地磁盘读取文件。<br>- `remote`：从远程存储读取文件。 |
    | `source.local.faissFile` | 源文件所在的目录路径。例如，`/db/faiss.index`。 |
- `target`
  
  
    | 参数 | 描述 |
    | --- | --- |
    | `target.create.collection.name` | Milvus 集合的名称。 |
    | `target.create.collection.shardsNums` | 要在集合中创建的分片数。有关分片的更多信息，请参阅[Terminology](https://milvus.io/docs/glossary.md#Shard)。 |
    | `target.create.collection.dim` | 向量字段的维度。 |
    | `target.create.collection.metricType` | 用于衡量向量之间相似性的度量类型。有关更多信息，请参阅[Terminology](https://milvus.io/docs/glossary.md#Metric-type)。 |
    | `target.mode` | 存储转储文件的位置。有效值：<br>- `local`：在本地磁盘上存储转储文件。<br>- `remote`：在对象存储上存储转储文件。 |
    | `target.remote.outputDir` | 云存储桶中的输出目录路径。 |
    | `target.remote.cloud` | 云存储服务提供商。示例值：`aws`，`gcp`，`azure`。 |
    | `target.remote.endpoint` | Milvus 2.x 存储的终端点。 |
    | `target.remote.region` | 云存储区域。如果使用本地 MinIO，则可以是任何值。 |
    | `target.remote.bucket` | 用于存储数据的存储桶名称。该值必须与 Milvus 2.x 中的配置相同。有关更多信息，请参阅[System Configurations](https://milvus.io/docs/configure_minio.md#miniobucketName)。 |
    | `target.remote.ak` | Milvus 2.x 存储的访问密钥。 |
    | `target.remote.sk` | Milvus 2.x 存储的秘密密钥。 |
    | `target.remote.useIAM` | 是否使用 IAM 角色进行连接。 |
    | `target.remote.useSSL` | 连接到 Milvus 2.x 时是否启用 SSL。有关更多信息，请参阅[Encryption in Transit](https://milvus.io/docs/tls.md#Encryption-in-Transit)。 |
    | `target.remote.checkBucket` | 是否检查对象存储中指定的存储桶是否存在。 |
    | `target.milvus2x.endpoint` | 目标 Milvus 服务器的地址。 |
    | `target.milvus2x.username` | Milvus 2.x 服务器的用户名。如果为您的 Milvus 服务器启用了用户身份验证，则需要此参数。有关更多信息，请参阅[Enable Authentication](https://milvus.io/docs/authenticate.md)。 |
    | `target.milvus2x.password` | Milvus 2.x 服务器的密码。如果为您的 Milvus 服务器启用了用户身份验证，则需要此参数。有关更多信息，请参阅[Enable Authentication](https://milvus.io/docs/authenticate.md)。 |

## 启动迁移任务

1. 使用以下命令启动迁移任务。将 `{YourConfigFilePath}` 替换为配置文件 `migration.yaml` 所在的本地目录。
   
    ```bash```
    运行上述命令将Faiss索引数据转换为NumPy文件，然后使用[bulkInsert](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/utility/do_bulk_insert.md)操作将数据写入目标存储桶。

2. 生成NumPy文件后，使用以下命令将这些文件导入Milvus 2.x。将`{YourConfigFilePath}`替换为存放配置文件`migration.yaml`的本地目录。

    ```bash
    ./milvus-migration  load  --config=/{YourConfigFilePath}/migration.yaml
    ```
    

## 验证结果

执行迁移任务后，您可以发起API调用或使用Attu查看已迁移实体的数量。更多信息，请参考[Attu](https://github.com/zilliztech/attu)和[get_collection_stats()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/get_collection_stats.md)。