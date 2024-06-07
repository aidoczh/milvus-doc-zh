---
id: milvus_lite.md
summary: 使用 Milvus Lite 的入门指南。
title: 在本地运行 Milvus Lite
---

# 在本地运行 Milvus Lite

本页面介绍了如何在本地运行 Milvus Lite 以进行开发和测试。

## 概述

Milvus Lite 是 [Milvus](https://github.com/milvus-io/milvus) 的轻量级版本，Milvus 是一个开源的向量数据库，通过向量嵌入和相似度搜索为 AI 应用提供支持。

Milvus Lite 可以被导入到您的 Python 应用程序中，提供 Milvus 的核心向量搜索功能。Milvus Lite 包含在 [Milvus 的 Python SDK](https://github.com/milvus-io/pymilvus) 中，因此可以通过 `pip install pymilvus` 简单部署。该存储库包含了 Milvus Lite 的核心组件。

Milvus Lite 与 Milvus 共享相同的 API，并覆盖了大部分 Milvus 的功能。它们共同为不同规模的环境提供一致的用户体验，适用于不同规模的用例。使用相同的客户端代码，您可以在 Milvus Lite 中快速演示少于百万个向量，或者在单台机器上托管 Milvus Docker 容器的小规模应用程序，最终在 Kubenetes 上进行大规模生产部署，以每秒数千次查询量为服务的数十亿向量。

## 先决条件

Milvus Lite 支持以下操作系统发行版和芯片类型：

- Ubuntu >= 20.04 (x86_64)
- MacOS >= 11.0 (Apple Silicon 和 x86_64)

请注意，Milvus Lite 适用于开始使用向量搜索或构建演示和原型。对于生产用例，我们建议在 [Docker](install_standalone-docker.md) 和 [Kubenetes](install_cluster-milvusoperator.md) 上使用 Milvus，或考虑在 [Zilliz Cloud](https://zilliz.com/cloud) 上使用完全托管的 Milvus。

## 设置 Milvus Lite

Milvus Lite 已经与 pymilvus 打包在一起，pymilvus 是 Milvus 的 Python SDK 库。要设置 Milvus Lite，请在终端中运行以下命令。

```
pip install "pymilvus>=2.4.2"
```

## 连接到 Milvus Lite

您可以按照以下方式连接到 Milvus Lite。

```python
from pymilvus import MilvusClient

client = MilvusClient("milvus_demo.db")
```

运行上述代码片段后，当前文件夹中将生成一个名为 **milvus_demo.db** 的数据库文件。

## 限制

在运行 Milvus Lite 时，请注意某些功能不受支持。以下表格总结了 Milvus Lite 的使用限制。

### 集合

| 方法 / 参数                                                                                                                   | Milvus Lite 中的支持情况                                      |
|----------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------|
| [create_collection()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md)       | 支持，但参数有限                                               |
| `collection_name`                                                                                                          | 是                                                             |
| `dimension`                                                                                                                | 是                                                             |
| `primary_field_name`                                                                                                       | 是                                                             |
| `id_type`                                                                                                                  | 是                                                             |
| `vector_field_name`                                                                                                        | 是                                                             |
| `metric_type`                                                                                                              | 是                                                             |
| `auto_id`                                                                                                                  | 是                                                             |
| `schema`                                                                                                                   | 是                                                             |
| `index_params`                                                                                                             | 是                                                             |
| `enable_dynamic_field`                                                                                                     | 是                                                             |
| `num_shards`                                                                                                               | 否                                                             |
| `partition_key_field`                                                                                                      | 否                                                             |
| `num_partitions`                                                                                                           | 否                                                             |
| `consistency_level`                                                                                                        | 否（仅支持`Strong`；任何配置都将被视为`Strong`。）          |
| [get_collection_stats()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/get_collection_stats.md) | 支持获取集合统计信息。                       |
| `collection_name`                                                                                                          | 是                                                            |
| `timeout`                                                                                                                  | 是                                                            |
| [describe_collection()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/describe_collection.md)   | 响应中的 `num_shards`、`consistency_level` 和 `collection_id` 无效。 |
| `timeout`                                                                                                                  | 是                                                            |
| [has_collection()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/has_collection.md)             | 支持检查集合是否存在。                                       |
| `collection_name`                                                                                                          | 是                                                            |
| `timeout`                                                                                                                  | 是                                                            |
| [list_collections()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/list_collections.md)         | 支持列出所有集合。                                           |
| [drop_collection()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/drop_collection.md)           | 支持删除集合。                                               |
| `collection_name`                                                                                                          | 是                                                            |
| `timeout`                                                                                                                  | 是                                                            |
| [rename_collection()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/rename_collection.md)       | 不支持重命名集合。                                           |

### 字段与模式

| 方法 / 参数                                                                                                     | 在 Milvus Lite 中支持        |
|--------------------------------------------------------------------------------------------------------------|---------------------------------|
| [create_schema()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md) | 支持有限参数 |
| `auto_id`                                                                                                    | 是                               |
| `enable_dynamic_field`                                                                                       | 是                              |
| `primary_field`                                                                                              | 是                              |
| `partition_key_field`                                                                                        | 否                              |
| [add_field()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/CollectionSchema/add_field.md)    | 支持，但参数有限                |
| `field_name`                                                                                                 | 是                              |
| `datatype`                                                                                                   | 是                              |
| `is_primary`                                                                                                 | 是                              |
| `max_length`                                                                                                 | 是                              |
| `element_type`                                                                                               | 是                              |
| `max_capacity`                                                                                               | 是                              |
| `dim`                                                                                                        | 是                              |
| `is_partition_key`                                                                                           | 否                              |

### 插入 & 搜索

| 方法 / 参数                                                                                      | 在 Milvus Lite 中支持         |
|-------------------------------------------------------------------------------------------|---------------------------------|
| [search()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/search.md) | 支持，但参数有限                |
| `collection_name`                                                                         | 是                              |
| `data`                                                                                    | 是                              |
| `filter`                                                                                  | 是                              |
| `limit`                                                                                   | 是                              |
| `output_fields`                                                                           | 是                              |
| `search_params`                                                                           | 是                              |
| `timeout`                                                                                 | 是                              |
| `partition_names`                                                                         | 否                              |
| `anns_field`                                                                              | 是                              |
| [query()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/query.md)   | 支持有限的参数                 |
| `collection_name`                                                                         | 是                              |
| `filter`                                                                                  | 是                              |
| `output_fields`                                                                           | 是                              |
| `timeout`                                                                                 | 是                              |
| `ids`                                                                                     | 是                              |
| `partition_names`                                                                         | 否                              |
| [get()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/get.md)       | 支持有限的参数                 |
| `collection_name`                                                                         | 是                              |
| `ids`                                                                                     | 是                              |
| `output_fields`                                                                           | 是                              |
| `timeout`                                                                                 | 是                              |
| `partition_names`                                                                         | 否                              |
| [delete()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/delete.md) | 支持有限的参数                 |
| `collection_name`                                                                         | 是                              |
| `ids`                                                                                     | 是                              |
| `timeout`                                                                                 | 是                              |
| `filter`                                                                                  | 是                              |
| `partition_name`                                                                          | 否                              |
### 插入 & 更新

| 方法 / 参数                                                                                                              | Milvus Lite 中支持的参数 |
|-----------------------------------------------------------------------------------------------------------------------|--------------------------|
| [insert()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/insert.md)       | 支持有限参数 |
| `collection_name`                                                                                                     | 是                        |
| `data`                                                                                                                | 是                        |
| `timeout`                                                                                                             | 是                        |
| `partition_name`                                                                                                      | 否                        |
| [upsert()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/upsert.md)       | 支持有限参数 |
| `collection_name`                                                                                                     | 是                        |
| `data`                                                                                                                | 是                        |
| `timeout`                                                                                                             | 是                        |
| `partition_name`                                                                                                      | 否                        |

### 载入 & 释放

| 方法 / 参数                                                                                                              | Milvus Lite 中支持的参数 |
|-----------------------------------------------------------------------------------------------------------------------|--------------------------|
| [load_collection()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/load_collection.md)       | 是                        |
| `collection_name`                                                                                                     | 是                        |
| `timeout`                                                                                                             | 是                        |
| [release_collection()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/release_collection.md) | 是                        |
| `collection_name`                                                                                                     | 是                        |
| `timeout`                                                                                                             | 是                        |
| [get_load_state()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/get_load_state.md)         | 不支持获取载入状态。            |
| [refresh_load()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/refresh_load.md)             | 不支持加载已卸载集合的数据。            |
| [close()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/close.md)                               | 是                        |

### 索引

| 方法 / 参数                                                                                                       | 在 Milvus Lite 中支持                                                                               |
|----------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| [list_indexes()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/list_collections.md) | 支持列出索引。                                                                          |
| `collection_name`                                                                                              | 是                                                                                                      |
| `field_name`                                                                                                   | 是                                                                                                      |
| [create_index()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/create_index.md)      | 仅支持部分索引类型：`FLAT`, `HNSW`, `BIN_FLAT`, `SPARSE_INVERTED_INDEX`, `SPARSE_WAND`. |
| `index_params`                                                                                                 | 是                                                                                                      |
| `timeout`                                                                                                      | 是                                                                                                      |
| [drop_index()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/drop_index.md)          | 支持删除索引。                                                                         |
| `collection_name`                                                                                              | 是                                                                                                      |
| `index_name`                                                                                                   | 是                                                                                                      |
| `timeout`                                                                                                      | 是                                                                                                      |
| [describe_index()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/describe_index.md)  | 支持描述索引。                                                                       |
| `collection_name`                                                                                              | 是                                                                                                      |
| `index_name`                                                                                                   | 是                                                                                                      |
| `timeout`                                                                                                      | 是                                                                                                      |

### 分区

Milvus Lite 不支持分区和与分区相关的方法。

### 用户 & 角色

Milvus Lite 不支持用户和角色以及相关方法。

### 别名

Milvus Lite 不支持别名和与别名相关的方法。

### 其他

对于上表中未列出的其他方法，Milvus Lite 不支持。

## 下一步

连接到 Milvus Lite 后，您可以：

- 查看[快速入门](quickstart.md)以了解 Milvus 的功能。

- 学习 Milvus 的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- [使用 Helm Chart 升级 Milvus](upgrade_milvus_cluster-helm.md)。
- [扩展您的 Milvus 集群](scaleout.md)。
- 在云上部署您的 Milvus 集群：
  - [Amazon EC2](aws.md)
  - [Amazon EKS](eks.md)
  - [Google Cloud](gcp.md)
  - [Google Cloud 存储](gcs.md)
  - [Microsoft Azure](azure.md)
  - [Microsoft Azure Blob 存储](abs.md)
- 探索[Milvus 备份](milvus_backup_overview.md)，这是一个用于 Milvus 数据备份的开源工具。
- 探索[Birdwatcher](birdwatcher_overview.md)，这是一个用于调试 Milvus 和动态配置更新的开源工具。
- 探索[Attu](https://milvus.io/docs/attu.md)，这是一个直观的 Milvus 管理工具。
- [使用 Prometheus 监控 Milvus](monitor.md)。