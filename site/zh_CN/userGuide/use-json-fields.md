---
id: use-json-fields.md
title: 使用 JSON 字段
---

# 使用 JSON 字段

本指南介绍如何使用 JSON 字段，包括插入 JSON 值以及在 JSON 字段中使用基本和高级运算符进行搜索和查询。

## 概述

JSON 是 JavaScript 对象表示法的缩写，是一种轻量且简单的基于文本的数据格式。JSON 中的数据以键值对的形式结构化，其中每个键都是一个映射到数字、字符串、布尔值、列表或数组的值的字符串。在 Milvus 集群中，可以将字典作为字段值存储在集合中。

例如，以下代码随机生成键值对，每个键值对包含一个具有键 __color__ 的 JSON 字段。

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
    current_coord = [random.randint(0, 40) for _ in range(3)]
    current_ref = [[random.choice(colors) for _ in range(3)] for _ in range(3)]
    data.append({
        "id": i,
        "vector": [random.uniform(-1, 1) for _ in range(5)],
        "color": {
            "label": current_color,
            "tag": current_tag,
            "coord": current_coord,
            "ref": current_ref
        }
    })

print(data[0])
```

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.Random;

import com.alibaba.fastjson.JSONObject;

// 3. 将随机生成的向量和 JSON 数据插入集合
List<String> colors = Arrays.asList("green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey");
List<JSONObject> data = new ArrayList<>();

for (int i=0; i<1000; i++) {
    Random rand = new Random();
    String current_color = colors.get(rand.nextInt(colors.size()-1));
    Integer current_tag = rand.nextInt(8999) + 1000;
    List<Integer> current_coord = Arrays.asList(rand.nextInt(40), rand.nextInt(40), rand.nextInt(40));
    List<List<String>> current_ref = Arrays.asList(
        Arrays.asList(colors.get(rand.nextInt(colors.size()-1)), colors.get(rand.nextInt(colors.size()-1)), colors.get(rand.nextInt(colors.size()-1))),
        Arrays.asList(colors.get(rand.nextInt(colors.size()-1)), colors.get(rand.nextInt(colors.size()-1)), colors.get(rand.nextInt(colors.size()-1))),
        Arrays.asList(colors.get(rand.nextInt(colors.size()-1)), colors.get(rand.nextInt(colors.size()-1)), colors.get(rand.nextInt(colors.size()-1)))
    );
    JSONObject row = new JSONObject();
    row.put("id", Long.valueOf(i));
    row.put("vector", Arrays.asList(rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat()));
    JSONObject color = new JSONObject();
    color.put("label", current_color);
    color.put("tag", current_tag);
    color.put("coord", current_coord);
    color.put("ref", current_ref);
    row.put("color", color);
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
    const current_coord = Array(3).fill(0).map(() => Math.floor(Math.random() * 40))
    const current_ref = [ Array(3).fill(0).map(() => colors[Math.floor(Math.random() * colors.length)]) ]

    data.push({
        id: i,
        vector: [Math.random(), Math.random(), Math.random(), Math.random(), Math.random()],
        color: {
            label: current_color,
            tag: current_tag,
            coord: current_coord,
            ref: current_ref
        }
    })
}

console.log(data[0])
```

您可以通过查看第一个条目来查看生成数据的结构。

```
{
    "id": 0,
    "vector": [
        -0.8017921296923975,
        0.550046715206634,
        0.764922589768134,
        0.6371433836123146,
        0.2705233937454232
    ],
    "color": {
        "label": "blue",
        "tag": 9927,
        "coord": [
            22,
            36,
            6
        ],
        "ref": [
            [
                "blue",
                "green",
                "white"
            ],
            [
                "black",
                "green",
                "pink"
            ],
            [
                "grey",
                "black",
                "brown"
            ]
        ]
    }
}
```

<div class="admonition note">

<p><b>注意事项</b></p>

<ul>
<li><p>确保列表或数组中的所有值都是相同的数据类型。</p></li>
<li><p>JSON 字段值中的任何嵌套字典将被视为字符串。</p></li>
<li><p>仅使用字母数字字符和下划线来命名 JSON 键，因为其他字符可能会导致过滤或搜索时出现问题。</p></li>
<li>目前，不支持对 JSON 字段进行索引，这可能会使过滤变得耗时。但是，这个限制将在即将发布的版本中解决。</li>
</ul>

</div>

## 定义 JSON 字段

要定义 JSON 字段，只需按照定义其他类型字段的相同步骤进行操作。

<div class="language-python">
```
更多参数信息，请参考 SDK 参考文档中的 [`MilvusClient`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md)、[`create_schema()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md)、[`add_field()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/CollectionSchema/add_field.md)、[`add_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/add_index.md)、[`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md) 和 [`get_load_state()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/get_load_state.md)。

</div>

<div class="language-java">

更多参数信息，请参考 SDK 参考文档中的 [`MilvusClientV2`](https://milvus.io/api-reference/java/v2.4.x/v2/Client/MilvusClientV2.md)、[`createSchema()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createSchema.md)、[`addField()`](https://milvus.io/api-reference/java/v2.4.x/v2/CollectionSchema/addField.md)、[`IndexParam`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/IndexParam.md)、[`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md) 和 [`getLoadState()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/getLoadState.md)。

</div>

<div class="language-javascript">

更多参数信息，请参考 SDK 参考文档中的 [`MilvusClient`](https://milvus.io/api-reference/node/v2.4.x/Client/MilvusClient.md)、[`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md) 和 [`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md)。

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
    enable_dynamic_field=False,
)

schema.add_field(field_name="id", datatype=DataType.INT64, is_primary=True)
schema.add_field(field_name="vector", datatype=DataType.FLOAT_VECTOR, dim=5)
# highlight-next-line
schema.add_field(field_name="color", datatype=DataType.JSON)

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

// highlight-start    
schema.addField(AddFieldReq.builder()
    .fieldName("color")
    .dataType(DataType.JSON)
    .build());
// highlight-end

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

// 2.4 使用模式和索引参数创建集合
CreateCollectionReq customizedSetupReq = CreateCollectionReq.builder()
    .collectionName("test_collection")
    .collectionSchema(schema)
    .indexParams(indexParams)         
    .build();

client.createCollection(customizedSetupReq);

// 2.5 检查集合是否已加载
GetLoadStateReq getLoadStateReq = GetLoadStateReq.builder()
    .collectionName("test_collection")
    .build();

Boolean isLoaded = client.getLoadState(getLoadStateReq);

System.out.println(isLoaded);

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
// highlight-start
    {
        name: "color",
        data_type: DataType.JSON,
    }
// highlight-end
]

// 2.2 准备索引参数
const index_params = [{
    field_name: "vector",
    index_type: "IVF_FLAT",
    metric_type: "IP",
    params: { nlist: 1024}
}]

// 2.3 使用字段和索引参数创建集合
res = await client.createCollection({
    collection_name: "test_collection",
    fields: fields, 
    index_params: index_params
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

<div class="language-python">

要了解更多关于参数的信息，请参考 [`MilvusClient`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md), [`create_schema()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md), [`add_field()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/CollectionSchema/add_field.md), [`add_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/add_index.md), [`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md), 以及 SDK 参考中的 [`get_load_state()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/get_load_state.md)。

</div>

<div class="language-java">

要了解更多关于参数的信息，请参考 [`MilvusClientV2`](https://milvus.io/api-reference/java/v2.4.x/v2/Client/MilvusClientV2.md), [`createSchema()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createSchema.md), [`addField()`](https://milvus.io/api-reference/java/v2.4.x/v2/CollectionSchema/addField.md), [`IndexParam`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/IndexParam.md), [`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md), 以及 SDK 参考中的 [`getLoadState()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/getLoadState.md)。

</div>

<div class="language-javascript">

要了解更多关于参数的信息，请参考 [`MilvusClient`](https://milvus.io/api-reference/node/v2.4.x/Client/MilvusClient.md), [`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md), 以及 SDK 参考中的 [`getLoadState()`](https://milvus.io/api-reference/node/v2.4.x/Management/getLoadState.md)。

</div>

## 插入字段值

在从 `CollectionSchema` 对象创建集合之后，可以将类似上面的字典插入其中。

<div class="language-python">

使用 [`insert()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/insert.md) 方法将数据插入集合。

</div>

<div class="language-java">

使用 [`insert()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/insert.md) 方法将数据插入集合。

</div>

<div class="language-javascript">

使用 [`insert()`](https://milvus.io/api-reference/node/v2.4.x/Vector/insert.md) 方法将数据插入集合。

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
// 3.1 向集合中插入数据
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
// 3. 向集合中插入随机生成的向量
const colors = ["green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey"]
var data = []

for (let i = 0; i < 1000; i++) {
    const current_color = colors[Math.floor(Math.random() * colors.length)]
    const current_tag = Math.floor(Math.random() * 8999 + 1000)
    const current_coord = Array(3).fill(0).map(() => Math.floor(Math.random() * 40))
    const current_ref = [ Array(3).fill(0).map(() => colors[Math.floor(Math.random() * colors.length)]) ]

    data.push({
        id: i,
        vector: [Math.random(), Math.random(), Math.random(), Math.random(), Math.random()],
        color: {
            label: current_color,
            tag: current_tag,
            coord: current_coord,
            ref: current_ref
        }
    })
}

console.log(data[0])

// 输出
// 
// {
//   id: 0,
//   vector: [
//     0.11455530974226114,
//     0.21704086958595314,
//     0.9430119822312437,
//     0.7802712923612023,
//     0.9106927960926137
//   ],
//   color: { label: 'grey', tag: 7393, coord: [ 22, 1, 22 ], ref: [ [Array] ] }
// }
// 

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

## 基本标量过滤

在添加完所有数据后，您可以使用 JSON 字段中的键进行搜索和查询，方式与标准标量字段相同。

<div class="language-python">

有关参数的更多信息，请参阅 SDK 参考中的 [`search()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/search.md)。

</div>

<div class="language-java">

有关参数的更多信息，请参阅 SDK 参考中的 [`search()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/search.md)。

</div>

<div class="language-javascript">

有关参数的更多信息，请参阅 SDK 参考中的 [`search()`](https://milvus.io/api-reference/node/v2.4.x/Vector/search.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```
```python
# 4. 使用 JSON 字段进行基本搜索
query_vectors = [ [ random.uniform(-1, 1) for _ in range(5) ]]

res = client.search(
    collection_name="test_collection",
    data=query_vectors,
    # highlight-next-line
    filter='color["label"] in ["red"]',
    search_params={
        "metric_type": "L2",
        "params": {"nprobe": 16}
    },
    output_fields=["id", "color"],
    limit=3
)

print(res)

# 输出
#
# [
#     [
#         {
#             "id": 460,
#             "distance": 0.4016231596469879,
#             "entity": {
#                 "id": 460,
#                 "color": {
#                     "label": "red",
#                     "tag": 5030,
#                     "coord": [14, 32, 40],
#                     "ref": [
#                         [ "pink", "green", "brown" ],
#                         [ "red", "grey", "black"],
#                         [ "red", "yellow", "orange"]
#                     ]
#                 }
#             }
#         },
#         {
#             "id": 785,
#             "distance": 0.451080858707428,
#             "entity": {
#                 "id": 785,
#                 "color": {
#                     "label": "red",
#                     "tag": 5290,
#                     "coord": [31, 13, 23],
#                     "ref": [
#                         ["yellow", "pink", "pink"],
#                         ["purple", "grey", "orange"],
#                         ["grey", "purple", "pink"]
#                     ]
#                 }
#             }
#         },
#         {
#             "id": 355,
#             "distance": 0.5839247703552246,
#             "entity": {
#                 "id": 355,
#                 "color": {
#                     "label": "red",
#                     "tag": 8725,
#                     "coord": [5, 10, 22],
#                     "ref": [
#                         ["white", "purple", "yellow"],
#                         ["white", "purple", "white"],
#                         ["orange", "white", "pink"]
#                     ]
#                 }
#             }
#         }
#     ]
# ]
```
```java
// 4. 使用 JSON 字段进行基本搜索
List<List<Float>> query_vectors = Arrays.asList(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f));

SearchReq searchReq = SearchReq.builder()
    .collectionName("test_collection")
    .data(query_vectors)
    // highlight-next-line
    .filter("color[\"label\"] in [\"red\"]")
    .outputFields(Arrays.asList("id", "color"))
    .topK(3)
    .build();

SearchResp searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));

// 输出:
// {"searchResults": [[
//     {
//         "distance": 1.2636482,
//         "id": 290,
//         "entity": {
//             "color": {
//                 "coord": [32,37,32],
//                 "ref": [
//                     ["green", "blue", "yellow"],
//                     ["yellow", "pink", "pink"],
//                     ["purple", "red", "brown"]
//                 ],
//                 "label": "red",
//                 "tag": 8949
//             },
//             "id": 290
//         }
//     },
//     {
//         "distance": 1.002122,
//         "id": 629,
//         "entity": {
//             "color": {
//                 "coord": [23,5,35],
//                 "ref": [
//                     ["black", ""yellow", "black"],
//                     ["black", "purple", "white"],
//                     ["black", "brown", "orange"]
//                 ],
//                 "label": "red",
//                 "tag": 5072
//             },
//             "id": 629
//         }
//     },
//     {
//         "distance": 0.9542817,
//         "id": 279,
//         "entity": {
//             "color": {
//                 "coord": [20,33,33],
//                 "ref": [
//                     ["yellow", "white", "brown"],
//                     ["black", "white", "purple"],
//                     ["green", "brown", "blue"]
//                 ],
//                 "label": "red",
//                 "tag": 4704
//             },
//             "id": 279
//         }
//     }
// ]]}
```
```javascript
// 4. 使用 JSON 字段进行基本搜索
query_vectors = [[0.6765405125697714, 0.759217474274025, 0.4122471841491111, 0.3346805565394215, 0.09679748345514638]]

res = await client.search({
    collection_name: "test_collection",
    data: query_vectors,
    // highlight-next-line
    filter: 'color["label"] in ["red"]',
    output_fields: ["color", "id"],
    limit: 3
})

console.log(JSON.stringify(res.results, null, 4))

// 输出
// 
// [
//     {
//         "score": 1.777988076210022,
//         "id": "595",
//         "color": {
//             "label": "red",
//             "tag": 7393,
//             "coord": [31,34,18],
//             "ref": [
//                 ["grey", "white", "orange"]
//             ]
//         }
//     },
//     {
//         "score": 1.7542595863342285,
//         "id": "82",
//         "color": {
//             "label": "red",
//             "tag": 8636,
//             "coord": [4,37,29],
//             "ref": [
//                 ["brown", "brown", "pink"]
//             ]
//         }
//     },
//     {
//         "score": 1.7537562847137451,
//         "id": "748",
//         "color": {
//             "label": "red",
//             "tag": 1626,
//             "coord": [31,4,25
//             ],
//             "ref": [
//                 ["grey", "green", "blue"]
//             ]
//         }
//     }
// ]
// 
```
## 高级标量过滤

Milvus提供了一组用于JSON字段中标量过滤的高级过滤器。这些过滤器包括`JSON_CONTAINS`、`JSON_CONTAINS_ALL`和`JSON_CONTAINS_ANY`。

- 过滤出所有具有`["blue", "brown", "grey"]`作为参考颜色集的实体。

    <div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
    </div>

    ```python
    # 5. 在JSON字段中进行高级搜索
    
    res = client.query(
        collection_name="test_collection",
        data=query_vectors,
        filter='JSON_CONTAINS(color["ref"], ["blue", "brown", "grey"])',
        output_fields=["id", "color"],
        limit=3
    )
    
    print(res)
    
    # 输出
    #
    # [
    #     {
    #         "id": 79,
    #         "color": {
    #             "label": "orange",
    #             "tag": 8857,
    #             "coord": [
    #                 10,
    #                 14,
    #                 5
    #             ],
    #             "ref": [
    #                 [
    #                     "yellow",
    #                     "white",
    #                     "green"
    #                 ],
    #                 [
    #                     "blue",
    #                     "purple",
    #                     "purple"
    #                 ],
    #                 [
    #                     "blue",
    #                     "brown",
    #                     "grey"
    #                 ]
    #             ]
    #         }
    #     },
    #     {
    #         "id": 371,
    #         "color": {
    #             "label": "black",
    #             "tag": 1324,
    #             "coord": [
    #                 2,
    #                 18,
    #                 32
    #             ],
    #             "ref": [
    #                 [
    #                     "purple",
    #                     "orange",
    #                     "brown"
    #                 ],
    #                 [
    #                     "blue",
    #                     "brown",
    #                     "grey"
    #                 ],
    #                 [
    #                     "purple",
    #                     "blue",
    #                     "blue"
    #                 ]
    #             ]
    #         }
    #     },
    #     {
    #         "id": 590,
    #         "color": {
    #             "label": "red",
    #             "tag": 3340,
    #             "coord": [
    #                 13,
    #                 21,
    #                 13
    #             ],
    #             "ref": [
    #                 [
    #                     "yellow",
    #                     "yellow",
    #                     "red"
    #                 ],
    #                 [
    #                     "blue",
    #                     "brown",
    #                     "grey"
    #                 ],
    #                 [
    #                     "pink",
```java
// 5. 在 JSON 字段中进行高级搜索
searchReq = SearchReq.builder()
    .collectionName("test_collection")
    .data(query_vectors)
    .filter("JSON_CONTAINS(color[\"ref\"], [\"purple\", \"pink\", \"orange\"])")
    .outputFields(Arrays.asList("id", "color"))
    .topK(3)
    .build();

searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));

// 输出:
// {"searchResults": [[
//     {
//         "distance": 1.1811467,
//         "id": 180,
//         "entity": {
//             "color": {
//                 "coord": [
//                     17,
//                     26,
//                     14
//                 ],
//                 "ref": [
//                     [
//                         "white",
//                         "black",
//                         "brown"
//                     ],
//                     [
//                         "purple",
//                         "pink",
//                         "orange"
//                     ],
//                     [
//                         "black",
//                         "pink",
//                         "red"
//                     ]
//                 ],
//                 "label": "green",
//                 "tag": 2470
//             },
//             "id": 180
//         }
//     },
//     {
//         "distance": 0.6487204,
//         "id": 331,
//         "entity": {
//             "color": {
//                 "coord": [
//                     16,
//                     32,
//                     23
//                 ],
//                 "ref": [
//                     [
//                         "purple",
//                         "pink",
//                         "orange"
//                     ],
//                     [
//                         "brown",
//                         "red",
//                         "orange"
//                     ],
//                     [
//                         "red",
//                         "yellow",
//                         "brown"
//                     ]
//                 ],
//                 "label": "white",
//                 "tag": 1236
//             },
//             "id": 331
//         }
//     },
//     {
//         "distance": 0.59387654,
//         "id": 483,
//         "entity": {
//             "color": {
//                 "coord": [
//                     8,
//                     33,
//                     2
//                 ],
```

```javascript
// 5. JSON字段内的高级搜索
res = await client.search({
    collection_name: "test_collection",
    data: query_vectors,
    filter: 'JSON_CONTAINS(color["ref"], ["blue", "brown", "grey"])',
    output_fields: ["color", "id"],
    limit: 3
})

console.log(JSON.stringify(res.results, null, 4))

// 输出
// 
// [
//     {
//         "id": 79,
//         "color": {
//             "label": "orange",
//             "tag": 8857,
//             "coord": [
//                 10,
//                 14,
//                 5
//             ],
//             "ref": [
//                 [
//                     "yellow",
//                     "white",
//                     "green"
//                 ],
//                 [
//                     "blue",
//                     "purple",
//                     "purple"
//                 ],
//                 [
//                     "blue",
//                     "brown",
//                     "grey"
//                 ]
//             ]
//         }
//     },
//     {
//         "id": 371,
//         "color": {
//             "label": "black",
//             "tag": 1324,
//             "coord": [
//                 2,
//                 18,
//                 32
//             ],
//             "ref": [
//                 [
//                     "purple",
//                     "orange",
//                     "brown"
//                 ],
//                 [
//                     "blue",
//                     "brown",
//                     "grey"
//                 ],
//                 [
//                     "purple",
//                     "blue",
//                     "blue"
//                 ]
//             ]
//         }
//     },
//     {
//         "id": 590,
//         "color": {
//             "label": "red",
//             "tag": 3340,
//             "coord": [
//                 13,
```python
res = client.query(
    collection_name="test_collection",
    data=query_vectors,
    filter='JSON_CONTAINS_ALL(color["coord"], [4, 5])',
    output_fields=["id", "color"],
    limit=3
)

print(res)

# 输出
#
# [
#     {
#         "id": 281,
#         "color": {
#             "label": "red",
#             "tag": 3645,
#             "coord": [
#                 5,
#                 33,
#                 4
#             ],
#             "ref": [
#                 [
#                     "orange",
#                     "blue",
#                     "pink"
#                 ],
#                 [
#                     "purple",
#                     "blue",
#                     "purple"
#                 ],
#                 [
#                     "black",
#                     "brown",
#                     "yellow"
#                 ]
#             ]
#         }
#     },
#     {
#         "id": 464,
#         "color": {
#             "label": "brown",
#             "tag": 6261,
#             "coord": [
#                 5,
#                 9,
#                 4
#             ],
#             "ref": [
#                 [
#                     "purple",
#                     "purple",
#                     "brown"
#                 ],
#                 [
#                     "black",
#                     "pink",
#                     "white"
#                 ],
#                 [
#                     "brown",
#                     "grey",
#                     "brown"
#                 ]
#             ]
#         }
#     },
#     {
#         "id": 567,
#         "color": {
#             "label": "green",
#             "tag": 4589,
#             "coord": [
#                 5,
#                 39,
#                 4
#             ],
#             "ref": [
```
```java
searchReq = SearchReq.builder()
    .collectionName("test_collection")
    .data(query_vectors)
    .filter("JSON_CONTAINS_ALL(color[\"coord\"], [4, 5])")
    .outputFields(Arrays.asList("id", "color"))
    .topK(3)
    .build();

searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));     

// 输出:
// {"searchResults": [[
//     {
//         "distance": 0.77485126,
//         "id": 304,
//         "entity": {
//             "color": {
//                 "coord": [
//                     4,
//                     5,
//                     13
//                 ],
//                 "ref": [
//                     [
//                         "purple",
//                         "pink",
//                         "brown"
//                     ],
//                     [
//                         "orange",
//                         "red",
//                         "blue"
//                     ],
//                     [
//                         "yellow",
//                         "blue",
//                         "purple"
//                     ]
//                 ],
//                 "label": "blue",
//                 "tag": 7228
//             },
//             "id": 304
//         }
//     },
//     {
//         "distance": 0.68138736,
//         "id": 253,
//         "entity": {
//             "color": {
//                 "coord": [
//                     5,
//                     38,
//                     4
//                 ],
//                 "ref": [
//                     [
//                         "black",
//                         "pink",
//                         "blue"
//                     ],
//                     [
//                         "pink",
//                         "brown",
//                         "pink"
//                     ],
//                     [
//                         "red",
//                         "pink",
//                         "orange"
//                     ]
//                 ],
//                 "label": "blue",
//                 "tag": 6935
//             },
//             "id": 253
//         }
```
```    
```javascript
res = await client.search({
    collection_name: "test_collection",
    data: query_vectors,
    filter: 'JSON_CONTAINS_ALL(color["coord"], [4, 5])',
    output_fields: ["color", "id"],
    limit: 3
})

console.log(JSON.stringify(res.results, null, 4))

// 输出
// 
// [
//     {
//         "score": 1.8944344520568848,
//         "id": "792",
//         "color": {
//             "label": "purple",
//             "tag": 8161,
//             "coord": [
//                 4,
//                 38,
//                 5
//             ],
//             "ref": [
//                 [
//                     "red",
//                     "white",
//                     "grey"
//                 ]
//             ]
//         }
//     },
//     {
//         "score": 1.2801706790924072,
//         "id": "489",
//         "color": {
//             "label": "red",
//             "tag": 4358,
//             "coord": [
//                 5,
//                 4,
//                 1
//             ],
//             "ref": [
//                 [
//                     "blue",
//                     "orange",
//                     "orange"
//                 ]
//             ]
//         }
//     },
//     {
//         "score": 1.2097992897033691,
//         "id": "656",
//         "color": {
//             "label": "red",
//             "tag": 7856,
//             "coord": [
//                 5,
//                 20,
//                 4
//             ],
//             "ref": [
//                 [
//                     "black",
//                     "orange",
//                     "white"
//                 ]
//             ]
//         }
//     }
// ]
```
```python
# 过滤包含协调员为 `4` 或 `5` 的实体。

<div class="multipleCode">
<a href="#python">Python </a>
<a href="#java">Java</a>
<a href="#javascript">Node.js</a>
</div>

```python
res = client.query(
    collection_name="test_collection",
    data=query_vectors,
    filter='JSON_CONTAINS_ANY(color["coord"], [4, 5])',
    output_fields=["id", "color"],
    limit=3
)

print(res)

# 输出
#
# [
#     {
#         "id": 0,
#         "color": {
#             "label": "yellow",
#             "tag": 6340,
#             "coord": [
#                 40,
#                 4,
#                 40
#             ],
#             "ref": [
#                 [
#                     "purple",
#                     "yellow",
#                     "orange"
#                 ],
#                 [
#                     "green",
#                     "grey",
#                     "purple"
#                 ],
#                 [
#                     "black",
#                     "white",
#                     "yellow"
#                 ]
#             ]
#         }
#     },
#     {
#         "id": 2,
#         "color": {
#             "label": "brown",
#             "tag": 9359,
#             "coord": [
#                 38,
#                 21,
#                 5
#             ],
#             "ref": [
#                 [
#                     "red",
#                     "brown",
#                     "white"
#                 ],
#                 [
#                     "purple",
#                     "red",
#                     "brown"
#                 ],
#                 [
#                     "pink",
#                     "grey",
#                     "black"
#                 ]
#             ]
#         }
#     },
#     {
#         "id": 7,
#         "color": {
#             "label": "green",
#             "tag": 3560,
#             "coord": [
#                 5,
#                 9,
#                 5
#             ],
#             "ref": [
#                 [
#                     "blue",
#                     "orange",
#                     "green"
#                 ],
#                 [
#                     "blue",
#                     "blue",
#                     "black"
#                 ],
#                 [
#                     "green",
#                     "purple",
#                     "green"
#                 ]
#             ]
#         }
#     }
# ]
```

```java
searchReq = SearchReq.builder()
```
```java
        .collectionName("test_collection")
        .data(query_vectors)
        .filter("JSON_CONTAINS_ANY(color[\"coord\"], [4, 5])")
        .outputFields(Arrays.asList("id", "color"))
        .topK(3)
        .build();

    searchResp = client.search(searchReq);

    System.out.println(JSONObject.toJSON(searchResp));   

    // 输出:
    // {"searchResults": [[
    //     {
    //         "distance": 1.002122,
    //         "id": 629,
    //         "entity": {
    //             "color": {
    //                 "coord": [
    //                     23,
    //                     5,
    //                     35
    //                 ],
    //                 "ref": [
    //                     [
    //                         "black",
    //                         "yellow",
    //                         "black"
    //                     ],
    //                     [
    //                         "black",
    //                         "purple",
    //                         "white"
    //                     ],
    //                     [
    //                         "black",
    //                         "brown",
    //                         "orange"
    //                     ]
    //                 ],
    //                 "label": "red",
    //                 "tag": 5072
    //             },
    //             "id": 629
    //         }
    //     },
    //     {
    //         "distance": 0.85788506,
    //         "id": 108,
    //         "entity": {
    //             "color": {
    //                 "coord": [
    //                     25,
    //                     5,
    //                     38
    //                 ],
    //                 "ref": [
    //                     [
    //                         "green",
    //                         "brown",
    //                         "pink"
    //                     ],
    //                     [
    //                         "purple",
    //                         "green",
    //                         "green"
    //                     ],
    //                     [
    //                         "green",
    //                         "pink",
    //                         "black"
    //                     ]
    //                 ],
    //                 "label": "orange",
    //                 "tag": 8982
    //             },
    //             "id": 108
    //         }
    //     },
    //     {
    //         "distance": 0.80550396,
    //         "id": 120,
    //         "entity": {
    //             "color": {
    //                 "coord": [
    //                     25,
    //                     16,
    //                     4
    //                 ],
    //                 "ref": [
    //                     [
    //                         "red",
    //                         "green",
    //                         "orange"
    //                     ],
    //                     [
    // ```
## JSON过滤器参考

在处理JSON字段时，您可以将JSON字段本身或其中的特定键用作过滤器。

<div class="admonition note">

<p><b>注意</b></p>

<ul>
<li>Milvus将字符串值存储在JSON字段中，不会执行语义转义或转换。</li>
</ul>
<p>例如，<code>'a"b'</code>，<code>"a'b"</code>，<code>'a\\\\'b'</code>和<code>"a\\\\"b"</code>将被保存为原样，而<code>'a'b'</code>和<code>"a"b"</code>将被视为无效值。</p>
<ul>
<li><p>要使用 JSON 字段构建过滤表达式，可以利用字段内的键。</p></li>
<li><p>如果键的值是整数或浮点数，可以将其与另一个整数或浮点数键或 INT32/64 或 FLOAT32/64 字段进行比较。</p></li>
<li><p>如果键的值是字符串，只能将其与另一个字符串键或 VARCHAR 字段进行比较。</p></li>
</ul>

</div>

### JSON 字段中的基本运算符

以下表格假定名为 `json_key` 的 JSON 字段的值具有名为 `A` 的键。在构建使用 JSON 字段键的布尔表达式时，请将其用作参考。

|  __运算符__    |  __示例__                                       |  __备注__                                                                                                                                                                                                                                                                                                       |
| -------------- | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|  __<__         |  `'json_field["A"] < 3'`                        |  如果 `json_field["A"]` 的值小于 `3`，则此表达式求值为 true。                                                                                                                                                                                                                                                 |
|  __>__         |  `'json_field["A"] > 1'`                        |  如果 `json_field["A"]` 的值大于 `1`，则此表达式求值为 true。                                                                                                                                                                                                                                                 |
|  __==__        |  `'json_field["A"] == 1'`                       |  如果 `json_field["A"]` 的值等于 `1`，则此表达式求值为 true。                                                                                                                                                                                                                                                 |
|  __!=__        |  `'json_field["A"][0]' != "abc"'`               |  当表达式为真时：<br/> - `json_field` 没有名为 `A` 的键。<br/> - `json_field` 有名为 `A` 的键，但 `json_field["A"]` 不是数组。<br/> - `json_field["A"]` 是一个空数组。<br/> - `json_field["A"]` 是一个数组，但第一个元素不是 `abc`。<br/> |
|  __<=__        |  `'json_field["A"] <= 5'`                       |  当 `json_field["A"]` 的值小于或等于 `5` 时，表达式为真。                                                                                                                                                                                                               |
|  __>=__        |  `'json_field["A"] >= 1'`                       |  当 `json_field["A"]` 的值大于或等于 `1` 时，表达式为真。                                                                                                                                                                                                            |
|  __not__       |  `'not json_field["A"] == 1'`                   |  当表达式为真时：<br/> - `json_field` 没有名为 `A` 的键。<br/> - `json_field["A"]` 不等于 `1`。<br/>                                                                                                                                                             |
|  __in__        |  `'json_field["A"] in [1, 2, 3]'`               |  当 `json_field["A"]` 的值为 `1`、`2` 或 `3` 时，表达式为真。                                                                                                                                                                                                                        |
|  __and (&&)__  |  `'json_field["A"] > 1 && json_field["A"] < 3'` |  当 `json_field["A"]` 的值大于 `1` 且小于 `3` 时，表达式为真。                                                                                                                                                                                                        |
|  __or (\|\|)__ |  `'json_field["A"] > 1 \|\| json_field["A"] < 3'` |  当 `json_field["A"]` 的值大于 `1` 或小于 `3` 时，表达式为真。                                                                                                                                                                                                       |
|  __exists__    |  `'exists json_field["A"]'`                     |  当 `json_field` 有名为 `A` 的键时，表达式为真。                                                                                                                                                                                                                                |

### 高级运算符
以下运算符专门用于 JSON 字段：

- `json_contains(identifier, jsonExpr)`

    此运算符用于过滤包含指定 JSON 表达式的实体。

    - 示例 1: `{"x": [1,2,3]}`

        ```python
        json_contains(x, 1) # => True (x 包含 1。)
        json_contains(x, "a") # => False (x 不包含成员 "a"。)
```

    - 示例 2: `{"x", [[1,2,3], [4,5,6], [7,8,9]]}`
    
        ```python
        json_contains(x, [1,2,3]) # => True (x 包含 [1,2,3]。)
        json_contains(x, [3,2,1]) # => False (x 不包含成员 [3,2,1]。)
        ```

- `json_contains_all(identifier, jsonExpr)`

    此运算符用于过滤包含 JSON 表达式所有成员的实体。

    示例: `{"x": [1,2,3,4,5,7,8]}`

    ```python
    json_contains_all(x, [1,2,8]) # => True (x 包含 1, 2, 和 8。)
    json_contains_all(x, [4,5,6]) # => False (x 不包含成员 6。)
    ```

- `json_contains_any(identifier, jsonExpr)`

    此运算符用于过滤包含 JSON 表达式任意成员的实体。

    示例: `{"x": [1,2,3,4,5,7,8]}`

    ```python
    json_contains_any(x, [1,2,8]) # => True (x 包含 1, 2, 和 8。)
    json_contains_any(x, [4,5,6]) # => True (x 包含 4 和 5。)
    json_contains_any(x, [6,9]) # => False (x 不包含 6 和 9 中的任何一个。)
    ```