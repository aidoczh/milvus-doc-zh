---
id: index-with-gpu.md
order: 3
summary: 本指南介绍了如何在 Milvus 中使用 GPU 支持构建索引，以提高搜索性能。
title: 使用 GPU 构建索引
---

# 使用 GPU 构建索引

本指南概述了在 Milvus 中使用 GPU 支持构建索引的步骤，可以显著提高高吞吐量和高召回率场景下的搜索性能。有关 Milvus 支持的 GPU 索引类型的详细信息，请参阅 [GPU 索引](gpu_index.md)。

## 配置 Milvus 设置以控制 GPU 内存

Milvus 使用全局图形内存池来分配 GPU 内存。

它在 [Milvus 配置文件](https://github.com/milvus-io/milvus/blob/master/configs/milvus.yaml#L767-L769) 中支持两个参数 `initMemSize` 和 `maxMemSize`。内存池的大小最初设置为 `initMemSize`，并在超过此限制后自动扩展到 `maxMemSize`。

默认情况下，`initMemSize` 在 Milvus 启动时设置为可用 GPU 内存的一半，而 `maxMemSize` 默认等于所有可用 GPU 内存。

```yaml
# 当使用 GPU 索引时，Milvus 将利用内存池来避免频繁的内存分配和释放。
# 在这里，您可以设置内存池占用的内存大小，单位为 MB。
# 请注意，当实际内存需求超过 maxMemSize 设置的值时，Milvus 可能会崩溃。
# 如果 initMemSize 和 MaxMemSize 都设置为零，
# Milvus 将自动初始化一半可用的 GPU 内存，
# maxMemSize 将默认设置为整个可用的 GPU 内存。
gpu:
  initMemSize: 0 # 设置初始内存池大小。
  maxMemSize: 0 # maxMemSize 设置最大内存使用限制。当内存使用超过 initMemSize 时，Milvus 将尝试扩展内存池。
```

## 构建索引

以下示例演示了如何构建不同类型的 GPU 索引。

### 准备索引参数

在设置 GPU 索引参数时，定义 __index_type__、__metric_type__ 和 __params__：

- __index_type__（_字符串_）：用于加速向量搜索的索引类型。有效选项包括 __GPU_CAGRA__、__GPU_IVF_FLAT__、__GPU_IVF_PQ__ 和 __GPU_BRUTE_FORCE__。

- __metric_type__（_字符串_）：用于衡量向量相似性的度量类型。有效选项为 __IP__ 和 __L2__。

- __params__（_字典_）：索引特定的构建参数。此参数的有效选项取决于索引类型。

以下是不同索引类型的示例配置：

- __GPU_CAGRA__ 索引

    ```python
    index_params = {
        "metric_type": "L2",
        "index_type": "GPU_CAGRA",
        "params": {
            'intermediate_graph_degree': 64,
            'graph_degree': 32
        }
    }
    ```

    __params__ 的可能选项包括：

    - __intermediate_graph_degree__（_整数_）：通过确定修剪前图的度数来影响召回率和构建时间。推荐值为 __32__ 或 __64__。

- __graph_degree__（_整数_）：通过在修剪后设置图的度数，影响搜索性能和召回率。通常情况下，它是__intermediate_graph_degree__的一半。这两个度数之间的差异越大，建立时间就越长。其值必须小于__intermediate_graph_degree__的值。

- __build_algo__（_字符串_）：在修剪之前选择图生成算法。可能的选项：

    - __IVF_PQ__：提供更高质量但建立时间较慢。

    - __NN_DESCENT__：提供更快的建立速度，但可能召回率较低。

- __cache_dataset_on_device__（_字符串_，__"true"__ | __"false"__）：决定是否将原始数据集缓存在 GPU 内存中。将其设置为__"true"__可以通过优化搜索结果来增强召回率，而将其设置为__"false"__可以节省 GPU 内存。

- __GPU_IVF_FLAT__或__GPU_IVF_PQ__索引

    ```python
    index_params = {
        "metric_type": "L2",
        "index_type": "GPU_IVF_FLAT", # 或 GPU_IVF_PQ
        "params": {
            "nlist": 1024
        }
    }
    ```

    __params__选项与__[IVF_FLAT](https://milvus.io/docs/index.md#IVF_FLAT)__和__[IVF_PQ](https://milvus.io/docs/index.md#IVF_PQ)__中使用的选项相同。

- __GPU_BRUTE_FORCE__索引

    ```python
    index_params = {
        'index_type': 'GPU_BRUTE_FORCE',
        'metric_type': 'L2',
        'params': {}
    }
    ```

    不需要额外的__params__配置。

### 建立索引

在__index_params__中配置索引参数后，调用[`create_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/Collection/create_index.md)方法来构建索引。

```python
# 获取现有集合
collection = Collection("YOUR_COLLECTION_NAME")

collection.create_index(
    field_name="vector", # 构建索引的向量字段名称
    index_params=index_params
)
```

## 搜索

建立 GPU 索引后，下一步是在进行搜索之前准备搜索参数。

### 准备搜索参数

以下是不同索引类型的示例配置：

- __GPU_BRUTE_FORCE__索引

    ```python
    search_params = {
        "metric_type": "L2",
        "params": {}
    }
    ```

    不需要额外的__params__配置。

- __GPU_CAGRA__索引

    ```python
    search_params = {
        "metric_type": "L2",
        "params": {
            "itopk_size": 128,
            "search_width": 4,
            "min_iterations": 0,
            "max_iterations": 0,
            "team_size": 0
        }
    }
    ```

    关键搜索参数包括：

    - __itopk_size__：确定搜索过程中保留的中间结果的大小。较大的值可能会提高召回率，但会降低搜索性能。它应至少等于最终的 top-k（__limit__）值，通常是 2 的幂（例如 16、32、64、128）。
    - __search_width__：指定在搜索过程中进入 CAGRA 图的入口数量。增加此值可以增强召回率，但可能会影响搜索性能。

    - __min_iterations__ / __max_iterations__：这些参数控制搜索迭代过程。默认情况下，它们设置为 __0__，CAGRA 会根据 __itopk_size__ 和 __search_width__ 自动确定迭代次数。手动调整这些值可以帮助平衡性能和准确性。

    - __team_size__：指定用于在 GPU 上计算度量距离的 CUDA 线程数。常见值是 2 的幂次方，最大为 32（例如 2、4、8、16、32）。它对搜索性能影响较小。默认值为 __0__，Milvus 会根据向量维度自动选择 __team_size__。

- __GPU_IVF_FLAT__ 或 __GPU_IVF_PQ__ 索引

    ```python
    search_params = {
        "metric_type": "L2", 
        "params": {"nprobe": 10}
    }
    ```

    这两种索引类型的搜索参数与 __[IVF_FLAT](https://milvus.io/docs/index.md#IVF_FLAT)__ 和 __[IVF_PQ](https://milvus.io/docs/index.md#IVF_PQ)__ 中使用的参数类似。有关更多信息，请参阅 [进行向量相似度搜索](https://milvus.io/docs/search.md#Prepare-search-parameters)。

### 进行搜索

使用 [`search()`](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/Collection/search.md) 方法在 GPU 索引上执行向量相似度搜索。

```python
# 将数据加载到内存中
collection.load()

collection.search(
    data=[[query_vector]], # 您的查询向量
    anns_field="vector", # 向量字段的名称
    param=search_params,
    limit=100 # 要返回的结果数量
)
```

## 限制

在使用 GPU 索引时，请注意以下限制：

- 对于 __GPU_IVF_FLAT__，__limit__ 的最大值为 256。

- 对于 __GPU_IVF_PQ__ 和 __GPU_CAGRA__，__limit__ 的最大值为 1024。

- 尽管 __GPU_BRUTE_FORCE__ 的 __limit__ 没有设定限制，但建议不要超过 4096，以避免潜在的性能问题。

- 目前，GPU 索引不支持余弦距离。如果需要余弦距离，应首先对数据进行归一化，然后可以使用内积距离作为替代。

- GPU 索引的 OOM 保护功能支持不完整，过多的数据可能导致 QueryNode 崩溃。

- GPU 索引不支持像 [范围搜索](https://milvus.io/docs/single-vector-search.md#Range-search) 和 [分组搜索](https://milvus.io/docs/single-vector-search.md#Grouping-searchh) 这样的搜索功能。

## 常见问题

- __何时适合使用 GPU 索引？__
GPU 索引在需要高吞吐量或高召回率的情况下特别有益。例如，在处理大批量数据时，GPU 索引的吞吐量可以比 CPU 索引高出多达 100 倍。在小批量数据场景中，GPU 索引在性能方面仍然明显优于 CPU 索引。此外，如果需要快速数据插入，引入 GPU 可大大加快构建索引的过程。

- **在哪些场景下 GPU 索引如 CAGRA、GPU_IVF_PQ、GPU_IVF_FLAT 和 GPU_BRUTE_FORCE 最适用？**

    CAGRA 索引非常适合需要提高性能的场景，尽管会消耗更多内存。对于内存保护至关重要的环境，GPU_IVF_PQ 索引可以帮助减少存储需求，尽管这会导致更高的精度损失。GPU_IVF_FLAT 索引提供了一个平衡的选择，在性能和内存使用之间取得了折衷。最后，GPU_BRUTE_FORCE 索引专为穷举搜索操作而设计，通过执行遍历搜索来保证召回率为 1。