---
id: configure_querycoord.md
related_key: configure
group: system_configuration.md
summary: 学习如何配置 Milvus 的查询协调器。
title: 查询协调器相关配置
---


# 查询协调器相关配置

本主题介绍了 Milvus 的查询协调器相关配置。

查询协调器（query coord）管理查询节点的拓扑结构和负载均衡，以及从增长段（growing segments）到封存段（sealed segments）的交接操作。

在本节中，您可以配置查询协调器地址、自动交接、自动负载均衡等。

## `queryCoord.address`

<table id="queryCoord.address">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>查询协调器的 TCP/IP 地址。</li>
        <li>如果将此参数设置为 <code>0.0.0.0</code>，则查询协调器监视所有 IPv4 地址。</li>
      </td>
      <td>localhost</td>
    </tr>
  </tbody>
</table>

## `queryCoord.port`

<table id="queryCoord.port">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>查询协调器的 TCP 端口。</td>
      <td>19531</td>
    </tr>
  </tbody>
</table>

## `queryCoord.activeStandby.enabled`

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
        查询协调器是否以主备模式工作。
      </td>
      <td>false</td>
    </tr>
  </tbody>
</table>

## `queryCoord.replicas`

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
        查询协调器 Pod 的数量。如果将 `queryCoord.activeStandby.enabled` 设置为 `true`，则此项为必填项。
      </td>
      <td>1</td>
    </tr>
  </tbody>
</table>

## `queryCoord.autoHandoff`

<table id="queryCoord.autoHandoff">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>控制是否在增长段达到封存阈值时自动用相应的索引封存段替换增长段的开关值。</li>
        <li>如果将此参数设置为 <code>false</code>，Milvus 将直接对增长段进行蛮力搜索。</li>
      </td>
      <td>true</td>
    </tr>
  </tbody>
</table>

## `queryCoord.autoBalance`

<table id="queryCoord.autoBalance">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        切换值以控制是否通过均匀分配段加载和释放操作来自动平衡查询节点之间的内存使用情况。
      </td>
      <td>true</td>
    </tr>
  </tbody>
</table>

## `queryCoord.overloadedMemoryThresholdPercentage`

<table id="queryCoord.overloadedMemoryThresholdPercentage">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        触发封存段平衡的查询节点内存使用阈值（以百分比表示）。
      </td>
      <td>90</td>
    </tr>
  </tbody>
</table>

## `queryCoord.balanceIntervalSeconds`

<table id="queryCoord.balanceIntervalSeconds">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>查询协调器在查询节点之间平衡内存使用情况的间隔。</li>
        <li>单位：秒</li>
      </td>
      <td>60</td>
    </tr>
  </tbody>
</table>


## `queryCoord.memoryUsageMaxDifferencePercentage`

<table id="queryCoord.memoryUsageMaxDifferencePercentage">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        触发封存段平衡的任意两个查询节点之间内存使用差异的阈值（以百分比表示）。
      </td>
      <td>30</td>
    </tr>
  </tbody>
</table>

## `queryCoord.grpc.serverMaxRecvSize`

<table id="queryCoord.grpc.serverMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>查询协调器可以接收的每个 RPC 请求的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>2147483647</td>
    </tr>
  </tbody>
</table>

## `queryCoord.grpc.serverMaxSendSize`

<table id="queryCoord.grpc.serverMaxSendSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在接收 RPC 请求时，查询协调器可以发送的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>2147483647</td>
    </tr>
  </tbody>
</table>


## `queryCoord.grpc.clientMaxRecvSize`

<table id="queryCoord.grpc.clientMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在发送 RPC 请求时，查询协调器可以接收的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>104857600</td>
    </tr>
  </tbody>
</table>

## `queryCoord.grpc.clientMaxSendSize`

<table id="queryCoord.grpc.clientMaxSendSize">
  <thead>
```markdown
<tr>
  <th class="width80">描述</th>
  <th class="width20">默认值</th> 
</tr>
</thead>
<tbody>
  <tr>
    <td>
      <li>查询协调器可以发送的每个 RPC 请求的最大大小。</li>
      <li>单位：字节</li>
    </td>
    <td>104857600</td>
  </tr>
</tbody>
</table>
```