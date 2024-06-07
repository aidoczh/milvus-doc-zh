---
id: configure_rocksmq.md
related_key: configure
group: system_configuration.md
summary: 学习如何为 Milvus 独立版配置 RocksMQ。
title: RocksMQ相关配置
---

# RocksMQ相关配置

本主题介绍了 Milvus 中与 RocksMQ 相关的配置。

RocksMQ 是支持 Milvus 独立版可靠存储和消息流发布/订阅的底层引擎，它是基于 RocksDB 实现的。

在本节中，您可以配置消息大小、保留时间和大小等。

## `rocksmq.path`

<table id="rocksmq.path">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>用于 Milvus 在 RocksMQ 中存储数据的键前缀。</li>
        <li>注意：在使用 Milvus 一段时间后更改此参数将影响您访问旧数据。</li>
        <li>建议在首次启动 Milvus 之前更改此参数。</li>
        <li>如果已存在 etcd 服务，请为 Milvus 设置一个易于识别的根键前缀。</li>
      </td>
      <td>/var/lib/milvus/rdb_data</td>
    </tr>
  </tbody>
</table>

## `rocksmq.rocksmqPageSize`

<table id="rocksmq.rocksmqPageSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>RocksMQ 中每个页面中消息的最大大小。基于此参数，RocksMQ 中的消息将按批次进行检查和清除（在过期时）。</li>
        <li>单位：字节</li>
      </td>
      <td>2147483648</td>
    </tr>
  </tbody>
</table>

## `rocksmq.retentionTimeInMinutes`

<table id="rocksmq.retentionTimeInMinutes">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>RocksMQ 中已确认消息的最大保留时间。RocksMQ 中的已确认消息将保留指定时间后被清除。</li>
        <li>单位：分钟</li>
      </td>
      <td>10080</td>
    </tr>
  </tbody>
</table>

## `rocksmq.retentionSizeInMB`

<table id="rocksmq.retentionSizeInMB">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>RocksMQ 中每个主题中已确认消息的最大保留大小。如果超过此参数，每个主题中的已确认消息将被清除。</li>
        <li>单位：MB</li>
      </td>
      <td>8192</td>
    </tr>
  </tbody>
</table>

## `rocksmq.compactionInterval`

<table id="rocksmq.compactionInterval">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
## `rocksmq.lrucacheratio`

<table id="rocksmq.lrucacheratio">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>触发 RocksDB 压缩以删除已删除数据的时间间隔。</li>
        <li>单位：秒</li>
      </td>
      <td>86400</td>
    </tr>
    <tr>
      <td>
        <li>RocksDB 缓存内存比例。</li>
      </td>
      <td>0.06</td>
    </tr>
  </tbody>
</table>