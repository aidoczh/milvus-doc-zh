---
id: configure-helm.md
label: Helm
related_key: configure
summary: Configure Milvus with Helm Charts.
title: Configure Milvus with Helm Charts
---

# 使用 Helm Charts 配置 Milvus

本主题介绍如何使用 Helm Charts 配置 Milvus 组件及其第三方依赖项。

<div class="alert note">
在当前版本中，所有参数仅在 Milvus 重新启动后生效。
</div>

## 通过配置文件配置 Milvus

您可以使用配置文件 `values.yaml` 来配置 Milvus。

### 下载配置文件

直接[下载](https://raw.githubusercontent.com/milvus-io/milvus-helm/master/charts/milvus/values.yaml) `values.yaml` 文件，或者使用以下命令下载：

```
$ wget https://raw.githubusercontent.com/milvus-io/milvus-helm/master/charts/milvus/values.yaml
```

### 修改配置文件

通过调整 `values.yaml` 中的相应参数，配置您的 Milvus 实例以适应您的应用场景。

具体来说，在 `values.yaml` 中搜索 `extraConfigFiles`，并按照以下方式将您的配置放入此部分：

```yaml
# milvus.yaml 的额外配置
# 如果设置，此配置将合并到 milvus.yaml 中
# 请遵循 milvus.yaml 中的配置结构
# 在 https://github.com/milvus-io/milvus/blob/master/configs/milvus.yaml 查看
# 注意：此配置将是最高优先级，将覆盖镜像和 Helm Charts 中的配置
extraConfigFiles:
  user.yaml: |+
    #    例如设置查询节点的优雅时间
    #    queryNodes:
    #      gracefulTime: 10
```

查看以下链接以获取有关每个参数的更多信息。

按照排序：

<div class="filter">
<a href="#component">组件或依赖项</a> <a href="#purpose">配置目的</a> 
</div>

<div class="filter-component table-wrapper">

<table id="component">
<thead>
  <tr>
    <th>依赖项</th>
    <th>组件</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>
        <ul>
            <li><a href="configure_etcd.md">etcd</a></li>
            <li><a href="configure_minio.md">MinIO 或 S3</a></li>
            <li><a href="configure_pulsar.md">Pulsar</a></li>
            <li><a href="configure_rocksmq.md">RocksMQ</a></li>
        </ul>
    </td>
    <td>
        <ul>
            <li><a href="configure_rootcoord.md">Root coord</a></li>
            <li><a href="configure_proxy.md">Proxy</a></li>
            <li><a href="configure_querycoord.md">Query coord</a></li>
            <li><a href="configure_querynode.md">Query node</a></li>
            <li><a href="configure_indexcoord.md">Index coord</a></li>
            <li><a href="configure_indexnode.md">Index node</a></li>
            <li><a href="configure_datacoord.md">Data coord</a></li>
            <li><a href="configure_datanode.md">Data node</a></li>
            <li><a href="configure_localstorage.md">Local storage</a></li>
            <li><a href="configure_log.md">Log</a></li>
```markdown
<li><a href="configure_messagechannel.md">消息通道</a></li>
<li><a href="configure_common.md">通用设置</a></li>
<li><a href="configure_knowhere.md">未知领域</a></li>
<li><a href="configure_quota_limits.md">配额和限制</a></li>
</ul>
</td>
</tr>
</tbody>
</table>

</div>

<div class="filter-purpose table-wrapper">
<table id="purpose">
<thead>
  <tr>
    <th>Purpose</th>
    <th>Parameters</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Performance tuning</td>
    <td>
        <ul>
            <li><a href="configure_querynode.md#queryNodegracefulTime"><code>queryNode.gracefulTime</code></a></li>
            <li><a href="configure_rootcoord.md#rootCoordminSegmentSizeToEnableIndex"><code>rootCoord.minSegmentSizeToEnableIndex</code></a></li>
            <li><a href="configure_datacoord.md#dataCoordsegmentmaxSize"><code>dataCoord.segment.maxSize</code></a></li>
            <li><a href="configure_datacoord.md#dataCoordsegmentsealProportion"><code>dataCoord.segment.sealProportion</code></a></li>
            <li><a href="configure_datanode.md#dataNodeflushinsertBufSize"><code>dataNode.flush.insertBufSize</code></a></li>
            <li><a href="configure_querycoord.md#queryCoordautoHandoff"><code>queryCoord.autoHandoff</code></a></li>
            <li><a href="configure_querycoord.md#queryCoordautoBalance"><code>queryCoord.autoBalance</code></a></li>
            <li><a href="configure_localstorage.md#localStorageenabled"><code>localStorage.enabled</code></a></li>
        </ul>
    </td>
  </tr>
  <tr>
    <td>Data and meta</td>
    <td>
        <ul>
            <li><a href="configure_common.md#commonretentionDuration"><code>common.retentionDuration</code></a></li>
            <li><a href="configure_rocksmq.md#rocksmqretentionTimeInMinutes"><code>rocksmq.retentionTimeInMinutes</code></a></li>
            <li><a href="configure_datacoord.md#dataCoordenableCompaction"><code>dataCoord.enableCompaction</code></a></li>
            <li><a href="configure_datacoord.md#dataCoordenableGarbageCollection"><code>dataCoord.enableGarbageCollection</code></a></li>
            <li><a href="configure_datacoord.md#dataCoordgcdropTolerance"><code>dataCoord.gc.dropTolerance</code></a></li>
        </ul>
    </td>
  </tr>
  <tr>
    <td>Administration</td>
    <td>
        <ul>
            <li><a href="configure_log.md#loglevel"><code>log.level</code></a></li>
            <li><a href="configure_log.md#logfilerootPath"><code>log.file.rootPath</code></a></li>
            <li><a href="configure_log.md#logfilemaxAge"><code>log.file.maxAge</code></a></li>
            <li><a href="configure_minio.md#minioaccessKeyID"><code>minio.accessKeyID</code></a></li>
            <li><a href="configure_minio.md#miniosecretAccessKey"><code>minio.secretAccessKey</code></a></li>
        </ul>
    </td>
  </tr>
  <tr>
    <td>Quota and Limits</td>
    <td>
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
    </td>
  </tr>
</tbody>
</table>


</div>

对于专门针对 Kubernetes 安装的其他参数，请参阅[Milvus Helm Chart Configuration](https://github.com/milvus-io/milvus-helm/tree/master/charts/milvus#configuration)。

### 启动 Milvus

完成修改配置文件后，您可以使用该文件启动 Milvus。

```
$ helm upgrade my-release milvus/milvus -f values.yaml
```

## 通过命令行配置 Milvus

或者，您可以直接使用 Helm 命令升级 Milvus 配置。

### 检查可配置参数

在升级之前，您可以使用 Helm charts 检查可配置参数。

```
$ helm show values milvus/milvus
```

### 启动 Milvus

通过在升级命令中添加 `--values` 或 `--set` 来配置并启动 Milvus。

```
# 例如，禁用压缩的情况下升级 Milvus 集群
$ helm upgrade my-release milvus/milvus --set dataCoord.enableCompaction=false
```

## 下一步操作
- 如果您想学习如何监控 Milvus 服务并创建警报：
  - 参阅 [在 Kubernetes 上使用 Prometheus Operator 监控 Milvus](monitor.md)
  - 参阅 [在 Grafana 中可视化 Milvus 指标](visualize.md)。

- 如果您正在寻找如何分配资源的指导：
  - [在 Kubernetes 上分配资源](allocate.md#standalone)