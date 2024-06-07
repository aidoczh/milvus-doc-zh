---
id: m2m.md
summary: 本指南提供了一个全面的、逐步的过程，用于将数据从 Milvus 1.x（包括 0.9.x 及以上版本）迁移至 Milvus 2.x。
title: 从 Milvus 1.x 开始
---

# 从 Milvus 1.x 开始

本指南提供了一个全面的、逐步的过程，用于将数据从 Milvus 1.x（包括 0.9.x 及以上版本）迁移至 Milvus 2.x。通过按照本指南操作，您将能够高效地转移数据，利用 Milvus 2.x 的高级功能和改进的性能。

## 先决条件

- **软件版本**：
    - 源 Milvus：0.9.x 至 1.x
    - 目标 Milvus：2.x
- **所需工具**：
    - [Milvus 迁移工具](https://github.com/zilliztech/milvus-migration)。有关安装详情，请参阅[安装迁移工具](milvusdm_install.md)。

## 导出源 Milvus 安装的元数据

为了准备将数据迁移至 Milvus 0.9.x 到 1.x，停止源 Milvus 或至少停止在其中执行任何 DML 操作。

1. 将源 Milvus 安装的元数据导出至 `meta.json`。

    - 对于那些使用 MySQL 作为后端的安装，请运行
    
    ```bash
    ./milvus-migration export -m "user:password@tcp(adderss)/milvus?charset=utf8mb4&parseTime=True&loc=Local" -o outputDir
    ```
    
    - 对于那些使用 SQLite 作为后端的安装，请运行
    
    ```bash
    ./milvus-migration export -s /milvus/db/meta.sqlite -o outputDir
    ```
    
2. 复制您的 Milvus 安装的 `tables` 文件夹，然后将 `meta.json` 和 `tables` 文件夹一起移动到一个空文件夹中。
    
    完成此步骤后，空文件夹的结构应如下所示：
    
    ```
    migration_data
    ├── meta.json
    └── tables
    ```
    
3. 将在上一步准备的文件夹上传至 S3 块存储桶，或直接在下一节中使用此本地文件夹。

## 配置迁移文件

将示例迁移配置文件保存为 `migration.yaml`，并根据实际情况修改配置。您可以将配置文件放在任何本地目录中。

```yaml
dumper:
  worker:
    limit: 2
    workMode: milvus1x
    reader:
      bufferSize: 1024
    writer:
      bufferSize: 1024
loader:
  worker:
    limit: 16
meta:
  mode: local
  localFile: /outputDir/test/meta.json
source:
  mode: local
  local:
    tablesDir: /db/tables/
target:
  mode: remote
  remote:
    outputDir: "migration/test/xx"
    ak: xxxx
    sk: xxxx
    cloud: aws
    region: us-west-2
    bucket: xxxxx
    useIAM: true
    checkBucket: false
  milvus2x:
    endpoint: "{yourMilvus2_xServerAddress}:{port}"
    username: xxxx
    password: xxxx
```

以下表格描述了示例配置文件中的参数。有关完整配置列表，请参阅[Milvus 迁移：Milvus1.x 至 Milvus 2.x](https://github.com/zilliztech/milvus-migration/blob/main/README_1X.md#migrationyaml-reference)。

- `dumper`
    
    
    | 参数 | 描述 |
    | --- | --- |
| `dumper.worker.limit` | 数据导出线程的并发数。 |
| `dumper.worker.workMode` | 迁移作业的操作模式。从 Milvus 1.x 迁移时设置为 `milvus1x`。 |
| `dumper.worker.reader.bufferSize` | 每个批次从 Milvus 1.x 中读取的缓冲区大小。单位：KB。 |
| `dumper.worker.writer.bufferSize` | 每个批次写入到 Milvus 2.x 的缓冲区大小。单位：KB。 |
- `loader`
    
    
| 参数 | 描述 |
| --- | --- |
| `loader.worker.limit` | 加载器线程的并发数。 |
- `meta`
    
    
| 参数 | 描述 |
| --- | --- |
| `meta.mode` | 指定从哪里读取 meta 文件 meta.json。有效值：`local`、`remote`、`mysql`、`sqlite`。 |
| `meta.localFile` | `meta.json` 文件所在的本地目录路径。仅当 `meta.mode` 设置为 `local` 时使用此配置。有关其他 meta 配置，请参阅 [README_1X](https://github.com/zilliztech/milvus-migration/blob/main/README_1X.md#meta)。 |
- `source`
    
    
| 参数 | 描述 |
| --- | --- |
| `source.mode` | 指定从哪里读取源文件。有效值：<br>- `local`：从本地磁盘读取文件。<br>- `remote`：从远程存储读取文件。 |
| `source.local.tablesDir` | 源文件所在的目录路径。例如，`/db/tables/`。 |
- `target`
    
    
| 参数 | 描述 |
| --- | --- |
| `target.mode` | 存储转储文件的位置。有效值：<br>- `local`：在本地磁盘上存储转储文件。<br>- `remote`：在对象存储上存储转储文件。 |
| `target.remote.outputDir` | 云存储桶中的输出目录路径。 |
| `target.remote.ak` | Milvus 2.x 存储的访问密钥。 |
| `target.remote.sk` | Milvus 2.x 存储的秘密密钥。 |
| `target.remote.cloud` | 云存储服务提供商。示例值：`aws`、`gcp`、`azure`。 |
| `target.remote.region` | 云存储区域。如果使用本地 MinIO，则可以是任何值。 |
| `target.remote.bucket` | 用于存储数据的存储桶名称。该值必须与 Milvus 2.x 中的配置相同。有关更多信息，请参阅 [系统配置](https://milvus.io/docs/configure_minio.md#miniobucketName)。 |
| `target.remote.useIAM` | 是否使用 IAM 角色进行连接。 |
| `target.remote.checkBucket` | 是否检查指定的存储桶是否存在于对象存储中。 |
| `target.milvus2x.endpoint` | 目标 Milvus 服务器的地址。 |
| `target.milvus2x.username` | Milvus 2.x 服务器的用户名。如果为 Milvus 服务器启用了用户身份验证，则需要此参数。有关更多信息，请参阅 [启用身份验证](https://milvus.io/docs/authenticate.md)。 |
    | `target.milvus2x.password` | Milvus 2.x 服务器的密码。如果为 Milvus 服务器启用了用户认证，则需要此参数。有关更多信息，请参考 [启用认证](https://milvus.io/docs/authenticate.md)。 |

## 开始迁移任务

1. 使用以下命令启动迁移任务。将 `{YourConfigFilePath}` 替换为存储配置文件 `migration.yaml` 的本地目录。
    
    ```bash
    ./milvus-migration  dump  --config=/{YourConfigFilePath}/migration.yaml
    ```
    
    上述命令将把 Milvus 1.x 中的源数据转换为 NumPy 文件，然后使用 [bulkInsert](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/utility/do_bulk_insert.md) 操作将数据写入目标存储桶。
    
2. 生成 NumPy 文件后，使用以下命令将这些文件导入到 Milvus 2.x。将 `{YourConfigFilePath}` 替换为存储配置文件 `migration.yaml` 的本地目录。
    
    ```bash
    ./milvus-migration  load  --config=/{YourConfigFilePath}/migration.yaml
    ```
    

## 验证结果

执行迁移任务后，您可以进行 API 调用或使用 Attu 查看已迁移的实体数量。有关更多信息，请参考 [Attu](https://github.com/zilliztech/attu) 和 [get_collection_stats()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/get_collection_stats.md)。