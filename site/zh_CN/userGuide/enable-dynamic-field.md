---
id: enable-dynamic-field.md
title: 启用动态字段
---

# 启用动态字段

本页面将解释如何在集合中使用动态字段进行灵活的数据插入和检索。

## 概述

Milvus允许您通过设置每个特定字段的名称和数据类型来定义集合的模式，以便您可以在这些字段上创建索引，以提高搜索性能。

一旦定义了字段，您需要在插入数据时包含这些字段。如果您的所有数据条目中并非始终包含某些字段，该怎么办？这就是动态字段发挥作用的地方。

集合中的动态字段是一个名为$meta的保留JSON字段。它可以保存非模式定义字段及其值，以键值对的形式。使用动态字段，您可以搜索和查询既包括模式定义字段，也包括它们可能具有的任何非模式定义字段。

## 启用动态字段

在为集合定义模式时，您可以将enable_dynamic_field设置为True，以启用保留的动态字段，表示稍后插入的任何非模式定义字段及其值将保存为键值对在保留的动态字段中。

以下代码片段创建了一个具有两个模式定义字段（即id和vector）并启用动态字段的集合。

```python
For more information on parameters, refer to [`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md) in the SDK reference.
```

```java
For more information on parameters, refer to [`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md) in the SDK reference.
```

```javascript
For more information on parameters, refer to [`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md) in the SDK reference.


<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```
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
    # highlight-next-line
    enable_dynamic_field=True,
)

schema.add_field(field_name="id", datatype=DataType.INT64, is_primary=True)
schema.add_field(field_name="vector", datatype=DataType.FLOAT_VECTOR, dim=5)

index_params = MilvusClient.prepare_index_params()

index_params.add_index(
    field_name="id",
    index_type="STL_SORT"
)

index_params.add_index(
    field_name="vector",
    index_type="IVF_FLAT",
    metric_type="L2",
    params={"nlist": 1024}
)

client.create_collection(
    collection_name="test_collection",
    schema=schema,
    index_params=index_params
)

res = client.get_load_state(
    collection_name="test_collection"
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
import io.milvus.v2.common.DataType;
import io.milvus.v2.common.IndexParam;
import io.milvus.v2.service.collection.request.AddFieldReq;
import io.milvus.v2.service.collection.request.CreateCollectionReq;
import io.milvus.v2.service.collection.request.GetLoadStateReq;

String CLUSTER_ENDPOINT = "http://localhost:19530";

// 1. 连接到 Milvus 服务器
ConnectConfig connectConfig = ConnectConfig.builder()
    .uri(CLUSTER_ENDPOINT)
    .build();

MilvusClientV2 client = new MilvusClientV2(connectConfig);

// 2. 在自定义设置模式下创建一个集合

// 2.1 创建模式
CreateCollectionReq.CollectionSchema schema = client.createSchema();

// 2.2 向模式中添加字段
schema.addField(AddFieldReq.builder().fieldName("id").dataType(DataType.Int64).isPrimaryKey(true).autoID(false).build());
schema.addField(AddFieldReq.builder().fieldName("vector").dataType(DataType.FloatVector).dimension(5).build());

// 2.3 准备索引参数
IndexParam indexParamForIdField = IndexParam.builder()
    .fieldName("id")
    .indexType(IndexParam.IndexType.STL_SORT)
    .build();

IndexParam indexParamForVectorField = IndexParam.builder()
    .fieldName("vector")
    .indexType(IndexParam.IndexType.IVF_FLAT)
    .metricType(IndexParam.MetricType.IP)
    .extraParams(Map.of("nlist", 1024))
    .build();

List<IndexParam> indexParams = new ArrayList<>();
indexParams.add(indexParamForIdField);
indexParams.add(indexParamForVectorField);

// 2.4 使用模式和索引参数创建一个集合
CreateCollectionReq customizedSetupReq = CreateCollectionReq.builder()
    .collectionName("customized_setup")
    .collectionSchema(schema)
    .indexParams(indexParams)
    // highlight-next-line
    .enableDynamicField(true)
    .build();

client.createCollection(customizedSetupReq);

Thread.sleep(5000);

// 2.5 获取集合的加载状态
GetLoadStateReq customSetupLoadStateReq1 = GetLoadStateReq.builder()
    .collectionName("customized_setup")
    .build();

boolean res = client.getLoadState(customSetupLoadStateReq1);

System.out.println(res);

// 输出:
// true
```
```javascript
const { MilvusClient, DataType, sleep } = require("@zilliz/milvus2-sdk-node")

const address = "http://localhost:19530"

async function main() {
// 1. 设置 Milvus 客户端
client = new MilvusClient({address}); 

// 2. 创建集合
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
]

// 2.2 准备索引参数
const index_params = [{
    field_name: "id",
    index_type: "STL_SORT"
},{
    field_name: "vector",
    index_type: "IVF_FLAT",
    metric_type: "IP",
    params: { nlist: 1024}
}]

// 2.3 使用字段和索引参数创建集合
res = await client.createCollection({
    collection_name: "test_collection",
    fields: fields, 
    index_params: index_params,
    // highlight-next-line
    enable_dynamic_field: true
})

console.log(res.error_code)

// 输出
// 
// 成功
// 

res = await client.getLoadState({
    collection_name: "test_collection",
})  

console.log(res.state)

// 输出
// 
// LoadStateLoaded
// 
```
## 插入动态数据

创建了集合之后，就可以开始向集合中插入数据，包括动态数据。

### 准备数据

在这一部分，您需要为稍后的插入准备一些随机生成的数据。

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
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

然后，您可以安全地将数据插入集合。

<div class="language-python">

有关参数的更多信息，请参考 SDK 参考中的 [`insert()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/insert.md)。

</div>

<div class="language-java">
要了解更多有关参数的信息，请参考 SDK 参考中的 [`insert()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/insert.md)。

</div>

<div class="language-javascript">

要了解更多有关参数的信息，请参考 SDK 参考中的 [`insert()`](https://milvus.io/api-reference/node/v2.4.x/Vector/insert.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
res = client.insert(
    collection_name="test_collection",
    data=data,
)

print(res)

# Output
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

time.sleep(5)
```

```java
// 3.1 Insert data into the collection
InsertReq insertReq = InsertReq.builder()
    .collectionName("customized_setup")
    .data(data)
    .build();

InsertResp insertResp = client.insert(insertReq);

System.out.println(JSONObject.toJSON(insertResp));

// Output:
// {"insertCnt": 1000}

Thread.sleep(5000);
```

```javascript
res = await client.insert({
    collection_name: "test_collection",
    data: data,
})

console.log(res.insert_cnt)

// Output
// 
// 1000
// 

await sleep(5000)
```

## 使用动态字段进行搜索

如果您已启用动态字段并插入了非架构定义字段，则可以在搜索或查询的过滤表达式中使用这些字段，如下所示。

<div class="language-python">

要了解更多有关参数的信息，请参考 SDK 参考中的 [`search()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/search.md)。

</div>

<div class="language-java">

要了解更多有关参数的信息，请参考 SDK 参考中的 [`search()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/search.md)。

</div>

<div class="language-javascript">

要了解更多有关参数的信息，请参考 SDK 参考中的 [`search()`](https://milvus.io/api-reference/node/v2.4.x/Vector/search.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```python
# 4. 使用动态字段进行搜索
query_vectors = [[0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]]

res = client.search(
    collection_name="test_collection",
    data=query_vectors,
    filter="color in [\"red\", \"green\"]",
    search_params={"metric_type": "L2", "params": {"nprobe": 10}},
    limit=3
)

print(res)

# 输出
#
# [
#     [
#         {
#             "id": 863,
#             "distance": 0.188413605093956,
#             "entity": {
#                 "id": 863,
#                 "color_tag": "red_2371"
#             }
#         },
#         {
#             "id": 799,
#             "distance": 0.29188022017478943,
#             "entity": {
#                 "id": 799,
#                 "color_tag": "red_2235"
#             }
#         },
#         {
#             "id": 564,
#             "distance": 0.3492690920829773,
#             "entity": {
#                 "id": 564,
#                 "color_tag": "red_9186"
#             }
#         }
#     ]
# ]
```
```java
// 4. 使用非模式定义字段进行搜索
List<List<Float>> queryVectors = Arrays.asList(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f));

SearchReq searchReq = SearchReq.builder()
    .collectionName("customized_setup")
    .data(queryVectors)
    .filter("$meta[\"color\"] in [\"red\", \"green\"]")
    .outputFields(List.of("id", "color_tag"))
    .topK(3)
    .build();

SearchResp searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));

// 输出:
// {"searchResults": [[
//     {
//         "distance": 1.3159835,
//         "id": 979,
//         "entity": {
//             "color_tag": "red_7155",
//             "id": 979
//         }
//     },
//     {
//         "distance": 1.0744804,
//         "id": 44,
//         "entity": {
//             "color_tag": "green_8006",
//             "id": 44
//         }
//     },
//     {
//         "distance": 1.0060014,
//         "id": 617,
//         "entity": {
//             "color_tag": "red_4056",
//             "id": 617
//         }
//     }
// ]]}
```

```javascript
// 4. 使用非模式定义字段进行搜索
const query_vectors = [[0.1, 0.2, 0.3, 0.4, 0.5]]

res = await client.search({
    collection_name: "test_collection",
    data: query_vectors,
    filter: "color in [\"red\", \"green\"]",
    output_fields: ["color_tag"],
    limit: 3
})

console.log(res.results)

// 输出
// 
// [
//   { score: 1.2284551858901978, id: '301', color_tag: 'red_1270' },
//   { score: 1.2195171117782593, id: '205', color_tag: 'red_2780' },
//   { score: 1.2055039405822754, id: '487', color_tag: 'red_6653' }
// ]
// 
```

## 总结

值得注意的是，在定义集合模式时，__color__、__tag__ 和 __color_tag__ 这些字段可能不存在，但在进行搜索和查询时，你可以将它们用作模式定义字段。

如果非模式定义字段的名称包含除数字、字母和下划线之外的字符，例如加号（+）、星号（*）或美元符号（\$），在布尔表达式中使用它或将其包含在输出字段中时，必须像下面的代码片段中所示，将键包含在 __$meta[]__ 中。

```python
... 
filter='$meta["$key"] in ["a", "b", "c"]', 
output_fields='$meta["$key"]'  
...
```