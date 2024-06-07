---
id: configure_querynode.md
related_key: configure
group: system_configuration.md
summary: 学习如何配置 Milvus 的查询节点。
title: 查询节点相关配置
---

# 查询节点相关配置

本主题介绍了 Milvus 查询节点的相关配置。

查询节点在增量数据和历史数据上执行向量和标量数据的混合搜索。

在本节中，您可以配置查询节点端口、优雅时间等。

## `queryNode.gracefulTime`

<table id="queryNode.gracefulTime">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>新插入数据可被搜索的最短时间。</li>
        <li>单位：毫秒</li>
        <li>当搜索消息的时间戳早于查询节点系统时间时，Milvus 直接执行搜索请求。</li>
        <li>当搜索消息的时间戳晚于查询节点系统时间时，Milvus 将等待直到查询节点系统时间和时间戳之间的时间差小于此参数，然后执行搜索请求。</li>
      </td>
      <td>0</td>
    </tr>
  </tbody>
</table>

## `queryNode.port`

<table id="queryNode.port">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>查询节点的 TCP 端口。</td>
      <td>21123</td>
    </tr>
  </tbody>
</table>

## `queryNode.grpc.serverMaxRecvSize`

<table id="queryNode.grpc.serverMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>查询节点可以接收的每个 RPC 请求的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>2147483647</td>
    </tr>
  </tbody>
</table>

## `queryNode.grpc.serverMaxSendSize`

<table id="queryNode.grpc.serverMaxSendSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在接收 RPC 请求时，查询节点可以发送的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>2147483647</td>
    </tr>
  </tbody>
</table>

## `queryNode.grpc.clientMaxRecvSize`

<table id="queryNode.grpc.clientMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在发送 RPC 请求时，查询节点可以接收的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>104857600</td>
    </tr>
  </tbody>
</table>

## `queryNode.grpc.clientMaxSendSize`

<table id="queryNode.grpc.clientMaxSendSize">
  <thead>
    <tr>
## `queryNode.replicas`

<table id="queryNode.replicas">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
     <td>
        在加载集合时，在查询节点上创建的数据段的内存副本数量。在独立部署中，最大值为1。有关更多信息，请参阅 <a href="https://milvus.io/docs/replica.md#In-Memory-Replica">内存副本</a>。
      </td>
      <td>1</td>
    </tr>
  </tbody>
</table>

## `queryNode.stats.publishInterval`

<table id="queryNode.stats.publishInterval">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        查询节点发布节点统计信息的间隔，包括段状态、CPU 使用率、内存使用率、健康状态等。
        <br/>
        单位：毫秒
      </td>
      <td>1000</td>
    </tr>
  </tbody>
</table>

## `queryNode.dataSync.flowGraph.maxQueueLength`

<table id="queryNode.dataSync.flowGraph.maxQueueLength">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        查询节点中流图中任务队列缓存的最大大小。
        <br/>
        单位：MB
        <br/>
        查询节点使用流图订阅和组织消息流。
      </td>
      <td>1024</td>
    </tr>
  </tbody>
</table>

## `queryNode.segcore.chunkRows`

<table id="queryNode.segcore.chunkRows">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
          Segcore 将段划分为块的行数。
      </td>
      <td>1024</td>
    </tr>
  </tbody>
</table>

## `queryNode.segcore.InterimIndex`

<table id="queryNode.segcore.chunkRows">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
          是否为正在增长的段和尚未索引的密封段创建临时索引，以提高搜索性能。
          <br/>
          <ul><li>
            Milvus 最终会密封并索引所有段，但启用此功能可优化数据插入后立即查询的搜索性能。
          </li>
          <li>
这个默认值是 `true`，表示 Milvus 会为不断增长的段和未被索引的封存段创建临时索引。