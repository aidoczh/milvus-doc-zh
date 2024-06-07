---
id: configure_datanode.md
related_key: configure
group: system_configuration.md
summary: 学习如何配置 Milvus 的数据节点。
title: 数据节点相关配置
---

# 数据节点相关配置

本主题介绍了 Milvus 的数据节点相关配置。

数据节点通过订阅日志代理获取增量日志数据，处理变异请求，并将日志数据打包成日志快照，存储在对象存储中。

在本节中，您可以配置数据节点端口等。

## `dataNode.port`

<table id="dataNode.port">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>数据节点的 TCP 端口。</td>
      <td>21124</td>
    </tr>
  </tbody>
</table>

## `dataNode.grpc.serverMaxRecvSize`

<table id="dataNode.grpc.serverMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>数据节点可以接收的每个 RPC 请求的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>2147483647</td>
    </tr>
  </tbody>
</table>

## `dataNode.grpc.serverMaxSendSize`

<table id="dataNode.grpc.serverMaxSendSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在接收 RPC 请求时，数据节点可以发送的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>2147483647</td>
    </tr>
  </tbody>
</table>

## `dataNode.grpc.clientMaxRecvSize`

<table id="dataNode.grpc.clientMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在发送 RPC 请求时，数据节点可以接收的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>104857600</td>
    </tr>
  </tbody>
</table>

## `dataNode.grpc.clientMaxSendSize`

<table id="dataNode.grpc.clientMaxSendSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>数据节点可以发送的每个 RPC 请求的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>104857600</td>
    </tr>
  </tbody>
</table>

## `dataNode.dataSync.flowGraph.maxQueueLength`

<table id="dataNode.dataSync.flowGraph.maxQueueLength">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>数据节点中流图中任务队列缓存的最大大小。</li>
        <li>单位：MB</li>
## `dataNode.flush.insertBufSize`

<table id="dataNode.flush.insertBufSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>数据节点使用流图来订阅和组织消息流。</li>
      </td>
      <td>1024</td>
    </tr>
  </tbody>
</table>

## `dataNode.flush.insertBufSize`

<table id="dataNode.flush.insertBufSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在内存中缓冲段中每个 binlog 文件的最大大小。超过此值的 binlog 文件将被刷新到MinIO或S3服务。</li>
        <li>单位：字节</li>
        <li>将此参数设置得太小会导致系统频繁存储少量数据。将其设置得太大会增加系统对内存的需求。</li>
      </td>
      <td>16777216</td>
    </tr>
  </tbody>
</table>