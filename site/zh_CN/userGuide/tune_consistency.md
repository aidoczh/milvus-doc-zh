---
id: tune_consistency.md
related_key: 调整一致性
title: 调整一致性
summary: 学习如何在 Milvus 中调整一致性级别。
---

# 调整一致性

Milvus 支持在创建集合、进行向量查询以及进行向量搜索（目前仅在 PyMilvus 中）时设置一致性级别。Milvus 支持四种一致性级别：`Strong`、`Eventual`、`Bounded` 和 `Session`。默认情况下，未指定一致性级别的集合将被设置为 `Bounded` 一致性级别。本主题描述了如何调整一致性级别。

## 配置参数

默认情况下，一致性级别设置为 `Bounded`，在此级别下，当进行搜索或查询请求时，Milvus 会读取一个较早更新的数据视图（通常是几秒钟之前）。您可以通过配置参数 `consistency_level` 来设置一致性级别，该参数在创建集合和进行搜索或查询时使用。有关背后机制，请参阅[在搜索请求中保证时间戳](https://github.com/milvus-io/milvus/blob/master/docs/developer_guides/how-guarantee-ts-works.md)。

<table class="language-python">
        <thead>
        <tr>
            <th>参数</th>
            <th>描述</th>
            <th>选项</th>
        </tr>
        </thead>
        <tbody>
        <tr>
            <td><code>consistency_level</code></td>
            <td>要创建的集合的一致性级别。</td>
            <td>
                <ul>
                    <li><code>Strong</code></li>
                    <li><code>Bounded</code></li>
                    <li><code>Session</code></li>
                    <li><code>Eventually</code></li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>

#### 示例

以下示例将一致性级别设置为 `Strong`，这意味着 Milvus 将在搜索或查询请求到达时精确时间点读取最新的数据视图。在搜索或查询请求中设置的一致性级别会覆盖在创建集合时设置的级别。在搜索或查询时未指定一致性级别时，Milvus 会采用集合的原始一致性级别。

- **创建集合**

```python
from pymilvus import Collection
collection = Collection(
    name=collection_name, 
    schema=schema, 
    using='default', 
    shards_num=2,
    consistency_level="Strong"
    )
```

- **进行向量搜索**

```python
result = hello_milvus.search(
        vectors_to_search,
        "embeddings",
        search_params,
        limit=3,
        output_fields=["random"],
        # 搜索将扫描插入到 Milvus 中的所有实体。
        consistency_level="Strong",
        )
```

- **进行向量查询**    

```python
res = collection.query(
  expr = "book_id in [2,4,6,8]", 
  output_fields = ["book_id", "book_intro"],
  consistency_level="Strong"
)
```