---
id: manage-collections.md
title: 管理集合
---


# 管理集合

本指南将指导您如何使用您选择的 SDK 创建和管理集合。

## 开始之前

- 您已安装了[Milvus 独立版](https://milvus.io/docs/install_standalone-docker.md)或[Milvus 集群版](https://milvus.io/docs/install_cluster-milvusoperator.md)。

- 您已安装了所需的 SDK。您可以选择多种语言，包括[Python](https://milvus.io/docs/install-pymilvus.md)、[Java](https://milvus.io/docs/install-java.md)、[Go](https://milvus.io/docs/install-go.md)和[Node.js](https://milvus.io/docs/install-node.md)。

## 概述

在 Milvus 中，您将向集合中存储您的向量嵌入。集合中的所有向量嵌入共享相同的维度和用于测量相似性的距离度量标准。

Milvus 集合支持动态字段（即，在模式中未预定义的字段）和主键的自动增加。

为了满足不同的偏好，Milvus 提供了两种创建集合的方法。一种提供快速设置，另一种允许详细定制集合模式和索引参数。

此外，您可以在必要时查看、加载、释放和删除集合。

## 创建集合

您可以通过以下任一方式创建集合：

- __快速设置__

    通过此方式，您只需为集合指定名称和要存储在该集合中的向量嵌入的维度即可创建集合。有关详细信息，请参阅[快速设置](manage-collections.md)。

- __自定义设置__

    您可以自行确定集合的__模式__和__索引参数__，而不是让 Milvus 几乎为您的集合决定一切。有关详细信息，请参阅[自定义设置](manage-collections.md)。

### 快速设置

在人工智能行业的飞速发展背景下，大多数开发人员只需一个简单而动态的集合即可开始。Milvus 允许快速设置这样一个集合，只需三个参数即可：

- 要创建的集合名称，

- 要插入的向量嵌入的维度，以及

- 用于测量向量嵌入之间相似性的度量类型。

<div class="language-python">

要进行快速设置，请使用[`MilvusClient`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md)类的[`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md)方法来创建具有指定名称和维度的集合。

</div>

<div class="language-java">

要进行快速设置，请使用[`MilvusClientV2`](https://milvus.io/api-reference/java/v2.4.x/v2/Client/MilvusClientV2.md)类的[`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md)方法来创建具有指定名称和维度的集合。

</div>

```python
从[`MilvusClient`](https://milvus.io/api-reference/node/v2.4.x/Client/MilvusClient.md)类的[`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md)方法快速设置，创建指定名称和维度的集合。

```

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

# 2. 在快速设置模式下创建集合
client.create_collection(
    collection_name="quick_setup",
    dimension=5
)

res = client.get_load_state(
    collection_name="quick_setup"
)

print(res)

# 输出
#
# {
#     "state": "<LoadState: Loaded>"
# }
```

```java
import io.milvus.v2.client.ConnectConfig;
import io.milvus.v2.client.MilvusClientV2;
import io.milvus.v2.service.collection.request.GetLoadStateReq;
import io.milvus.v2.service.collection.request.CreateCollectionReq;

String CLUSTER_ENDPOINT = "http://localhost:19530";

// 1. 连接到 Milvus 服务器
ConnectConfig connectConfig = ConnectConfig.builder()
    .uri(CLUSTER_ENDPOINT)
    .build();

MilvusClientV2 client = new MilvusClientV2(connectConfig);

// 2. 在快速设置模式下创建集合
CreateCollectionReq quickSetupReq = CreateCollectionReq.builder()
    .collectionName("quick_setup")
    .dimension(5)
    .build();

client.createCollection(quickSetupReq);

// Thread.sleep(5000);

GetLoadStateReq quickSetupLoadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .build();

Boolean res = client.getLoadState(quickSetupLoadStateReq);

System.out.println(res);

// 输出:
// true
```

```javascript
address = "http://localhost:19530"

// 1. 设置 Milvus 客户端
client = new MilvusClient({address});

// 2. 在快速设置模式下创建集合
let res = await client.createCollection({
    collection_name: "quick_setup",
    dimension: 5,
});  

console.log(res.error_code)

// 输出
// 
// Success
// 

res = await client.getLoadState({
    collection_name: "quick_setup"
})

console.log(res.state)

// 输出
// 
// LoadStateLoaded
// 
```

以上代码生成的集合仅包含两个字段：`id`（作为主键）和`vector`（作为向量字段），默认启用`auto_id`和`enable_dynamic_field`设置。

- `auto_id` 

    启用此设置可确保主键自动递增。在数据插入期间无需手动提供主键。

- `enable_dynamic_field`

    启用后，要插入的数据中除`id`和`vector`之外的所有字段都被视为动态字段。这些额外字段保存在名为`$meta`的特殊字段中的键值对中。此功能允许在数据插入期间包含额外字段。
```
自动索引并加载的集合已经准备好立即插入数据。

### 自定义设置

您可以自行确定集合的 __模式__ 和 __索引参数__，而不是让 Milvus 几乎决定一切。

#### 步骤 1：设置模式

模式定义了集合的结构。在模式中，您可以选择启用或禁用 `enable_dynamic_field`，添加预定义字段，并为每个字段设置属性。有关该概念和可用数据类型的详细解释，请参阅[模式解释](schema.md)。

<div class="language-python">

要设置模式，请使用 [`create_schema()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md) 创建一个模式对象，并使用 [`add_field()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/CollectionSchema/add_field.md) 向模式中添加字段。

</div>

<div class="language-java">

要设置模式，请使用 [`createSchema()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createSchema.md) 创建一个模式对象，并使用 [`addField()`](https://milvus.io/api-reference/java/v2.4.x/v2/CollectionSchema/addField.md) 向模式中添加字段。

</div>

<div class="language-javascript">

要设置模式，请使用 [`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md)。

</div>

<div class="multipleCode">
  <a href="#python">Python </a>
  <a href="#java">Java</a>
  <a href="#javascript">Node.js</a>
</div>

```python
# 3. 在自定义设置模式下创建集合

# 3.1 创建模式
schema = MilvusClient.create_schema(
    auto_id=False,
    enable_dynamic_field=True,
)

# 3.2 向模式中添加字段
schema.add_field(field_name="my_id", datatype=DataType.INT64, is_primary=True)
schema.add_field(field_name="my_vector", datatype=DataType.FLOAT_VECTOR, dim=5)
```

```java
import io.milvus.v2.common.DataType;
import io.milvus.v2.service.collection.request.CreateCollectionReq;

// 3. 在自定义设置模式下创建集合

// 3.1 创建模式
CreateCollectionReq.CollectionSchema schema = client.createSchema();

// 3.2 向模式中添加字段
schema.addField(AddFieldReq.builder()
    .fieldName("my_id")
    .dataType(DataType.Int64).
    isPrimaryKey(true)
    .autoID(false)
    .build());

schema.addField(AddFieldReq.builder()
    .fieldName("my_vector")
    .dataType(DataType.FloatVector)
    .dimension(5)
    .build());
```

```javascript
// 3. 在自定义设置模式下创建集合
// 3.1 定义字段
const fields = [
    {
        name: "my_id",
        data_type: DataType.Int64,
        is_primary_key: true,
        auto_id: false
    },
    {
        name: "my_vector",
        data_type: DataType.FloatVector,
        dim: 5
    },
]
```
在提供的 Python 代码片段中，`enable_dynamic_field` 被设置为 `True`，并且为主键启用了 `auto_id`。此外，引入了一个 `vector` 字段，配置了维度为 768，还包括四个标量字段，每个字段都有其各自的属性。

#### 第二步：设置索引参数

索引参数决定了 Milvus 如何在集合中组织数据。您可以通过调整它们的 `metric_type` 和 `index_type` 来为特定字段定制索引过程。对于向量字段，您可以灵活选择 `COSINE`、`L2` 或 `IP` 作为 `metric_type`。

<div class="language-python">

要设置索引参数，请使用 [`prepare_index_params()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/prepare_index_params.md) 准备索引参数，使用 [`add_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/add_index.md) 添加索引。

</div>

<div class="language-java">

要设置索引参数，请使用 [IndexParam](https://milvus.io/api-reference/java/v2.4.x/v2/Management/IndexParam.md)。

</div>

<div class="language-javascript">

要设置索引参数，请使用 [`createIndex()`](https://milvus.io/api-reference/node/v2.4.x/Management/createIndex.md)。

</div>

<div class="multipleCode">
  <a href="#python">Python </a>
  <a href="#java">Java</a>
  <a href="#javascript">Node.js</a>
</div>

```python
# 3.3. 准备索引参数
index_params = client.prepare_index_params()

# 3.4. 添加索引
index_params.add_index(
    field_name="my_id",
    index_type="STL_SORT"
)

index_params.add_index(
    field_name="my_vector", 
    index_type="IVF_FLAT",
    metric_type="IP",
    params={ "nlist": 128 }
)
```

```java
import io.milvus.v2.common.IndexParam;

// 3.3 准备索引参数
IndexParam indexParamForIdField = IndexParam.builder()
    .fieldName("my_id")
    .indexType(IndexParam.IndexType.STL_SORT)
    .build();

IndexParam indexParamForVectorField = IndexParam.builder()
    .fieldName("my_vector")
    .indexType(IndexParam.IndexType.IVF_FLAT)
    .metricType(IndexParam.MetricType.L2)
    .extraParams(Map.of("nlist", 1024))
    .build();

List<IndexParam> indexParams = new ArrayList<>();
indexParams.add(indexParamForIdField);
indexParams.add(indexParamForVectorField);
```

```javascript
// 3.2 准备索引参数
const index_params = [{
    field_name: "my_id",
    index_type: "STL_SORT"
},{
    field_name: "my_vector",
    index_type: "IVF_FLAT",
    metric_type: "IP",
    params: { nlist: 1024}
}]
```

上述代码片段演示了如何分别为向量字段和标量字段设置索引参数。对于向量字段，设置指标类型和索引类型。对于标量字段，只设置索引类型。建议为经常用于过滤的向量字段和任何标量字段创建索引。

#### 第三步：创建集合
你可以选择分别创建一个集合和一个索引文件，或者在创建时同时加载索引。

使用 [create_collection()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md) 来创建具有指定模式和索引参数的集合，使用 [get_load_state()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/get_load_state.md) 来检查集合的加载状态。

使用 [createCollection()](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md) 来创建具有指定模式和索引参数的集合，使用 [getLoadState()](https://milvus.io/api-reference/java/v2.4.x/v2/Management/getLoadState.md) 来检查集合的加载状态。

使用 [createCollection()](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md) 来创建具有指定模式和索引参数的集合，使用 [getLoadState()](https://milvus.io/api-reference/node/v2.4.x/Management/getLoadState.md) 来检查集合的加载状态。

- __在创建时同时加载索引的集合__

    <div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
    </div>

    ```python
    # 3.5. 创建一个同时加载索引的集合
    client.create_collection(
        collection_name="customized_setup_1",
        schema=schema,
        index_params=index_params
    )
    
    time.sleep(5)
    
    res = client.get_load_state(
        collection_name="customized_setup_1"
    )
    
    print(res)
    
    # 输出
    #
    # {
    #     "state": "<LoadState: Loaded>"
    # }
    ```

    ```java
    import io.milvus.v2.service.collection.request.CreateCollectionReq;
    import io.milvus.v2.service.collection.request.GetLoadStateReq;
    
    // 3.4 创建一个带有模式和索引参数的集合
    CreateCollectionReq customizedSetupReq1 = CreateCollectionReq.builder()
        .collectionName("customized_setup_1")
        .collectionSchema(schema)
        .indexParams(indexParams)
        .build();
    
    client.createCollection(customizedSetupReq1);
    
    // Thread.sleep(5000);
    
    // 3.5 获取集合的加载状态
    GetLoadStateReq customSetupLoadStateReq1 = GetLoadStateReq.builder()
        .collectionName("customized_setup_1")
        .build();
    
    res = client.getLoadState(customSetupLoadStateReq1);
    
    System.out.println(res);
    
    // 输出:
    // true
    ```

    ```javascript
    // 3.3 创建一个带有字段和索引参数的集合
    res = await client.createCollection({
        collection_name: "customized_setup_1",
        fields: fields,
        index_params: index_params,
    })
    
    console.log(res.error_code)
    ```
```markdown
// 输出
// 
// 成功
// 

res = await client.getLoadState({
    collection_name: "customized_setup_1"
})

console.log(res.state)

// 输出
// 
// LoadStateLoaded
//   

```

上述创建的集合会自动加载。要了解更多关于加载和释放集合的信息，请参考[加载和释放集合](manage-collections.md)。

- __分别创建集合和索引文件。__

<div class="multipleCode">
<a href="#python">Python </a>
<a href="#java">Java</a>
<a href="#javascript">Node.js</a>
</div>

```python
# 3.6 创建集合并单独对其建立索引
client.create_collection(
    collection_name="customized_setup_2",
    schema=schema,
)

res = client.get_load_state(
    collection_name="customized_setup_2"
)

print(res)

# 输出
#
# {
#     "state": "<LoadState: NotLoad>"
# }
```

```java
// 3.6 创建集合并单独对其建立索引
CreateCollectionReq customizedSetupReq2 = CreateCollectionReq.builder()
    .collectionName("customized_setup_2")
    .collectionSchema(schema)
    .build();

client.createCollection(customizedSetupReq2);
```

```javascript
// 3.4 创建集合并单独对其建立索引
res = await client.createCollection({
    collection_name: "customized_setup_2",
    fields: fields,
})

console.log(res.error_code)

// 输出
// 
// 成功
// 

res = await client.getLoadState({
    collection_name: "customized_setup_2"
})

console.log(res.state)

// 输出
// 
// LoadStateNotLoad
// 
```

上述创建的集合不会自动加载。您可以按以下方式为集合创建索引。以单独方式为集合创建索引不会自动加载集合。详情请参考[加载和释放集合](manage-collections.md)。

<div class="multipleCode">
<a href="#python">Python </a>
<a href="#java">Java</a>
<a href="#javascript">Node.js</a>
</div>

```python
# 3.6 创建索引
client.create_index(
    collection_name="customized_setup_2",
    index_params=index_params
)

res = client.get_load_state(
    collection_name="customized_setup_2"
)

print(res)

# 输出
#
# {
#     "state": "<LoadState: NotLoad>"
# }
```

```java
CreateIndexReq  createIndexReq = CreateIndexReq.builder()
    .collectionName("customized_setup_2")
    .indexParams(indexParams)
    .build();

client.createIndex(createIndexReq);

// Thread.sleep(1000);

// 3.7 获取集合的加载状态
GetLoadStateReq customSetupLoadStateReq2 = GetLoadStateReq.builder()
    .collectionName("customized_setup_2")
    .build();
```
```python
res = client.getLoadState(customSetupLoadStateReq2);

print(res);

// 输出:
// false
```

```javascript
// 3.5 创建索引
res = await client.createIndex({
    collection_name: "customized_setup_2",
    field_name: "my_vector",
    index_type: "IVF_FLAT",
    metric_type: "IP",
    params: { nlist: 1024}
})

res = await client.getLoadState({
    collection_name: "customized_setup_2"
})

console.log(res.state)

// 输出
// 
// LoadStateNotLoad
//
```

## 查看集合

<div class="language-python">

要查看现有集合的详细信息，请使用 [describe_collection()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/describe_collection.md)。

</div>

<div class="language-java">

要查看现有集合的详细信息，请使用 [describeCollection()](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/describeCollection.md)。

</div>

<div class="language-javascript">

要查看现有集合的详细信息，请使用 [describeCollection()](https://milvus.io/api-reference/node/v2.4.x/Collections/describeCollection.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 5. 查看集合
res = client.describe_collection(
    collection_name="customized_setup_2"
)

print(res)

# 输出
#
# {
#     "collection_name": "customized_setup_2",
#     "auto_id": false,
#     "num_shards": 1,
#     "description": "",
#     "fields": [
#         {
#             "field_id": 100,
#             "name": "my_id",
#             "description": "",
#             "type": 5,
#             "params": {},
#             "element_type": 0,
#             "is_primary": true
#         },
#         {
#             "field_id": 101,
#             "name": "my_vector",
#             "description": "",
#             "type": 101,
#             "params": {
#                 "dim": 5
#             },
#             "element_type": 0
#         }
#     ],
#     "aliases": [],
#     "collection_id": 448143479230158446,
#     "consistency_level": 2,
#     "properties": {},
#     "num_partitions": 1,
#     "enable_dynamic_field": true
# }

```
```java
import io.milvus.v2.service.collection.request.DescribeCollectionReq;
import io.milvus.v2.service.collection.response.DescribeCollectionResp;

// 4. 查看集合
DescribeCollectionReq describeCollectionReq = DescribeCollectionReq.builder()
    .collectionName("customized_setup_2")
    .build();

DescribeCollectionResp describeCollectionRes = client.describeCollection(describeCollectionReq);

System.out.println(JSONObject.toJSON(describeCollectionRes));

// 输出:
// {
//     "createTime": 449005822816026627,
//     "collectionSchema": {"fieldSchemaList": [
//         {
//             "autoID": false,
//             "dataType": "Int64",
//             "name": "my_id",
//             "description": "",
//             "isPrimaryKey": true,
//             "maxLength": 65535,
//             "isPartitionKey": false
//         },
//         {
//             "autoID": false,
//             "dataType": "FloatVector",
//             "name": "my_vector",
//             "description": "",
//             "isPrimaryKey": false,
//             "dimension": 5,
//             "maxLength": 65535,
//             "isPartitionKey": false
//         }
//     ]},
//     "vectorFieldName": ["my_vector"],
//     "autoID": false,
//     "fieldNames": [
//         "my_id",
//         "my_vector"
//     ],
//     "description": "",
//     "numOfPartitions": 1,
//     "primaryFieldName": "my_id",
//     "enableDynamicField": true,
//     "collectionName": "customized_setup_2"
// }
```
```javascript
// 5. 查看集合
res = await client.describeCollection({
    collection_name: "customized_setup_2"
})

console.log(res)

// 输出
// 
// {
//   virtual_channel_names: [ 'by-dev-rootcoord-dml_13_449007919953017716v0' ],
//   physical_channel_names: [ 'by-dev-rootcoord-dml_13' ],
//   aliases: [],
//   start_positions: [],
//   properties: [],
//   status: {
//     extra_info: {},
//     error_code: 'Success',
//     reason: '',
//     code: 0,
//     retriable: false,
//     detail: ''
//   },
//   schema: {
//     fields: [ [Object], [Object] ],
//     properties: [],
//     name: 'customized_setup_2',
//     description: '',
//     autoID: false,
//     enable_dynamic_field: false
//   },
//   collectionID: '449007919953017716',
//   created_timestamp: '449024569603784707',
//   created_utc_timestamp: '1712892797866',
//   shards_num: 1,
//   consistency_level: 'Bounded',
//   collection_name: 'customized_setup_2',
//   db_name: 'default',
//   num_partitions: '1'
// }
// 
```

要列出所有现有的集合，可以按照以下步骤操作：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 6. 列出所有集合名称
res = client.list_collections()

print(res)

# 输出
#
# [
#     "customized_setup_2",
#     "quick_setup",
#     "customized_setup_1"
# ]
```

```java
import io.milvus.v2.service.collection.response.ListCollectionsResp;

// 5. 列出所有集合名称
ListCollectionsResp listCollectionsRes = client.listCollections();

System.out.println(listCollectionsRes.getCollectionNames());

// 输出:
// [
//     "customized_setup_2",
//     "quick_setup",
//     "customized_setup_1"
// ]
```

```javascript
// 5. 列出所有集合名称
ListCollectionsResp listCollectionsRes = client.listCollections();

System.out.println(listCollectionsRes.getCollectionNames());

// 输出:
// [
//     "customized_setup_1",
//     "quick_setup",
//     "customized_setup_2"
// ]
```

## 加载和释放集合

在加载集合的过程中，Milvus将集合的索引文件加载到内存中。相反，在释放集合时，Milvus会将索引文件从内存中卸载。在对集合进行搜索之前，请确保该集合已加载。

### 加载集合

<div class="language-python">

要加载集合，请使用 [`load_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/load_collection.md) 方法，指定集合名称。您还可以设置 `replica_number` 来确定在加载集合时在查询节点上创建多少数据段的内存副本。

- Milvus Standalone: `replica_number` 的最大允许值为 1。
```
- Milvus 集群：最大值不应超过 Milvus 配置中设置的 `queryNode.replicas`。有关更多详细信息，请参阅 [Query Node-related Configurations](https://milvus.io/docs/configure_querynode.md#Query-Node-related-Configurations)。

</div>

<div class="language-java">

要加载一个集合，请使用 [`loadCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/loadCollection.md) 方法，并指定集合名称。

</div>

<div class="language-javascript">

要加载一个集合，请使用 [`loadCollection()`](https://milvus.io/api-reference/node/v2.4.x/Management/loadCollection.md) 方法，并指定集合名称。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 7. 加载集合
client.load_collection(
    collection_name="customized_setup_2",
    replica_number=1 # 在查询节点上创建的副本数。对于 Milvus Standalone，最大值为 1；对于 Milvus 集群，不超过 `queryNode.replicas`。
)

res = client.get_load_state(
    collection_name="customized_setup_2"
)

print(res)

# 输出
#
# {
#     "state": "<LoadState: Loaded>"
# }
```

```java
import io.milvus.v2.service.collection.request.LoadCollectionReq;

// 6. 加载集合
LoadCollectionReq loadCollectionReq = LoadCollectionReq.builder()
    .collectionName("customized_setup_2")
    .build();

client.loadCollection(loadCollectionReq);

// Thread.sleep(5000);

// 7. 获取集合加载状态
GetLoadStateReq loadStateReq = GetLoadStateReq.builder()
    .collectionName("customized_setup_2")
    .build();

res = client.getLoadState(loadStateReq);

System.out.println(res);

// 输出:
// true
```

```javascript
// 7. 加载集合
res = await client.loadCollection({
    collection_name: "customized_setup_2"
})

console.log(res.error_code)

// 输出
// 
// Success
// 

await sleep(3000)

res = await client.getLoadState({
    collection_name: "customized_setup_2"
})

console.log(res.state)

// 输出
// 
// LoadStateLoaded
// 
```

### 释放集合

<div class="language-python">

要释放一个集合，请使用 [`release_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/release_collection.md) 方法，并指定集合名称。

</div>

<div class="language-java">

要释放一个集合，请使用 [`releaseCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/releaseCollection.md) 方法，并指定集合名称。

</div>

<div class="language-javascript">

要释放一个集合，请使用 [`releaseCollection()`](https://milvus.io/api-reference/node/v2.4.x/Management/releaseCollection.md) 方法，并指定集合名称。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```python
# 8. 释放数据集
client.release_collection(
    collection_name="customized_setup_2"
)

res = client.get_load_state(
    collection_name="customized_setup_2"
)

print(res)

# 输出
#
# {
#     "state": "<LoadState: NotLoad>"
# }
```
```java
import io.milvus.v2.service.collection.request.ReleaseCollectionReq;

// 8. 释放集合
ReleaseCollectionReq releaseCollectionReq = ReleaseCollectionReq.builder()
    .collectionName("customized_setup_2")
    .build();

client.releaseCollection(releaseCollectionReq);

// Thread.sleep(1000);

res = client.getLoadState(loadStateReq);

System.out.println(res);

// 输出:
// false
```

```javascipt
// 8. 释放集合
res = await client.releaseCollection({
    collection_name: "customized_setup_2"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 

res = await client.getLoadState({
    collection_name: "customized_setup_2"
})

console.log(res.state)

// 输出
// 
// 未加载
// 
```

## 设置别名

您可以为集合分配别名，以使它们在特定上下文中更有意义。您可以为一个集合分配多个别名，但多个集合不能共享一个别名。

### 创建别名

<div class="language-python">

要创建别名，请使用 [`create_alias()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_alias.md) 方法，指定集合名称和别名。

</div>

<div class="language-java">

要创建别名，请使用 [`createAlias()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createAlias.md) 方法，指定集合名称和别名。

</div>

<div class="language-javascript">

要创建别名，请使用 [`createAlias()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createAlias.md) 方法，指定集合名称和别名。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 9.1. 创建别名
client.create_alias(
    collection_name="customized_setup_2",
    alias="bob"
)

client.create_alias(
    collection_name="customized_setup_2",
    alias="alice"
)
```

```java
import io.milvus.v2.service.utility.request.CreateAliasReq;

// 9. 管理别名

// 9.1 创建别名
CreateAliasReq createAliasReq = CreateAliasReq.builder()
    .collectionName("customized_setup_2")
    .alias("bob")
    .build();

client.createAlias(createAliasReq);

createAliasReq = CreateAliasReq.builder()
    .collectionName("customized_setup_2")
    .alias("alice")
    .build();

client.createAlias(createAliasReq);
```

```javascript
// 9. 管理别名
// 9.1 创建别名
res = await client.createAlias({
    collection_name: "customized_setup_2",
    alias: "bob"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 

res = await client.createAlias({
    collection_name: "customized_setup_2",
    alias: "alice"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 
```

### 列出别名

<div class="language-python">
```
要列出别名，请使用 [`list_aliases()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/list_aliases.md) 方法，并指定集合名称。

</div>

<div class="language-java">

要列出别名，请使用 [`listAliases()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/listAliases.md) 方法，并指定集合名称。

</div>

<div class="language-javascript">

要列出别名，请使用 [`listAliases()`](https://milvus.io/api-reference/node/v2.4.x/Collections/listAliases.md) 方法，并指定集合名称。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 9.2. 列出别名
res = client.list_aliases(
    collection_name="customized_setup_2"
)

print(res)

# 输出
#
# {
#     "aliases": [
#         "bob",
#         "alice"
#     ],
#     "collection_name": "customized_setup_2",
#     "db_name": "default"
# }
```

```java
import io.milvus.v2.service.utility.request.ListAliasesReq;
import io.milvus.v2.service.utility.response.ListAliasResp;

// 9.2 列出别名
ListAliasesReq listAliasesReq = ListAliasesReq.builder()
    .collectionName("customized_setup_2")
    .build();

ListAliasResp listAliasRes = client.listAliases(listAliasesReq);

System.out.println(listAliasRes.getAlias());

// 输出:
// [
//     "bob",
//     "alice"
// ]
```

```javascript
// 9.2 列出别名
res = await client.listAliases({
    collection_name: "customized_setup_2"
})

console.log(res.aliases)

// 输出
// 
// [ 'bob', 'alice' ]
// 
```

### 描述别名

<div class="language-python">

要描述别名，请使用 [`describe_alias()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/describe_alias.md) 方法，并指定别名。

</div>

<div class="language-java">

要描述别名，请使用 [`describeAlias()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/describeAlias.md) 方法，并指定别名。

</div>

<div class="language-javascript">

要描述别名，请使用 [`describeAlias()`](https://milvus.io/api-reference/node/v2.4.x/Collections/describeAlias.md) 方法，并指定别名。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 9.3. 描述别名
res = client.describe_alias(
    alias="bob"
)

print(res)

# 输出
#
# {
#     "alias": "bob",
#     "collection_name": "customized_setup_2",
#     "db_name": "default"
# }
```
```java
import io.milvus.v2.service.utility.request.DescribeAliasReq;
import io.milvus.v2.service.utility.response.DescribeAliasResp;

// 9.3 描述别名
DescribeAliasReq describeAliasReq = DescribeAliasReq.builder()
    .alias("bob")
    .build();

DescribeAliasResp describeAliasRes = client.describeAlias(describeAliasReq);

System.out.println(JSONObject.toJSON(describeAliasRes));

// 输出:
// {
//     "alias": "bob",
//     "collectionName": "customized_setup_2"
// }
```
```javascript
// 9.3 描述别名
res = await client.describeAlias({
    collection_name: "customized_setup_2",
    alias: "bob"
})

console.log(res)

// 输出
// 
// {
//   status: {
//     extra_info: {},
//     error_code: 'Success',
//     reason: '',
//     code: 0,
//     retriable: false,
//     detail: ''
//   },
//   db_name: 'default',
//   alias: 'bob',
//   collection: 'customized_setup_2'
// }
// 
```

### 重新分配别名

<div class="language-python">

要将别名重新分配给其他集合，请使用 [`alter_alias()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/alter_alias.md) 方法，指定集合名称和别名。

</div>

<div class="language-java">

要将别名重新分配给其他集合，请使用 [`alterAlias()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/alterAlias.md) 方法，指定集合名称和别名。

</div>

<div class="language-javascript">

要将别名重新分配给其他集合，请使用 [`alterAlias()`](https://milvus.io/api-reference/node/v2.4.x/Collections/alterAlias.md) 方法，指定集合名称和别名。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 9.4 将别名重新分配给其他集合
client.alter_alias(
    collection_name="customized_setup_1",
    alias="alice"
)

res = client.list_aliases(
    collection_name="customized_setup_1"
)

print(res)

# 输出
#
# {
#     "aliases": [
#         "alice"
#     ],
#     "collection_name": "customized_setup_1",
#     "db_name": "default"
# }

res = client.list_aliases(
    collection_name="customized_setup_2"
)

print(res)

# 输出
#
# {
#     "aliases": [
#         "bob"
#     ],
#     "collection_name": "customized_setup_2",
#     "db_name": "default"
# }
```

```java
import io.milvus.v2.service.utility.request.AlterAliasReq;

// 9.4 将别名重新分配给其他集合
AlterAliasReq alterAliasReq = AlterAliasReq.builder()
    .collectionName("customized_setup_1")
    .alias("alice")
    .build();

client.alterAlias(alterAliasReq);

listAliasesReq = ListAliasesReq.builder()
    .collectionName("customized_setup_1")
    .build();

listAliasRes = client.listAliases(listAliasesReq);

System.out.println(listAliasRes.getAlias());

// 输出:
// ["alice"]

listAliasesReq = ListAliasesReq.builder()
    .collectionName("customized_setup_2")
    .build();

listAliasRes = client.listAliases(listAliasesReq);

System.out.println(listAliasRes.getAlias());

// 输出:
// ["bob"]
```
```
```javascript
// 9.4 重新分配别名给其他集合
res = await client.alterAlias({
    collection_name: "customized_setup_1",
    alias: "alice"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 

res = await client.listAliases({
    collection_name: "customized_setup_1"
})

console.log(res.aliases)

// 输出
// 
// [ 'alice' ]
// 

res = await client.listAliases({
    collection_name: "customized_setup_2"
})

console.log(res.aliases)

// 输出
// 
// [ 'bob' ]
// 
```
### 删除别名

使用 [`drop_alias()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/drop_alias.md) 方法来删除别名，指定要删除的别名。

使用 [`dropAlias()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/dropAlias.md) 方法来删除别名，指定要删除的别名。

使用 [`dropAlias()`](https://milvus.io/api-reference/node/v2.4.x/Collections/dropAlias.md) 方法来删除别名，指定要删除的别名。

```python
# 9.5 删除别名
client.drop_alias(
    alias="bob"
)

client.drop_alias(
    alias="alice"
)
```

```java
import io.milvus.v2.service.utility.request.DropAliasReq;

// 9.5 删除别名
DropAliasReq dropAliasReq = DropAliasReq.builder()
    .alias("bob")
    .build();

client.dropAlias(dropAliasReq);

dropAliasReq = DropAliasReq.builder()
    .alias("alice")
    .build();

client.dropAlias(dropAliasReq);
```

```javascript
// 9.5 删除别名
res = await client.dropAlias({
    alias: "bob"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 

res = await client.dropAlias({
    alias: "alice"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 
```

## 删除集合

如果不再需要某个集合，可以删除该集合。

使用 [`drop_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/drop_collection.md) 方法来删除集合，指定要删除的集合名称。

使用 [`dropCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/dropCollection.md) 方法来删除集合，指定要删除的集合名称。

使用 [`dropCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/dropCollection.md) 方法来删除集合，指定要删除的集合名称。

```python
# 10. 删除集合
client.drop_collection(
    collection_name="quick_setup"
)

client.drop_collection(
    collection_name="customized_setup_1"
)

client.drop_collection(
    collection_name="customized_setup_2"
)
```
```java
import io.milvus.v2.service.collection.request.DropCollectionReq;

// 10. 删除集合

DropCollectionReq dropQuickSetupParam = DropCollectionReq.builder()
    .collectionName("quick_setup")
    .build();

client.dropCollection(dropQuickSetupParam);

DropCollectionReq dropCustomizedSetupParam = DropCollectionReq.builder()
    .collectionName("customized_setup_1")
    .build();

client.dropCollection(dropCustomizedSetupParam);

dropCustomizedSetupParam = DropCollectionReq.builder()
    .collectionName("customized_setup_2")
    .build();

client.dropCollection(dropCustomizedSetupParam);
```
```javascript
// 10. 删除集合
res = await client.dropCollection({
    collection_name: "customized_setup_2"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 

res = await client.dropCollection({
    collection_name: "customized_setup_1"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 

res = await client.dropCollection({
    collection_name: "quick_setup"
})

console.log(res.error_code)

// 输出
// 
// 成功
// 
```