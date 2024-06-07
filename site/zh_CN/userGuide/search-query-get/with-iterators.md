---
id: with-iterators.md
order: 4
summary: Milvus提供了用于迭代具有大量实体结果的搜索和查询迭代器。
title: 使用迭代器
---

# 使用迭代器

Milvus提供了用于迭代具有大量实体结果的搜索和查询迭代器。

## 概述

迭代器是一种强大的工具，通过使用主键值和布尔表达式，帮助您浏览大型数据集。这可以显著改善您检索数据的方式。与传统的使用 __offset__ 和 __limit__ 参数不同，随着时间推移，这种方法可能变得不那么高效，迭代器提供了一种更具可扩展性的解决方案。

### 使用迭代器的好处

- __简单性__：消除了复杂的 __offset__ 和 __limit__ 设置。

- __效率__：通过仅获取所需数据，提供可扩展的数据检索。

- __一致性__：通过布尔过滤器确保一致的数据集大小。

<div class="admonition note">

<p><b>注意事项</b></p>

<ul>

<li>此功能适用于 Milvus 2.3.x 或更高版本。</li>

</ul>

</div>

## 准备工作

以下步骤重新配置代码以连接到 Milvus，快速设置一个集合，并将超过10,000个随机生成的实体插入到集合中。

### 步骤1：创建一个集合

<div class="language-python">

使用 [`MilvusClient`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md) 连接到 Milvus 服务器，并使用 [`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md) 创建一个集合。

</div>

<div class="language-java">

使用 [`MilvusClientV2`](https://milvus.io/api-reference/java/v2.4.x/v2/Client/MilvusClientV2.md) 连接到 Milvus 服务器，并使用 [`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md) 创建一个集合。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
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
import io.milvus.client.MilvusServiceClient;
import io.milvus.param.ConnectParam;
import io.milvus.param.highlevel.collection.CreateSimpleCollectionParam;

String CLUSTER_ENDPOINT = "http://localhost:19530";

// 1. 连接到 Milvus 服务器
ConnectParam connectParam = ConnectParam.newBuilder()
        .withUri(CLUSTER_ENDPOINT)
        .build();

MilvusServiceClient client  = new MilvusServiceClient(connectParam);

// 2. 创建一个集合
CreateSimpleCollectionParam createCollectionParam = CreateSimpleCollectionParam.newBuilder()
        .withCollectionName("quick_setup")
        .withDimension(5)
        .build();

client.createCollection(createCollectionParam);
```

### 步骤2：插入随机生成的实体

<div class="language-python">
使用 [`insert()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/insert.md) 将实体插入集合中。

</div>

<div class="language-java">

使用 [`insert()`](https://milvus.io/api-reference/java/v2.4.x/v2/Vector/insert.md) 将实体插入集合中。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
</div>

```python
# 3. 插入随机生成的向量
colors = ["green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey"]
data = []

for i in range(10000):
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
#         -0.5705990742218152,
#         0.39844925120642083,
#         -0.8791287928610869,
#         0.024163154953680932,
#         0.6837669917169638
#     ],
#     "color": "purple",
#     "tag": 7774,
#     "color_tag": "purple_7774"
# }

res = client.insert(
    collection_name="quick_setup",
    data=data,
)

print(res)

# 输出
#
# {
#     "insert_count": 10000,
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
#         "(9990 more items hidden)"
#     ]
# }
```

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Random;

import com.alibaba.fastjson.JSONObject;

import io.milvus.param.R;
import io.milvus.param.dml.InsertParam;
import io.milvus.response.MutationResultWrapper;
import io.milvus.grpc.MutationResult;


// 3. 将随机生成的向量插入集合
List<String> colors = Arrays.asList("green", "blue", "yellow", "red", "black", "white", "purple", "pink", "orange", "brown", "grey");
List<JSONObject> data = new ArrayList<>();

for (int i=0; i<10000; i++) {
    Random rand = new Random();
    String current_color = colors.get(rand.nextInt(colors.size()-1));
    JSONObject row = new JSONObject();
    row.put("id", Long.valueOf(i));
    row.put("vector", Arrays.asList(rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat(), rand.nextFloat()));
    row.put("color_tag", current_color + "_" + String.valueOf(rand.nextInt(8999) + 1000));
    data.add(row);
}

InsertParam insertParam = InsertParam.newBuilder()
    .withCollectionName("quick_setup")
    .withRows(data)
    .build();

R<MutationResult> insertRes = client.insert(insertParam);

if (insertRes.getStatus() != R.Status.Success.getCode()) {
    System.err.println(insertRes.getMessage());
}

MutationResultWrapper wrapper = new MutationResultWrapper(insertRes.getData());
System.out.println(wrapper.getInsertCount());
```

## 使用迭代器进行搜索
迭代器使相似性搜索更具可扩展性。要使用迭代器进行搜索，请按照以下步骤操作：

1. 初始化搜索迭代器以定义搜索参数和输出字段。

2. 在循环中使用 __next()__ 方法来分页浏览搜索结果。

    - 如果该方法返回一个空数组，则表示循环结束，没有更多页面可用。

    - 所有结果都携带指定的输出字段。

3. 当检索到所有数据后，手动调用 __close()__ 方法关闭迭代器。

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
</div>

```python
from pymilvus import Collection

# 4. 使用迭代器进行搜索
connections.connect(uri=CLUSTER_ENDPOINT, token=TOKEN)
collection = Collection("quick_setup")

query_vectors = [[0.3580376395471989, -0.6023495712049978, 0.18414012509913835, -0.26286205330961354, 0.9029438446296592]]
search_params = {
    "metric_type": "IP",
    "params": {"nprobe": 10}
}

iterator = collection.search_iterator(
    data=query_vectors,
    anns_field="vector",
    batch_size=10,
    param=search_params,
    output_fields=["color_tag"],
    limit=3
)

results = []

while True:
    result = iterator.next()
    if not result:
        iterator.close()
        break
        
    results.extend(result)
    
    for hit in result:
        results.append(hit.to_dict())

print(results)

# 输出
#
# [
#     {
#         "id": 1756,
#         "distance": 2.0642056465148926,
#         "entity": {
#             "color_tag": "black_9109"
#         }
#     },
#     {
#         "id": 6488,
#         "distance": 1.9437453746795654,
#         "entity": {
#             "color_tag": "purple_8164"
#         }
#     },
#     {
#         "id": 3338,
#         "distance": 1.9107104539871216,
#         "entity": {
#             "color_tag": "brown_8121"
#         }
#     }
# ]
```
```java
import io.milvus.param.dml.QueryIteratorParam;
import io.milvus.param.dml.SearchIteratorParam;
import io.milvus.response.QueryResultsWrapper;
import io.milvus.orm.iterator.SearchIterator;

// 4. 使用迭代器进行搜索
SearchIteratorParam iteratorParam = SearchIteratorParam.newBuilder()
    .withCollectionName("quick_setup")
    .withVectorFieldName("vector")
    // 在兼容 Milvus 2.4.x 的集群中使用 withFloatVectors()
    .withVectors(Arrays.asList(0.3580376395471989f, -0.6023495712049978f, 0.18414012509913835f, -0.26286205330961354f, 0.9029438446296592f))
    .withBatchSize(10L)
    .withParams("{\"metric_type\": \"COSINE\", \"params\": {\"level\": 1}}")
    .build();
        

R<SearchIterator> searchIteratorRes = client.searchIterator(iteratorParam);

if (searchIteratorRes.getStatus() != R.Status.Success.getCode()) {
    System.err.println(searchIteratorRes.getMessage());
}

SearchIterator searchIterator = searchIteratorRes.getData();
List<QueryResultsWrapper.RowRecord> results = new ArrayList<>();

while (true) {
    List<QueryResultsWrapper.RowRecord> batchResults = searchIterator.next();
    if (batchResults.isEmpty()) {
        searchIterator.close();
        break;
    }
    for (QueryResultsWrapper.RowRecord rowRecord : batchResults) {
        results.add(rowRecord);
    }
}

System.out.println(results.size());
```
## 使用迭代器进行查询

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
</div>

```python
# 6. 使用迭代器进行查询
iterator = collection.query_iterator(
    batch_size=10, # 控制每次调用 next() 返回的大小
    expr="color_tag like \"brown_8\"",
    output_fields=["color_tag"]
)

results = []

while True:
    result = iterator.next()
    if not result:
        iterator.close()
        break
        
    results.extend(result)
    
# 8. 检查搜索结果
print(len(results))

print(results[:3])

# 输出
#
# [
#     {
#         "color_tag": "brown_8785",
#         "id": 94
#     },
#     {
#         "color_tag": "brown_8568",
#         "id": 176
#     },
#     {
#         "color_tag": "brown_8721",
#         "id": 289
#     }
# ]
```

```java
import io.milvus.param.dml.QueryIteratorParam;
import io.milvus.orm.iterator.QueryIterator;

// 5. 使用迭代器进行查询

try {
    Files.write(Path.of("results.json"), JSON.toJSONString(new ArrayList<>()).getBytes(), StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING);
} catch (Exception e) {
    // TODO: 处理异常
    e.printStackTrace();
}

QueryIteratorParam queryIteratorParam = QueryIteratorParam.newBuilder()
    .withCollectionName("quick_setup")
    .withExpr("color_tag like \"brown_8%\"")
    .withBatchSize(50L)
    .addOutField("vector")
    .addOutField("color_tag")
    .build();

R<QueryIterator> queryIteratRes = client.queryIterator(queryIteratorParam);

if (queryIteratRes.getStatus() != R.Status.Success.getCode()) {
    System.err.println(queryIteratRes.getMessage());
}

QueryIterator queryIterator = queryIteratRes.getData();

while (true) {
    List<QueryResultsWrapper.RowRecord> batchResults = queryIterator.next();
    if (batchResults.isEmpty()) {
        queryIterator.close();
        break;
    }

    String jsonString = "";
    List<JSONObject> jsonObject = new ArrayList<>();
    try {
        jsonString = Files.readString(Path.of("results.json"));
        jsonObject = JSON.parseArray(jsonString).toJavaList(null);
    } catch (IOException e) {
        // TODO: 处理异常
        e.printStackTrace();
    }

    for (QueryResultsWrapper.RowRecord queryResult : batchResults) {
        JSONObject row = new JSONObject();
        row.put("id", queryResult.get("id"));
        row.put("vector", queryResult.get("vector"));
        row.put("color_tag", queryResult.get("color_tag"));
        jsonObject.add(row);
    }

    try {
        Files.write(Path.of("results.json"), JSON.toJSONString(jsonObject).getBytes(), StandardOpenOption.WRITE);
    } catch (IOException e) {
        // TODO: 处理异常
        e.printStackTrace();
    }
}
```