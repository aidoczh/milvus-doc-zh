---
id: index-scalar-fields.md
order: 2
summary: 本指南将引导您创建和配置标量字段的索引，例如整数、字符串等。
title: 标量字段索引
---

# 标量字段索引

在 Milvus 中，标量索引用于加速通过特定非向量字段值进行的元过滤，类似于传统数据库索引。本指南将引导您创建和配置标量字段的索引，例如整数、字符串等。

## 标量索引类型

- __[自动索引](https://milvus.io/docs/index-scalar-fields.md#Auto-indexing)__: Milvus 根据标量字段的数据类型自动决定索引类型。当您不需要控制特定索引类型时，这是合适的选择。

- __[自定义索引](https://milvus.io/docs/index-scalar-fields.md#Custom-indexing)__: 您可以指定精确的索引类型，例如倒排索引。这提供了对索引类型选择更多的控制。

## 自动索引

<div class="language-python">

要使用自动索引，在 [`add_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/add_index.md) 中省略 __index_type__ 参数，这样 Milvus 可以根据标量字段类型推断索引类型。

</div>

<div class="language-java">

要使用自动索引，在 [`IndexParam`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/IndexParam.md) 中省略 __indexType__ 参数，这样 Milvus 可以根据标量字段类型推断索引类型。

</div>

<div class="language-javascript">

要使用自动索引，在 [`createIndex()`](https://milvus.io/api-reference/node/v2.4.x/Management/createIndex.md) 中省略 __index_type__ 参数，这样 Milvus 可以根据标量字段类型推断索引类型。

</div>

有关标量数据类型与默认索引算法之间的映射，请参阅[标量字段索引算法](https://milvus.io/docs/scalar_index.md#Scalar-field-indexing-algorithms)。

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 自动索引
client = MilvusClient(
    uri="http://localhost:19530"
)

index_params = client.create_index_params() # 准备一个空的 IndexParams 对象，无需指定任何索引参数

index_params.add_index(
    field_name="scalar_1", # 要索引的标量字段的名称
    index_type="", # 要创建的索引类型。对于自动索引，保持为空或省略此参数。
    index_name="default_index" # 要创建的索引的名称
)

client.create_index(
  collection_name="test_scalar_index", # 指定集合名称
  index_params=index_params
)
```
```java
import io.milvus.v2.common.IndexParam;
import io.milvus.v2.service.index.request.CreateIndexReq;

IndexParam indexParamForScalarField = IndexParam.builder()
    .fieldName("scalar_1") // 要建立索引的标量字段的名称
    .indexName("default_index") // 要创建的索引的名称
    .indexType("") // 要创建的索引类型。对于自动索引，请将其保留为空或省略此参数。
    .build();

List<IndexParam> indexParams = new ArrayList<>();
indexParams.add(indexParamForVectorField);

CreateIndexReq createIndexReq = CreateIndexReq.builder()
    .collectionName("test_scalar_index") // 指定集合名称
    .indexParams(indexParams)
    .build();

client.createIndex(createIndexReq);
```
```javascript
await client.createIndex({
    collection_name: "test_scalar_index", // 指定集合名称
    field_name: "scalar_1", // 要创建索引的标量字段名称
    index_name: "default_index", // 要创建的索引名称
    index_type: "" // 要创建的索引类型。对于自动索引，请将其保留为空或省略此参数。
})
```

## 自定义索引

<div class="language-python">

要使用自定义索引，请在 [`add_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/add_index.md) 中使用 __index_type__ 参数指定特定的索引类型。

</div>

<div class="language-java">

要使用自定义索引，请在 [`IndexParam`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/IndexParam.md) 中使用 __indexType__ 参数指定特定的索引类型。

</div>

<div class="language-javascript">

要使用自定义索引，请在 [`createIndex()`](https://milvus.io/api-reference/node/v2.4.x/Management/createIndex.md) 中使用 __index_type__ 参数指定特定的索引类型。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
index_params = client.create_index_params() # 准备一个 IndexParams 对象

index_params.add_index(
    field_name="scalar_2", # 要创建索引的标量字段名称
    index_type="INVERTED", # 要创建的索引类型
    index_name="inverted_index" # 要创建的索引名称
)

client.create_index(
  collection_name="test_scalar_index", # 指定集合名称
  index_params=index_params
)
```

```java
import io.milvus.v2.common.IndexParam;
import io.milvus.v2.service.index.request.CreateIndexReq;

IndexParam indexParamForScalarField = IndexParam.builder()
    .fieldName("scalar_1") // 要创建索引的标量字段名称
    .indexName("inverted_index") // 要创建的索引名称
    .indexType("INVERTED") // 要创建的索引类型
    .build();

List<IndexParam> indexParams = new ArrayList<>();
indexParams.add(indexParamForVectorField);

CreateIndexReq createIndexReq = CreateIndexReq.builder()
    .collectionName("test_scalar_index") // 指定集合名称
    .indexParams(indexParams)
    .build();

client.createIndex(createIndexReq);
```

```javascript
await client.createIndex({
    collection_name: "test_scalar_index", // 指定集合名称
    field_name: "scalar_1", // 要创建索引的标量字段名称
    index_name: "inverted_index", // 要创建的索引名称
    index_type: "INVERTED" // 要创建的索引类型
})
```

<div class="language-python">

__方法和参数__

- __create_index_params()__

    准备一个 __IndexParams__ 对象。

- __add_index()__

    将索引配置添加到 __IndexParams__ 对象中。

    - __field_name__（_字符串_）

        要索引的标量字段的名称。

    - __index_type__（_字符串_）： 

### 创建标量索引

要创建的标量索引的类型。对于隐式索引，请将其保留为空或省略此参数。

对于自定义索引，有效值包括：

- __INVERTED__：（推荐）倒排索引包含一个按字母顺序排列的包含所有标记化单词的术语词典。有关详细信息，请参阅标量索引。

- __STL_SORT__：使用标准模板库排序算法对标量字段进行排序。支持布尔值和数值字段（例如，INT8、INT16、INT32、INT64、FLOAT、DOUBLE）。

- __Trie__：用于快速前缀搜索和检索的树数据结构。支持VARCHAR字段。

- __index_name__（_字符串_）

    要创建的标量索引的名称。每个标量字段支持一个索引。

- __create_index()__

    在指定的集合中创建索引。

    - __collection_name__（_字符串_）

        创建索引的集合的名称。

    - __index_params__

        包含索引配置的__IndexParams__对象。

</div>

<div class="language-java">

__方法和参数__

- __IndexParam__
  准备一个IndexParam对象。
  - __fieldName__（_字符串_）
    要索引的标量字段的名称。
  - __indexName__（_字符串_）
    要创建的标量索引的名称。每个标量字段支持一个索引。
  - __indexType__（_字符串_）
    要创建的标量索引的类型。对于隐式索引，请将其保留为空或省略此参数。
    对于自定义索引，有效值包括：
    - __INVERTED__：（推荐）倒排索引包含一个按字母顺序排列的包含所有标记化单词的术语词典。有关详细信息，请参阅标量索引。
    - __STL_SORT__：使用标准模板库排序算法对标量字段进行排序。支持布尔值和数值字段（例如，INT8、INT16、INT32、INT64、FLOAT、DOUBLE）。
    - __Trie__：用于快速前缀搜索和检索的树数据结构。支持VARCHAR字段。
- __CreateIndexReq__
  在指定的集合中创建索引。
  - __collectionName__（_字符串_）
    创建索引的集合的名称。
  - __indexParams__（_List<IndexParam>_）
    包含索引配置的IndexParam对象列表。

</div>

<div class="language-javascript">

__方法和参数__
- __createIndex__

  在指定的集合中创建索引。
  - __collection_name__（_字符串_）
    创建索引的集合的名称。
  - __field_name__（_字符串_）
    要索引的标量字段的名称。
  - __index_name__（_字符串_）
    要创建的标量索引的名称。每个标量字段支持一个索引。
  - __index_type__（_字符串_）
    要创建的标量索引的类型。对于隐式索引，请将其保留为空或省略此参数。
    对于自定义索引，有效值包括：
    - __倒排索引__: （推荐）倒排索引由包含所有按字母顺序排序的标记化单词的词典组成。详情请参考[标量索引](scalar_index.md)。  
    - __STL_SORT__: 使用标准模板库排序算法对标量字段进行排序。支持布尔和数值字段（例如，INT8，INT16，INT32，INT64，FLOAT，DOUBLE）。  
    - __Trie__: 用于快速前缀搜索和检索的树数据结构。支持VARCHAR字段。  

</div>

## 验证结果

<div class="language-python">

使用 [`list_indexes()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/list_indexes.md) 方法来验证标量索引的创建：

</div>

<div class="language-java">

使用 `listIndexes()` 方法来验证标量索引的创建：

</div>

<div class="language-javascript">

使用 `listIndexes()` 方法来验证标量索引的创建：

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
client.list_indexes(
    collection_name="test_scalar_index"  # 指定集合名称
)

# 输出:
# ['default_index','inverted_index']
```

```java
import java.util.List;
import io.milvus.v2.service.index.request.ListIndexesReq;

ListIndexesReq listIndexesReq = ListIndexesReq.builder()
    .collectionName("test_scalar_index")  // 指定集合名称
    .build();

List<String> indexNames = client.listIndexes(listIndexesReq);

System.out.println(indexNames);

// 输出:
// [
//     "default_index",
//     "inverted_index"
// ]
```

```javascript
res = await client.listIndexes({
    collection_name: 'test_scalar_index'
})

console.log(res.indexes)

// 输出:
// [
//     "default_index",
//     "inverted_index"
// ]   
```

## 限制

- 目前，标量索引支持 INT8、INT16、INT32、INT64、FLOAT、DOUBLE、BOOL 和 VARCHAR 数据类型，但不支持 JSON 和 ARRAY 类型。