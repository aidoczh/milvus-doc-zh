---
id: single-vector-search.md
order: 1
summary: 本文介绍如何使用单一查询向量在 Milvus 集合中搜索向量。
title: 单一向量搜索
---

# 单一向量搜索

在插入数据后，下一步是在 Milvus 集合中执行相似性搜索。

Milvus 允许您进行两种类型的搜索，取决于您的集合中向量字段的数量：

- **单一向量搜索**：如果您的集合只有一个向量字段，请使用 [`search()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/search.md) 方法查找最相似的实体。该方法将您的查询向量与集合中现有向量进行比较，并返回最接近的匹配项的 ID，以及它们之间的距离。可选地，它还可以返回结果的向量值和元数据。
- **多向量搜索**：对于具有两个或更多向量字段的集合，请使用 [`hybrid_search()`](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/Collection/hybrid_search.md) 方法。该方法执行多个近似最近邻（ANN）搜索请求，并组合结果以在重新排序后返回最相关的匹配项。

本指南重点介绍如何在 Milvus 中执行单一向量搜索。有关多向量搜索的详细信息，请参阅 [混合搜索](https://milvus.io/docs/multi-vector-search.md)。

## 概述

有各种搜索类型可满足不同需求：

- [基本搜索](https://milvus.io/docs/single-vector-search.md#Basic-search)：包括单一向量搜索、批量向量搜索、分区搜索以及带有指定输出字段的搜索。

- [过滤搜索](https://milvus.io/docs/single-vector-search.md#Filtered-search)：根据标量字段应用过滤条件以细化搜索结果。

- [范围搜索](https://milvus.io/docs/single-vector-search.md#Range-search)：查找与查询向量在特定距离范围内的向量。

- [分组搜索](https://milvus.io/docs/single-vector-search.md#Grouping-search)：根据特定字段对搜索结果进行分组，以确保结果的多样性。

## 准备工作

下面的代码片段重新利用现有代码，建立与 Milvus 的连接并快速设置一个集合。

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```python
# 1. 设置 Milvus 客户端
client = MilvusClient(
    uri=CLUSTER_ENDPOINT,
    token=TOKEN 
)

# 2. 创建集合
client.create_collection(
    collection_name="quick_setup",
    dimension=5,
    metric_type="IP"
)

# 3. 插入随机生成的向量
colors = ["green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey"]
data = []

for i in range(1000):
    current_color = random.choice(colors)
    data.append({
        "id": i,
        "vector": [ random.uniform(-1, 1) for _ in range(5) ],
        "color": current_color,
        "color_tag": f"{current_color}_{str(random.randint(1000, 9999))}"
    })

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

# 6.1 创建分区
client.create_partition(
    collection_name="quick_setup",
    partition_name="red"
)

client.create_partition(
    collection_name="quick_setup",
    partition_name="blue"
)

# 6.1 将数据插入分区
red_data = [ {"id": i, "vector": [ random.uniform(-1, 1) for _ in range(5) ], "color": "red", "color_tag": f"red_{str(random.randint(1000, 9999))}" } for i in range(500) ]
blue_data = [ {"id": i, "vector": [ random.uniform(-1, 1) for _ in range(5) ], "color": "blue", "color_tag": f"blue_{str(random.randint(1000, 9999))}" } for i in range(500) ]

res = client.insert(
    collection_name="quick_setup",
    data=red_data,
    partition_name="red"
)

print(res)

# 输出
#
# {
#     "insert_count": 500,
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
#         "(490 more items hidden)"
#     ]
# }

res = client.insert(
    collection_name="quick_setup",
    data=blue_data,
    partition_name="blue"
)

print(res)

# 输出
#
# {
#     "insert_count": 500,
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
#         "(490 more items hidden)"
#     ]
# }
```
### 英文段落如下：

---

**Title: A Brief Introduction to Image Compression**

Image compression is a type of data compression applied to digital images, to reduce their cost for storage or transmission. Algorithms may take advantage of visual perception and the statistical properties of image data to provide superior results compared with generic data compression methods. Image compression may be lossy or lossless. Lossless compression is preferred for archival purposes and often for medical imaging, technical drawings, clip art, or comics. Lossy compression methods, especially when used at low bit rates, introduce compression artifacts. 

![Image Compression](https://www.example.com/image_compression.jpg)

One common type of image compression is JPEG (Joint Photographic Experts Group), which is a lossy compression method. Another example is PNG (Portable Network Graphics), which uses a lossless compression algorithm. Other image compression formats include GIF (Graphics Interchange Format), TIFF (Tagged Image File Format), and BMP (Bitmap Image File).

Research in image compression continues to evolve, with new algorithms and techniques being developed to improve compression efficiency while maintaining image quality. Deep learning approaches have also been applied to image compression, with promising results in achieving high compression ratios with minimal loss of image quality [20].

---

### 中文翻译如下：

---

**标题：图像压缩简介**

图像压缩是一种应用于数字图像的数据压缩类型，旨在降低图像的存储或传输成本。算法可以利用视觉感知和图像数据的统计特性，相较于通用数据压缩方法，提供更优越的结果。图像压缩可以是有损的或无损的。无损压缩通常用于档案目的，也常用于医学成像、技术绘图、剪贴画或漫画。有损压缩方法，特别是在低比特率下使用时，会引入压缩伪影。

![图像压缩](https://www.example.com/image_compression.jpg)

一种常见的图像压缩类型是 JPEG（联合图像专家组），这是一种有损压缩方法。另一个例子是 PNG（便携式网络图形），它使用无损压缩算法。其他图像压缩格式包括 GIF（图形交换格式）、TIFF（标记图像文件格式）和 BMP（位图图像文件）。

图像压缩领域的研究不断发展，不断有新的算法和技术被开发出来，以提高压缩效率同时保持图像质量。深度学习方法也被应用于图像压缩，取得了令人期待的成果，实现了高压缩比且最小程度损失图像质量 [20]。
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.Random;

import com.alibaba.fastjson.JSONObject;

import io.milvus.v2.client.ConnectConfig;
import io.milvus.v2.client.MilvusClientV2;
import io.milvus.v2.service.collection.request.CreateCollectionReq;
import io.milvus.v2.service.collection.request.GetLoadStateReq;
import io.milvus.v2.service.vector.request.InsertReq;
import io.milvus.v2.service.vector.response.InsertResp; 

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

GetLoadStateReq loadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .build();

boolean state = client.getLoadState(loadStateReq);

System.out.println(state);

// 输出:
// true

// 3. 将随机生成的向量插入集合
List<String> colors = Arrays.asList("green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey");
List<JSONObject> data = new ArrayList<>();

for (int i=0; i<1000; i++) {
    Random rand = new Random();
    String current_color = colors.get(rand.nextInt(colors.size()-1));
    JSONObject row = new JSONObject();
    row.put("id", Long.valueOf(i));
    row.put("vector", Arrays.asList(rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat()));
    row.put("color_tag", current_color + "_" + String.valueOf(rand.nextInt(8999) + 1000));
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

// 6.1. 创建一个分区
CreatePartitionReq partitionReq = CreatePartitionReq.builder()
    .collectionName("quick_setup")
    .partitionName("red")
    .build();

client.createPartition(partitionReq);

partitionReq = CreatePartitionReq.builder()
    .collectionName("quick_setup")
    .partitionName("blue")
    .build();

client.createPartition(partitionReq);

// 6.2 将数据插入分区
data = new ArrayList<>();

for (int i=1000; i<1500; i++) {
    Random rand = new Random();
    String current_color = "red";
    JSONObject row = new JSONObject();
    row.put("id", Long.valueOf(i));
    row.put("vector", Arrays.asList(rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat()));
    row.put("color", current_color);
    row.put("color_tag", current_color + "_" + String.valueOf(rand.nextInt(8999) + 1000));
    data.add(row);
}     

insertReq = InsertReq.builder()
    .collectionName("quick_setup")
    .data(data)
    .partitionName("red")
    .build();

insertResp = client.insert(insertReq);

System.out.println(JSONObject.toJSON(insertResp));

// 输出:
// {"insertCnt": 500}

data = new ArrayList<>();

for (int i=1500; i<2000; i++) {
    Random rand = new Random();
    String current_color = "blue";
    JSONObject row = new JSONObject();
    row.put("id", Long.valueOf(i));
    row.put("vector", Arrays.asList(rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat()));
    row.put("color", current_color);
    row.put("color_tag", current_color + "_" + String.valueOf(rand.nextInt(8999) + 1000));
    data.add(row);
}

insertReq = InsertReq.builder()
    .collectionName("quick_setup")
    .data(data)
    .partitionName("blue")
    .build();

insertResp = client.insert(insertReq);

System.out.println(JSONObject.toJSON(insertResp));

// 输出:
// {"insertCnt": 500}
```
```javascript
const { MilvusClient, DataType, sleep } = require("@zilliz/milvus2-sdk-node")

const address = "http://localhost:19530"

// 1. 设置 Milvus 客户端
client = new MilvusClient({address});

// 2. 在快速设置模式下创建一个集合
await client.createCollection({
    collection_name: "quick_setup",
    dimension: 5,
    metric_type: "IP"
});  

// 3. 插入随机生成的向量
const colors = ["green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey"]
data = []

for (let i = 0; i < 1000; i++) {
    current_color = colors[Math.floor(Math.random() * colors.length)]
    data.push({
        id: i,
        vector: [Math.random(), Math.random(), Math.random(), Math.random(), Math.random()],
        color: current_color,
        color_tag: `${current_color}_${Math.floor(Math.random() * 8999) + 1000}`
    })
}

var res = await client.insert({
    collection_name: "quick_setup",
    data: data
})

console.log(res.insert_cnt)

// 输出
// 
// 1000
// 

await client.createPartition({
    collection_name: "quick_setup",
    partition_name: "red"
})

await client.createPartition({
    collection_name: "quick_setup",
    partition_name: "blue"
})

// 6.1 将数据插入分区
var red_data = []
var blue_data = []

for (let i = 1000; i < 1500; i++) {
    red_data.push({
        id: i,
        vector: [Math.random(), Math.random(), Math.random(), Math.random(), Math.random()],
        color: "red",
        color_tag: `red_${Math.floor(Math.random() * 8999) + 1000}`
    })
}

for (let i = 1500; i < 2000; i++) {
    blue_data.push({
        id: i,
        vector: [Math.random(), Math.random(), Math.random(), Math.random(), Math.random()],
        color: "blue",
        color_tag: `blue_${Math.floor(Math.random() * 8999) + 1000}`
    })
}

res = await client.insert({
    collection_name: "quick_setup",
    data: red_data,
    partition_name: "red"
})

console.log(res.insert_cnt)

// 输出
// 
// 500
// 

res = await client.insert({
    collection_name: "quick_setup",
    data: blue_data,
    partition_name: "blue"
})

console.log(res.insert_cnt)

// 输出
// 
// 500
// 

```

## 基本搜索

在发送 `search` 请求时，您可以提供一个或多个向量值，表示您的查询嵌入，以及一个 `limit` 值，指示要返回的结果数量。

根据您的数据和查询向量，可能会获得少于 `limit` 个结果。当 `limit` 大于查询的可能匹配向量数量时，就会发生这种情况。

### 单向量搜索

单向量搜索是 Milvus 中最简单的 `search` 操作形式，旨在找到与给定查询向量最相似的向量。

要执行单向量搜索，请指定目标集合名称、查询向量和所需的结果数量（`limit`）。此操作返回一个结果集，包括最相似的向量、它们的 ID 和与查询向量的距离。
以下是搜索与查询向量最相似的前5个实体的示例：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 单向量搜索
res = client.search(
    collection_name="test_collection", # 替换为您的集合实际名称
    # 替换为您的查询向量
    data=[[0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]],
    limit=5, # 返回的最大搜索结果数
    search_params={"metric_type": "IP", "params": {}} # 搜索参数
)

# 将输出转换为格式化的 JSON 字符串
result = json.dumps(res, indent=4)
print(result)
```

```java
// 4. 单向量搜索
List<List<Float>> query_vectors = Arrays.asList(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f));

SearchReq searchReq = SearchReq.builder()
    .collectionName("quick_setup")
    .data(query_vectors)
    .topK(3) // 要返回的结果数
    .build();

SearchResp searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));
```

```javascript
// 4. 单向量搜索
var query_vector = [0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592],

res = await client.search({
    collection_name: "quick_setup",
    data: [query_vector],
    limit: 3, // 要返回的结果数
})

console.log(res.results)
```

输出类似于以下内容：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
[
    [
        {
            "id": 0,
            "distance": 1.4093276262283325,
            "entity": {}
        },
        {
            "id": 4,
            "distance": 0.9902134537696838,
            "entity": {}
        },
        {
            "id": 1,
            "distance": 0.8519943356513977,
            "entity": {}
        },
        {
            "id": 5,
            "distance": 0.7972343564033508,
            "entity": {}
        },
        {
            "id": 2,
            "distance": 0.5928734540939331,
            "entity": {}
        }
    ]
]
```
```java
{"searchResults": [[
    {
        "score": 1.263043,
        "fields": {
            "vector": [
                0.9533119,
                0.02538395,
                0.76714665,
                0.35481733,
                0.9845762
            ],
            "id": 740
        }
    },
    {
        "score": 1.2377806,
        "fields": {
            "vector": [
                0.7411156,
                0.08687937,
                0.8254139,
                0.08370924,
                0.99095553
            ],
            "id": 640
        }
    },
    {
        "score": 1.1869997,
        "fields": {
            "vector": [
                0.87928146,
                0.05324632,
                0.6312755,
                0.28005534,
                0.9542448
            ],
            "id": 455
        }
    }
]]}
```
```javascript
[
  { score: 1.7463608980178833, id: '854' },
  { score: 1.744946002960205, id: '425' },
  { score: 1.7258622646331787, id: '718' }
]
```

输出展示了与您的查询向量最接近的前5个邻居，包括它们的唯一ID和计算出的距离。

### 批量向量搜索

批量向量搜索扩展了[单向量搜索](https://milvus.io/docs/single-vector-search.md#Single-Vector-Search)的概念，允许在单个请求中搜索多个查询向量。这种搜索类型非常适用于需要为一组查询向量找到相似向量的场景，显著减少了所需的时间和计算资源。

在批量向量搜索中，您可以在`data`字段中包含多个查询向量。系统会并行处理这些向量，为每个查询向量返回一个单独的结果集，每个集合包含在集合中找到的最接近的匹配项。

以下是搜索两组不同的最相似实体的示例查询向量：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 批量向量搜索
res = client.search(
    collection_name="test_collection", # 替换为您的集合实际名称
    data=[
        [0.19886812562848388, 0.06023560599112088, 0.6976963061752597, 0.2614474506242501, 0.838729485096104],
        [0.3172005263489739, 0.9719044792798428, -0.36981146090600725, -0.4860894583077995, 0.95791889146345]
    ], # 替换为您的查询向量
    limit=2, # 返回的最大搜索结果数
    search_params={"metric_type": "IP", "params": {}} # 搜索参数
)

result = json.dumps(res, indent=4)
print(result)
```

```java
// 5. 批量向量搜索
query_vectors = Arrays.asList(
    Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f),
    Arrays.asList(0.19886812562848388f, 0.06023560599112088f, 0.6976963061752597f, 0.2614474506242501f, 0.838729485096104f)
);

searchReq = SearchReq.builder()
    .collectionName("quick_setup")
    .data(query_vectors)
    .topK(2)
    .build();

searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));
```

```javascript
// 5. 批量向量搜索
var query_vectors = [
    [0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592],
    [0.19886812562848388, 0.06023560599112088, 0.6976963061752597, 0.2614474506242501, 0.838729485096104]
]

res = await client.search({
    collection_name: "quick_setup",
    data: query_vectors,
    limit: 2,
})

console.log(res.results)
```

输出类似于以下内容：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```
```python
[
    [
        {
            "id": 1,
            "distance": 1.3017789125442505,
            "entity": {}
        },
        {
            "id": 7,
            "distance": 1.2419954538345337,
            "entity": {}
        }
    ], # 结果集 1
    [
        {
            "id": 3,
            "distance": 2.3358664512634277,
            "entity": {}
        },
        {
            "id": 8,
            "distance": 0.5642921924591064,
            "entity": {}
        }
    ] # 结果集 2
]
```
```java
// 两组向量的返回结果如预期所示

{"searchResults": [
    [
        {
            "score": 1.263043,
            "fields": {
                "vector": [
                    0.9533119,
                    0.02538395,
                    0.76714665,
                    0.35481733,
                    0.9845762
                ],
                "id": 740
            }
        },
        {
            "score": 1.2377806,
            "fields": {
                "vector": [
                    0.7411156,
                    0.08687937,
                    0.8254139,
                    0.08370924,
                    0.99095553
                ],
                "id": 640
            }
        }
    ],
    [
        {
            "score": 1.8654699,
            "fields": {
                "vector": [
                    0.4671427,
                    0.8378432,
                    0.98844475,
                    0.82763994,
                    0.9729997
                ],
                "id": 638
            }
        },
        {
            "score": 1.8581753,
            "fields": {
                "vector": [
                    0.735541,
                    0.60140246,
                    0.86730254,
                    0.93152493,
                    0.98603314
                ],
                "id": 855
            }
        }
    ]
]}
```

```javascript
[
  [
    { score: 2.3590476512908936, id: '854' },
    { score: 2.2896690368652344, id: '59' }
  [
    { score: 2.664059638977051, id: '59' },
    { score: 2.59483003616333, id: '854' }
  ]
]
```

搜索结果包括两组最近邻居，每组对应一个查询向量，展示了批量向量搜索的效率，能够一次处理多个查询向量。

### 分区搜索

分区搜索将您的搜索范围缩小到集合的特定子集或分区。这对于数据被分成逻辑或分类部分的有序数据集特别有用，通过减少需要扫描的数据量，可以加快搜索操作的速度。

要进行分区搜索，只需在搜索请求的 `partition_names` 中包含目标分区的名称。这指定 `search` 操作仅考虑指定分区内的向量。

以下是在 `red` 中搜索实体的示例：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 6.2 在分区内搜索
query_vector = [0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]

res = client.search(
    collection_name="quick_setup",
    data=[query_vector],
    limit=5,
    search_params={"metric_type": "IP", "params": {"level": 1}},
    partition_names=["red"]
)

print(res)
```
```
```java
// 6.3 在分区内搜索
query_vectors = Arrays.asList(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f));

searchReq = SearchReq.builder()
    .collectionName("quick_setup")
    .data(query_vectors)
    .partitionNames(Arrays.asList("red"))
    .topK(5)
    .build();

searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));
```
```javascript
// 6.2 在分区内搜索
query_vector = [0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]

res = await client.search({
    collection_name: "quick_setup",
    data: [query_vector],
    partition_names: ["red"],
    limit: 5,
})

console.log(res.results)
```

输出结果类似于以下内容：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
[
    [
        {
            "id": 16,
            "distance": 0.9200337529182434,
            "entity": {}
        },
        {
            "id": 14,
            "distance": 0.4505271911621094,
            "entity": {}
        },
        {
            "id": 15,
            "distance": 0.19924677908420563,
            "entity": {}
        },
        {
            "id": 17,
            "distance": 0.0075093843042850494,
            "entity": {}
        },
        {
            "id": 13,
            "distance": -0.14609718322753906,
            "entity": {}
        }
    ]
]
```
```java
{"searchResults": [
    [
        {
            "score": 1.1677284,
            "fields": {
                "vector": [
                    0.9986977,
                    0.17964739,
                    0.49086612,
                    0.23155272,
                    0.98438674
                ],
                "id": 1435
            }
        },
        {
            "score": 1.1476475,
            "fields": {
                "vector": [
                    0.6952647,
                    0.13417172,
                    0.91045254,
                    0.119336545,
                    0.9338931
                ],
                "id": 1291
            }
        },
        {
            "score": 1.0969629,
            "fields": {
                "vector": [
                    0.3363194,
                    0.028906643,
                    0.6675426,
                    0.030419827,
                    0.9735209
                ],
                "id": 1168
            }
        },
        {
            "score": 1.0741848,
            "fields": {
                "vector": [
                    0.9980543,
                    0.36063594,
                    0.66427994,
                    0.17359233,
                    0.94954175
                ],
                "id": 1164
            }
        },
        {
            "score": 1.0584627,
            "fields": {
                "vector": [
                    0.7187005,
                    0.12674773,
                    0.987718,
                    0.3110777,
                    0.86093885
                ],
                "id": 1085
            }
        }
    ],
    [
        {
            "score": 1.8030131,
            "fields": {
                "vector": [
                    0.59726167,
                    0.7054632,
                    0.9573117,
                    0.94529945,
                    0.8664103
                ],
                "id": 1203
            }
        },
        {
            "score": 1.7728865,
            "fields": {
                "vector": [
                    0.6672442,
                    0.60448086,
                    0.9325822,
                    0.80272985,
                    0.8861626
                ],
                "id": 1448
            }
        },
        {
            "score": 1.7536311,
            "fields": {
                "vector": [
                    0.59663296,
                    0.77831805,
                    0.8578314,
                    0.88818026,
                    0.9030075
                ],
                "id": 1010
            }
        },
        {
            "score": 1.7520742,
            "fields": {
                "vector": [
                    0.854198,
                    0.72294194,
                    0.9245805,
                    0.86126596,
                    0.7969224
                ],
                "id": 1219
            }
        },
        {
            "score": 1.7452049,
            "fields": {
                "vector": [
                    0.96419,
                    0.943535,
                    0.87611496,
                    0.8268136,
                    0.79786557
                ],
                "id": 1149
            }
        }
    ]
]}
```
```javascript
[
  { score: 3.0258803367614746, id: '1201' },
  { score: 3.004319190979004, id: '1458' },
  { score: 2.880324363708496, id: '1187' },
  { score: 2.8246407508850098, id: '1347' },
  { score: 2.797295093536377, id: '1406' }
]
```

然后，在`blue`中搜索实体：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
res = client.search(
    collection_name="quick_setup",
    data=[query_vector],
    limit=5,
    search_params={"metric_type": "IP", "params": {"level": 1}},
    partition_names=["blue"]
)

print(res)
```

```java
searchReq = SearchReq.builder()
    .collectionName("quick_setup")
    .data(query_vectors)
    .partitionNames(Arrays.asList("blue"))
    .topK(5)
    .build();

searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));
```

```javascript
res = await client.search({
    collection_name: "quick_setup",
    data: [query_vector],
    partition_names: ["blue"],
    limit: 5,
})

console.log(res.results)
```

输出结果类似于以下内容：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
[
    [
        {
            "id": 20,
            "distance": 2.363696813583374,
            "entity": {}
        },
        {
            "id": 26,
            "distance": 1.0665391683578491,
            "entity": {}
        },
        {
            "id": 23,
            "distance": 1.066049575805664,
            "entity": {}
        },
        {
            "id": 29,
            "distance": 0.8353596925735474,
            "entity": {}
        },
        {
            "id": 28,
            "distance": 0.7484277486801147,
            "entity": {}
        }
    ]
]
```
```java
{"searchResults": [
    [
        {
            "score": 1.1628494,
            "fields": {
                "vector": [
                    0.7442872,
                    0.046407282,
                    0.71031404,
                    0.3544345,
                    0.9819991
                ],
                "id": 1992
            }
        },
        {
            "score": 1.1470042,
            "fields": {
                "vector": [
                    0.5505825,
                    0.04367262,
                    0.9985836,
                    0.18922359,
                    0.93255126
                ],
                "id": 1977
            }
        },
        {
            "score": 1.1450152,
            "fields": {
                "vector": [
                    0.89994013,
                    0.052991092,
                    0.8645576,
                    0.6406729,
                    0.95679337
                ],
                "id": 1573
            }
        },
        {
            "score": 1.1439825,
            "fields": {
                "vector": [
                    0.9253267,
                    0.15890503,
                    0.7999555,
                    0.19126713,
                    0.898583
                ],
                "id": 1552
            }
        },
        {
            "score": 1.1029172,
            "fields": {
                "vector": [
                    0.95661926,
                    0.18777144,
                    0.38115507,
                    0.14323527,
                    0.93137646
                ],
                "id": 1823
            }
        }
    ],
    [
        {
            "score": 1.8005109,
            "fields": {
                "vector": [
                    0.5953582,
                    0.7794224,
                    0.9388869,
                    0.79825854,
                    0.9197286
                ],
                "id": 1888
            }
        },
        {
            "score": 1.7714822,
            "fields": {
                "vector": [
                    0.56805456,
                    0.89422905,
                    0.88187534,
                    0.914824,
                    0.8944365
                ],
                "id": 1648
            }
        },
        {
            "score": 1.7561421,
            "fields": {
                "vector": [
                    0.83421993,
                    0.39865613,
                    0.92319834,
                    0.42695504,
                    0.96633124
                ],
                "id": 1688
            }
        },
        {
            "score": 1.7553532,
            "fields": {
                "vector": [
                    0.89994013,
                    0.052991092,
                    0.8645576,
                    0.6406729,
                    0.95679337
                ],
                "id": 1573
            }
        },
        {
            "score": 1.7543385,
            "fields": {
                "vector": [
                    0.16542226,
                    0.38248396,
                    0.9888778,
                    0.80913955,
                    0.9501492
                ],
                "id": 1544
            }
        }
    ]
]}
```
```javascript
[
  { score: 2.8421106338500977, id: '1745' },
  { score: 2.838560104370117, id: '1782' },
  { score: 2.8134000301361084, id: '1511' },
  { score: 2.718268871307373, id: '1679' },
  { score: 2.7014894485473633, id: '1597' }
]
```

`红色`中的数据与`蓝色`中的数据不同。因此，搜索结果将受限于指定的分区，反映该子集的独特特征和数据分布。

### 带有输出字段的搜索

带有输出字段的搜索允许您指定匹配向量中应包含在搜索结果中的哪些属性或字段。

您可以在请求中指定`output_fields`以返回具有特定字段的结果。

以下是返回具有`color`属性值的结果的示例：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 使用输出字段进行搜索
res = client.search(
    collection_name="test_collection", # 替换为实际集合的名称
    data=[[0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]],
    limit=5, # 返回的最大搜索结果数
    search_params={"metric_type": "IP", "params": {}}, # 搜索参数
    output_fields=["color"] # 要返回的输出字段
)

result = json.dumps(res, indent=4)
print(result)
```

```java
// 7. 使用输出字段进行搜索
query_vectors = Arrays.asList(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f));

searchReq = SearchReq.builder()
    .collectionName("quick_setup")
    .data(query_vectors)
    .outputFields(Arrays.asList("color"))
    .topK(5)
    .build();

searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));
```

```javascript
// 7. 使用输出字段进行搜索
query_vector = [0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]

res = await client.search({
    collection_name: "quick_setup",
    data: [query_vector],
    limit: 5,
    output_fields: ["color"],
})

console.log(res.results)
```

输出类似于以下内容：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```
```python
[
    [
        {
            "id": 0,
            "distance": 1.4093276262283325,
            "entity": {
                "color": "粉红色_8682"
            }
        },
        {
            "id": 16,
            "distance": 1.0159327983856201,
            "entity": {
                "color": "黄色_1496"
            }
        },
        {
            "id": 4,
            "distance": 0.9902134537696838,
            "entity": {
                "color": "红色_4794"
            }
        },
        {
            "id": 14,
            "distance": 0.9803846478462219,
            "entity": {
                "color": "绿色_2899"
            }
        },
        {
            "id": 1,
            "distance": 0.8519943356513977,
            "entity": {
                "color": "红色_7025"
            }
        }
    ]
]
```
```java
{"searchResults": [
    [
        {
            "score": 1.263043,
            "fields": {}
        },
        {
            "score": 1.2377806,
            "fields": {}
        },
        {
            "score": 1.1869997,
            "fields": {}
        },
        {
            "score": 1.1748955,
            "fields": {}
        },
        {
            "score": 1.1720343,
            "fields": {}
        }
    ]
]}
```

```javascript

[
  { score: 3.036271572113037, id: '59', color: 'orange' },
  { score: 3.0267879962921143, id: '1745', color: 'blue' },
  { score: 3.0069446563720703, id: '854', color: 'black' },
  { score: 2.984386682510376, id: '718', color: 'black' },
  { score: 2.916019916534424, id: '425', color: 'purple' }
]
```

在最近邻的结果旁边，搜索结果将包括指定的字段 `color`，为每个匹配向量提供更丰富的信息集。

## 过滤搜索

过滤搜索对向量搜索应用标量过滤器，允许您根据特定条件细化搜索结果。您可以在[布尔表达式规则](https://milvus.io/docs/boolean.md)中了解有关过滤表达式的更多信息，并在[获取和标量查询](https://milvus.io/docs/get-and-scalar-query.md)中查看示例。

### 使用 `like` 运算符

`like` 运算符通过评估包括前缀、中缀和后缀在内的模式来增强字符串搜索：

- __前缀匹配__：要查找以特定前缀开头的值，请使用语法 `'like "prefix%"'`。
- __中缀匹配__：要查找包含字符串任何位置的特定字符序列的值，请使用语法 `'like "%infix%"'`。
- __后缀匹配__：要查找以特定后缀结尾的值，请使用语法 `'like "%suffix"'`。

对于单字符匹配，下划线 (`_`) 作为一个字符的通配符，例如，`'like "y_llow"'`。

### 搜索字符串中的特殊字符

如果要搜索包含特殊字符（如下划线 `_` 或百分号 `%`）的字符串，这些字符通常用作搜索模式中的通配符（`_` 代表任意单个字符，`%` 代表任意字符序列），则必须转义这些字符以将其视为字面字符。使用反斜杠 (`\`) 来转义特殊字符，并记得转义反斜杠本身。例如：

- 要搜索字面下划线，请使用 `\\_`。
- 要搜索字面百分号，请使用 `\\%`。

因此，如果您需要搜索文本 `"_version_"`，您的查询应格式为 `'like "\\_version\\_"'`，以确保下划线被视为搜索项的一部分，而不是通配符。

过滤结果中 __color__ 以 __red__ 为前缀的：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```
```python
# 使用过滤器进行搜索
res = client.search(
    collection_name="test_collection",  # 用您的集合实际名称替换
    data=[[0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]],
    limit=5,  # 返回的最大搜索结果数
    search_params={"metric_type": "IP", "params": {}},  # 搜索参数
    output_fields=["color"],  # 要返回的输出字段
    filter='color like "red%"'
)

result = json.dumps(res, indent=4)
print(result)
```
```java
// 8. 过滤搜索
query_vectors = Arrays.asList(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f));

searchReq = SearchReq.builder()
    .collectionName("quick_setup")
    .data(query_vectors)
    .outputFields(Arrays.asList("color_tag"))
    .filter("color_tag like \"red%\"")
    .topK(5)
    .build();

searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));
```

```javascript
// 8. 过滤搜索
// 8.1 使用 "like" 运算符和前缀通配符进行过滤
query_vector = [0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]

res = await client.search({
    collection_name: "quick_setup",
    data: [query_vector],
    limit: 5,
    filters: "color_tag like \"red%\"",
    output_fields: ["color_tag"]
})

console.log(res.results)
```

输出类似于以下内容：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
[
    [
        {
            "id": 4,
            "distance": 0.9902134537696838,
            "entity": {
                "color": "red_4794"
            }
        },
        {
            "id": 1,
            "distance": 0.8519943356513977,
            "entity": {
                "color": "red_7025"
            }
        },
        {
            "id": 6,
            "distance": -0.4113418459892273,
            "entity": {
                "color": "red_9392"
            }
        }
    ]
]
```

```java
{"searchResults": [
    [
        {
            "score": 1.1869997,
            "fields": {"color_tag": "red_3026"}
        },
        {
            "score": 1.1677284,
            "fields": {"color_tag": "red_9030"}
        },
        {
            "score": 1.1476475,
            "fields": {"color_tag": "red_3744"}
        },
        {
            "score": 1.0969629,
            "fields": {"color_tag": "red_4168"}
        },
        {
            "score": 1.0741848,
            "fields": {"color_tag": "red_9678"}
        }
    ]
]}
```

```javascript
[
  { score: 2.5080761909484863, id: '1201', color_tag: 'red_8904' },
  { score: 2.491129159927368, id: '425', color_tag: 'purple_8212' },
  { score: 2.4889798164367676, id: '1458', color_tag: 'red_6891' },
  { score: 2.42964243888855, id: '724', color_tag: 'black_9885' },
  { score: 2.4004223346710205, id: '854', color_tag: 'black_5990' }
]
```

过滤包含字符串中任何位置有 __ll__ 字母的 __color__ 结果：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```
```python
# 针对颜色字段的中缀匹配
res = client.search(
    collection_name="test_collection",  # 替换为您的集合实际名称
    data=[[0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]],
    limit=5,  # 返回的最大搜索结果数
    search_params={"metric_type": "IP", "params": {}},  # 搜索参数
    output_fields=["color"],  # 要返回的输出字段
    filter='color like "%ll%"'  # 在颜色字段上进行过滤，中缀匹配"ll"
)

result = json.dumps(res, indent=4)
print(result)
```
```java
// 8. 过滤搜索
query_vectors = Arrays.asList(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f));

searchReq = SearchReq.builder()
    .collectionName("quick_setup")
    .data(query_vectors)
    .outputFields(Arrays.asList("color_tag"))
    .filter("color like \"%ll%\"")
    .topK(5)
    .build();

searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));
```

```javascript
// 8. 过滤搜索
// 8.1 使用 "like" 运算符和前缀通配符进行过滤
query_vector = [0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]

res = await client.search({
    collection_name: "quick_setup",
    data: [query_vector],
    limit: 5,
    filters: "color_tag like \"%ll%\"",
    output_fields: ["color_tag"]
})

console.log(res.results)
```

输出类似于以下内容：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
[
    [
        {
            "id": 5,
            "distance": 0.7972343564033508,
            "entity": {
                "color": "yellow_4222"
            }
        }
    ]
]
```

```java
{"searchResults": [
    [
        {
            "score": 1.1869997,
            "fields": {"color_tag": "yellow_4222"}
        }
    ]
]}
```

```javascript
[
  { score: 2.5080761909484863, id: '1201', color_tag: 'yellow_4222' }
]
```

## 范围搜索

范围搜索允许您查找与查询向量在指定距离范围内的向量。

通过设置 `radius` 和可选的 `range_filter`，您可以调整搜索的广度，以包括与查询向量略有相似的向量，从而提供潜在匹配的更全面视图。

- `radius`: 定义搜索空间的外部边界。只有距离查询向量在此距离内的向量才被视为潜在匹配项。

- `range_filter`: 虽然 `radius` 设置了搜索的外部限制，但可以选择使用 `range_filter` 来定义内部边界，创建一个距离范围，在此范围内的向量才被视为匹配项。

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>
```
```python
# 进行范围搜索
search_params = {
    "metric_type": "IP",
    "params": {
        "radius": 0.8, # 搜索圆的半径
        "range_filter": 1.0 # 范围过滤器，用于过滤出不在搜索圆内的向量
    }
}

res = client.search(
    collection_name="test_collection", # 用实际集合名称替换
    data=[[0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]],
    limit=3, # 返回的最大搜索结果数
    search_params=search_params, # 搜索参数
    output_fields=["color"], # 要返回的输出字段
)

result = json.dumps(res, indent=4)
print(result)
```
```java
// 9. 范围搜索
query_vectors = Arrays.asList(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f));

searchReq = SearchReq.builder()
    .collectionName("quick_setup")
    .data(query_vectors)
    .outputFields(Arrays.asList("color_tag"))
    .searchParams(Map.of("radius", 0.1, "range", 1.0))
    .topK(5)
    .build();

searchResp = client.search(searchReq);

System.out.println(JSONObject.toJSON(searchResp));
```

```javascript
// 9. 范围搜索
query_vector = [0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]

res = await client.search({
    collection_name: "quick_setup",
    data: [query_vector],
    limit: 5,
    params: {
        radius: 0.1,
        range: 1.0
    },
    output_fields: ["color_tag"]
})

console.log(res.results)
```

输出类似于以下内容：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
[
    [
        {
            "id": 4,
            "distance": 0.9902134537696838,
            "entity": {
                "color": "red_4794"
            }
        },
        {
            "id": 14,
            "distance": 0.9803846478462219,
            "entity": {
                "color": "green_2899"
            }
        },
        {
            "id": 1,
            "distance": 0.8519943356513977,
            "entity": {
                "color": "red_7025"
            }
        }
    ]
]
```

```java
{"searchResults": [
    [
        {
            "score": 1.263043,
            "fields": {"color_tag": "green_2052"}
        },
        {
            "score": 1.2377806,
            "fields": {"color_tag": "purple_3709"}
        },
        {
            "score": 1.1869997,
            "fields": {"color_tag": "red_3026"}
        },
        {
            "score": 1.1748955,
            "fields": {"color_tag": "black_1646"}
        },
        {
            "score": 1.1720343,
            "fields": {"color_tag": "green_4853"}
        }
    ]
]}
```

```javascript
[
  { score: 2.3387961387634277, id: '718', color_tag: 'black_7154' },
  { score: 2.3352415561676025, id: '1745', color_tag: 'blue_8741' },
  { score: 2.290485382080078, id: '1408', color_tag: 'red_2324' },
  { score: 2.285870313644409, id: '854', color_tag: 'black_5990' },
  { score: 2.2593345642089844, id: '1309', color_tag: 'red_8458' }
]
```

您会发现，返回的所有实体与查询向量的距离均在0.8到1.0的范围内。

`radius` 和 `range_filter` 的参数设置随所使用的度量类型而变化。

|  __度量类型__ |  __特征__                             |  __范围搜索设置__                                                                               |
| ---------------- | ------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
|  `L2`            |  较小的 L2 距离表示更高的相似度。 |  要排除结果中最接近的向量，请确保：<br/> `range_filter` <= 距离 < `radius` |
|  `IP`            |  较大的 IP 距离表示更高的相似度。  |  要排除结果中最接近的向量，请确保：<br/> `radius` < 距离 <= `range_filter` |

## 分组搜索

在 Milvus 中，通过特定字段进行分组搜索可以避免结果中相同字段项的冗余。您可以针对特定字段获得各异的结果集。

考虑一个文档集合，每个文档分为各种段落。每个段落由一个向量嵌入表示，并属于一个文档。为了找到相关文档而不是相似段落，您可以在 `search()` 操作中包含 `group_by_field` 参数，按文档 ID 对结果进行分组。这有助于返回最相关且唯一的文档，而不是来自同一文档的单独段落。

以下是按字段分组搜索结果的示例代码：

```python
# 连接到 Milvus
client = MilvusClient(uri='http://localhost:19530') # Milvus 服务器地址

# 将数据加载到集合中
client.load_collection("group_search") # 集合名称

# 分组搜索结果
res = client.search(
    collection_name="group_search", # 集合名称
    data=[[0.14529211512077012, 0.9147257273453546, 0.7965055218724449, 0.7009258593102812, 0.5605206522382088]], # 查询向量
    search_params={
    "metric_type": "L2",
    "params": {"nprobe": 10},
    }, # 搜索参数
    limit=10, # 返回的最大搜索结果数
    group_by_field="doc_id", # 按文档 ID 对结果进行分组
    output_fields=["doc_id", "passage_id"]
)

# 提取 `doc_id` 列中的值
doc_ids = [result['entity']['doc_id'] for result in res[0]]

print(doc_ids)
```

输出类似于以下内容：

```python
[5, 10, 1, 7, 9, 6, 3, 4, 8, 2]
```

在给定的输出中，可以观察到返回的实体不包含任何重复的 `doc_id` 值。

为了比较，让我们将 `group_by_field` 注释掉并进行常规搜索：
```python
# 连接到 Milvus
client = MilvusClient(uri='http://localhost:19530') # Milvus 服务器地址

# 将数据加载到集合中
client.load_collection("group_search") # 集合名称

# 在没有 `group_by_field` 的情况下进行搜索
res = client.search(
    collection_name="group_search", # 集合名称
    data=query_passage_vector, # 用您的查询向量替换
    search_params={
    "metric_type": "L2",
    "params": {"nprobe": 10},
    }, # 搜索参数
    limit=10, # 返回的搜索结果的最大数量
    # group_by_field="doc_id", # 按文档 ID 对结果进行分组
    output_fields=["doc_id", "passage_id"]
)

# 提取 `doc_id` 列中的值
doc_ids = [result['entity']['doc_id'] for result in res[0]]

print(doc_ids)
```
输出类似于以下内容：

```python
[1, 10, 3, 10, 1, 9, 4, 4, 8, 6]
```

在给定的输出中，可以观察到返回的实体包含重复的 `doc_id` 值。

__限制__

- __索引__: 此分组功能仅适用于使用 __HNSW__、__IVF_FLAT__ 或 __FLAT__ 类型进行索引的集合。有关更多信息，请参阅[内存索引](https://milvus.io/docs/index.md#HNSW)。

- __向量__: 目前，分组搜索不支持 __BINARY_VECTOR__ 类型的向量字段。有关数据类型的更多信息，请参阅[支持的数据类型](https://milvus.io/docs/schema.md#Supported-data-types)。

- __字段__: 目前，分组搜索仅允许单个列。您不能在 `group_by_field` 配置中指定多个字段名。此外，分组搜索与 JSON、FLOAT、DOUBLE、ARRAY 或向量字段的数据类型不兼容。

- __性能影响__: 请注意，随着查询向量数量的增加，性能会下降。以具有 2 个 CPU 核心和 8 GB 内存的集群为例，分组搜索的执行时间会随着输入查询向量数量的增加而成比例增加。

- __功能__: 目前，分组搜索不受[范围搜索](https://milvus.io/docs/single-vector-search.md#Range-search)、[搜索迭代器](https://milvus.io/docs/with-iterators.md#Search-with-iterator)或[多向量搜索](multi-vector-search.md)的支持。

## 搜索参数

在上述搜索中，除了范围搜索外，适用默认搜索参数。在正常情况下，您无需手动设置搜索参数。

```python
# 在正常情况下，您无需手动设置搜索参数
# 除了范围搜索。
search_parameters = {
    'metric_type': 'L2',
    'params': {
        'nprobe': 10,
        'level': 1，
        'radius': 1.0
        'range_filter': 0.8
    }
}
```

下表列出了搜索参数中所有可能的设置。

|  __参数名称__         |  __参数描述__                                                                                                                                      |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|  `metric_type`         |  衡量向量嵌入之间相似性的方法。<br/> 可能的值为 `IP`、`L2` 和 `COSINE`，默认为加载的索引文件的值。      |
|  `params.nprobe`       |  查询期间要查询的单位数。<br/> 该值在范围 [1, nlist<sub>[1]</sub>] 内。                                                     |
|  `params.level`        |  搜索精度级别。<br/> 可能的值为 `1`、`2` 和 `3`，默认为 `1`。较高的值会产生更准确的结果，但性能较慢。  |
|  `params.radius`       |  查询向量和候选向量之间的最小相似度。<br/> 该值的范围为 [1, nlist<sub>[1]</sub>]。                              |
|  `params.range_filter` |  相似度范围，可选择性地细化在范围内查找向量。<br/> 该值的范围为 [top-K<sub>[2]</sub>, ∞]。          |

<div class="admonition note">

<p><b>notes</b></p>

<p>[1] 索引后的聚类单元数量。在对集合进行索引时，Milvus 将向量数据细分为多个聚类单元，其数量随实际索引设置而变化。</p>
<p>[2] 在搜索中返回的实体数量。</p>

</div>