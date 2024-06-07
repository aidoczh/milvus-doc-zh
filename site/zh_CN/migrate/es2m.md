---
id: es2m.md
summary: 本指南提供了从Elasticsearch迁移数据到Milvus 2.x的全面，逐步过程。
title: 从Elasticsearch开始
---

# 从Elasticsearch开始

本指南提供了从Elasticsearch迁移数据到Milvus 2.x的全面，逐步过程。通过遵循本指南，您将能够高效地转移数据，利用Milvus 2.x的高级功能和改进的性能。

## 先决条件

- **软件版本**：
    - 源Elasticsearch版本：7.x或8.x
    - 目标Milvus版本：2.x
    - 安装详情，请参阅[Elasticsearch安装](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html)和[Milvus安装](https://milvus.io/docs/install_standalone-docker.md)。
- **所需工具**：
    - [Milvus-migration](https://github.com/zilliztech/milvus-migration) 工具。安装详情，请参阅[安装迁移工具](milvusdm_install.md)。
- **支持的迁移数据类型**：从源Elasticsearch索引迁移的字段属于以下类型 - [dense_vector](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/dense-vector.html#dense-vector), [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/keyword.html#keyword-field-type), [text](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/text.html#text-field-type), [long](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/number.html), [integer](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/number.html), [double](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/number.html), [float](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/number.html), [boolean](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/boolean.html), [object](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/object.html)。这里未列出的数据类型目前不支持迁移。有关Milvus集合和Elasticsearch索引之间数据映射的详细信息，请参阅[字段映射参考](#field-mapping-reference)。
- **Elasticsearch索引要求**：
    - 源Elasticsearch索引必须包含一个`dense_vector`类型的向量字段。没有向量字段，迁移无法启动。

## 配置迁移文件

将示例迁移配置文件保存为`migration.yaml`，根据实际情况修改配置。您可以将配置文件放在任何本地目录中。
```yaml
dumper: # 迁移作业的配置。
  worker:
    workMode: "elasticsearch" # 迁移作业的操作模式。
    reader:
      bufferSize: 2500 # 每个批次从 Elasticsearch 中读取的缓冲区大小。建议范围为 2000 到 4000。
meta: # 源 Elasticsearch 索引和目标 Milvus 2.x 集合的元数据配置。
  mode: "config" # 指定元数据配置的来源。目前仅支持 `config`。
  version: "8.9.1"
  index: "qatest_index" # 标识要从中迁移数据的 Elasticsearch 索引。
  fields: # 要迁移的 Elasticsearch 索引中的字段。
  - name: "my_vector" # Elasticsearch 字段的名称。
    type: "dense_vector" # Elasticsearch 字段的数据类型。
    dims: 128 # 向量字段的维度。仅在 `type` 为 `dense_vector` 时需要。
  - name: "id"
    pk: true # 指定该字段是否作为主键。
    type: "long"
  - name: "num"
    type: "integer"
  - name: "double1"
    type: "double"
  - name: "text1"
    maxLen: 1000 # 数据字段的最大长度。仅对 `keyword` 和 `text` 数据类型必需。
    type: "text"
  - name: "bl1"
    type: "boolean"
  - name: "float1"
    type: "float"
  milvus: # 用于在 Milvus 2.x 中创建集合的特定配置
    collection: "Collection_01" # Milvus 集合的名称。如果未指定，默认为 Elasticsearch 索引名称。
    closeDynamicField: false # 指定是否在集合中禁用动态字段。默认为 `false`。
    shardNum: 2 # 要在集合中创建的分片数。
    consistencyLevel: Strong # Milvus 集合的一致性级别。
source: # 源 Elasticsearch 服务器的连接配置
  es:
    urls:
    - "http://10.15.1.***:9200" # 源 Elasticsearch 服务器的地址。
    username: "" # Elasticsearch 服务器的用户名。
    password: "" # Elasticsearch 服务器的密码。
target:
  mode: "remote" # 存储转储文件的位置。有效值：`remote` 和 `local`。
  remote: # 远程存储的配置
    outputDir: "migration/milvus/test" # 云存储桶中的输出目录路径。
    cloud: "aws" # 云存储服务提供商。例如：`aws`、`gcp`、`azure` 等。
    region: "us-west-2" # 云存储的地区；如果使用本地 Minio，则可以是任何值。
    bucket: "zilliz-aws-us-****-*-********" # 用于存储数据的存储桶名称；必须与 Milvus 2.x 的 milvus.yaml 中的配置相匹配。
    useIAM: true # 是否使用 IAM 角色进行连接。
    checkBucket: false # 检查存储中是否存在指定的存储桶。
  milvus2x: # 目标 Milvus 2.x 服务器的连接配置
    endpoint: "http://10.102.*.**:19530" # 目标 Milvus 服务器的地址。
    username: "****" # Milvus 2.x 服务器的用户名。
    password: "******" # Milvus 2.x 服务器的密码。
```
以下表格描述了示例配置文件中的参数。要查看完整的配置列表，请参考[Milvus 迁移：从 Elasticsearch 到 Milvus 2.x](https://github.com/zilliztech/milvus-migration/blob/main/README_ES.md#migrationyaml-reference)。

- `dumper`
    
    
    | 参数 | 描述 |
    | --- | --- |
    | `dumper.worker.workMode` | 迁移作业的操作模式。从 Elasticsearch 索引迁移时设置为 `elasticsearch`。 |
    | `dumper.worker.reader.bufferSize` | 每个批次从 Elasticsearch 中读取的缓冲区大小。单位：KB。 |
- `meta`
    
    
    | 参数 | 描述 |
    | --- | --- |
    | `meta.mode` | 指定元数据配置的来源。目前仅支持 `config`。 |
    | `meta.index` | 标识要从中迁移数据的 Elasticsearch 索引。 |
    | `meta.fields` | 要迁移的 Elasticsearch 索引中的字段。 |
    | `meta.fields.name` | Elasticsearch 字段的名称。 |
    | `meta.fields.maxLen` | 字段的最大长度。仅当 `meta.fields.type` 为 `keyword` 或 `text` 时需要此参数。 |
    | `meta.fields.pk` | 指定该字段是否作为主键。 |
    | `meta.fields.type` | Elasticsearch 字段的数据类型。目前支持 Elasticsearch 中以下数据类型：[dense_vector](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/dense-vector.html#dense-vector), [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/keyword.html#keyword-field-type), [text](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/text.html#text-field-type), [long](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/number.html), [integer](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/number.html), [double](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/number.html), [float](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/number.html), [boolean](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/boolean.html), [object](https://www.elastic.co/guide/en/elasticsearch/reference/8.13/object.html)。 |
    | `meta.fields.dims` | 向量字段的维度。仅当 `meta.fields.type` 为 `dense_vector` 时需要此参数。 |
    | `meta.milvus` | 用于在 Milvus 2.x 中创建集合的特定配置。 |
    | `meta.milvus.collection` | Milvus 集合的名称。如果未指定，默认为 Elasticsearch 索引名称。 |
    | `meta.milvus.closeDynamicField` | 指定是否在集合中禁用动态字段。默认为 `false`。有关动态字段的更多信息，请参考[启用动态字段](https://milvus.io/docs/enable-dynamic-field.md#Enable-Dynamic-Field)。 |
    | `meta.milvus.shardNum` | 要在集合中创建的分片数。有关分片的更多信息，请参考[术语表](https://milvus.io/docs/glossary.md#Shard)。 |
    | `meta.milvus.consistencyLevel` | Milvus中集合的一致性级别。更多信息，请参考[Consistency](https://milvus.io/docs/consistency.md)。 |
- `source`
    
    
    | 参数 | 描述 |
    | --- | --- |
    | `source.es` | 源Elasticsearch服务器的连接配置。 |
    | `source.es.urls` | 源Elasticsearch服务器的地址。 |
    | `source.es.username` | Elasticsearch服务器的用户名。 |
    | `source.es.password` | Elasticsearch服务器的密码。 |
- `target`
    
    
    | 参数 | 描述 |
    | --- | --- |
    | `target.mode` | 存储转储文件的位置。有效值：<br>- `local`：将转储文件存储在本地磁盘上。<br>- `remote`：将转储文件存储在对象存储上。 |
    | `target.remote.outputDir` | 云存储桶中的输出目录路径。 |
    | `target.remote.cloud` | 云存储服务提供商。示例值：`aws`，`gcp`，`azure`。 |
    | `target.remote.region` | 云存储区域。如果使用本地MinIO，则可以是任何值。 |
    | `target.remote.bucket` | 用于存储数据的存储桶名称。该值必须与Milvus 2.x中的配置相同。更多信息，请参考[System Configurations](https://milvus.io/docs/configure_minio.md#miniobucketName)。 |
    | `target.remote.useIAM` | 是否使用IAM角色进行连接。 |
    | `target.remote.checkBucket` | 是否检查指定的存储桶是否存在于对象存储中。 |
    | `target.milvus2x` | 目标Milvus 2.x服务器的连接配置。 |
    | `target.milvus2x.endpoint` | 目标Milvus服务器的地址。 |
    | `target.milvus2x.username` | Milvus 2.x服务器的用户名。如果为您的Milvus服务器启用了用户身份验证，则此参数是必需的。更多信息，请参考[Enable Authentication](https://milvus.io/docs/authenticate.md)。 |
    | `target.milvus2x.password` | Milvus 2.x服务器的密码。如果为您的Milvus服务器启用了用户身份验证，则此参数是必需的。更多信息，请参考[Enable Authentication](https://milvus.io/docs/authenticate.md)。 |

## 启动迁移任务

使用以下命令启动迁移任务。将`{YourConfigFilePath}`替换为配置文件`migration.yaml`所在的本地目录。

```bash
./milvus-migration start --config=/{YourConfigFilePath}/migration.yaml
```

以下是成功迁移日志输出的示例：
```bash
[任务/load_base_task.go:94] ["[LoadTasker] 正在解析任务-------------->"] [计数=0] [文件名=testfiles/output/zwh/migration/test_mul_field4/data_1_1.json] [任务ID=442665677354739304]
[任务/load_base_task.go:76] ["[LoadTasker] 任务进度 --------------->"] [文件名=testfiles/output/zwh/migration/test_mul_field4/data_1_1.json] [任务ID=442665677354739304]
[数据库客户端/cus_field_milvus2x.go:86] ["[Milvus2x] 开始展示集合行数"]
[加载器/cus_milvus2x_loader.go:66] ["[Loader] 静态: "] [集合=test_mul_field4_rename1] [之前数量=50000] [之后数量=100000] [增加数量=50000]
[加载器/cus_milvus2x_loader.go:66] ["[Loader] 静态总计"] ["总集合数"=1] [之前总数=50000] [之后总数=100000] [总增加数量=50000]
[迁移/es_starter.go:25] ["[Starter] ES到Milvus迁移完成!!!"] [耗时=80.009174459]
[启动器/starter.go:106] ["[Starter] 迁移成功!"] [耗时=80.00928425]
[清理器/remote_cleaner.go:27] ["[Remote Cleaner] 开始清理文件"] [存储桶=a-bucket] [根路径=testfiles/output/zwh/migration]
[命令/start.go:32] ["[Cleaner] 清理文件成功!"]
```
## 验证结果

执行迁移任务后，您可以发起 API 调用或使用 Attu 查看已迁移实体的数量。更多信息，请参考 [Attu](https://github.com/zilliztech/attu) 和 [get_collection_stats()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/get_collection_stats.md)。

## 字段映射参考

请查看下表，了解 Elasticsearch 索引中的字段类型如何映射到 Milvus 集合中的字段类型。

有关 Milvus 支持的数据类型的更多信息，请参考 [支持的数据类型](https://milvus.io/docs/schema.md#Supported-data-types)。

| Elasticsearch 字段类型 | Milvus 字段类型 | 描述 |
| --- | --- | --- |
| dense_vector | FloatVector | 向量维度在迁移过程中保持不变。 |
| keyword | VarChar | 设置最大长度（1 到 65,535）。超出限制的字符串可能触发迁移错误。 |
| text | VarChar | 设置最大长度（1 到 65,535）。超出限制的字符串可能触发迁移错误。 |
| long | Int64 | - |
| integer | Int32 | - |
| double | Double | - |
| float | Float | - |
| boolean | Bool | - |
| object | JSON | - |