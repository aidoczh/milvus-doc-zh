---
id: get-and-scalar-query.md
order: 3
summary: 本指南演示了如何通过ID获取实体并进行标量过滤.
title: 获取与标量查询
---

# 获取与标量查询

本指南演示了如何通过ID获取实体并进行标量过滤。标量过滤是根据指定的过滤条件检索与之匹配的实体。

## 概述

标量查询使用布尔表达式基于定义的条件过滤集合中的实体。查询结果是一组与定义条件匹配的实体。与矢量搜索不同，矢量搜索识别与集合中给定矢量最接近的矢量，而查询根据特定标准过滤实体。

在 Milvus 中，__过滤器始终是由运算符连接的字段名组成的字符串__。在本指南中，您将找到各种过滤器示例。要了解更多关于运算符细节，请查看[参考](https://milvus.io/docs/get-and-scalar-query.md#Reference-on-scalar-filters)部分。

## 准备工作

以下步骤重新配置代码以连接到 Milvus，快速设置一个集合，并将超过 1,000 个随机生成的实体插入到集合中。

### 步骤 1：创建集合

<div class="language-python">

使用 [`MilvusClient`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md) 连接到 Milvus 服务器，并使用 [`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md) 创建一个集合。

</div>

<div class="language-java">

使用 [`MilvusClientV2`](https://milvus.io/api-reference/java/v2.4.x/v2/Client/MilvusClientV2.md) 连接到 Milvus 服务器，并使用 [`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md) 创建一个集合。

</div>

<div class="language-javascript">

使用 [`MilvusClient`](https://milvus.io/api-reference/node/v2.4.x/Client/MilvusClient.md) 连接到 Milvus 服务器，并使用 [`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md) 创建一个集合。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
from pymilvus import MilvusClient

# 1. 设置 Milvus 客户端
client = MilvusClient(
    uri="http://localhost:19530"
)

# 2. 创建一个集合
client.create_collection(
    collection_name="quick_setup",
    dimension=5,
)
```

```java
String CLUSTER_ENDPOINT = "http://localhost:19530";

// 1. 连接到 Milvus 服务器
ConnectConfig connectConfig = ConnectConfig.builder()
    .uri(CLUSTER_ENDPOINT)
    .build();

MilvusClientV2 client = new MilvusClientV2(connectConfig);  

// 2. 在快速设置模式下创建一个集合
CreateCollectionReq quickSetupReq = CreateCollectionReq.builder()
    .collectionName("quick_setup")
    .dimension(5)
    .metricType("IP")
    .build();

client.createCollection(quickSetupReq);
```
```javascript
const { MilvusClient, DataType, sleep } = require("@zilliz/milvus2-sdk-node")

const address = "http://localhost:19530"

// 1. 设置一个 Milvus 客户端
client = new MilvusClient({address}); 

// 2. 在快速设置模式下创建一个集合
await client.createCollection({
    collection_name: "quick_setup",
    dimension: 5,
}); 
```
### 第二步：插入随机生成的实体

<div class="language-python">

使用 [`insert()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/insert.md) 将实体插入到集合中。

</div>

<div class="language-java">

使用 [`insert()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/insert.md) 将实体插入到集合中。

</div>

<div class="language-javascript">

使用 [`insert()`](https://milvus.io/api-reference/node/v2.4.x/Vector/insert.md) 将实体插入到集合中。

</div>

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

# 输出
#
# {
#     "id": 0,
#     "vector": [
#         0.7371107800002366,
#         -0.7290389773227746,
#         0.38367002049157417,
#         0.36996000494220627,
#         -0.3641898951462792
#     ],
#     "color": "yellow",
#     "tag": 6781,
#     "color_tag": "yellow_6781"
# }

res = client.insert(
    collection_name="quick_setup",
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
// 3. 将随机生成的向量插入到集合中
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
    row.put("color_tag", current_color + '_' + String.valueOf(rand.nextInt(8999) + 1000));
    data.add(row);
}

InsertReq insertReq = InsertReq.builder()
    .collectionName("quick_setup")
    .data(data)
    .build();

InsertResp insertResp = client.insert(insertReq);

System.out.println(JSONObject.toJSON(insertResp));

// 输出:
// {"insertCnt": 1000}
```
```javascript
// 3. 插入随机生成的向量
const colors = ["green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey"]
var data = []

for (let i = 0; i < 1000; i++) {
    current_color = colors[Math.floor(Math.random() * colors.length)]
    current_tag = Math.floor(Math.random() * 8999 + 1000)
    data.push({
        "id": i,
        "vector": [Math.random(), Math.random(), Math.random(), Math.random(), Math.random()],
        "color": current_color,
        "tag": current_tag,
        "color_tag": `${current_color}_${current_tag}`
    })
}

console.log(data[0])

// 输出
// 
// {
//   id: 0,
//   vector: [
//     0.16022394821966035,
//     0.6514875214491056,
//     0.18294484964044666,
//     0.30227694168725394,
//     0.47553087493572255
//   ],
//   color: 'blue',
//   tag: 8907,
//   color_tag: 'blue_8907'
// }
// 

res = await client.insert({
    collection_name: "quick_setup",
    data: data
})

console.log(res.insert_cnt)

// 输出
// 
// 1000
// 
```
### 第三步：创建分区并插入更多实体

<div class="language-python">

使用 [`create_partition()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Partitions/create_partition.md) 来创建分区，使用 [`insert()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/insert.md) 来向集合中插入更多实体。

</div>

<div class="language-java">

使用 [`createPartition()`](https://milvus.io/api-reference/java/v2.4.x/v2/Partitions/createPartition.md) 来创建分区，使用 [`insert()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/insert.md) 来向集合中插入更多实体。

</div>

<div class="language-javascript">

使用 [`createPartition()`](https://milvus.io/api-reference/node/v2.4.x/Partitions/createPartition.md) 来创建分区，使用 [`insert()`](https://milvus.io/api-reference/node/v2.4.x/Vector/insert.md) 来向集合中插入更多实体。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 4. 创建分区并插入更多实体
client.create_partition(
    collection_name="quick_setup",
    partition_name="partitionA"
)

client.create_partition(
    collection_name="quick_setup",
    partition_name="partitionB"
)

data = []

for i in range(1000, 1500):
    current_color = random.choice(colors)
    data.append({
        "id": i,
        "vector": [ random.uniform(-1, 1) for _ in range(5) ],
        "color": current_color,
        "tag": current_tag,
        "color_tag": f"{current_color}_{str(current_tag)}"
    })

res = client.insert(
    collection_name="quick_setup",
    data=data,
    partition_name="partitionA"
)

print(res)

# 输出
#
# {
#     "insert_count": 500,
#     "ids": [
#         1000,
#         1001,
#         1002,
#         1003,
#         1004,
#         1005,
#         1006,
#         1007,
#         1008,
#         1009,
#         "(隐藏了 490 个更多项)"
#     ]
# }

data = []

for i in range(1500, 2000):
    current_color = random.choice(colors)
    data.append({
        "id": i,
        "vector": [ random.uniform(-1, 1) for _ in range(5) ],
        "color": current_color,
        "tag": current_tag,
        "color_tag": f"{current_color}_{str(current_tag)}"
    })

res = client.insert(
    collection_name="quick_setup",
    data=data,
    partition_name="partitionB"
)

print(res)

# 输出
#
# {
#     "insert_count": 500,
#     "ids": [
#         1500,
#         1501,
#         1502,
#         1503,
#         1504,
#         1505,
#         1506,
#         1507,
#         1508,
#         1509,
#         "(隐藏了 490 个更多项)"
#     ]
# }
```
```java
// 4. 创建分区并插入更多数据
CreatePartitionReq createPartitionReq = CreatePartitionReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionA")
    .build();

client.createPartition(createPartitionReq);

createPartitionReq = CreatePartitionReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionB")
    .build();

client.createPartition(createPartitionReq);

data.clear();

for (int i=1000; i<1500; i++) {
    Random rand = new Random();
    String current_color = colors.get(rand.nextInt(colors.size()-1));
    int current_tag = rand.nextInt(8999) + 1000;
    JSONObject row = new JSONObject();
    row.put("id", Long.valueOf(i));
    row.put("vector", Arrays.asList(rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat()));
    row.put("color", current_color);
    row.put("tag", current_tag);
    data.add(row);
}

insertReq = InsertReq.builder()
    .collectionName("quick_setup")
    .data(data)
    .partitionName("partitionA")
    .build();

insertResp = client.insert(insertReq);

System.out.println(JSONObject.toJSON(insertResp));

// 输出:
// {"insertCnt": 500}

data.clear();

for (int i=1500; i<2000; i++) {
    Random rand = new Random();
    String current_color = colors.get(rand.nextInt(colors.size()-1));
    int current_tag = rand.nextInt(8999) + 1000;
    JSONObject row = new JSONObject();
    row.put("id", Long.valueOf(i));
    row.put("vector", Arrays.asList(rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat()));
    row.put("color", current_color);
    row.put("tag", current_tag);
    data.add(row);
}

insertReq = InsertReq.builder()
    .collectionName("quick_setup")
    .data(data)
    .partitionName("partitionB")
    .build();

insertResp = client.insert(insertReq);

System.out.println(JSONObject.toJSON(insertResp));

// 输出:
// {"insertCnt": 500}
```
```javascript
// 4. 创建分区并插入更多实体
await client.createPartition({
    collection_name: "quick_setup",
    partition_name: "partitionA"
})

await client.createPartition({
    collection_name: "quick_setup",
    partition_name: "partitionB"
})

data = []

for (let i = 1000; i < 1500; i++) {
    current_color = colors[Math.floor(Math.random() * colors.length)]
    current_tag = Math.floor(Math.random() * 8999 + 1000)
    data.push({
        "id": i,
        "vector": [Math.random(), Math.random(), Math.random(), Math.random(), Math.random()],
        "color": current_color,
        "tag": current_tag,
        "color_tag": `${current_color}_${current_tag}`
    })
}

res = await client.insert({
    collection_name: "quick_setup",
    data: data,
    partition_name: "partitionA"
})

console.log(res.insert_cnt)

// 输出
// 
// 500
// 

await sleep(5000)

data = []

for (let i = 1500; i < 2000; i++) {
    current_color = colors[Math.floor(Math.random() * colors.length)]
    current_tag = Math.floor(Math.random() * 8999 + 1000)
    data.push({
        "id": i,
        "vector": [Math.random(), Math.random(), Math.random(), Math.random(), Math.random()],
        "color": current_color,
        "tag": current_tag,
        "color_tag": `${current_color}_${current_tag}`
    })
}

res = await client.insert({
    collection_name: "quick_setup",
    data: data,
    partition_name: "partitionB"
})

console.log(res.insert_cnt)

// 输出
// 
// 500
// 
```

## 根据 ID 获取实体

<div class="language-python">

如果您知道感兴趣的实体的 ID，您可以使用 [`get()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/get.md) 方法。

</div>

<div class="language-java">

如果您知道感兴趣的实体的 ID，您可以使用 [`get()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/get.md) 方法。

</div>

<div class="language-javascript">

如果您知道感兴趣的实体的 ID，您可以使用 [`get()`](https://milvus.io/api-reference/node/v2.4.x/Vector/get.md) 方法。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```
```python
# 4. 通过ID获取实体
res = client.get(
    collection_name="quick_setup",
    ids=[0, 1, 2]
)

print(res)

# 输出
#
# [
#     {
#         "id": 0,
#         "vector": [
#             0.68824464,
#             0.6552274,
#             0.33593303,
#             -0.7099536,
#             -0.07070546
#         ],
#         "color_tag": "green_2006",
#         "color": "green"
#     },
#     {
#         "id": 1,
#         "vector": [
#             -0.98531723,
#             0.33456197,
#             0.2844234,
#             0.42886782,
#             0.32753858
#         ],
#         "color_tag": "white_9298",
#         "color": "white"
#     },
#     {
#         "id": 2,
#         "vector": [
#             -0.9886812,
#             -0.44129863,
#             -0.29859528,
#             0.06059075,
#             -0.43817034
#         ],
#         "color_tag": "grey_5312",
#         "color": "grey"
#     }
# ]
```
```java
// 5. 通过 ID 获取实体
GetReq getReq = GetReq.builder()
    .collectionName("quick_setup")
    .ids(Arrays.asList(0L, 1L, 2L))
    .build();

GetResp entities = client.get(getReq);

System.out.println(JSONObject.toJSON(entities));

// 输出:
// {"getResults": [
//     {"entity": {
//         "color": "white",
//         "color_tag": "white_4597",
//         "vector": [
//             0.09665024,
//             0.1163497,
//             0.0701347,
//             0.32577968,
//             0.40943468
//         ],
//         "tag": 8946,
//         "id": 0
//     }},
//     {"entity": {
//         "color": "green",
//         "color_tag": "green_3039",
//         "vector": [
//             0.90689456,
//             0.4377399,
//             0.75387514,
//             0.36454988,
//             0.8702918
//         ],
//         "tag": 2341,
//         "id": 1
//     }},
//     {"entity": {
//         "color": "white",
//         "color_tag": "white_8708",
//         "vector": [
//             0.9757728,
//             0.13974023,
//             0.8023141,
//             0.61947155,
//             0.8290197
//         ],
//         "tag": 9913,
//         "id": 2
//     }}
// ]}
```

```javascript
// 5. 通过 id 获取实体
res = await client.get({
    collection_name: "quick_setup",
    ids: [0, 1, 2],
    output_fields: ["vector", "color_tag"]
})

console.log(res.data)

// 输出
// 
// [
//   {
//     vector: [
//       0.16022394597530365,
//       0.6514875292778015,
//       0.18294484913349152,
//       0.30227693915367126,
//       0.47553086280822754
//     ],
//     '$meta': { color: 'blue', tag: 8907, color_tag: 'blue_8907' },
//     id: '0'
//   },
//   {
//     vector: [
//       0.2459285855293274,
//       0.4974019527435303,
//       0.2154673933982849,
//       0.03719571232795715,
//       0.8348019123077393
//     ],
//     '$meta': { color: 'grey', tag: 3710, color_tag: 'grey_3710' },
//     id: '1'
//   },
//   {
//     vector: [
//       0.9404329061508179,
//       0.49662265181541443,
//       0.8088793158531189,
//       0.9337621331214905,
//       0.8269071578979492
//     ],
//     '$meta': { color: 'blue', tag: 2993, color_tag: 'blue_2993' },
//     id: '2'
//   }
// ]
// 
```

### 从分区获取实体

您还可以从特定分区获取实体。

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```
```python
# 5. 从分区获取实体
res = client.get(
    collection_name="quick_setup",
    ids=[1000, 1001, 1002],
    partition_names=["partitionA"]
)

print(res)

# 输出
#
# [
#     {
#         "color": "green",
#         "tag": 1995,
#         "color_tag": "green_1995",
#         "id": 1000,
#         "vector": [
#             0.7807706,
#             0.8083741,
#             0.17276904,
#             -0.8580777,
#             0.024156934
#         ]
#     },
#     {
#         "color": "red",
#         "tag": 1995,
#         "color_tag": "red_1995",
#         "id": 1001,
#         "vector": [
#             0.065074645,
#             -0.44882354,
#             -0.29479212,
#             -0.19798489,
#             -0.77542555
#         ]
#     },
#     {
#         "color": "green",
#         "tag": 1995,
#         "color_tag": "green_1995",
#         "id": 1002,
#         "vector": [
#             0.027934508,
#             -0.44199976,
#             -0.40262738,
#             -0.041511405,
#             0.024782438
#         ]
#     }
# ]
```
```java
// 5. 通过分区中的ID获取实体
getReq = GetReq.builder()
    .collectionName("quick_setup")
    .ids(Arrays.asList(1001L, 1002L, 1003L))
    .partitionName("partitionA")
    .build();

entities = client.get(getReq);

System.out.println(JSONObject.toJSON(entities));

// 输出:
// {"getResults": [
//     {"entity": {
//         "color": "yellow",
//         "vector": [
//             0.4300114,
//             0.599917,
//             0.799163,
//             0.75395125,
//             0.89947814
//         ],
//         "id": 1001,
//         "tag": 5803
//     }},
//     {"entity": {
//         "color": "blue",
//         "vector": [
//             0.009218454,
//             0.64637834,
//             0.19815737,
//             0.30519038,
//             0.8218663
//         ],
//         "id": 1002,
//         "tag": 7212
//     }},
//     {"entity": {
//         "color": "black",
//         "vector": [
//             0.76521933,
//             0.7818409,
//             0.16976339,
//             0.8719652,
//             0.1434964
//         ],
//         "id": 1003,
//         "tag": 1710
//     }}
// ]}
```

```javascript
// 5.1 通过分区中的ID获取实体
res = await client.get({
    collection_name: "quick_setup",
    ids: [1000, 1001, 1002],
    partition_names: ["partitionA"],
    output_fields: ["vector", "color_tag"]
})

console.log(res.data)

// 输出
// 
// [
//   {
//     id: '1000',
//     vector: [
//       0.014254206791520119,
//       0.5817716121673584,
//       0.19793470203876495,
//       0.8064294457435608,
//       0.7745839357376099
//     ],
//     '$meta': { color: 'white', tag: 5996, color_tag: 'white_5996' }
//   },
//   {
//     id: '1001',
//     vector: [
//       0.6073881983757019,
//       0.05214758217334747,
//       0.730999231338501,
//       0.20900958776474,
//       0.03665429726243019
//     ],
//     '$meta': { color: 'grey', tag: 2834, color_tag: 'grey_2834' }
//   },
//   {
//     id: '1002',
//     vector: [
//       0.48877206444740295,
//       0.34028753638267517,
//       0.6527213454246521,
//       0.9763909578323364,
//       0.8031482100486755
//     ],
//     '$meta': { color: 'pink', tag: 9107, color_tag: 'pink_9107' }
//   }
// ]
// 
```


## 使用基本运算符

在本节中，您将找到如何在标量过滤中使用基本运算符的示例。您也可以将这些过滤器应用于[向量搜索](https://milvus.io/docs/single-vector-search.md#Filtered-search)和[数据删除](https://milvus.io/docs/insert-update-delete.md#Delete-entities)中。

<div class="language-python">

有关更多信息，请参阅 SDK 参考中的[`query()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/query.md)。

</div>

<div class="language-java">

有关更多信息，请参阅 SDK 参考中的[`query()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/query.md)。

</div>

<div class="language-javascript">
```
要了解更多信息，请参考 SDK 参考中的 [`query()`](https://milvus.io/api-reference/node/v2.4.x/Vector/query.md)。

</div>

- 过滤标签值在 1,000 到 1,500 之间的实体。

    <div class="multipleCode">
       <a href="#python">Python </a>
       <a href="#java">Java</a>
       <a href="#javascript">Node.js</a>
    </div>

    ```python
    # 6. 使用基本运算符

    res = client.query(
        collection_name="quick_setup",
        # highlight-start
        filter="1000 < tag < 1500",
        output_fields=["color_tag"],
        # highlight-end
        limit=3
    )

    print(res)

    # 输出
    #
    # [
    #     {
    #         "id": 1,
    #         "color_tag": "pink_1023"
    #     },
    #     {
    #         "id": 41,
    #         "color_tag": "red_1483"
    #     },
    #     {
    #         "id": 44,
    #         "color_tag": "grey_1146"
    #     }
    # ]
    ```

    ```java
    // 6. 使用基本运算符

    QueryReq queryReq = QueryReq.builder()
        .collectionName("quick_setup")
        .filter("1000 < tag < 1500")
        .outputFields(Arrays.asList("color_tag"))
        .limit(3)
        .build();

    QueryResp queryResp = client.query(queryReq);

    System.out.println(JSONObject.toJSON(queryResp));

    // 输出:
    // {"queryResults": [
    //     {"entity": {
    //         "color_tag": "white_7588",
    //         "id": 34
    //     }},
    //     {"entity": {
    //         "color_tag": "orange_4989",
    //         "id": 64
    //     }},
    //     {"entity": {
    //         "color_tag": "white_3415",
    //         "id": 73
    //     }}
    // ]}
    ```

    ```javascript
    // 6. 使用基本运算符
    res = await client.query({
        collection_name: "quick_setup",
        filter: "1000 < tag < 1500",
        output_fields: ["color_tag"],
        limit: 3
    })

    console.log(res.data)

    // 输出
    // 
    // [
    //   {
    //     '$meta': { color: 'pink', tag: 1050, color_tag: 'pink_1050' },
    //     id: '6'
    //   },
    //   {
    //     '$meta': { color: 'purple', tag: 1174, color_tag: 'purple_1174' },
    //     id: '24'
    //   },
    //   {
    //     '$meta': { color: 'orange', tag: 1023, color_tag: 'orange_1023' },
    //     id: '40'
    //   }
    // ]
    // 
    ```

- 过滤颜色值设为__brown__的实体。

    <div class="multipleCode">
       <a href="#python">Python </a>
       <a href="#java">Java</a>
       <a href="#javascript">Node.js</a>
    </div>

    ```python
    res = client.query(
        collection_name="quick_setup",
        # highlight-start
        filter='color == "brown"',
        output_fields=["color_tag"],
        # highlight-end
        limit=3
    )
    
    print(res)
    
    # 输出
    #
    # [
    #     {
    #         "color_tag": "brown_5343",
    #         "id": 15
    #     },
    #     {
    #         "color_tag": "brown_3167",
    #         "id": 27
    #     },
    #     {
```python
res = client.query(
    collection_name="quick_setup",
    # highlight-start
    filter='color not in ["green", "purple"]',
    output_fields=["color_tag"],
    # highlight-end
    limit=3
)

print(res)

# Output
#
# [
#     {
#         "color_tag": "yellow_6781",
#         "id": 0
#     },
#     {
#         "color_tag": "pink_1023",
#         "id": 1
#     },
#     {
#         "color_tag": "blue_3972",
#         "id": 2
#     }
# ]
```

```java
queryReq = QueryReq.builder()
    .collectionName("quick_setup")
    .filter("color not in [\"green\", \"purple\"]")
    .outputFields(Arrays.asList("color_tag"))
    .limit(3)
    .build();

queryResp = client.query(queryReq);

System.out.println(JSONObject.toJSON(queryResp));   

// Output:
// {"queryResults": [
//     {"entity": {
//         "color_tag": "white_4597",
//         "id": 0
//     }},
//     {"entity": {
//         "color_tag": "white_8708",
//         "id": 2
//     }},
//     {"entity": {
//         "color_tag": "brown_7792",
//         "id": 3
//     }}
// ]}
```

```javascript
res = await client.query({
    collection_name: "quick_setup",
    filter: 'color not in ["green", "purple"]',
    output_fields: ["color_tag"],
    limit: 3
})

console.log(res.data)

// Output
// 
// [
//   {
//     '$meta': { color: 'yellow', tag: 6781, color_tag: 'yellow_6781' },
//     id: '0'
//   },
//   {
//     '$meta': { color: 'pink', tag: 1023, color_tag: 'pink_1023' },
//     id: '1'
//   },
//   {
//     '$meta': { color: 'blue', tag: 3972, color_tag: 'blue_3972' },
//     id: '2'
//   }
// ]
// 
```
```    
    过滤器: '颜色不在["green", "purple"]',
    输出字段: ["color_tag"],
    限制: 3
})

console.log(res.data)

// 输出
// 
// [
//   {
//     '$meta': { color: 'blue', tag: 8907, color_tag: 'blue_8907' },
//     id: '0'
//   },
//   {
//     '$meta': { color: 'grey', tag: 3710, color_tag: 'grey_3710' },
//     id: '1'
//   },
//   {
//     '$meta': { color: 'blue', tag: 2993, color_tag: 'blue_2993' },
//     id: '2'
//   }
// ]
// 
```

- 过滤颜色标签以 __red__ 开头的文章。

    <div class="multipleCode">
       <a href="#python">Python </a>
       <a href="#java">Java</a>
       <a href="#javascript">Node.js</a>
    </div>

    ```python
    res = client.query(
        collection_name="quick_setup",
        # highlight-start
        filter='color_tag like "red%"',
        output_fields=["color_tag"],
        # highlight-end
        limit=3
    )

    print(res)

    # 输出
    #
    # [
    #     {
    #         "color_tag": "red_6443",
    #         "id": 17
    #     },
    #     {
    #         "color_tag": "red_1483",
    #         "id": 41
    #     },
    #     {
    #         "color_tag": "red_4348",
    #         "id": 47
    #     }
    # ]
    ```

    ```java
    queryReq = QueryReq.builder()
        .collectionName("quick_setup")
        .filter("color_tag like \"red%\"")
        .outputFields(Arrays.asList("color_tag"))
        .limit(3)
        .build();

    queryResp = client.query(queryReq);

    System.out.println(JSONObject.toJSON(queryResp));  

    // 输出:
    // {"queryResults": [
    //     {"entity": {
    //         "color_tag": "red_4929",
    //         "id": 9
    //     }},
    //     {"entity": {
    //         "color_tag": "red_8284",
    //         "id": 13
    //     }},
    //     {"entity": {
    //         "color_tag": "red_3021",
    //         "id": 44
    //     }}
    // ]}
    ```

    ```javascript
    res = await client.query({
        collection_name: "quick_setup",
        filter: 'color_tag like "red%"',
        output_fields: ["color_tag"],
        limit: 3
    })

    console.log(res.data)

    // 输出
    // 
    // [
    //   {
    //     '$meta': { color: 'red', tag: 8773, color_tag: 'red_8773' },
    //     id: '17'
    //   },
    //   {
    //     '$meta': { color: 'red', tag: 9197, color_tag: 'red_9197' },
    //     id: '34'
    //   },
    //   {
    //     '$meta': { color: 'red', tag: 7914, color_tag: 'red_7914' },
    //     id: '46'
    //   }
    // ]
    // 
    ```

- 过滤颜色设置为红色且标签值在 1,000 到 1,500 范围内的实体。

    <div class="multipleCode">
       <a href="#python">Python </a>
       <a href="#java">Java</a>
       <a href="#javascript">Node.js</a>
    </div>

    ```python
    res = client.query(
        collection_name="quick_setup",
        # highlight-start
## 使用高级运算符

在本节中，您将找到如何在标量过滤中使用高级运算符的示例。您也可以将这些过滤器应用于[向量搜索](https://milvus.io/docs/single-vector-search.md#Filtered-search)和[数据删除](https://milvus.io/docs/insert-update-delete.md#Delete-entities)。

### 计算实体数量

- 计算集合中实体的总数。

    <div class="multipleCode">
       <a href="#python">Python </a>
       <a href="#java">Java</a>
       <a href="#javascript">Node.js</a>
    </div>

    ```python
    # 7. 使用高级运算符
    
    # 计算集合中实体的总数
    res = client.query(
        collection_name="quick_setup",
        # highlight-start
        output_fields=["count(*)"]
        # highlight-end
    )
    
    print(res)
    
    # 输出
    #
    # [
    #     {
    #         "count(*)": 2000
    #     }
    # ]
    ```

    ```java
    // 7. 使用高级运算符
    // 计算集合中实体的总数
    queryReq = QueryReq.builder()
        .collectionName("quick_setup")
        .filter("")
        .outputFields(Arrays.asList("count(*)"))
```    

```javascript
// 7. 使用高级操作符
// 计算集合中实体的总数
res = await client.query({
    collection_name: "quick_setup",
    output_fields: ["count(*)"]
})

console.log(res.data)   

// 输出
// 
// [ { 'count(*)': '2000' } ]
// 
```

- 计算特定分区中实体的总数。

<div class="multipleCode">
   <a href="#python">Python </a>
   <a href="#java">Java</a>
   <a href="#javascript">Node.js</a>
</div>

```python
# 计算分区中实体的数量
res = client.query(
    collection_name="quick_setup",
    # highlight-start
    output_fields=["count(*)"],
    partition_names=["partitionA"]
    # highlight-end
)

print(res)

# 输出
#
# [
#     {
#         "count(*)": 500
#     }
# ]
```

```java
// 计算分区中实体的数量
queryReq = QueryReq.builder()
    .collectionName("quick_setup")
    .partitionNames(Arrays.asList("partitionA"))
    .filter("")
    .outputFields(Arrays.asList("count(*)"))
    .build();

queryResp = client.query(queryReq);

System.out.println(JSONObject.toJSON(queryResp));

// 输出:
// {"queryResults": [{"entity": {"count(*)": 500}}]}
```

```javascript
// 计算分区中实体的数量
res = await client.query({
    collection_name: "quick_setup",
    output_fields: ["count(*)"],
    partition_names: ["partitionA"]
})

console.log(res.data)     

// 输出
// 
// [ { 'count(*)': '500' } ]
// 
```

- 计算符合筛选条件的实体数量

<div class="multipleCode">
   <a href="#python">Python </a>
   <a href="#java">Java</a>
   <a href="#javascript">Node.js</a>
</div>

```python
# 计算符合特定筛选条件的实体数量
res = client.query(
    collection_name="quick_setup",
    # highlight-start
    filter='(color == "red") and (1000 < tag < 1500)',
    output_fields=["count(*)"],
    # highlight-end
)

print(res)

# 输出
#
# [
#     {
#         "count(*)": 3
#     }
# ]
```

```java
// 计算符合特定筛选条件的实体数量
queryReq = QueryReq.builder()
    .collectionName("quick_setup")
    .filter("(color == \"red\") and (1000 < tag < 1500)")
    .outputFields(Arrays.asList("count(*)"))
    .build();

queryResp = client.query(queryReq);

System.out.println(JSONObject.toJSON(queryResp));

// 输出:
// {"queryResults": [{"entity": {"count(*)": 7}}]}
```
```
```javascript
// 计算符合特定过滤器条件的实体数量
res = await client.query({
    collection_name: "quick_setup",
    filter: '(color == "red") and (1000 < tag < 1500)',
    output_fields: ["count(*)"]
})

console.log(res.data)   

// 输出
// 
// [ { 'count(*)': '10' } ]
// 
```

## 标量过滤器参考

### 基本运算符

__布尔表达式__ 总是 __由字段名称通过运算符连接而成的字符串__. 在本节中，您将了解更多关于基本运算符的信息。

|  __运算符__     |  __描述__                                                                                                                            |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
|  __add (&&)__   |  如果两个操作数都为真，则为真                                                                                                             |
|  __or (\|\|)__  |  如果任一操作数为真，则为真                                                                                                             |
|  __+, -, *, /__ |  加法、减法、乘法和除法                                                                                        |
|  __**__         |  指数                                                                                                                                   |
|  __%__          |  取模                                                                                                                                    |
|  __<, >__       |  小于、大于                                                                                                                    |
|  __==, !=__     |  等于、不等于                                                                                                                     |
|  __<=, >=__     |  小于等于、大于等于                                                                                            |
|  __not__        |  反转给定条件的结果。                                                                                                  |
|  __like__       |  使用通配符运算符将一个值与类似值进行比较。<br/> 例如，like "prefix%" 匹配以 "prefix" 开头的字符串。 |
|  __in__         |  测试表达式是否与值列表中的任何值匹配。                                                                              |

### 高级运算符

- `count(*)`

    计算集合中实体的确切数量。将其用作输出字段，以获取集合或分区中实体的确切数量。

    <div class="admonition note">

    <p><b>notes</b></p>
```
<p>这适用于已加载的集合。您应将其用作唯一的输出字段。</p>

</div>