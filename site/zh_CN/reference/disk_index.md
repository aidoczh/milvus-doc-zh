---
id: disk_index.md
related_key: disk_index
summary: Milvus 中的磁盘索引机制。
title: 磁盘索引
---

# 磁盘索引

本文介绍了一种名为 DiskANN 的磁盘索引算法。基于 Vamana 图，DiskANN 可以在大型数据集内实现高效搜索。

为了提高查询性能，您可以为每个向量字段[指定索引类型](index-vector-fields.md)。

<div class="alert note"> 
目前，一个向量字段仅支持一种索引类型。当切换索引类型时，Milvus 会自动删除旧索引。
</div>

## 先决条件

要使用 DiskANN，请注意：
- DiskANN 默认已启用。如果您更喜欢内存索引而不是磁盘索引，建议您禁用此功能以获得更好的性能。
  - 要禁用它，您可以在 milvus 配置文件中将 `queryNode.enableDisk` 更改为 `false`。
  - 要重新启用它，您可以将 `queryNode.enableDisk` 设置为 `true`。
- Milvus 实例在 Ubuntu 18.04.6 或更高版本上运行。
- Milvus 数据路径应挂载到 NVMe SSD 以获得最佳性能：
  - 对于 Milvus 独立实例，数据路径应为实例运行的容器中的 **/var/lib/milvus/data**。
  - 对于 Milvus 集群实例，数据路径应为 QueryNodes 和 IndexNodes 运行的容器中的 **/var/lib/milvus/data**。

## 限制

要使用 DiskANN，请确保：
- 数据中仅使用至少 1 维的浮点向量。
- 仅使用欧氏距离（L2）或内积（IP）来衡量向量之间的距离。

## 索引和搜索设置

 - 索引构建参数

   在构建 DiskANN 索引时，使用 `DISKANN` 作为索引类型。不需要任何索引参数。

- 搜索参数

  | 参数          | 描述                               | 范围                                           |
  | ------------- | ----------------------------------- | ----------------------------------------------- |
  | `search_list` | 候选列表的大小，较大的大小提供更高的召回率，但性能会下降。 | [topk, int32_max] |

## 与 DiskANN 相关的 Milvus 配置

DiskANN 是可调节的。您可以修改 `${MILVUS_ROOT_PATH}/configs/milvus.yaml` 中的 DiskANN 相关参数以提高其性能。

```YAML
...
DiskIndex:
  MaxDegree: 56
  SearchListSize: 100
  PQCodeBugetGBRatio: 0.125
  SearchCacheBudgetGBRatio: 0.125
  BeamWidthRatio: 4.0
...
```

| 参数        | 描述       | 值范围     | 默认值 |
| ---         | ---        | ---        | ---    |
| `MaxDegree` | Vamana 图的最大度数。 <br> 较大的值提供更高的召回率，但会增加索引的大小和构建时间。 | [1, 512] | 56 | 
| `SearchListSize` | 候选列表的大小。 <br> 较大的值会增加构建索引的时间，但提供更高的召回率。 <br> 将其设置为小于 `MaxDegree` 的值，除非您需要减少构建索引的时间。 | [1, int32_max] | 100 |
| `PQCodeBugetGBRatio` | PQ码的大小限制。较大的值提供更高的召回率，但会增加内存使用量。 | (0.0, 0.25] | 0.125 |
| `SearchCacheBudgetGBRatio` | 缓存节点数与原始数据的比率。较大的值可以提高索引构建性能，但会增加内存使用量。 | [0.0, 0.3) | 0.10 |
| `BeamWidthRatio` | 每次搜索迭代中最大IO请求数与CPU数量之间的比率。 | [1, max(128 / CPU数量, 16)] | 4.0 |

## 故障排除

- 如何处理 `io_setup() failed; returned -11, errno=11:Resource temporarily unavailable` 错误？

  Linux内核提供了异步非阻塞I/O（AIO）功能，允许进程同时启动多个I/O操作，而无需等待它们中的任何一个完成。这有助于提高那些可以重叠处理和I/O的应用程序的性能。

  可以通过proc文件系统中的 `/proc/sys/fs/aio-max-nr` 虚拟文件来调整性能。`aio-max-nr` 参数确定了允许的最大并发请求数。

  `aio-max-nr` 默认为 `65535`，您可以将其设置为 `10485760`。