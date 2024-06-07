---
id: configure_datacoord.md
related_key: configure
group: system_configuration.md
summary: 学习如何配置 Milvus 的数据协调器。
title: 数据协调器相关配置
---

# 数据协调器相关配置

本主题介绍了 Milvus 的数据协调器相关配置。

数据协调器（data coord）管理数据节点的拓扑结构，维护元数据，并触发刷新、压缩和其他后台数据操作。

在本节中，您可以配置数据协调器地址、段设置、压缩、垃圾回收等。


## `dataCoord.address`

<table id="dataCoord.address">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>数据协调器的 TCP/IP 地址。</li>
        <li>如果将此参数设置为 <code>0.0.0.0</code>，数据协调器将监视所有 IPv4 地址。</li>
      </td>
      <td>localhost</td>
    </tr>
  </tbody>
</table>

## `dataCoord.port`

<table id="dataCoord.port">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>数据协调器的 TCP 端口。</td>
      <td>13333</td>
    </tr>
  </tbody>
</table>

## `dataCoord.grpc.serverMaxRecvSize`

<table id="dataCoord.grpc.serverMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>数据协调器可以接收的每个 RPC 请求的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>2147483647</td>
    </tr>
  </tbody>
</table>

## `dataCoord.grpc.serverMaxSendSize`

<table id="dataCoord.grpc.serverMaxSendSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在接收 RPC 请求时，数据协调器可以发送的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>2147483647</td>
    </tr>
  </tbody>
</table>


## `dataCoord.grpc.clientMaxRecvSize`

<table id="dataCoord.grpc.clientMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在发送 RPC 请求时，数据协调器可以接收的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>104857600</td>
    </tr>
  </tbody>
</table>


## `dataCoord.grpc.clientMaxSendSize`

<table id="dataCoord.grpc.clientMaxSendSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>数据协调器可以发送的每个 RPC 请求的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>104857600</td>
    </tr>
  </tbody>
</table>

## `dataCoord.activeStandby.enabled`

<table id="rootCoord.dmlChannelNum">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        数据协调器是否以主备模式工作。
      </td>
      <td>false</td>
    </tr>
  </tbody>
</table>

## `dataCoord.replicas`

<table id="rootCoord.dmlChannelNum">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        数据协调器 Pod 的数量。如果 `dataCoord.activeStandby.enabled` 设置为 `true`，则此项为必填项。
      </td>
      <td>1</td>
    </tr>
  </tbody>
</table>

## `dataCoord.enableCompaction`

<table id="dataCoord.enableCompaction">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>控制是否启用段压缩的开关。</li>
        <li>压缩会将小型段合并为大型段，并清除超出时间旅行保留期限的已删除实体。</li>
      </td>
      <td>true</td>
    </tr>
  </tbody>
</table>


## `dataCoord.enableGarbageCollection`

<table id="dataCoord.enableGarbageCollection">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        控制是否启用垃圾收集以清除 MinIO 或 S3 服务中的废弃数据的开关。
      </td>
      <td>true</td>
    </tr>
  </tbody>
</table>

## `dataCoord.segment.maxSize`

<table id="dataCoord.segment.maxSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>段的最大大小。</li>
        <li>单位：MB</li>
        <li><code>datacoord.segment.maxSize</code> 和 <code>datacoord.segment.sealProportion</code> 共同决定是否可以封存一个段。</li>
      </td>
      <td>512</td>
    </tr>
  </tbody>
</table>

## `dataCoord.segment.sealProportion`

<table id="dataCoord.segment.sealProportion">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>封存一个段所需的最小比例，相对于 <code>datacoord.segment.maxSize</code>。</li>
        <li><code>datacoord.segment.maxSize</code> 和 <code>datacoord.segment.sealProportion</code> 共同决定是否可以封存一个段。</li>
      </td>
      <td>0.23</td>
    </tr>
  </tbody>
</table>


## `dataCoord.segment.assignmentExpiration`

<table id="dataCoord.segment.maxSize">
  <thead>
    <tr>
## `dataCoord.compaction.enableAutoCompaction`

<table id="dataCoord.compaction.enableAutoCompaction">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>控制是否启用自动段压缩的开关值，数据协调器在后台定位并合并可压缩段。</li>
        <li>此配置仅在<code>dataCoord.enableCompaction</code>设置为<code>true</code>时生效。</li>
      </td>
      <td>true</td>
    </tr>
  </tbody>
</table>


## `dataCoord.gc.interval`

<table id="dataCoord.gc.interval">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>数据协调器执行垃圾回收的间隔。</li>
        <li>单位：秒</li>
        <li>此配置仅在<code>dataCoord.enableGarbageCollection</code>设置为<code>true</code>时生效。</li>
      </td>
      <td>3600</td>
    </tr>
  </tbody>
</table>

## `dataCoord.gc.missingTolerance`

<table id="dataCoord.gc.missingTolerance">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>未记录的二进制日志（binlog）文件的保留持续时间。</li>
        <li>为此参数设置一个相当大的值可避免错误地删除缺少元数据的新创建的binlog文件。</li>
        <li>单位：秒</li>
        <li>此配置仅在<code>dataCoord.enableGarbageCollection</code>设置为<code>true</code>时生效。</li>
      </td>
      <td>86400</td>
    </tr>
  </tbody>
</table>

## `dataCoord.gc.dropTolerance`

<table id="dataCoord.gc.dropTolerance">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>已删除段的binlog文件在清除之前的保留持续时间。</li>
        <li>单位：秒</li>
        <li>此配置仅在<code>dataCoord.enableGarbageCollection</code>设置为<code>true</code>时生效。</li>
      </td>
      <td>86400</td>
    </tr>
  </tbody>
</table>