---
id: manage-partitions.md
title: 管理分区
---

# 管理分区

本指南将指导您如何在集合中创建和管理分区。

## 概述

在 Milvus 中，一个分区代表集合的一个子部分。这种功能允许将集合的物理存储划分为多个部分，通过将焦点缩小到数据的一个较小子集，有助于提高查询性能，而不是整个集合。

在创建集合时，至少会自动创建一个名为 ___default__ 的默认分区。您可以在一个集合中创建最多 4,096 个分区。

<div class="admonition note">

<p><b>注意</b></p>

<p>Milvus 引入了一个名为 <strong>Partition Key</strong> 的功能，利用底层分区根据特定字段的哈希值存储实体。该功能有助于实现多租户，提升搜索性能。详情请阅读 <a href="https://milvus.io/docs/use-partition-key.md">使用分区键</a>。</p>
<p>如果在集合中启用了 <strong>Partition Key</strong> 功能，Milvus 将负责管理所有分区，减轻您的责任。</p>

</div>

## 准备工作

下面的代码片段重新利用现有代码，建立与 Milvus 的连接并以快速设置模式创建集合，指示在创建时加载集合。

<div class="language-python">

对于准备工作，使用 [`MiluvsClient`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md) 连接到 Milvus，并使用 [`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md) 在快速设置模式下创建集合。

</div>

<div class="language-java">

对于准备工作，使用 [`MilvusClientV2`](https://milvus.io/api-reference/java/v2.4.x/v2/Client/MilvusClientV2.md) 连接到 Milvus，并使用 [`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Collections/createCollection.md) 在快速设置模式下创建集合。

</div>

<div class="language-javascript">

对于准备工作，使用 [`MilvusClient`](https://milvus.io/api-reference/node/v2.4.x/Client/MilvusClient.md) 连接到 Milvus，并使用 [`createCollection()`](https://milvus.io/api-reference/node/v2.4.x/Collections/createCollection.md) 在快速设置模式下创建集合。

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

# 2. 创建集合
client.create_collection(
    collection_name="quick_setup",
    dimension=5,
)

```
```java
import io.milvus.v2.client.ConnectConfig;
import io.milvus.v2.client.MilvusClientV2;
import io.milvus.v2.service.collection.request.CreateCollectionReq;

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
    .build();

client.createCollection(quickSetupReq);
```
```javascript
const address = "http://localhost:19530"

// 1. 设置 Milvus 客户端
client = new MilvusClient({address});

// 2. 在快速设置模式下创建集合
await client.createCollection({
    collection_name: "quick_setup",
    dimension: 5,
});  
```

<div class="admonition note">

<p><b>注意</b></p>

<p>在上述代码片段中，已经在创建集合的同时创建了集合的索引，表明集合在创建时已经被加载。</p>

</div>

## 列出分区

一旦集合准备就绪，您可以列出其分区。

<div class="language-python">

要列出分区，请使用 [`list_partitions()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Partitions/list_partitions.md)。

</div>

<div class="language-java">

要列出分区，请使用 [`listPartitions()`](https://milvus.io/api-reference/java/v2.4.x/v2/Partitions/listPartitions.md)。

</div>

<div class="language-javascript">

要列出分区，请使用 [`listPartitions()`](https://milvus.io/api-reference/node/v2.4.x/Partitions/listPartitions.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 3. 列出分区
res = client.list_partitions(collection_name="quick_setup")
print(res)

# 输出
#
# ["_default"]
```

```java
import io.milvus.v2.service.partition.request.ListPartitionsReq;

// 3. 列出集合中的所有分区
ListPartitionsReq listPartitionsReq = ListPartitionsReq.builder()
    .collectionName("quick_setup")
    .build();

List<String> partitionNames = client.listPartitions(listPartitionsReq);

System.out.println(partitionNames);

// 输出:
// ["_default"]
```

```javascript
// 3. 列出分区
res = await client.listPartitions({
    collection_name: "quick_setup"
})

console.log(res.partition_names)

// 输出
// 
// [ '_default' ]
// 
```

上述代码片段的输出包括了指定集合中分区的名称。

<div class="admonition note">

<p><b>注意</b></p>

<p>如果您在集合中设置了字段作为分区键，Milvus 会在创建集合时至少创建 <strong>64</strong> 个分区。当列出分区时，结果可能与上述代码片段的输出不同。</p>
<p>详情请参阅 <a href="https://milvus.io/docs/use-partition-key.md">使用分区键</a>。</p>

</div>

## 创建分区

您可以向集合添加更多分区。一个集合最多可以有 64 个分区。

<div class="language-python">

要创建分区，请使用 [`create_partition()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Partitions/create_partition.md)。

</div>

<div class="language-java">

要创建分区，请使用 [`createPartition()`](https://milvus.io/api-reference/java/v2.4.x/v2/Partitions/createPartition.md)。

</div>

<div class="language-javascript">
```
要创建分区，请使用[`createPartition()`](https://milvus.io/api-reference/node/v2.4.x/Partitions/createPartition.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 4. 创建更多分区
client.create_partition(
    collection_name="quick_setup",
    partition_name="partitionA"
)

client.create_partition(
    collection_name="quick_setup",
    partition_name="partitionB"
)

res = client.list_partitions(collection_name="quick_setup")
print(res)

# 输出
#
# ["_default", "partitionA", "partitionB"]
```

```java
import io.milvus.v2.service.partition.request.CreatePartitionReq;

// 4. 创建更多分区
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

listPartitionsReq = ListPartitionsReq.builder()
    .collectionName("quick_setup")
    .build();

partitionNames = client.listPartitions(listPartitionsReq);

System.out.println(partitionNames);

// 输出:
// [
//     "_default",
//     "partitionA",
//     "partitionB"
// ]
```

```javascript
// 4. 创建更多分区
await client.createPartition({
    collection_name: "quick_setup",
    partition_name: "partitionA"
})

await client.createPartition({
    collection_name: "quick_setup",
    partition_name: "partitionB"
})

res = await client.listPartitions({
    collection_name: "quick_setup"
})

console.log(res.partition_names)

// 输出
// 
// [ '_default', 'partitionA', 'partitionB' ]
// 
```

上面的代码片段在集合中创建一个分区，并列出该集合的分区。

<div class="admonition note">

<p><b>注意</b></p>

<p>如果在集合中将字段设置为分区键，Milvus会负责管理集合中的分区。因此，在尝试创建分区时可能会遇到提示的错误。</p>
<p>详情请参阅<a href="https://milvus.io/docs/use-partition-key.md">使用分区键</a>。</p>

</div>

## 检查特定分区

您还可以检查特定分区的存在。

<div class="language-python">

要检查特定分区，请使用[`has_partition()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Partitions/has_partition.md)。

</div>

<div class="language-java">

要检查特定分区，请使用[`hasPartition()`](https://milvus.io/api-reference/java/v2.4.x/v2/Partitions/hasPartition.md)。

</div>

<div class="language-javascript">

要检查特定分区，请使用[`hasPartition()`](https://milvus.io/api-reference/node/v2.4.x/Partitions/hasPartition.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
```python
# 5. 检查分区是否存在
res = client.has_partition(
    collection_name="quick_setup",
    partition_name="partitionA"
)
print(res)

# 输出
#
# True

res = client.has_partition(
    collection_name="quick_setup",
    partition_name="partitionC"
)
print(res)

# 输出
#
# False
```

```java
import io.milvus.v2.service.partition.request.HasPartitionReq;

// 5. 检查分区是否存在
HasPartitionReq hasPartitionReq = HasPartitionReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionA")
    .build();

boolean exists = client.hasPartition(hasPartitionReq);

System.out.println(exists);

// 输出:
// true

hasPartitionReq = HasPartitionReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionC")
    .build();

exists = client.hasPartition(hasPartitionReq);

System.out.println(exists);

// 输出:
// false
```

```javascript
// 5. 检查分区是否存在
res = await client.hasPartition({
    collection_name: "quick_setup",
    partition_name: "partitionA"
})

console.log(res.value)

// 输出
// 
// true
// 

res = await client.hasPartition({
    collection_name: "quick_setup",
    partition_name: "partitionC"
})

console.log(res.value)

// 输出
// 
// false
// 
```

上面的代码片段检查了集合是否有名为 `partitionA` 和 `partitionC` 的分区。

## 加载和释放分区

您可以加载和释放特定的分区，使其对搜索和查询可用或不可用。

### 获取加载状态

<div class="language-python">

要检查集合及其分区的加载状态，请使用 [`get_load_state()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/get_load_state.md)。

</div>

<div class="language-java">

要检查集合及其分区的加载状态，请使用 [`getLoadState()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/getLoadState.md)。

</div>

<div class="language-javascript">

要检查集合及其分区的加载状态，请使用 [`getLoadState()`](https://milvus.io/api-reference/node/v2.4.x/Management/getLoadState.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 释放集合
client.release_collection(collection_name="quick_setup")

# 检查加载状态
res = client.get_load_state(collection_name="quick_setup")
print(res)

# 输出
#
# {
#     "state": "<LoadState: Loaded>"
# }

res = client.get_load_state(
    collection_name="quick_setup", 
    partition_name="partitionA"
)

print(res)

# 输出
#
# {
#     "state": "<LoadState: Loaded>"
# }

res = client.get_load_state(
    collection_name="quick_setup", 
    partition_name="partitionB"
)

print(res)

# 输出
#
# {
#     "state": "<LoadState: NotLoad>"
# }

```
```java
import io.milvus.v2.service.collection.request.GetLoadStateReq;
import io.milvus.v2.service.collection.request.ReleaseCollectionReq;
import io.milvus.v2.service.partition.request.LoadPartitionsReq;
import io.milvus.v2.service.partition.request.ReleasePartitionsReq;

// 6. 单独加载一个分区
// 6.1 释放集合
ReleaseCollectionReq releaseCollectionReq = ReleaseCollectionReq.builder()
    .collectionName("quick_setup")
    .build();

client.releaseCollection(releaseCollectionReq);

// 6.2 加载 partitionA
LoadPartitionsReq loadPartitionsReq = LoadPartitionsReq.builder()
    .collectionName("quick_setup")
    .partitionNames(List.of("partitionA"))
    .build();

client.loadPartitions(loadPartitionsReq);

Thread.sleep(3000);

// 6.3 检查集合及其分区的加载状态
GetLoadStateReq getLoadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .build();

boolean state = client.getLoadState(getLoadStateReq);

System.out.println(state);

// 输出:
// true

getLoadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionA")
    .build();

state = client.getLoadState(getLoadStateReq);

System.out.println(state);

// 输出:
// true

getLoadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionB")
    .build();

state = client.getLoadState(getLoadStateReq);

System.out.println(state);

// 输出:
// false
```
```javascript
// 6. 单独加载分区
await client.releaseCollection({
    collection_name: "quick_setup"
})

res = await client.getLoadState({
    collection_name: "quick_setup"
})

console.log(res.state)

// 输出
// 
// LoadStateNotLoad
// 

await client.loadPartitions({
    collection_name: "quick_setup",
    partition_names: ["partitionA"]
})

await sleep(3000)

res = await client.getLoadState({
    collection_name: "quick_setup"
})

console.log(res.state)

// 输出
// 
// LoadStateLoaded
// 

res = await client.getLoadState({
    collection_name: "quick_setup",
    partition_name: "partitionA"
})

console.log(res.state)

// 输出
// 
// LoadStateLoaded
// 

res = await client.getLoadState({
    collection_name: "quick_setup",
    partition_name: "partitionB"
})

console.log(res.state)

// 输出
// 
// LoadStateLoaded
// 
```

可能的加载状态可能是以下之一

- __Loaded__

    如果至少有一个分区已加载，则将集合标记为`Loaded`。

- __NotLoad__

    如果没有一个分区被加载，则将集合标记为`NotLoad`。

- __Loading__

    如果至少有一个分区正在加载过程中，则将集合标记为Loading。

### 加载分区

<div class="language-python">

要加载集合的所有分区，只需调用[`load_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/load_collection.md)。要加载集合的特定分区，请使用[`load_partitions()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Partitions/load_partitions.md)。

</div>

<div class="language-java">

要加载集合的所有分区，只需调用[`loadCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/loadCollection.md)。要加载集合的特定分区，请使用[`loadPartitions()`](https://milvus.io/api-reference/java/v2.4.x/v2/Partitions/loadPartitions.md)。

</div>

<div class="language-javascript">

要加载集合的所有分区，只需调用[`loadCollection()`](https://milvus.io/api-reference/node/v2.4.x/Management/loadCollection.md)。要加载集合的特定分区，请使用[`loadPartitions()`](https://milvus.io/api-reference/node/v2.4.x/Partitions/loadPartitions.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
client.load_partitions(
    collection_name="quick_setup",
    partition_names=["partitionA"]
)

res = client.get_load_state(collection_name="quick_setup")
print(res)

# 输出
#
# {
#     "state": "<LoadState: Loaded>"
# }
```
```java
LoadPartitionsReq loadPartitionsReq = LoadPartitionsReq.builder()
    .collectionName("quick_setup")
    .partitionNames(List.of("partitionA"))
    .build();

client.loadPartitions(loadPartitionsReq);

getLoadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionA")
    .build();

state = client.getLoadState(getLoadStateReq);

System.out.println(state);

// 输出:
// true
```
```javascript
await client.loadPartitions({
    collection_name: "quick_setup",
    partition_names: ["partitionA"]
})

res = await client.getLoadState({
    collection_name: "quick_setup",
    partition_name: "partitionA"
})

console.log(res.state)

// 输出
// 
// LoadStateLoaded
//
```

要一次加载多个分区，请按照以下步骤操作：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
client.load_partitions(
    collection_name="quick_setup",
    partition_names=["partitionA", "partitionB"]
)

res = client.get_load_status(
    collection_name="quick_setup",
    partition_name="partitionA"
)

# 输出
#
# {
#     "state": "<LoadState: Loaded>"
# }

res = client.get_load_status(
    collection_name="quick_setup",
    partition_name="partitionB"
)

# 输出
#
# {
#     "state": "<LoadState: Loaded>"
# }
```

```java
LoadPartitionsReq loadPartitionsReq = LoadPartitionsReq.builder()
    .collectionName("quick_setup")
    .partitionNames(List.of("partitionA", "partitionB"))
    .build();

client.loadPartitions(loadPartitionsReq);

getLoadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionA")
    .build();

state = client.getLoadState(getLoadStateReq);

System.out.println(state);

// 输出:
// true

getLoadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionB")
    .build();

state = client.getLoadState(getLoadStateReq);

System.out.println(state);

// 输出:
// true
```

```javascript
await client.loadPartitions({
    collection_name: "quick_setup",
    partition_names: ["partitionA", "partitionB"]
})

res = await client.getLoadState({
    collection_name: "quick_setup",
    partition_name: "partitionA"
})

console.log(res)

// 输出
// 
// LoadStateLoaded
// 

res = await client.getLoadState({
    collection_name: "quick_setup",
    partition_name: "partitionB"
})

console.log(res)

// 输出
// 
// LoadStateLoaded
// 
```

### 释放分区

<div class="language-python">

要释放集合的所有分区，只需调用 [`release_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/release_collection.md)。要释放集合的特定分区，请使用 [`release_partitions()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Partitions/release_partitions.md)。

</div>

<div class="language-java">

要释放集合的所有分区，只需调用 [`releaseCollection()`](https://milvus.io/api-reference/java/v2.4.x/v2/Management/releaseCollection.md)。要释放集合的特定分区，请使用 [`releasePartitions()`](https://milvus.io/api-reference/java/v2.4.x/v2/Partitions/releasePartitions.md)。

</div>

<div class="language-javascript">
```
释放集合的所有分区，只需调用 [`releaseCollection()`](https://milvus.io/api-reference/node/v2.4.x/Management/releaseCollection.md)。要释放集合的特定分区，请使用 [`releasePartitions()`](https://milvus.io/api-reference/node/v2.4.x/Partitions/releasePartitions.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 7. 释放一个分区
client.release_partitions(
    collection_name="quick_setup",
    partition_names=["partitionA"]
)

res = client.get_load_state(
    collection_name="quick_setup", 
    partition_name="partitionA"
)

print(res)

# 输出
#
# {
#     "state": "<LoadState: NotLoad>"
# }

```

```java
import io.milvus.v2.service.partition.request.ReleasePartitionsReq;

// 7. 释放一个分区
ReleasePartitionsReq releasePartitionsReq = ReleasePartitionsReq.builder()
    .collectionName("quick_setup")
    .partitionNames(List.of("partitionA"))
    .build();

client.releasePartitions(releasePartitionsReq);

getLoadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .partitionName("partitionA")
    .build();

state = client.getLoadState(getLoadStateReq);

System.out.println(state);

// 输出:
// false
```

```javascript
// 7. 释放一个分区
await client.releasePartitions({
    collection_name: "quick_setup",
    partition_names: ["partitionA"]
})

res = await client.getLoadState({
    collection_name: "quick_setup"
})

console.log(res.state)

// 输出
// 
// LoadStateNotLoad
// 
```

要一次释放多个分区，请按照以下步骤进行：

```python
client.release_partitions(
    collection_name="quick_setup",
    partition_names=["_default", "partitionA", "partitionB"]
)

res = client.get_load_status(
    collection_name="quick_setup",
)

# 输出
#
# {
#     "state": "<LoadState: NotLoad>"
# }
```

## 删除分区

一旦释放了一个分区，如果不再需要，可以将其删除。

<div class="language-python">

要删除一个分区，请使用 [`drop_partition()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Partitions/drop_partition.md)。

</div>

<div class="language-java">

要删除一个分区，请使用 [`dropPartition()`](https://milvus.io/api-reference/java/v2.4.x/v2/Partitions/dropPartition.md)。

</div>

<div class="language-javascript">

要删除一个分区，请使用 [`dropPartition()`](https://milvus.io/api-reference/node/v2.4.x/Partitions/dropPartition.md)。

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
# 8. 删除一个分区
client.drop_partition(
    collection_name="quick_setup",
    partition_name="partitionB"
)

res = client.list_partitions(collection_name="quick_setup")
print(res)

# 输出
#
# ["_default", "partitionA"]
```
```java
import io.milvus.v2.service.partition.request.ReleasePartitionsReq;

ReleasePartitionsReq releasePartitionsReq = ReleasePartitionsReq.builder()
    .collectionName("quick_setup")
    .partitionNames(List.of("_default", "partitionA", "partitionB"))
    .build();

client.releasePartitions(releasePartitionsReq);

getLoadStateReq = GetLoadStateReq.builder()
    .collectionName("quick_setup")
    .build();

state = client.getLoadState(getLoadStateReq);

System.out.println(state);

// 输出:
// false
```
```javascript
await client.releasePartitions({
    collection_name: "quick_setup",
    partition_names: ["_default", "partitionA", "partitionB"]
})

res = await client.getLoadState({
    collection_name: "quick_setup"
})

console.log(res)

// 输出
// 
// {
//   status: {
//     error_code: 'Success',
//     reason: '',
//     code: 0,
//     retriable: false,
//     detail: ''
//   },
//   state: 'LoadStateNotLoad'
// }
// 
```

<div class="admonition note">

<p><b>注意</b></p>

<p>在删除分区之前，您需要将其从内存中释放。</p>

</div>

## 常见问题

- __一个分区可以存储多少数据？__

    建议在一个分区中存储少于1B的数据。

- __最多可以创建多少个分区？__

    默认情况下，Milvus允许创建最多4,096个分区。您可以通过配置 `rootCoord.maxPartitionNum` 来调整最大分区数。有关详细信息，请参考[系统配置](https://milvus.io/docs/configure_rootcoord.md#rootCoordmaxPartitionNum)。

- __如何区分分区和分区键？__

    分区是物理存储单元，而分区键是根据指定列自动将数据分配到特定分区的逻辑概念。

    例如，在Milvus中，如果您有一个以 `color` 字段定义为分区键的集合，系统会根据每个实体 `color` 字段的哈希值自动将数据分配到分区。这个自动化过程使用户无需在插入或搜索数据时手动指定分区。

    另一方面，当手动创建分区时，您需要根据分区键的标准将数据分配到每个分区。如果您有一个带有 `color` 字段的集合，您将手动将具有 `color` 值为 `red` 的实体分配到 `分区A`，将具有 `color` 值为 `blue` 的实体分配到 `分区B`。这种手动管理需要更多的工作。

    总之，分区和分区键都用于优化数据计算和提高查询效率。重要的是要意识到启用分区键意味着放弃对分区数据插入和加载的手动管理控制，因为这些过程完全自动化并由Milvus处理。
```