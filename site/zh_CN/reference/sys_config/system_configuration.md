---
id: system_configuration.md
related_key: configure
group: system_configuration.md
summary: 了解 Milvus 的系统配置。
title: Milvus 系统配置清单
---

# Milvus 系统配置清单

本主题介绍了 Milvus 系统配置的一般部分。

Milvus 维护了大量参数来配置系统。每个配置都有一个默认值，可以直接使用。您可以灵活修改这些参数，以便 Milvus 能够更好地为您的应用提供服务。更多信息请参见[配置 Milvus](configure-docker.md)。

<div class="alert note">
在当前版本中，所有参数只有在 Milvus 启动时配置后才会生效。
</div>

## 部分

为了方便维护，Milvus 根据其组件、依赖关系和一般用途将其配置分为 17 个部分。

### `etcd`

etcd 是支持 Milvus 元数据存储和访问的元数据引擎。

在此部分下，您可以配置 etcd 端点、相关键前缀等。

有关此部分下每个参数的详细描述，请参见[与 etcd 相关的配置](configure_etcd.md)。

### `minio`

Milvus 支持 MinIO 和 Amazon S3 作为数据持久化的存储引擎，用于插入日志文件和索引文件。虽然 MinIO 是 S3 兼容性的事实标准，但您可以直接在 MinIO 部分下配置 S3 参数。

在此部分下，您可以配置 MinIO 或 S3 地址、相关访问密钥等。

有关此部分下每个参数的详细描述，请参见[与 MinIO 相关的配置](configure_minio.md)。

### `pulsar`

Pulsar 是支持 Milvus 集群可靠存储和消息流发布/订阅的底层引擎。

在此部分下，您可以配置 Pulsar 地址、消息大小等。

有关此部分下每个参数的详细描述，请参见[Pulsar 相关的配置](configure_pulsar.md)。

### `rocksmq`

RocksMQ 是支持 Milvus 独立运行的可靠存储和消息流发布/订阅的底层引擎，它是基于 RocksDB 实现的。

在此部分下，您可以配置消息大小、保留时间和大小等。

有关此部分下每个参数的详细描述，请参见[RocksMQ 相关的配置](configure_rocksmq.md)。

### `nats`

NATS 是一种面向消息的中间件，允许应用程序和服务之间以消息的形式交换数据。Milvus 使用 NATS 作为可靠存储和消息流发布/订阅的底层引擎。您可以将其用作 RocksMQ 的替代方案。

在此部分下，您可以配置 NATS 服务器、监控属性、保留时间和大小等。

有关此部分下每个参数的详细描述，请参见[NATS 相关的配置](configure_nats.md)。

### `kafka`
Apache Kafka 是一个开源的分布式事件流平台，用于高性能数据管道、流式分析、数据集成和关键应用程序。它作为可靠存储和消息流发布/订阅的 RocksMQ 和 Pulsar 的替代方案。

详细了解此部分下每个参数的描述，请参阅[Kafka相关配置](configure_kafka.md)。

### `rootCoord`

根协调器（root coord）处理数据定义语言（DDL）和数据控制语言（DCL）请求，管理 TSO（时间戳 Oracle）并发布时间标记消息。

在此部分下，您可以配置根协调器地址、索引构建阈值等。

详细了解此部分下每个参数的描述，请参阅[根协调器相关配置](configure_rootcoord.md)。

### `proxy`

代理是系统的访问层和用户端点。它验证客户端请求并减少返回结果。

在此部分下，您可以配置代理端口、系统限制等。

详细了解此部分下每个参数的描述，请参阅[代理相关配置](configure_proxy.md)。

### `queryCoord`

查询协调器（query coord）管理查询节点的拓扑和负载平衡，并将操作从增长段移交到封存段。

在此部分下，您可以配置查询协调器地址、自动移交、自动负载平衡等。

详细了解此部分下每个参数的描述，请参阅[查询协调器相关配置](configure_querycoord.md)。

### `queryNode`

查询节点在增量和历史数据上执行向量和标量数据的混合搜索。

在此部分下，您可以配置查询节点端口、优雅时间等。

详细了解此部分下每个参数的描述，请参阅[查询节点相关配置](configure_querynode.md)。

### `indexNode`

索引节点为向量构建索引。

在此部分下，您可以配置索引节点端口等。

详细了解此部分下每个参数的描述，请参阅[索引节点相关配置](configure_indexnode.md)。

### `dataCoord`

数据协调器（data coord）管理数据节点的拓扑，维护元数据，并触发刷新、压缩和其他后台数据操作。

在此部分下，您可以配置数据协调器地址、段设置、压缩、垃圾回收等。

详细了解此部分下每个参数的描述，请参阅[数据协调器相关配置](configure_datacoord.md)。

### `dataNode`

数据节点通过订阅日志代理检索增量日志数据，处理变更请求，并将日志数据打包成日志快照并存储在对象存储中。

在此部分下，您可以配置数据节点端口等。

详细了解此部分下每个参数的描述，请参阅[数据节点相关配置](configure_datanode.md)。

### `localStorage`
Milvus 在搜索或查询时将向本地存储中存储向量数据，以避免重复访问 MinIO 或 S3 服务。

在这个部分，您可以启用本地存储，并配置路径等。

请参阅[与本地存储相关的配置](configure_localstorage.md)以获取本部分下每个参数的详细描述。

### `log`

使用 Milvus 会生成一系列日志。默认情况下，Milvus 使用日志记录调试或更高级别的信息，输出到标准输出（stdout）和标准错误（stderr）。

在这个部分，您可以配置系统日志输出。

请参阅[与日志相关的配置](configure_log.md)以获取本部分下每个参数的详细描述。

### `msgChannel`

在这个部分，您可以配置消息通道名称前缀和组件订阅名称前缀。

请参阅[与消息通道相关的配置](configure_messagechannel.md)以获取本部分下每个参数的详细描述。

### `common`

在这个部分，您可以配置分区和索引的默认名称，以及 Milvus 的时间旅行（数据保留）跨度。

请参阅[通用配置](configure_common.md)以获取本部分下每个参数的详细描述。

### `knowhere`

[Knowhere](https://github.com/milvus-io/milvus/blob/master/docs/design_docs/knowhere_design.md) 是 Milvus 的搜索引擎。

在这个部分，您可以配置系统的默认 SIMD 指令集类型。

请参阅[与 Knowhere 相关的配置](configure_knowhere.md)以获取本部分下每个参数的详细描述。

## 经常使用的参数

以下列出了一些根据修改目的分类的经常使用的参数。

### 性能调优

以下参数控制影响索引创建和向量相似性搜索性能的系统行为。

<ul>
    <li><a href="configure_querynode.md#queryNode.gracefulTime"><code>queryNode.gracefulTime</code></a></li>
    <li><a href="configure_rootcoord.md#rootCoord.minSegmentSizeToEnableIndex"><code>rootCoord.minSegmentSizeToEnableIndex</code></a></li>
    <li><a href="configure_datacoord.md#dataCoord.segment.maxSize"><code>dataCoord.segment.maxSize</code></a></li>
    <li><a href="configure_datacoord.md#dataCoord.segment.sealProportion"><code>dataCoord.segment.sealProportion</code></a></li>
    <li><a href="configure_datanode.md#dataNode.flush.insertBufSize"><code>dataNode.flush.insertBufSize</code></a></li>
    <li><a href="configure_querycoord.md#queryCoord.autoHandoff"><code>queryCoord.autoHandoff</code></a></li>
    <li><a href="configure_querycoord.md#queryCoord.autoBalance"><code>queryCoord.autoBalance</code></a></li>
    <li><a href="configure_localstorage.md#localStorage.enabled"><code>localStorage.enabled</code></a></li>
</ul>

### 数据和元数据保留

以下参数控制数据和元数据的保留。

<ul>
<ul>
    <li><a href="configure_common.md#common.retentionDuration"><code>common.retentionDuration</code></a></li>
    <li><a href="configure_rocksmq.md#rocksmq.retentionTimeInMinutes"><code>rocksmq.retentionTimeInMinutes</code></a></li>
    <li><a href="configure_datacoord.md#dataCoord.enableCompaction"><code>dataCoord.enableCompaction</code></a></li>
    <li><a href="configure_datacoord.md#dataCoord.enableGarbageCollection"><code>dataCoord.enableGarbageCollection</code></a></li>
    <li><a href="configure_datacoord.md#dataCoord.gc.dropTolerance"><code>dataCoord.gc.dropTolerance</code></a></li>
</ul>


### 管理

以下参数控制日志输出和对象存储访问。

<ul>
    <li><a href="configure_log.md#log.level"><code>log.level</code></a></li>
    <li><a href="configure_log.md#log.file.rootPath"><code>log.file.rootPath</code></a></li>
    <li><a href="configure_log.md#log.file.maxAge"><code>log.file.maxAge</code></a></li>
    <li><a href="configure_minio.md#minio.accessKeyID"><code>minio.accessKeyID</code></a></li>
    <li><a href="configure_minio.md#minio.secretAccessKey"><code>minio.secretAccessKey</code></a></li>
</ul>

### 配额和限制

<ul>
    <li><a href="configure_quota_limits.md#quotaAndLimitsddlenabled"><code>quotaAndLimits.ddl.enabled</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsddlcollectionRate"><code>quotaAndLimits.ddl.collectionRate</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsddlpartitionRate"><code>quotaAndLimits.ddl.partitionRate</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsindexRateenabled"><code>quotaAndLimits.indexRate.enabled</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsindexRatemax"><code>quotaAndLimits.indexRate.max</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsflushRateenabled"><code>quotaAndLimits.flushRate.enabled</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsflushmax"><code>quotaAndLimits.flush.max</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitscompationenabled"><code>quotaAndLimits.compation.enabled</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitscompactionmax"><code>quotaAndLimits.compaction.max</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsdmlenabled"><code>quotaAndLimits.dml.enabled</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsdmlinsertRatemax"><code>quotaAndLimits.dml.insertRate.max</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsdmldeleteRatemax"><code>quotaAndLimits.dml.deleteRate.max</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsdqlenabled"><code>quotaAndLimits.dql.enabled</code></a></li>
    <li><a href="configure_quota_limits.md#quotaAndLimitsdqlsearchRatemax"><code>quotaAndLimits.dql.searchRate.max</code></a></li>
</ul>
<ul>
<li><a href="configure_quota_limits.md#quotaAndLimitsdqlqueryRatemax"><code>quotaAndLimits.dql.queryRate.max</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingttProtectionenabled"><code>quotaAndLimits.limitWriting.ttProtection.enabled</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingttProtectionmaxTimeTickDelay"><code>quotaAndLimits.limitWriting.ttProtection.maxTimeTickDelay</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectionenabled"><code>quotaAndLimits.limitWriting.memProtection.enabled</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectiondataNodeMemoryLowWaterLevel"><code>quotaAndLimits.limitWriting.memProtection.dataNodeMemoryLowWaterLevel</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectionqueryNodeMemoryLowWaterLevel"><code>quotaAndLimits.limitWriting.memProtection.queryNodeMemoryLowWaterLevel</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectiondataNodeMemoryHighWaterLevel"><code>quotaAndLimits.limitWriting.memProtection.dataNodeMemoryHighWaterLevel</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectionqueryNodeMemoryHighWaterLevel"><code>quotaAndLimits.limitWriting.memProtection.queryNodeMemoryHighWaterLevel</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingdiskProtectionenabled"><code>quotaAndLimits.limitWriting.diskProtection.enabled</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingdiskProtectiondiskQuota"><code>quotaAndLimits.limitWriting.diskProtection.diskQuota</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingforceDeny"><code>quotaAndLimits.limitWriting.forceDeny</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitReadingqueueProtectionenabled"><code>quotaAndLimits.limitReading.queueProtection.enabled</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitReadingqueueProtectionnqInQueueThreshold"><code>quotaAndLimits.limitReading.queueProtection.nqInQueueThreshold</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitReadingqueueProtectionqueueLatencyThreshold"><code>quotaAndLimits.limitReading.queueProtection.queueLatencyThreshold</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitReadingresultProtectionenabled"><code>quotaAndLimits.limitReading.resultProtection.enabled</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitReadingresultProtectionmaxReadResultRate"><code>quotaAndLimits.limitReading.resultProtection.maxReadResultRate</code></a></li>
<li><a href="configure_quota_limits.md#quotaAndLimitslimitReadingforceDeny"><code>quotaAndLimits.limitReading.forceDeny</code></a></li>
</ul>

## 接下来是什么？
- 在安装之前，学习如何[配置 Milvus](configure-docker.md)。

- 了解更多关于 Milvus 安装的信息：
  - [安装独立版 Milvus](install_standalone-docker.md)