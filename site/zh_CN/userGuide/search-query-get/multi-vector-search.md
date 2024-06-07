---
id: multi-vector-search.md
order: 2
summary: 本指南演示了如何在 Milvus 中执行混合搜索，并理解结果的重新排序。
title: 混合搜索
---

# 混合搜索

自 Milvus 2.4 版本开始，我们引入了多向量支持和混合搜索框架，这意味着用户可以将多个向量字段（最多达到 10 个）引入单个集合中。不同的向量字段可以代表不同的方面、不同的嵌入模型，甚至是表征同一实体的不同数据模态，极大地扩展了信息的丰富性。这一特性在综合搜索场景中特别有用，比如基于各种属性（如图片、声音、指纹等）识别向量库中最相似的人。

混合搜索使得可以在各种向量字段上执行搜索请求，并利用重新排序策略（如 Reciprocal Rank Fusion（RRF）和加权评分）合并结果。要了解更多有关重新排序策略的信息，请参考 [重新排序](reranking.md)。

在本教程中，您将学习如何：

- 为不同向量字段上的相似性搜索创建多个 `AnnSearchRequest` 实例；

- 配置重新排序策略，以合并和重新排序来自多个 `AnnSearchRequest` 实例的搜索结果；

- 使用 [`hybrid_search()`](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/Collection/hybrid_search.md) 方法执行混合搜索。

<div class="alert note">

本页上的代码片段使用 [PyMilvus ORM 模块](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/Connections/connect.md) 与 Milvus 进行交互。将很快提供使用新的 [MilvusClient SDK](https://milvus.io/api-reference/pymilvus/v2.4.x/About.md) 的代码片段。

</div>

## 准备工作

在开始混合搜索之前，请确保您有一个包含多个向量字段的集合。

以下是一个示例，展示了如何创建一个名为 `test_collection` 的集合，其中包含两个向量字段 `filmVector` 和 `posterVector`，并向其中插入随机实体。

```python
from pymilvus import connections, Collection, FieldSchema, CollectionSchema, DataType
import random

# 连接到 Milvus
connections.connect(
    host="10.102.7.3", # 请替换为您的 Milvus 服务器 IP
    port="19530"
)

# 创建模式
fields = [
    FieldSchema(name="film_id", dtype=DataType.INT64, is_primary=True),
    FieldSchema(name="filmVector", dtype=DataType.FLOAT_VECTOR, dim=5), # 电影向量的向量字段
    FieldSchema(name="posterVector", dtype=DataType.FLOAT_VECTOR, dim=5)] # 海报向量的向量字段

schema = CollectionSchema(fields=fields,enable_dynamic_field=False)

# 创建集合
collection = Collection(name="test_collection", schema=schema)

# 为每个向量字段创建索引
index_params = {
    "metric_type": "L2",
    "index_type": "IVF_FLAT",
    "params": {"nlist": 128},
}

collection.create_index("filmVector", index_params)
collection.create_index("posterVector", index_params)

# 生成随机实体进行插入
entities = []

for _ in range(1000):
    # 为模式中的每个字段生成随机值
    film_id = random.randint(1, 1000)
    film_vector = [ random.random() for _ in range(5) ]
    poster_vector = [ random.random() for _ in range(5) ]

    # 为每个实体创建一个字典
    entity = {
        "film_id": film_id,
        "filmVector": film_vector,
        "posterVector": poster_vector
    }

    # 将实体添加到列表中
    entities.append(entity)
    
collection.insert(entities)
```
## 第一步：创建多个 AnnSearchRequest 实例

多向量搜索使用 `hybrid_search()` API 在单个调用中执行多个近似最近邻（ANN）搜索请求。每个 `AnnSearchRequest` 表示对特定向量字段进行的单个搜索请求。

以下示例创建了两个 `AnnSearchRequest` 实例，以在两个向量字段上执行各自的相似度搜索。

```python
from pymilvus import AnnSearchRequest

# 为 filmVector 创建 ANN 搜索请求 1
query_filmVector = [[0.8896863042430693, 0.370613100114602, 0.23779315077113428, 0.38227915951132996, 0.5997064603128835]]

search_param_1 = {
    "data": query_filmVector, # 查询向量
    "anns_field": "filmVector", # 向量字段名称
    "param": {
        "metric_type": "L2", # 此参数值必须与集合模式中使用的值相同
        "params": {"nprobe": 10}
    },
    "limit": 2 # 在此 AnnSearchRequest 中返回的搜索结果数量
}
request_1 = AnnSearchRequest(**search_param_1)

# 为 posterVector 创建 ANN 搜索请求 2
query_posterVector = [[0.02550758562349764, 0.006085637357292062, 0.5325251250159071, 0.7676432650114147, 0.5521074424751443]]
search_param_2 = {
    "data": query_posterVector, # 查询向量
    "anns_field": "posterVector", # 向量字段名称
    "param": {
        "metric_type": "L2", # 此参数值必须与集合模式中使用的值相同
        "params": {"nprobe": 10}
    },
    "limit": 2 # 在此 AnnSearchRequest 中返回的搜索结果数量
}
request_2 = AnnSearchRequest(**search_param_2)

# 将这两个请求存储为 `reqs` 中的列表
reqs = [request_1, request_2]
```

参数：

- `AnnSearchRequest`（_对象_）

    表示一个 ANN 搜索请求的类。每个混合搜索可以同时包含 1 到 1,024 个 `ANNSearchRequest` 对象。

- `data`（_列表_）

    要在单个 `AnnSearchRequest` 中搜索的查询向量。当前，此参数接受包含单个查询向量的列表，例如 `[[0.5791814851218929, 0.5792985702614121, 0.8480776460143558, 0.16098005945243, 0.2842979317256803]]`。将来，此参数将扩展为接受多个查询向量。

- `anns_field`（_字符串_）

    在单个 `AnnSearchRequest` 中使用的向量字段的名称。

- `param`（_字典_）

    单个 `AnnSearchRequest` 的搜索参数字典。这些搜索参数与单向量搜索的参数相同。有关更多信息，请参阅[搜索参数](https://milvus.io/docs/single-vector-search.md#Search-parameters)。

- `limit`（_整数_）

    在单个 `ANNSearchRequest` 中包含的最大搜索结果数量。
这个参数仅影响在单个`ANNSearchRequest`中返回的搜索结果数量，并不决定`hybrid_search`调用返回的最终结果。在混合搜索中，最终结果是通过合并和重新排名来自多个`ANNSearchRequest`实例的结果来确定的。

## 步骤2：配置重新排名策略

创建`AnnSearchRequest`实例后，配置一个重新排名策略来合并和重新排名结果。目前有两个选项：`WeightedRanker`和`RRFRanker`。有关重新排名策略的更多信息，请参阅[Reranking](reranking.md)。

- 使用加权评分

    `WeightedRanker`用于为每个向量字段搜索结果分配指定权重的重要性。如果您优先考虑某些向量字段，`WeightedRanker(value1, value2, ..., valueN)`可以在组合搜索结果中反映这一点。

    ```python
    from pymilvus import WeightedRanker
    # 使用WeightedRanker来使用指定权重合并结果
    # 将0.8的权重分配给文本搜索，0.2分配给图像搜索
    rerank = WeightedRanker(0.8, 0.2)  
    ```

    使用`WeightedRanker`时，请注意：

  - 每个权重值的范围从0（最不重要）到1（最重要），影响最终的聚合分数。
  - 在`WeightedRanker`中提供的权重值总数应等于您创建的`AnnSearchRequest`实例的数量。

- 使用倒数秩融合（RFF）

    ```python
    # 或者，使用RRFRanker进行倒数秩融合重新排名
    from pymilvus import RRFRanker
    
    rerank = RRFRanker()
    ```

## 步骤3：执行混合搜索

设置了`AnnSearchRequest`实例和重新排名策略后，使用`hybrid_search()`方法执行混合搜索。

```python
# 在进行混合搜索之前，将集合加载到内存中。
collection.load()

res = collection.hybrid_search(
    reqs, # 在步骤1中创建的AnnSearchRequest列表
    rerank, # 步骤2中指定的重新排名策略
    limit=2 # 返回的最终搜索结果数量
)

print(res)
```

参数：

- `reqs`（_list_）

  一个搜索请求列表，其中每个请求都是一个`ANNSearchRequest`对象。每个请求可以对应不同的向量字段和不同的搜索参数集。

- `rerank`（_object_）

  用于混合搜索的重新排名策略。可能的值：`WeightedRanker(value1, value2, ..., valueN)`和`RRFRanker()`。

    有关重新排名策略的更多信息，请参阅[Reranking](reranking.md)。

- `limit`（_int_）

    在混合搜索中返回的最终结果的最大数量。

输出类似于以下内容：

```python
["['id: 844, distance: 0.006047376897186041, entity: {}', 'id: 876, distance: 0.006422005593776703, entity: {}']"]
```

## 限制
- 通常，每个集合默认允许最多有4个向量字段。但是，您可以调整 `proxy.maxVectorFieldNum` 配置以扩展集合中向量字段的最大数量，最大限制为每个集合最多10个向量字段。更多信息请参阅[代理相关配置](https://milvus.io/docs/configure_proxy.md#Proxy-related-Configurations)。

- 集合中部分索引或加载的向量字段将导致错误。

- 目前，混合搜索中每个 `AnnSearchRequest` 只能携带一个查询向量。

## 常见问题解答

- **在哪些场景下推荐使用混合搜索？**

    混合搜索非常适用于需要高准确性的复杂情况，特别是当一个实体可以由多个不同的向量表示时。这适用于同一数据（如句子）通过不同的嵌入模型处理，或者将多模态信息（如个体的图像、指纹和声纹）转换为不同的向量格式的情况。通过为这些向量分配权重，它们的综合影响可以显著丰富召回率，并提高搜索结果的有效性。

- **加权排序器如何归一化不同向量字段之间的距离？**

    加权排序器使用为每个字段分配的权重来归一化向量字段之间的距离。它根据权重计算每个向量字段的重要性，优先考虑那些具有较高权重的字段。建议在ANN搜索请求中使用相同的度量类型以确保一致性。这种方法确保被视为更重要的向量对整体排序产生更大影响。

- **是否可以使用类似 Cohere Ranker 或 BGE Ranker 的替代排序器？**

    目前，仅支持提供的排序器。计划在未来更新中包括其他排序器。

- **是否可以同时执行多个混合搜索操作？**

    是的，支持同时执行多个混合搜索操作。

- **我可以在多个 AnnSearchRequest 对象中使用相同的向量字段执行混合搜索吗？**

    从技术上讲，可以在多个 AnnSearchRequest 对象中使用相同的向量字段执行混合搜索。对于混合搜索，不需要拥有多个向量字段。