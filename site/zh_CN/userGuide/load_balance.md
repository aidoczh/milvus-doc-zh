---
id: load_balance.md
related_key: 负载均衡
summary: 学习如何在Milvus中平衡查询负载。
title: 平衡查询负载
---

# 平衡查询负载

本主题介绍了如何在Milvus中平衡查询负载。

Milvus默认支持自动负载均衡。您可以[配置](configure-docker.md)您的Milvus以启用或禁用[自动负载均衡](configure_querycoord.md#queryCoordautoBalance)。通过指定[`queryCoord.balanceIntervalSeconds`](configure_querycoord.md#queryCoordbalanceIntervalSeconds)、[`queryCoord.overloadedMemoryThresholdPercentage`](configure_querycoord.md#queryCoordoverloadedMemoryThresholdPercentage)和[`queryCoord.memoryUsageMaxDifferencePercentage`](configure_querycoord.md#queryCoordmemoryUsageMaxDifferencePercentage)，您可以更改触发自动负载均衡的阈值。

如果禁用了自动负载均衡，您仍然可以手动平衡负载。

## 检查段信息

获取您希望传输的已封存段的`segmentID`以及您希望将段传输到的查询节点的`nodeID`。

{{fragments/multiple_code.md}}

```python
from pymilvus import utility
utility.get_query_segment_info("book")
```

```go
// 此功能正在GO客户端上积极开发中。
```

```java
milvusClient.getQuerySegmentInfo(
  GetQuerySegmentInfoParam.newBuilder()
    .withCollectionName("book")
    .build()
);
```

```javascript
await getQuerySegmentInfo({
    collectionName: "book",
});
```

```shell
show query_segment -c book
```


<table class="language-python">
<thead>
<tr>
<th>参数</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>collection_name</code></td>
<td>要检查段信息的集合名称。</td>
</tr>
</tbody>
</table>

<table class="language-javascript">
<thead>
<tr>
<th>参数</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>collectionName</code></td>
<td>要检查段信息的集合名称。</td>
</tr>
</tbody>
</table>

<table class="language-java">
<thead>
<tr>
<th>参数</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>CollectionName</code></td>
<td>要检查段信息的集合名称。</td>
</tr>
</tbody>
</table>

<table class="language-shell">
    <thead>
        <tr>
            <th>选项</th>
            <th>描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>-c</td>
            <td>要检查段信息的集合名称。</td>
        </tr>
    </tbody>
</table>

## 传输段

传输带有当前查询节点和新查询节点的`segmentID`和`nodeID`的已封存段。

{{fragments/multiple_code.md}}

```python
utility.load_balance(
  src_node_id=3, 
  dst_node_ids=[4], 
  sealed_segment_ids=[431067441441538050]
)
```

```go
// 此功能正在GO客户端上积极开发中。
```
```java
milvusClient.loadBalance(
  LoadBalanceParam.newBuilder()
    .withSourceNodeID(3L)
    .addDestinationNodeID(4L)
    .addSegmentID(431067441441538050L)
    .build()
);
```
```javascript
await loadBalance({
  src_nodeID: 3,
  dst_nodeIDs: [4],
  sealed_segmentIDs: [431067441441538050]
});
```

```shell
load_balance -s 3 -d 4 -ss 431067441441538050
```

<table class="language-python">
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>src_node_id</code></td>
<td>要从中转移片段的查询节点的ID。</td>
</tr>
<tr>
<td><code>dst_node_ids</code> (可选)</td>
<td>要将片段转移至的查询节点的ID。如果此参数留空，Milvus会自动将片段转移至其他查询节点。</td>
</tr>
<tr>
<td><code>sealed_segment_ids</code> (可选)</td>
<td>要转移的片段的ID。如果此参数留空，Milvus会自动将源查询节点中的所有密封片段转移至其他查询节点。</td>
</tr>
</tbody>
</table>

<table class="language-javascript">
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>src_nodeID</code></td>
<td>要从中转移片段的查询节点的ID。</td>
</tr>
<tr>
<td><code>dst_nodeIDs</code> (可选)</td>
<td>要将片段转移至的查询节点的ID。如果此参数留空，Milvus会自动将片段转移至其他查询节点。</td>
</tr>
<tr>
<td><code>sealed_segmentIDs</code> (可选)</td>
<td>要转移的片段的ID。如果此参数留空，Milvus会自动将源查询节点中的所有密封片段转移至其他查询节点。</td>
</tr>
</tbody>
</table>

<table class="language-java">
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>SourceNodeID</code></td>
<td>要从中转移片段的查询节点的ID。</td>
</tr>
<tr>
<td><code>DestinationNodeID</code> (可选)</td>
<td>要将片段转移至的查询节点的ID。如果此参数留空，Milvus会自动将片段转移至其他查询节点。</td>
</tr>
<tr>
<td><code>SegmentID</code> (可选)</td>
<td>要转移的片段的ID。如果此参数留空，Milvus会自动将源查询节点中的所有密封片段转移至其他查询节点。</td>
</tr>
</tbody>
</table>

<table class="language-shell">
<thead>
<tr>
<th>Option</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>-s</code></td>
<td>要从中转移片段的查询节点的ID。</td>
</tr>
<tr>
<td><code>-d</code> (多个)</td>
<td>要将片段转移至的查询节点的ID。</td>
</tr>
<tr>
<td><code>-ss</code> (多个)</td>
<td>要转移的片段的ID。</td>
</tr>
</tbody>
</table>

## 接下来做什么

- 了解更多 Milvus 的基本操作：
  - [插入、更新和删除](insert-update-delete.md)
  - [管理分区](manage-partitions.md)
  - [索引向量字段](index-vector-fields.md)
- [索引标量字段](index-scalar-fields.md)
- [单向量搜索](single-vector-search.md)
- [多向量搜索](multi-vector-search.md)