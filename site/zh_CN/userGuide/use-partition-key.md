---
id: use-partition-key.md
title: 使用分区键
---

# 使用分区键

本指南将指导您如何使用分区键加速从集合中检索数据。

## 概述

您可以将集合中的特定字段设置为分区键，使得 Milvus 根据该字段中各自的分区值将传入的实体分布到不同的分区中。这样可以将具有相同键值的实体分组在一个分区中，通过避免在按键字段过滤时扫描不相关的分区，加速搜索性能。与传统的过滤方法相比，分区键可以极大地提升查询性能。

您可以使用分区键来实现多租户。有关多租户的详细信息，请阅读 [多租户](https://milvus.io/docs/multi_tenancy.md)。

## 启用分区键

以下代码片段演示了如何将一个字段设置为分区键。

在下面的示例代码中，`num_partitions` 决定将创建的分区数，默认值为 `64`。我们建议您保留默认值。

<div class="language-python">

有关参数的更多信息，请参考[`MilvusClient`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md)、[`create_schema()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md) 和 [`add_field()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/CollectionSchema/add_field.md) 在 SDK 参考中。

</div>

<div class="language-java">

有关参数的更多信息，请参考[`MilvusClientV2`](https://milvus.io/api-reference/java/v2.4.x/v2/Client/MilvusClientV2.md)、[`createSchema()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createSchema.md) 和 [`addField()`](https://milvus.io/api-reference/java/v2.4.x/v2/CollectionSchema/addField.md) 在 SDK 参考中。

</div>

<div class="language-javascript">

有关参数的更多信息，请参考[`MilvusClient`](https://milvus.io/api-reference/node/v2.4.x/Client/MilvusClient.md) 和 [`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md) 在 SDK 参考中。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
import random, time
from pymilvus import connections, MilvusClient, DataType

CLUSTER_ENDPOINT = "http://localhost:19530"

# 1. 设置一个 Milvus 客户端
client = MilvusClient(
    uri=CLUSTER_ENDPOINT
)

# 2. 创建一个集合
schema = MilvusClient.create_schema(
    auto_id=False,
    enable_dynamic_field=True,
    # highlight-next-line
    partition_key_field="color",
    num_partitions=64 # 分区数，默认为 64。
)

schema.add_field(field_name="id", datatype=DataType.INT64, is_primary=True)
schema.add_field(field_name="vector", datatype=DataType.FLOAT_VECTOR, dim=5)
schema.add_field(field_name="color", datatype=DataType.VARCHAR, max_length=512)
```
```java
import io.milvus.v2.client.ConnectConfig;
import io.milvus.v2.client.MilvusClientV2;
import io.milvus.v2.common.DataType;
import io.milvus.v2.common.IndexParam;
import io.milvus.v2.service.collection.request.AddFieldReq;
import io.milvus.v2.service.collection.request.CreateCollectionReq;

String CLUSTER_ENDPOINT = "http://localhost:19530";

// 1. 连接到 Milvus 服务器
ConnectConfig connectConfig = ConnectConfig.builder()
    .uri(CLUSTER_ENDPOINT)
    .build();

MilvusClientV2 client = new MilvusClientV2(connectConfig);

// 2. 在自定义设置模式下创建集合

// 2.1 创建模式
CreateCollectionReq.CollectionSchema schema = client.createSchema();

// 2.2 向模式添加字段
schema.addField(AddFieldReq.builder()
    .fieldName("id")
    .dataType(DataType.Int64)
    .isPrimaryKey(true)
    .autoID(false)
    .build());

schema.addField(AddFieldReq.builder()
    .fieldName("vector")
    .dataType(DataType.FloatVector)
    .dimension(5)
    .build());
    
schema.addField(AddFieldReq.builder()
    .fieldName("color")
    .dataType(DataType.VarChar)
    .maxLength(512)
    // highlight-next-line
    .isPartitionKey(true)
    .build());
```

```javascript
const { MilvusClient, DataType, sleep } = require("@zilliz/milvus2-sdk-node")

const address = "http://localhost:19530"

async function main() {
// 1. 设置 Milvus 客户端
client = new MilvusClient({address}); 

// 2. 创建一个集合
// 2.1 定义字段
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
    {
        name: "color",
        data_type: DataType.VarChar,
        max_length: 512,
        // highlight-next-line
        is_partition_key: true
    }
]
```

定义字段后，设置索引参数。

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
index_params = MilvusClient.prepare_index_params()

index_params.add_index(
    field_name="id",
    index_type="STL_SORT"
)

index_params.add_index(
    field_name="color",
    index_type="Trie"
)

index_params.add_index(
    field_name="vector",
    index_type="IVF_FLAT",
    metric_type="L2",
    params={"nlist": 1024}
)
```

```java
// 2.3 准备索引参数
IndexParam indexParamForVectorField = IndexParam.builder()
    .fieldName("vector")
    .indexType(IndexParam.IndexType.IVF_FLAT)
    .metricType(IndexParam.MetricType.IP)
    .extraParams(Map.of("nlist", 1024))
    .build();

List<IndexParam> indexParams = new ArrayList<>();
indexParams.add(indexParamForVectorField);
```

```javascript
// 2.2 准备索引参数
const index_params = [{
    field_name: "color",
    index_type: "Trie"
},{
    field_name: "id",
    index_type: "STL_SORT"
},{
    field_name: "vector",
    index_type: "IVF_FLAT",
    metric_type: "IP",
    params: { nlist: 1024}
}]
```
最后，您可以创建一个集合。

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
client.create_collection(
    collection_name="test_collection",
    schema=schema,
    index_params=index_params
)
```

```java
// 2.4 使用模式和索引参数创建集合
CreateCollectionReq customizedSetupReq = CreateCollectionReq.builder()
    .collectionName("test_collection")
    .collectionSchema(schema)
    .indexParams(indexParams)          
    .build();

client.createCollection(customizedSetupReq);
```

```javascript
// 2.3 使用字段和索引参数创建集合
res = await client.createCollection({
    collection_name: "test_collection",
    fields: fields, 
    index_params: index_params,
})

console.log(res.error_code)

// 输出
// 
// 成功
//
```

## 列出分区

一旦集合的一个字段被用作分区键，Milvus会根据您指定的数量创建分区，并代表您管理这些分区。因此，您将无法再操作该集合中的分区。

以下代码片段演示了一旦集合中的一个字段被用作分区键，该集合中会创建 64 个分区。


## 插入数据

一旦集合准备就绪，可以按以下方式开始插入数据：

### 准备数据

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 2.1 列出集合中的所有分区
partition_names = client.list_partitions(
    collection_name="test_collection"
)

print(partition_names)

# 输出
#
# [
#     "_default_0",
#     "_default_1",
#     "_default_2",
#     "_default_3",
#     "_default_4",
#     "_default_5",
#     "_default_6",
#     "_default_7",
#     "_default_8",
#     "_default_9",
#     "(隐藏了 54 个更多项)"
# ]
```

```java
// 2.5 列出集合中的所有分区
List<String> partitionNames = client.listPartitions(ListPartitionsReq.builder()
    .collectionName("test_collection")
    .build());

System.out.println(partitionNames);

// 输出:
// [
//     "_default_0",
//     "_default_1",
//     "_default_2",
//     "_default_3",
//     "_default_4",
//     "_default_5",
//     "_default_6",
//     "_default_7",
//     "_default_8",
//     "_default_9",
//     "(隐藏了 54 个元素)"
// ]
```
```javascript
// 2.1 列出分区
res = await client.listPartitions({
    collection_name: "test_collection",
})

console.log(res.partition_names)

// 输出
// 
// [
//   '_default_0',  '_default_1',  '_default_2',  '_default_3',
//   '_default_4',  '_default_5',  '_default_6',  '_default_7',
//   '_default_8',  '_default_9',  '_default_10', '_default_11',
//   '_default_12', '_default_13', '_default_14', '_default_15',
//   '_default_16', '_default_17', '_default_18', '_default_19',
//   '_default_20', '_default_21', '_default_22', '_default_23',
//   '_default_24', '_default_25', '_default_26', '_default_27',
//   '_default_28', '_default_29', '_default_30', '_default_31',
//   '_default_32', '_default_33', '_default_34', '_default_35',
//   '_default_36', '_default_37', '_default_38', '_default_39',
//   '_default_40', '_default_41', '_default_42', '_default_43',
//   '_default_44', '_default_45', '_default_46', '_default_47',
//   '_default_48', '_default_49', '_default_50', '_default_51',
//   '_default_52', '_default_53', '_default_54', '_default_55',
//   '_default_56', '_default_57', '_default_58', '_default_59',
//   '_default_60', '_default_61', '_default_62', '_default_63'
// ]
// 
```
## 插入数据

准备好集合后，可以按照以下步骤开始插入数据：

### 准备数据

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 3. 插入随机生成的向量
colors = ["green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey"]
data = []

for i in range(1000):
    current_color = random.choice(colors)
    current_tag = random.randint(1000, 9999)
    data.append({
        "id": i,
        "vector": [ random.uniform(-1, 1) for _ in range(5) ],
        "color": current_color,
        "tag": current_tag,
        "color_tag": f"{current_color}_{str(current_tag)}"
    })

print(data[0])
```

```java
// 3. 插入随机生成的向量
List<String> colors = Arrays.asList("green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey");
List<JSONObject> data = new ArrayList<>();

for (int i=0; i<1000; i++) {
    Random rand = new Random();
    String current_color = colors.get(rand.nextInt(colors.size()-1));
    int current_tag = rand.nextInt(8999) + 1000;
    JSONObject row = new JSONObject();
    row.put("id", Long.valueOf(i));
    row.put("vector", Arrays.asList(rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat()));
    row.put("color", current_color);
    row.put("tag", current_tag);
    row.put("color_tag", current_color + "_" + String.valueOf(rand.nextInt(8999) + 1000));
    data.add(row);
}

System.out.println(JSONObject.toJSON(data.get(0)));   
```

```javascript
// 3. 插入随机生成的向量
const colors = ["green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey"]
var data = []

for (let i = 0; i < 1000; i++) {
    const current_color = colors[Math.floor(Math.random() * colors.length)]
    const current_tag = Math.floor(Math.random() * 8999 + 1000)
    data.push({
        id: i,
        vector: [Math.random(), Math.random(), Math.random(), Math.random(), Math.random()],
        color: current_color,
        tag: current_tag,
        color_tag: `${current_color}_${current_tag}`
    })
}

console.log(data[0])
```

您可以通过查看第一个条目来查看生成数据的结构。

```
{
    id: 0,
    vector: [
        0.1275656405044483,
        0.47417858592773277,
        0.13858264437643286,
        0.2390904907020377,
        0.8447862593689635
    ],
    color: 'blue',
    tag: 2064,
    color_tag: 'blue_2064'
}
```

### 插入数据

<div class="language-python">

使用 [`insert()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/insert.md) 方法将数据插入集合。

</div>

<div class="language-java">

使用 [`insert()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/insert.md) 方法将数据插入集合。

</div>

<div class="language-javascript">
使用 [`insert()`](https://milvus.io/api-reference/node/v2.4.x/Vector/insert.md) 方法将数据插入集合中。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
res = client.insert(
    collection_name="test_collection",
    data=data
)

print(res)

# 输出
#
# {
#     "insert_count": 1000,
#     "ids": [
#         0,
#         1,
#         2,
#         3,
#         4,
#         5,
#         6,
#         7,
#         8,
#         9,
#         "(990 more items hidden)"
#     ]
# }
```

```java
// 3.1 将数据插入集合
InsertReq insertReq = InsertReq.builder()
    .collectionName("test_collection")
    .data(data)
    .build();

InsertResp insertResp = client.insert(insertReq);

System.out.println(JSONObject.toJSON(insertResp));

// 输出:
// {"insertCnt": 1000}
```

```javascript
res = await client.insert({
    collection_name: "test_collection",
    data: data,
})

console.log(res.insert_cnt)

// 输出
// 
// 1000
// 
```

## 使用分区键

一旦您已经对集合进行了索引和加载，并插入了数据，您可以使用分区键进行相似度搜索。

<div class="language-python">

有关参数的更多信息，请参阅 SDK 参考中的 [`search()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/search.md)。

</div>

<div class="language-java">

有关参数的更多信息，请参阅 SDK 参考中的 [`search()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/search.md)。

</div>

<div class="language-javascript">

有关参数的更多信息，请参阅 SDK 参考中的 [`search()`](https://milvus.io/api-reference/node/v2.4.x/Vector/search.md)。

</div>

<div class="admonition note">

<p><b>注意</b></p>

<p>要使用分区键进行相似度搜索，您应该在搜索请求的布尔表达式中包含以下之一：</p>
<ul>
<li><p><code>expr='&lt;partition_key&gt;=="xxxx"'</code></p></li>
<li><p><code>expr='&lt;partition_key&gt; in ["xxx", "xxx"]'</code></p></li>
</ul>
<p>请将 <code>&lt;partition_key&gt;</code> 替换为指定为分区键的字段名称。</p>

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```python
# 4. 使用分区键进行搜索
查询向量 = [[0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]]

res = client.search(
    collection_name="test_collection",
    data=query_vectors,
    filter="color == 'green'",
    search_params={"metric_type": "L2", "params": {"nprobe": 10}},
    output_fields=["id", "color_tag"],
    limit=3
)

print(res)

# 输出
#
# [
#     [
#         {
#             "id": 970,
#             "distance": 0.5770174264907837,
#             "entity": {
#                 "id": 970,
#                 "color_tag": "green_9828"
#             }
#         },
#         {
#             "id": 115,
#             "distance": 0.6898155808448792,
#             "entity": {
#                 "id": 115,
#                 "color_tag": "green_4073"
#             }
#         },
#         {
#             "id": 899,
#             "distance": 0.7028976678848267,
#             "entity": {
#                 "id": 899,
#                 "color_tag": "green_9897"
#             }
#         }
#     ]
# ]
```
```java
// 4. 使用分区键进行搜索
List<List<Float>> query_vectors = Arrays.asList(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f));

SearchReq searchReq = SearchReq.builder()
    .collectionName("test_collection")
    .data(query_vectors)
    .filter("color == \"green\"")
    .topK(3)
    .build();

SearchResp searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));   

// 输出:
// {"searchResults": [[
//     {
//         "distance": 1.0586997,
//         "id": 414,
//         "entity": {}
//     },
//     {
//         "distance": 0.981384,
//         "id": 293,
//         "entity": {}
//     },
//     {
//         "distance": 0.9548756,
//         "id": 325,
//         "entity": {}
//     }
// ]]}
```

```javascript
// 4. 使用分区键进行搜索
const query_vectors = [0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]

res = await client.search({
    collection_name: "test_collection",
    data: query_vectors,
    filter: "color == 'green'",
    output_fields: ["color_tag"],
    limit: 3
})

console.log(res.results)

// 输出
// 
// [
//   { score: 2.402090549468994, id: '135', color_tag: 'green_2694' },
//   { score: 2.3938629627227783, id: '326', color_tag: 'green_7104' },
//   { score: 2.3235254287719727, id: '801', color_tag: 'green_3162' }
// ]
// 
```

## 典型用例

您可以利用分区键功能来实现更好的搜索性能并启用多租户支持。这可以通过为每个实体分配一个特定于租户的值作为分区键字段来实现。在搜索或查询集合时，您可以通过在布尔表达式中包含分区键字段来按租户特定值过滤实体。这种方法确保了租户之间的数据隔离，并避免扫描不必要的分区。