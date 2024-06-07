---
id: array_data_type.md
title: 使用数组字段
---


# 使用数组字段

本指南解释了如何使用数组字段，包括插入数组值以及在数组字段中使用基本和高级运算符进行搜索和查询。

## 概述

Milvus支持数组作为字段数据类型之一。如果要将数组字段添加到模式中，必须为其元素设置数据类型以及该字段可以包含的最大元素数量。这表明Milvus集合中的数组应始终具有相同数据类型的元素。

例如，以下代码片段生成一个包含数组字段的随机数据集。

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
    current_coord = [ random.randint(0, 40) for _ in range(random.randint(3, 5)) ]
    data.append({
        "id": i,
        "vector": [ random.uniform(-1, 1) for _ in range(5) ],
        "color": current_color,
        "color_tag": current_tag,
        "color_coord": current_coord,
    })

print(data[0])
```

```java
List<String> colors = Arrays.asList("green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey");
List<JSONObject> data = new ArrayList<>();

for (int i=0; i<1000; i++) {
    Random rand = new Random();
    String current_color = colors.get(rand.nextInt(colors.size()-1));
    Long current_tag = rand.nextLong(8999L) + 1000L;

    // 生成一个随机大小的数组
    Long capacity = rand.nextLong(5L) + 1L;
    List<Long> current_coord = new ArrayList<>();
    current_coord.add(rand.nextLong(40L) + 1L);
    current_coord.add(rand.nextLong(40L) + 1L);
    for (int j=3; j<capacity; j++) {
        current_coord.add(rand.nextLong(40L) + 1L);
    }

    JSONObject row = new JSONObject();
    row.put("id", Long.valueOf(i));
    row.put("vector", Arrays.asList(rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat()));
    row.put("color", current_color);
    row.put("color_tag", current_tag);
    row.put("color_coord", current_coord);
    data.add(row);
}

System.out.println(JSONObject.toJSON(data.get(0)));   
```
```javascript
const colors = ["绿色", "蓝色", "黄色", "红色", "黑色", "白色", "紫色", "粉色", "橙色", "棕色", "灰色"]
var data = []

for (let i = 0; i < 1000; i++) {
    const current_color = colors[Math.floor(Math.random() * colors.length)]
    const current_tag = Math.floor(Math.random() * 8999 + 1000)
    const current_coord = Array(Math.floor(Math.random() * 5 + 1)).fill(0).map(() => Math.floor(Math.random() * 40))

    data.push({
        id: i,
        vector: Array(5).fill(0).map(() => Math.random()),
        color: current_color,
        color_tag: current_tag,
        color_coord: current_coord,
    })
}

console.log(data[0])
```
您可以通过查看生成数据的第一个条目来查看其结构。

```
{
    id: 0,
    vector: [
        0.0338537420906162,
        0.6844108238358322,
        0.28410588909961754,
        0.09752595400212116,
        0.22671013058761114
    ],
    color: 'orange',
    color_tag: 5677,
    color_coord: [ 3, 0, 18, 29 ]
}
```

<div class="alert note">

- 数组字段中的元素应为相同的数据类型。
- 数组字段中的元素数量应小于或等于数组字段的指定最大容量。

</div>

## 定义数组字段

要定义数组字段，只需按照定义其他数据类型字段的相同步骤进行操作。

<div class="language-python">

有关参数的更多信息，请参考[`MilvusClient`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md)、[`create_schema()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md)、[`add_field()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/CollectionSchema/add_field.md)、[`add_index()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/add_index.md)、[`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md)和[`get_load_state()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/get_load_state.md)在 SDK 参考中。

</div>

<div class="language-java">

有关参数的更多信息，请参考[`MilvusClientV2`](https://milvus.io/api-reference/java/v2.4.x/v2/Client/MilvusClientV2.md)、[`createSchema()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createSchema.md)、[`addField()`](https://milvus.io/api-reference/java/v2.4.x/v2/CollectionSchema/addField.md)、[`IndexParam`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/IndexParam.md)、[`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md)和[`getLoadState()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/getLoadState.md)在 SDK 参考中。

</div>

<div class="language-javascript">

有关参数的更多信息，请参考[`MilvusClient`](https://milvus.io/api-reference/node/v2.4.x/Client/MilvusClient.md)、[`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md)和[`getLoadState()`](https://milvus.io/api-reference/node/v2.4.x/Management/getLoadState.md)在 SDK 参考中。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```python
import random, time
from pymilvus import MilvusClient, DataType

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
schema.add_field(field_name="color", datatype=DataType.VARCHAR, max_length=512)
schema.add_field(field_name="color_tag", datatype=DataType.INT64)
# highlight-next-line
schema.add_field(field_name="color_coord", datatype=DataType.ARRAY, element_type=DataType.INT64, max_capacity=5)

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
    .build());

schema.addField(AddFieldReq.builder()
    .fieldName("color_tag")
    .dataType(DataType.Int64)
    .build());

// highlight-start
schema.addField(AddFieldReq.builder()
    .fieldName("color_coord")
    .dataType(DataType.Array)
    .elementType(DataType.Int64)
    .maxCapacity(5)
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

IndexParam indexParamForColorField = IndexParam.builder()
    .fieldName("color")
    .indexType(IndexParam.IndexType.TRIE)
    .build();

List<IndexParam> indexParams = new ArrayList<>();
indexParams.add(indexParamForIdField);
indexParams.add(indexParamForVectorField);
indexParams.add(indexParamForColorField);

// 2.4 使用模式和索引参数创建一个集合
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
    {
        name: "color",
        data_type: DataType.VarChar,
        max_length: 512
    },
    {
        name: "color_tag",
        data_type: DataType.Int64,
    },
// highlight-start
    {
        name: "color_coord",
        data_type: DataType.Array,
        element_type: DataType.Int64,
        max_capacity: 5
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
## 插入字段数值

在创建集合后，您可以插入诸如在[概览](#overview)中演示的数组。

<div class="language-python">

有关参数的更多信息，请参阅 SDK 参考中的[`insert()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/insert.md)。

</div>

<div class="language-java">

有关参数的更多信息，请参阅 SDK 参考中的[`insert()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/insert.md)。

</div>

<div class="language-javascript">

有关参数的更多信息，请参阅 SDK 参考中的[`insert()`](https://milvus.io/api-reference/node/v2.4.x/Vector/insert.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 3. 插入数据
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

time.sleep(5)
```

```java
import com.alibaba.fastjson.JSONObject;

import io.milvus.v2.service.vector.request.InsertReq;
import io.milvus.v2.service.vector.response.InsertResp;

// 3.1 将数据插入集合
InsertReq insertReq = InsertReq.builder()
    .collectionName("test_collection")
    .data(data)
    .build();

InsertResp insertResp = client.insert(insertReq);

System.out.println(JSONObject.toJSON(insertResp));

// 输出:
// {"insertCnt": 1000}

Thread.sleep(5000);   // 等待一段时间以确保数据已被索引
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

await sleep(5000)
```

## 基本标量过滤

一旦您的所有数据都已添加，您可以使用数组字段中的元素进行搜索和查询，方式与标准标量字段相同。

<div class="language-python">

有关参数的更多信息，请参阅 SDK 参考中的[`search()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/search.md)。

</div>

<div class="language-java">

有关参数的更多信息，请参阅 SDK 参考中的[`search()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/search.md)。

</div>

<div class="language-javascript">

有关参数的更多信息，请参阅 SDK 参考中的[`search()`](https://milvus.io/api-reference/node/v2.4.x/Vector/search.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```python
# 4. 使用数组字段进行基本搜索
query_vectors = [ [ random.uniform(-1, 1) for _ in range(5) ]]

res = client.search(
    collection_name="test_collection",
    data=query_vectors,
    filter="color_coord[0] < 10",
    search_params={
        "metric_type": "L2",
        "params": {"nprobe": 16}
    },
    output_fields=["id", "color", "color_tag", "color_coord"],
    limit=3
)

print(res)

# 输出
#
# [
#     [
#         {
#             "id": 993,
#             "distance": 0.1538649946451187,
#             "entity": {
#                 "color_coord": [
#                     5,
#                     37,
#                     39,
#                     18
#                 ],
#                 "id": 993,
#                 "color": "black",
#                 "color_tag": 6785
#             }
#         },
#         {
#             "id": 452,
#             "distance": 0.2353954315185547,
#             "entity": {
#                 "color_coord": [
#                     2,
#                     27,
#                     34,
#                     32,
#                     30
#                 ],
#                 "id": 452,
#                 "color": "brown",
#                 "color_tag": 2075
#             }
#         },
#         {
#             "id": 862,
#             "distance": 0.27913951873779297,
#             "entity": {
#                 "color_coord": [
#                     0,
#                     19,
#                     0,
#                     26
#                 ],
#                 "id": 862,
#                 "color": "brown",
#                 "color_tag": 1787
#             }
#         }
#     ]
# ]
```
```java
// 4. 使用数组字段进行基本搜索

QueryReq queryReq = QueryReq.builder()
    .collectionName("test_collection")
    .filter("color_coord[0] in [7, 8, 9]")
    .outputFields(Arrays.asList("id", "color", "color_tag", "color_coord"))
    .limit(3L)
    .build();

QueryResp queryResp = client.query(queryReq);

System.out.println(JSONObject.toJSON(queryResp));

// 输出:
// {"queryResults": [
//     {"entity": {
//         "color": "orange",
//         "color_tag": 2464,
//         "id": 18,
//         "color_coord": [
//             9,
//             30
//         ]
//     }},
//     {"entity": {
//         "color": "pink",
//         "color_tag": 2602,
//         "id": 22,
//         "color_coord": [
//             8,
//             34,
//             16
//         ]
//     }},
//     {"entity": {
//         "color": "pink",
//         "color_tag": 1243,
//         "id": 42,
//         "color_coord": [
//             9,
//             20
//         ]
//     }}
// ]}
```

```javascript
// 4. 使用数组字段进行基本搜索
const query_vectors = [Array(5).fill(0).map(() => Math.random())]

res = await client.search({
    collection_name: "test_collection",
    data: query_vectors,
    filter: "color_coord[0] < 10",
    output_fields: ["id", "color", "color_tag", "color_coord"],
    limit: 3
})

console.log(JSON.stringify(res.results, null, 4))

// 输出
// 
// [
//     {
//         "score": 2.015894889831543,
//         "id": "260",
//         "color": "green",
//         "color_tag": "5320",
//         "color_coord": [
//             "1",
//             "7",
//             "33",
//             "13",
//             "23"
//         ]
//     },
//     {
//         "score": 2.006500720977783,
//         "id": "720",
//         "color": "green",
//         "color_tag": "4939",
//         "color_coord": [
//             "0",
//             "19",
//             "5",
//             "30",
//             "15"
//         ]
//     },
//     {
//         "score": 1.9539016485214233,
//         "id": "243",
//         "color": "red",
//         "color_tag": "2403",
//         "color_coord": [
//             "4"
//         ]
//     }
// ]
// 
```

## 高级过滤

与 JSON 字段类似，Milvus 还为数组提供了高级过滤运算符，包括 `ARRAY_CONTAINS`、`ARRAY_CONTAINS_ALL`、`ARRAY_CONTAINS_ANY` 和 `ARRAY_LENGTH`。

- 过滤出所有颜色坐标中包含 `10` 的实体。

    <div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
    </div>
    
    ```python
    # 5. 在数组字段中进行高级搜索
    
    res = client.query(
        collection_name="test_collection",
        filter="ARRAY_CONTAINS(color_coord, 10)",
        output_fields=["id", "color", "color_tag", "color_coord"],
        limit=3
    )
    
    print(res)
    
    # 输出
    #
    # [
    #     {
    #         "id": 21,
    #         "color": "white",
```
```markdown
#         "color_tag": 4202,
#         "color_coord": [
#             10,
#             5,
#             5
#         ]
#     },
#     {
#         "id": 31,
#         "color": "grey",
#         "color_tag": 7386,
#         "color_coord": [
#             8,
#             10,
#             23,
#             7,
#             31
#         ]
#     },
#     {
#         "id": 45,
#         "color": "purple",
#         "color_tag": 6126,
#         "color_coord": [
#             0,
#             10,
#             24
#         ]
#     }
# ]
```

```java
// 5. 高级搜索数组字段内的内容
queryReq = QueryReq.builder()
    .collectionName("test_collection")
    .filter("ARRAY_CONTAINS(color_coord, 10)")
    .outputFields(Arrays.asList("id", "color", "color_tag", "color_coord"))
    .limit(3)
    .build();

queryResp = client.query(queryReq);

System.out.println(JSONObject.toJSON(queryResp));

// 输出:
// {"queryResults": [
//     {"entity": {
//         "color": "blue",
//         "color_tag": 4337,
//         "id": 17,
//         "color_coord": [
//             11,
//             33,
//             10,
//             20
//         ]
//     }},
//     {"entity": {
//         "color": "white",
//         "color_tag": 5219,
//         "id": 25,
//         "color_coord": [
//             10,
//             15
//         ]
//     }},
//     {"entity": {
//         "color": "red",
//         "color_tag": 7120,
//         "id": 35,
//         "color_coord": [
//             19,
//             10,
//             10,
//             14
//         ]
//     }}
// ]}
```

```javascript
// 5. 在数组字段中进行高级搜索
res = await client.search({
    collection_name: "test_collection",
    data: query_vectors,
    filter: "ARRAY_CONTAINS(color_coord, 10)",
    output_fields: ["id", "color", "color_tag", "color_coord"],
    limit: 3
})

console.log(JSON.stringify(res.results, null, 4))

// 输出
// 
// [
//     {
//         "score": 1.7962548732757568,
//         "id": "696",
//         "color": "red",
//         "color_tag": "1798",
//         "color_coord": [
//             "33",
//             "10",
//             "37"
//         ]
//     },
//     {
//         "score": 1.7126177549362183,
//         "id": "770",
//         "color": "red",
//         "color_tag": "1962",
//         "color_coord": [
//             "21",
//             "23",
//             "10"
//         ]
//     },
//     {
//         "score": 1.6707111597061157,
//         "id": "981",
//         "color": "yellow",
```
```    

- 过滤所有颜色坐标中同时包含 `7` 和 `8` 的实体。

    <div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
    </div>

    ```python
    res = client.query(
        collection_name="test_collection",
        filter="ARRAY_CONTAINS_ALL(color_coord, [7, 8])",
        output_fields=["id", "color", "color_tag", "color_coord"],
        limit=3
    )

    print(res)

    # 输出
    #
    # [
    #     {
    #         "color": "grey",
    #         "color_tag": 7386,
    #         "color_coord": [
    #             8,
    #             10,
    #             23,
    #             7,
    #             31
    #         ],
    #         "id": 31
    #     },
    #     {
    #         "color": "purple",
    #         "color_tag": 7823,
    #         "color_coord": [
    #             38,
    #             8,
    #             36,
    #             38,
    #             7
    #         ],
    #         "id": 258
    #     },
    #     {
    #         "color": "purple",
    #         "color_tag": 6356,
    #         "color_coord": [
    #             34,
    #             32,
    #             11,
    #             8,
    #             7
    #         ],
    #         "id": 348
    #     }
    # ]
```

    ```java
    queryReq = QueryReq.builder()
        .collectionName("test_collection")
        .filter("ARRAY_CONTAINS_ALL(color_coord, [7, 8, 9])")
        .outputFields(Arrays.asList("id", "color", "color_tag", "color_coord"))
        .limit(3)
        .build();
    
    queryResp = client.query(queryReq);
    
    System.out.println(JSONObject.toJSON(queryResp));     
    
    // 输出:
    // {"queryResults": [{"entity": {
    //     "color": "red",
    //     "color_tag": 6986,
    //     "id": 423,
    //     "color_coord": [
    //         26,
    //         7,
    //         8,
    //         9
    //     ]
    // }}]}
    ```    
    
    ```javascript
    res = await client.search({
        collection_name: "test_collection",
        data: query_vectors,
        filter: "ARRAY_CONTAINS_ALL(color_coord, [7, 8])",
        output_fields: ["id", "color", "color_tag", "color_coord"],
        limit: 3
    })
    
    console.log(JSON.stringify(res.results, null, 4))
    
    // 输出
    // 
    // [
    //     {
    //         "score": 0.8267516493797302,
    //         "id": "913",
    //         "color": "brown",
    //         "color_tag": "8897",
    //         "color_coord": [
    //             "39",
    //             "31",
    //             "8",
    //             "29",
    //             "7"
    //         ]
    //     },
    //     {
    //         "score": 0.6889009475708008,
    //         "id": "826",
    //         "color": "blue",```
```json
//         "color_tag": "4903",
//         "color_coord": [
//             "7",
//             "25",
//             "5",
//             "12",
//             "8"
//         ]
//     },
//     {
//         "score": 0.5851659774780273,
//         "id": "167",
//         "color": "blue",
//         "color_tag": "1550",
//         "color_coord": [
//             "8",
//             "27",
//             "7"
//         ]
//     }
// ]
// 
```

- 过滤所有颜色坐标中包含7、8或9的实体。

<div class="multipleCode">
<a href="#python">Python </a>
<a href="#java">Java</a>
<a href="#javascript">Node.js</a>
</div>

```python
res = client.query(
    collection_name="test_collection",
    filter="ARRAY_CONTAINS_ANY(color_coord, [7, 8, 9])",
    output_fields=["id", "color", "color_tag", "color_coord"],
    limit=3
)

print(res)

# 输出
#
# [
#     {
#         "id": 0,
#         "color": "green",
#         "color_tag": 9212,
#         "color_coord": [
#             37,
#             36,
#             36,
#             7,
#             9
#         ]
#     },
#     {
#         "id": 5,
#         "color": "blue",
#         "color_tag": 9643,
#         "color_coord": [
#             8,
#             20,
#             20,
#             11
#         ]
#     },
#     {
#         "id": 12,
#         "color": "blue",
#         "color_tag": 3075,
#         "color_coord": [
#             29,
#             7,
#             17
#         ]
#     }
# ]
```

```java
queryReq = QueryReq.builder()
    .collectionName("test_collection")
    .filter("ARRAY_CONTAINS_ANY(color_coord, [7, 8, 9])")
    .outputFields(Arrays.asList("id", "color", "color_tag", "color_coord"))
    .limit(3)
    .build();

queryResp = client.query(queryReq);

System.out.println(JSONObject.toJSON(queryResp));   

// 输出:
// {"queryResults": [
//     {"entity": {
//         "color": "orange",
//         "color_tag": 2464,
//         "id": 18,
//         "color_coord": [
//             9,
//             30
//         ]
//     }},
//     {"entity": {
//         "color": "pink",
//         "color_tag": 2602,
//         "id": 22,
//         "color_coord": [
//             8,
//             34,
//             16
//         ]
//     }},
//     {"entity": {
//         "color": "pink",
//         "color_tag": 1243,
//         "id": 42,
//         "color_coord": [
//             9,
//             20
//         ]
//     }}
// ]}
```

```javascript
res = await client.search({
    collection_name: "test_collection",
```
```   

- 过滤具有恰好四个元素的实体。

    <div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
    </div>

    ```python
    res = client.query(
        collection_name="test_collection",
        filter="ARRAY_LENGTH(color_coord) == 4",
        output_fields=["id", "color", "color_tag", "color_coord"],
        limit=3
    )

    print(res)

    # 输出
    #
    # [
    #     {
    #         "id": 1,
    #         "color": "pink",
    #         "color_tag": 6708,
    #         "color_coord": [
    #             15,
    #             36,
    #             38,
    #             2
    #         ]
    #     },
    #     {
    #         "id": 4,
    #         "color": "green",
    #         "color_tag": 5386,
    #         "color_coord": [
    #             13,
    #             32,
    #             35,
    #             5
    #         ]
    #     },
    #     {
    #         "id": 5,
    #         "color": "blue",
    #         "color_tag": 9643,
    #         "color_coord": [
    #             8,
    #             20,
    #             20,
    #             11
    #         ]
    #     }
    # ]
```

    ```java
    queryReq = QueryReq.builder()
        .collectionName("test_collection")
        .filter("ARRAY_LENGTH(color_coord) == 4")
        .outputFields(Arrays.asList("id", "color", "color_tag", "color_coord"))
        .limit(3)
        .build();
    
    queryResp = client.query(queryReq);
    
    System.out.println(JSONObject.toJSON(queryResp));   
    
    // 输出:
    // {"queryResults": [
    //     {"entity": {
    //         "color": "green",
    //         "color_tag": 2984,
    //         "id": 2,
    //         "color_coord": [
    //             27,
    //             31,
```   

```javascript
res = await client.search({
collection_name: "test_collection",
data: query_vectors,
filter: "ARRAY_LENGTH(color_coord) == 4",
output_fields: ["id", "color", "color_tag", "color_coord"],
limit: 3
})

console.log(JSON.stringify(res.results, null, 4))

// 输出
// 
// [
//     {
//         "score": 2.0404388904571533,
//         "id": "439",
//         "color": "橙色",
//         "color_tag": "7096",
//         "color_coord": [
//             "27",
//             "34",
//             "26",
//             "39"
//         ]
//     },
//     {
//         "score": 1.9059759378433228,
//         "id": "918",
//         "color": "紫色",
//         "color_tag": "2903",
//         "color_coord": [
//             "28",
//             "19",
//             "36",
//             "35"
//         ]
//     },
//     {
//         "score": 1.8385567665100098,
//         "id": "92",
//         "color": "黄色",
//         "color_tag": "4693",
//         "color_coord": [
//             "1",
//             "23",
//             "2",
//             "3"
//         ]
//     }
// ]
// 
```