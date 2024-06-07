---
id: configure_quota_limits.md
related_key: configure
group: system_configuration.md
summary: 学习如何配置配额和限制。
title: 配额和限制相关配置
---

# 配额和限制相关配置

本主题介绍了Milvus中与配额和限制相关的配置项。

其中一些配置项用于设置Milvus的阈值，以主动限制与集合、分区、索引等相关的DDL/DML/DQL请求。

另一些配置项用于设置背压信号，强制Milvus降低DDL/DML/DQL请求的速率。

## `quotaAndLimits.limits.maxCollectionNumPerDB`

<table id="quotaAndLimits.ddl.enabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>每个数据库中的最大集合数。</td>
      <td>64</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.ddl.enabled`

<table id="quotaAndLimits.ddl.enabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>DDL请求节流是否启用。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.ddl.collectionRate`

<table id="quotaAndLimits.ddl.collectionRate">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒钟与集合相关的DDL请求的最大数量。</li>
        <li>将此项设置为<code>10</code>表示Milvus每秒钟处理不超过10个与集合相关的DDL请求，包括集合创建请求、集合删除请求、集合加载请求和集合释放请求。</li>
        <li>要使用此设置，请同时将<code>quotaAndLimits.ddl.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.ddl.partitionRate`

<table id="quotaAndLimits.ddl.partitionRate">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒钟与分区相关的DDL请求的最大数量。</li>
        <li>将此项设置为<code>10</code>表示Milvus每秒钟处理不超过10个与分区相关的请求，包括分区创建请求、分区删除请求、分区加载请求和分区释放请求。</li>
        <li>要使用此设置，请同时将<code>quotaAndLimits.ddl.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.indexRate.enabled`

<table id="quotaAndLimits.indexRate.enabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
```markdown
<th class="width20">默认值</th> 
</tr>
</thead>
<tbody>
<tr>
<td>是否启用与索引相关的请求节流。</td>
<td>False</td>
</tr>
</tbody>
</table>

## `quotaAndLimits.indexRate.max`

<table id="quotaAndLimits.indexRate.max">
<thead>
<tr>
<th class="width80">描述</th>
<th class="width20">默认值</th> 
</tr>
</thead>
<tbody>
<tr>
<td>
<li>每秒钟与索引相关的请求的最大数量。</li>
<li>将此项设置为<code>10</code>表示Milvus每秒钟处理不超过10个与分区相关的请求，包括索引创建请求和索引删除请求。</li>
<li>要使用此设置，请同时将<code>quotaAndLimits.indexRate.enabled</code>设置为<code>true</code>。</li>
</td>
<td>∞</td>
</tr>
</tbody>
</table>

## `quotaAndLimits.flushRate.enabled`

<table id="quotaAndLimits.flushRate.enabled">
<thead>
<tr>
<th class="width80">描述</th>
<th class="width20">默认值</th> 
</tr>
</thead>
<tbody>
<tr>
<td>是否启用刷新请求节流。</td>
<td>False</td>
</tr>
</tbody>
</table>

## `quotaAndLimits.flush.max`

<table id="quotaAndLimits.flush.max">
<thead>
<tr>
<th class="width80">描述</th>
<th class="width20">默认值</th> 
</tr>
</thead>
<tbody>
<tr>
<td>
<li>每秒钟刷新请求的最大数量。</li>
<li>将此项设置为<code>10</code>表示Milvus每秒钟处理不超过10个刷新请求。</li>
<li>要使用此设置，请同时将<code>quotaAndLimits.flushRate.enabled</code>设置为<code>true</code>。</li>
</td>
<td>∞</td>
</tr>
</tbody>
</table>

## `quotaAndLimits.compaction.enabled`

<table id="quotaAndLimits.compaction.enabled">
<thead>
<tr>
<th class="width80">描述</th>
<th class="width20">默认值</th> 
</tr>
</thead>
<tbody>
<tr>
<td>是否启用压缩请求节流。</td>
<td>False</td>
</tr>
</tbody>
</table>

## `quotaAndLimits.compaction.max`

<table id="quotaAndLimits.compaction.max">
<thead>
<tr>
<th class="width80">描述</th>
<th class="width20">默认值</th> 
</tr>
</thead>
<tbody>
<tr>
<td>
<li>每秒钟手动压缩请求的最大数量。</li>
<li>将此项设置为<code>10</code>表示Milvus每秒钟处理不超过10个手动压缩请求。</li>
<li>要使用此设置，请同时将<code>quotaAndLimits.compaction.enabled</code>设置为<code>true</code>。</li>
</td>
<td>∞</td>
</tr>
</tbody>
</table>

## `quotaAndLimits.dml.enabled`

<table id="quotaAndLimits.dml.enabled">
<thead>
<tr>
<th class="width80">描述</th>
```
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>是否启用 DML 请求节流。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.dml.insertRate.max`

<table id="quotaAndLimits.dml.insertRate.max">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒最高数据插入速率。</li>
        <li>将此项设置为 <code>5</code> 表示 Milvus 仅允许以 5 MB/s 的速率进行数据插入。</li>
        <li>要使用此设置，同时将 <code>quotaAndLimits.dml.enabled</code> 设置为 <code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.dml.insertRate.collection.max`

<table id="quotaAndLimits.dml.insertRate.collection.max">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒每个集合的最高数据插入速率。</li>
        <li>将此项设置为 <code>5</code> 表示 Milvus 仅允许以 5 MB/s 的速率向任何集合插入数据。</li>
        <li>要使用此设置，同时将 <code>quotaAndLimits.dml.enabled</code> 设置为 <code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.dml.deleteRate.max`

<table id="quotaAndLimits.dml.deleteRate.max">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒最高数据删除速率。</li>
        <li>将此项设置为 <code>0.1</code> 表示 Milvus 仅允许以 0.1 MB/s 的速率进行数据删除。</li>
        <li>要使用此设置，同时将 <code>quotaAndLimits.dml.enabled</code> 设置为 <code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.dml.deleteRate.collection.max`

<table id="quotaAndLimits.dml.deleteRate.collection.max">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒每个集合的最高数据删除速率。</li>
        <li>将此项设置为 <code>0.1</code> 表示 Milvus 仅允许以 0.1 MB/s 的速率从任何集合删除数据。</li>
        <li>要使用此设置，同时将 <code>quotaAndLimits.dml.enabled</code> 设置为 <code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.dql.enabled`

<table id="quotaAndLimits.dql.enabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
<table>
      <td>是否启用了 DQL 请求节流。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.dql.searchRate.max`

<table id="quotaAndLimits.dql.searchRate.max">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒搜索的最大向量数。</li>
        <li>将此项设置为<code>100</code>表示 Milvus 每秒只允许搜索 100 个向量，无论这 100 个向量是在一次搜索中还是分散在多次搜索中。</li>
        <li>要使用此设置，同时将<code>quotaAndLimits.dql.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.dql.searchRate.collection.max`

<table id="quotaAndLimits.dql.searchRate.collection.max">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒每个集合搜索的最大向量数。</li>
        <li>将此项设置为<code>100</code>表示 Milvus 每秒每个集合只允许搜索 100 个向量，无论这 100 个向量是在一次搜索中还是分散在多次搜索中。</li>
        <li>要使用此设置，同时将<code>quotaAndLimits.dql.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.dql.queryRate.max`

<table id="quotaAndLimits.dql.queryRate.max">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒查询的最大次数。</li>
        <li>将此项设置为<code>100</code>表示 Milvus 每秒只允许进行 100 次查询。</li>
        <li>要使用此设置，同时将<code>quotaAndLimits.dql.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.dql.queryRate.collection.max`

<table id="quotaAndLimits.dql.queryRate.collection.max">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每秒每个集合查询的最大次数。</li>
        <li>将此项设置为<code>100</code>表示 Milvus 每秒每个集合只允许进行 100 次查询。</li>
        <li>要使用此设置，同时将<code>quotaAndLimits.dql.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.ttProtection.enabled`

<table id="quotaAndLimits.limitWriting.ttProtection.enabled">
  <thead>
    <tr>
## `quotaAndLimits.limitWriting.ttProtection.maxTimeTickDelay`

<table id="quotaAndLimits.limitWriting.ttProtection.maxTimeTickDelay">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>最大时间滴答延迟。时间滴答延迟是RootCoord TSO与DataNodes和QueryNodes上所有流图的最小时间滴答之间的差异。</li>
        <li>将此项设置为<code>300</code>表示Milvus随着延迟增加而降低DML请求速率，并在延迟达到设定的最大值（以秒为单位）时丢弃所有DML请求。</li>
        <li>要使用此设置，同时将<code>quotaAndLimits.limitWriting.ttProtection.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>300</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.memProtection.enabled`

<table id="quotaAndLimits.limitWriting.memProtection.enabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>是否启用基于内存水位的背压。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.memProtection.dataNodeMemoryLowWaterLevel`

<table id="quotaAndLimits.limitWriting.memProtection.dataNodeMemoryLowWaterLevel">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>DataNodes上的低内存水位。内存水位是DataNodes上已使用内存与总内存之比。</li>
        <li>将此项设置为<code>0.85</code>表示Milvus随着DataNodes上的内存水位达到设定值而降低DML请求速率。</li>
        <li>要使用此设置，同时将<code>quotaAndLimits.limitWriting.memProtection.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>0.85</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.memProtection.queryNodeMemoryLowWaterLevel`

<table id="quotaAndLimits.limitWriting.memProtection.queryNodeMemoryLowWaterLevel">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>QueryNodes上的低内存水位。内存水位是QueryNodes上已使用内存与总内存之比。</li>
## `quotaAndLimits.limitWriting.memProtection.dataNodeMemoryHighWaterLevel`

<table id="quotaAndLimits.limitWriting.memProtection.dataNodeMemoryHighWaterLevel">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>DataNodes 上的高内存水位。内存水位是 DataNodes 上已使用内存与总内存的比率。</li>
        <li>将此项设置为 <code>0.95</code> 表示当 DataNodes 上的内存水位达到设定值时，Milvus 将放弃所有 DML 请求。</li>
        <li>要使用此设置，同时将 <code>quotaAndLimits.limitWriting.memProtection.enabled</code> 设置为 <code>true</code>。</li>
      </td>
      <td>0.95</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.memProtection.queryNodeMemoryHighWaterLevel`

<table id="quotaAndLimits.limitWriting.memProtection.queryNodeMemoryHighWaterLevel">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>QueryNodes 上的高内存水位。内存水位是 QueryNodes 上已使用内存与总内存的比率。</li>
        <li>将此项设置为 <code>0.95</code> 表示当 QueryNodes 上的内存水位达到设定值时，Milvus 将放弃所有 DML 请求。</li>
        <li>要使用此设置，同时将 <code>quotaAndLimits.limitWriting.memProtection.enabled</code> 设置为 <code>true</code>。</li>
      </td>
      <td>0.95</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.diskProtection.enabled`

<table id="quotaAndLimits.limitWriting.diskProtection.enabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>基于磁盘配额的背压是否已启用。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.diskProtection.diskQuota`

<table id="quotaAndLimits.limitWriting.diskProtection.diskQuota">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>分配给 binlog 的磁盘配额。</li>
        <li>将此项设置为 <code>8192</code> 表示当 binlog 的大小达到设定值时，Milvus 将放弃所有 DML 请求。</li>
## `quotaAndLimits.limitWriting.diskProtection.enabled`

<table id="quotaAndLimits.limitWriting.diskProtection.enabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>启用此设置，同时将 <code>quotaAndLimits.limitWriting.diskProtection.enabled</code> 设置为 <code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.diskProtection.diskQuotaPerCollection`

<table id="quotaAndLimits.limitWriting.diskProtection.diskQuotaPerCollection">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>每个集合分配给 binlog 的磁盘配额。</li>
        <li>将此项设置为 <code>8192</code> 表示当集合的 binlog 大小达到设定值时，Milvus 会丢弃该集合中的所有 DML 请求。</li>
        <li>启用此设置，同时将 <code>quotaAndLimits.limitWriting.diskProtection.enabled</code> 设置为 <code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.forceDeny`

<table id="quotaAndLimits.limitWriting.forceDeny">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>是否手动配置 Milvus 以丢弃所有 DML 请求。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitReading.queueProtection.enabled`

<table id="quotaAndLimits.limitReading.queueProtection.enabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>基于搜索和查询队列长度的背压是否已启用。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitReading.queueProtection.nqInQueueThreshold`

<table id="quotaAndLimits.limitReading.queueProtection.nqInQueueThreshold">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>搜索向量或查询的最大数量。请注意，包含多个搜索向量的搜索请求被视为多个搜索，而查询与仅包含一个搜索向量的搜索请求相同。</li>
        <li>将此项设置为 <code>10000</code> 表示当搜索和查询的数量达到设定的最大值时，Milvus 会降低 DQL 请求速率，当数量降至设定值以下时，背压得到解决。降低速率由 <code>quotaAndLimits.limitReading.coolOffSpeed</code> 决定。</li>
        <li>启用此设置，同时将 <code>quotaAndLimits.limitReading.queueProtection.enabled</code> 设置为 <code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitReading.queueProtection.queueLatencyThreshold`
<table id="quotaAndLimits.limitReading.queueProtection.queueLatencyThreshold">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>排队搜索和查询的平均延迟。请注意，包含多个搜索向量的搜索请求被视为多个搜索，而查询与仅包含一个搜索向量的搜索请求相同。</li>
        <li>将此项设置为<code>200</code>表示当平均延迟达到设定的最大值（以毫秒为单位）时，Milvus会降低 DQL 请求速率，并在数字降至设定值以下时解决背压。降低速率由<code>quotaAndLimits.limitReading.coolOffSpeed</code>确定。</li>
        <li>要使用此设置，同时将<code>quotaAndLimits.limitReading.queueProtection.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitReading.resultProtection.enabled`

<table id="quotaAndLimits.limitReading.resultProtection.enabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>基于查询结果速率的背压是否已启用。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitReading.resultProtection.maxReadResultRate`

<table id="quotaAndLimits.limitReading.resultProtection.maxReadResultRate">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>返回给客户端的数据速率。</li>
        <li>将此项设置为<code>2</code>表示当数据速率达到设定的最大值（以 MB/s 为单位）时，Milvus会降低 DQL 请求速率，并在数字降至设定值以下时解决背压。降低速率由<code>quotaAndLimits.limitReading.coolOffSpeed</code>确定。</li>
        <li>要使用此设置，同时将<code>quotaAndLimits.limitReading.resultProtection.enabled</code>设置为<code>true</code>。</li>
      </td>
      <td>∞</td>
    </tr>
  </tbody>
</table>

## `quotaAndLimits.limitWriting.forceDeny`

<table id="quotaAndLimits.limitWriting.forceDeny">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>是否手动配置 Milvus 以丢弃所有 DQL 请求。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>