---
id: index-vector-fields.md
order: 1
summary: 本指南将带您了解在集合中创建和管理向量字段索引的基本操作。
title: 索引向量字段
---

# 索引向量字段

本指南将带您了解在集合中创建和管理向量字段索引的基本操作。

## 概述

Milvus利用索引文件中存储的元数据，以专门的结构组织数据，从而在搜索或查询过程中快速检索所需信息。

Milvus提供多种索引类型和度量标准，以对字段值进行排序，实现高效的相似性搜索。以下表格列出了不同向量字段类型支持的索引类型和度量标准。详情请参考[内存索引](index.md)和[相似性度量](metric.md)。

<div class="filter">
  <a href="#floating">浮点嵌入</a>
  <a href="#binary">二进制嵌入</a>
  <a href="#sparse">稀疏嵌入</a>
</div>

<div class="filter-floating table-wrapper" markdown="block">

<table class="tg">
<thead>
  <tr>
    <th class="tg-0pky" style="width: 204px;">度量类型</th>
    <th class="tg-0pky">索引类型</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-0pky"><ul><li>欧氏距离（L2）</li><li>内积（IP）</li><li>余弦相似度（COSINE）</li></td>
    <td class="tg-0pky" rowspan="2"><ul><li>FLAT</li><li>IVF_FLAT</li><li>IVF_SQ8</li><li>IVF_PQ</li><li>GPU_IVF_FLAT</li><li>GPU_IVF_PQ</li><li>HNSW</li><li>DISKANN</li></ul></td>
  </tr>
</tbody>
</table>

</div>

<div class="filter-binary table-wrapper" markdown="block">

<table class="tg">
<thead>
  <tr>
    <th class="tg-0pky" style="width: 204px;">度量类型</th>
    <th class="tg-0pky">索引类型</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-0pky"><ul><li>Jaccard</li><li>Hamming</li></ul></td>
    <td class="tg-0pky"><ul><li>BIN_FLAT</li><li>BIN_IVF_FLAT</li></ul></td>
  </tr>
</tbody>
</table>

</div>

<div class="filter-sparse table-wrapper" markdown="block">

<table class="tg">
<thead>
  <tr>
    <th class="tg-0pky" style="width: 204px;">度量类型</th>
    <th class="tg-0pky">索引类型</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-0pky">IP</td>
    <td class="tg-0pky"><ul><li>SPARSE_INVERTED_INDEX</li><li>SPARSE_WAND</li></ul></td>
  </tr>
</tbody>
</table>

</div>

建议为经常访问的向量字段和标量字段创建索引。

## 准备工作

如[管理集合](manage-collections.md)中所述，如果在创建集合时指定以下任一条件，则Milvus会自动生成索引并将其加载到内存中：

- 向量字段的维度和度量类型，或

- 模式和索引参数。
下面的代码片段重新利用现有代码来建立与 Milvus 实例的连接，并创建一个没有指定索引参数的集合。在这种情况下，该集合缺乏索引并保持未加载状态。

<div class="language-python">

要为索引做准备，使用 [`MilvusClient`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md) 连接到 Milvus 服务器，并通过 [`create_schema()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md)、[`add_field()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/CollectionSchema/add_field.md) 和 [`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md) 设置集合。

</div>

<div class="language-java">

要为索引做准备，使用 [`MilvusClientV2`](https://milvus.io/api-reference/java/v2.4.x/v2/Client/MilvusClientV2.md) 连接到 Milvus 服务器，并通过 [`createSchema()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createSchema.md)、[`addField()`](https://milvus.io/api-reference/java/v2.4.x/v2/CollectionSchema/addField.md) 和 [`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md) 设置集合。

</div>

<div class="language-javascript">

要为索引做准备，使用 [`MilvusClient`](https://milvus.io/api-reference/node/v2.4.x/Client/MilvusClient.md) 连接到 Milvus 服务器，并通过 [`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md) 设置集合。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
from pymilvus import MilvusClient, DataType

# 1. 设置 Milvus 客户端
client = MilvusClient(
    uri="http://localhost:19530"
)

# 2. 创建模式
# 2.1. 创建模式
schema = MilvusClient.create_schema(
    auto_id=False,
    enable_dynamic_field=True,
)

# 2.2. 向模式添加字段
schema.add_field(field_name="id", datatype=DataType.INT64, is_primary=True)
schema.add_field(field_name="vector", datatype=DataType.FLOAT_VECTOR, dim=5)

# 3. 创建集合
client.create_collection(
    collection_name="customized_setup", 
    schema=schema, 
)
```
```java
import io.milvus.v2.client.ConnectConfig;
import io.milvus.v2.client.MilvusClientV2;
import io.milvus.v2.common.DataType;
import io.milvus.v2.service.collection.request.CreateCollectionReq;

String CLUSTER_ENDPOINT = "http://localhost:19530";

// 1. 连接到 Milvus 服务器
ConnectConfig connectConfig = ConnectConfig.builder()
    .uri(CLUSTER_ENDPOINT)
    .build();

MilvusClientV2 client = new MilvusClientV2(connectConfig);

// 2. 创建一个集合

// 2.1 创建模式
CreateCollectionReq.CollectionSchema schema = client.createSchema();

// 2.2 向模式添加字段
schema.addField(AddFieldReq.builder().fieldName("id").dataType(DataType.Int64).isPrimaryKey(true).autoID(false).build());
schema.addField(AddFieldReq.builder().fieldName("vector").dataType(DataType.FloatVector).dimension(5).build());

// 3. 创建一个不带模式和索引参数的集合
CreateCollectionReq customizedSetupReq = CreateCollectionReq.builder()
.collectionName("customized_setup")
.collectionSchema(schema)
.build();

client.createCollection(customizedSetupReq);
```
```javascript
// 1. 设置 Milvus 客户端
client = new MilvusClient({address, token});

// 2. 为集合定义字段
const fields = [
    {
        name: "id",
        data_type: DataType.Int64,
        is_primary_key: true,
        auto_id: false
    },
    {
        name: "vector",
        data_type: DataType.FloatVector,
        dim: 5
    },
]

// 3. 创建集合
res = await client.createCollection({
    collection_name: "customized_setup",
    fields: fields,
})

console.log(res.error_code)  

// 输出
// 
// 成功
// 
```

## 为集合建立索引

<div class="language-python">

要为集合创建索引或对集合建立索引，请使用 [`prepare_index_params()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/prepare_index_params.md) 准备索引参数，使用 [`create_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/create_index.md) 创建索引。

</div>

<div class="language-java">

要为集合创建索引或对集合建立索引，请使用 [`IndexParam`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/IndexParam.md) 准备索引参数，使用 [`createIndex()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/createIndex.md) 创建索引。

</div>

<div class="language-javascript">

要为集合创建索引或对集合建立索引，请使用 [`createIndex()`](https://milvus.io/api-reference/node/v2.4.x/Management/createIndex.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 4.1. 设置索引参数
index_params = MilvusClient.prepare_index_params()

# 4.2. 在向量字段上添加索引
index_params.add_index(
    field_name="vector",
    metric_type="COSINE",
    index_type="IVF_FLAT",
    index_name="vector_index",
    params={ "nlist": 128 }
)

# 4.3. 创建索引文件
client.create_index(
    collection_name="customized_setup",
    index_params=index_params
)
```

```java
import io.milvus.v2.common.IndexParam;
import io.milvus.v2.service.index.request.CreateIndexReq;

// 4 准备索引参数

// 4.2 为向量字段 "vector" 添加索引
IndexParam indexParamForVectorField = IndexParam.builder()
    .fieldName("vector")
    .indexName("vector_index")
    .indexType(IndexParam.IndexType.IVF_FLAT)
    .metricType(IndexParam.MetricType.COSINE)
    .extraParams(Map.of("nlist", 128))
    .build();

List<IndexParam> indexParams = new ArrayList<>();
indexParams.add(indexParamForVectorField);

// 4.3 创建索引文件
CreateIndexReq createIndexReq = CreateIndexReq.builder()
    .collectionName("customized_setup")
    .indexParams(indexParams)
    .build();

client.createIndex(createIndexReq);
```
```
```javascript
// 4. 为集合设置索引
// 4.1. 设置索引参数
res = await client.createIndex({
    collection_name: "customized_setup",
    field_name: "vector",
    index_type: "AUTOINDEX",
    metric_type: "COSINE",   
    index_name: "vector_index",
    params: { "nlist": 128 }
})

console.log(res.error_code)

// 输出
// 
// 成功
// 
```

<div class="admonition note">

<p><b>注意</b></p>

<p>目前，您只能为集合中的每个字段创建一个索引文件。</p>

</div>

## 检查索引详情

创建索引后，您可以检查其详情。

<div class="language-python">

要检查索引详情，请使用[`list_indexes()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/list_indexes.md)列出索引名称，使用[`describe_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/describe_index.md)获取索引详情。

</div>

<div class="language-java">

要检查索引详情，请使用[`describeIndex()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/describeIndex.md)获取索引详情。

</div>

<div class="language-javascript">

要检查索引详情，请使用[`describeIndex()`](https://milvus.io/api-reference/node/v2.4.x/Management/describeIndex.md)获取索引详情。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 5. 描述索引
res = client.list_indexes(
    collection_name="customized_setup"
)

print(res)

# 输出
#
# [
#     "vector_index",
# ]

res = client.describe_index(
    collection_name="customized_setup",
    index_name="vector_index"
)

print(res)

# 输出
#
# {
#     "index_type": ,
#     "metric_type": "COSINE",
#     "field_name": "vector",
#     "index_name": "vector_index"
# }
```

```java
import io.milvus.v2.service.index.request.DescribeIndexReq;
import io.milvus.v2.service.index.response.DescribeIndexResp;

// 5. 描述索引
// 5.1 列出索引名称
ListIndexesReq listIndexesReq = ListIndexesReq.builder()
    .collectionName("customized_setup")
    .build();

List<String> indexNames = client.listIndexes(listIndexesReq);

System.out.println(indexNames);

// 输出:
// [
//     "vector_index"
// ]

// 5.2 描述索引
DescribeIndexReq describeIndexReq = DescribeIndexReq.builder()
    .collectionName("customized_setup")
    .indexName("vector_index")
    .build();

DescribeIndexResp describeIndexResp = client.describeIndex(describeIndexReq);

System.out.println(JSONObject.toJSON(describeIndexResp));

// 输出:
// {
//     "metricType": "COSINE",
//     "indexType": "AUTOINDEX",
//     "fieldName": "vector",
//     "indexName": "vector_index"
// }
```
```javascript
// 5. 描述索引
res = await client.describeIndex({
    collection_name: "customized_setup",
    index_name: "vector_index"
})

console.log(JSON.stringify(res.index_descriptions, null, 2))

// 输出
// 
// [
//   {
//     "params": [
//       {
//         "key": "index_type",
//         "value": "AUTOINDEX"
//       },
//       {
//         "key": "metric_type",
//         "value": "COSINE"
//       }
//     ],
//     "index_name": "vector_index",
//     "indexID": "449007919953063141",
//     "field_name": "vector",
//     "indexed_rows": "0",
//     "total_rows": "0",
//     "state": "Finished",
//     "index_state_fail_reason": "",
//     "pending_index_rows": "0"
//   }
// ]
// 
```
你可以检查在特定字段上创建的索引文件，并收集使用该索引文件索引的行数统计。

## 删除索引

如果不再需要索引，可以简单地删除它。

<div class="language-python">

要删除索引，请使用 [`drop_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/drop_index.md)。

</div>

<div class="language-java">

要删除索引，请使用 [`dropIndex()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/dropIndex.md)。

</div>

<div class="language-javascript">

要删除索引，请使用 [`dropIndex()`](https://milvus.io/api-reference/node/v2.4.x/Management/dropIndex.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 6. 删除索引
client.drop_index(
    collection_name="customized_setup",
    index_name="vector_index"
)
```

```java
// 6. 删除索引

DropIndexReq dropIndexReq = DropIndexReq.builder()
    .collectionName("customized_setup")
    .indexName("vector_index")
    .build();

client.dropIndex(dropIndexReq);
```

```javascript
// 6. 删除索引
res = await client.dropIndex({
    collection_name: "customized_setup",
    index_name: "vector_index"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 
```
