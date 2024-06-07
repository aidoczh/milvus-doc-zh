---
id: configure_operator.md
label: Milvus Operator
related_key: Milvus Operator
summary: 学习如何使用 Milvus Operator 配置 Milvus。
title: 使用 Milvus Operator 配置 Milvus
---

# 使用 Milvus Operator 配置 Milvus

在生产环境中，您需要根据机器类型和工作负载为 Milvus 集群分配资源。您可以在部署期间进行配置，也可以在集群运行时更新配置。

本主题介绍了如何在安装 Milvus 时使用 Milvus Operator 配置 Milvus 集群。

本主题假定您已部署了 Milvus Operator。有关更多信息，请参阅[部署 Milvus Operator](install_cluster-milvusoperator.md)。

使用 Milvus Operator 配置 Milvus 集群包括：
- 全局资源配置
- 私有资源配置

<div class="alert note">
私有资源配置将覆盖全局资源配置。如果您在全局配置资源的同时指定某个组件的私有资源，该组件将优先考虑并响应私有配置。
</div>

## 配置全局资源

使用 Milvus Operator 启动 Milvus 集群时，您需要指定一个配置文件。这里的示例使用默认配置文件。

```yaml
kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_cluster_default.yaml
```

配置文件的详细信息如下：

```yaml
apiVersion: milvus.io/v1beta1
kind: Milvus
metadata:
  name: my-release
  labels:
    app: milvus
spec:
  mode: cluster
  dependencies: {}
  components: {}
  config: {}
```

字段 `spec.components` 包括所有 Milvus 组件的全局和私有资源配置。以下是用于配置全局资源的四个常用字段。
- `image`：所使用的 Milvus Docker 镜像。
- `resources`：分配给每个组件的计算资源。
- `tolerations` 和 `nodeSelector`：K8s 集群中每个 Milvus 组件的调度规则。有关更多信息，请参阅[tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) 和 [nodeSelector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)。
- `env`：环境变量。

如果您想配置更多字段，请参阅[此处的文档](https://pkg.go.dev/github.com/zilliztech/milvus-operator/apis/milvus.io/v1beta1#ComponentSpec)。

要为 Milvus 集群配置全局资源，请创建一个 `milvuscluster_resource.yaml` 文件。

### 示例

以下示例为 Milvus 集群配置全局资源。
```
apiVersion: milvus.io/v1beta1
kind: Milvus
metadata:
  name: my-release
  labels:
    app: milvus
spec:
  mode: cluster
  components:
    image: milvusdb/milvus:v2.1.0
    nodeSelector: {}
    tolerations: {}
    env: {}
    resources:
      limits:
        cpu: '4'
        memory: 8Gi
      requests:
        cpu: 200m
        memory: 512Mi
```
运行以下命令以应用新配置：

```bash
kubectl apply -f milvuscluster_resource.yaml
```

<div class="alert note">
如果在 K8s 集群中存在名为<code>my-release</code>的 Milvus 集群，集群资源将根据配置文件进行更新。否则，将创建一个新的 Milvus 集群。
</div>

## 配置私有资源

在 Milvus 2.0 中，Milvus 集群包括七个组件：代理（proxy）、根协调器（root coord）、数据协调器（data coord）、查询协调器（query coord）、索引节点（index node）、数据节点（data node）和查询节点（query node）。然而，随着 Milvus 2.1.0 的发布，新增了一个名为混合协调器（mix coord）的组件。混合协调器包含所有协调器组件。因此，启动混合协调器意味着您无需安装和启动其他协调器，包括根协调器、数据协调器和查询协调器。

用于配置每个组件的常见字段包括：
- `replica`：每个组件的副本数量。
- `port`：每个组件的监听端口号。
- 全局资源配置中常用的四个字段：`image`、`env`、`nodeSelector`、`tolerations`、`resources`（见上文）。有关更多可配置字段，请单击[此文档](https://pkg.go.dev/github.com/zilliztech/milvus-operator/apis/milvus.io/v1beta1#MilvusComponents)中的每个组件。

<div class="alert note">
此外，在配置代理时，还有一个额外的字段称为`serviceType`。该字段定义了 Milvus 在 K8s 集群中提供的服务类型。
</div>

要为特定组件配置资源，请首先在`spec.componets`下的字段中添加组件名称，然后配置其私有资源。

<div class="filter">
<a href="#component">组件或依赖关系</a> <a href="#purpose">配置目的</a> 
</div>

<div class="filter-component table-wrapper">

<table id="component">
<thead>
  <tr>
    <th>依赖关系</th>
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
            <li><a href="configure_rootcoord.md">根协调器</a></li>
            <li><a href="configure_proxy.md">代理</a></li>
            <li><a href="configure_querycoord.md">查询协调器</a></li>
            <li><a href="configure_querynode.md">查询节点</a></li>
            <li><a href="configure_indexcoord.md">索引协调器</a></li>
            <li><a href="configure_indexnode.md">索引节点</a></li>
            <li><a href="configure_datacoord.md">数据协调器</a></li>
            <li><a href="configure_datanode.md">数据节点</a></li>
            <li><a href="configure_localstorage.md">本地存储</a></li>
            <li><a href="configure_log.md">日志</a></li>
            <li><a href="configure_messagechannel.md">消息通道</a></li>
```html
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
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitsmaxCollectionNumPerDB"><code>quotaAndLimits.limits.maxCollectionNumPerDB</code></a></li>
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
            <li><a href="configure_quota_limits.md#quotaAndLimitsdmlinsertRatecollectionmax"><code>quotaAndLimits.dml.insertRate.collection.max</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitsdmldeleteRatemax"><code>quotaAndLimits.dml.deleteRate.max</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitsdmldeleteRatecollectionmax"><code>quotaAndLimits.dml.deleteRate.collection.max</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitsdqlenabled"><code>quotaAndLimits.dql.enabled</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitsdqlsearchRatemax"><code>quotaAndLimits.dql.searchRate.max</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitsdqlsearchRatecollectionmax"><code>quotaAndLimits.dql.searchRate.collection.max</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitsdqlqueryRatemax"><code>quotaAndLimits.dql.queryRate.max</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitsdqlqueryRatecollectionmax"><code>quotaAndLimits.dql.queryRate.collection.max</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingttProtectionenabled"><code>quotaAndLimits.limitWriting.ttProtection.enabled</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingttProtectionmaxTimeTickDelay"><code>quotaAndLimits.limitWriting.ttProtection.maxTimeTickDelay</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectionenabled"><code>quotaAndLimits.limitWriting.memProtection.enabled</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectiondataNodeMemoryLowWaterLevel"><code>quotaAndLimits.limitWriting.memProtection.dataNodeMemoryLowWaterLevel</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectionqueryNodeMemoryLowWaterLevel"><code>quotaAndLimits.limitWriting.memProtection.queryNodeMemoryLowWaterLevel</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectiondataNodeMemoryHighWaterLevel"><code>quotaAndLimits.limitWriting.memProtection.dataNodeMemoryHighWaterLevel</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingmemProtectionqueryNodeMemoryHighWaterLevel"><code>quotaAndLimits.limitWriting.memProtection.queryNodeMemoryHighWaterLevel</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingdiskProtectionenabled"><code>quotaAndLimits.limitWriting.diskProtection.enabled</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingdiskProtectiondiskQuota"><code>quotaAndLimits.limitWriting.diskProtection.diskQuota</code></a></li>
            <li><a href="configure_quota_limits.md#quotaAndLimitslimitWritingdiskProtectiondiskQuotaPerCollection"><code>quotaAndLimits.limitWriting.diskProtection.diskQuotaPerCollection</code></a></li>
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


下面的示例配置了`milvuscluster.yaml`文件中代理和数据节点的副本和计算资源。

```
apiVersion: milvus.io/v1beta1
kind: Milvus
metadata:
  name: my-release
  labels:
    app: milvus
spec:
  mode: cluster
  components:
    image: milvusdb/milvus:v2.1.0
    resources:
      limits:
        cpu: '4'
        memory: 8Gi
      requests:
        cpu: 200m
        memory: 512Mi
    rootCoord: 
      replicas: 1
      port: 8080
      resources:
        limits:
          cpu: '6'
          memory: '10Gi'
    dataCoord: {}
    queryCoord: {}
    indexCoord: {}
    dataNode: {}
    indexNode: {}
    queryNode: {}
    proxy:
      replicas: 1
      serviceType: ClusterIP
      resources:
        limits:
          cpu: '2'
          memory: 4Gi
        requests:
          cpu: 100m
          memory: 128Mi
  config: {}
  dependencies: {}
```

<div class="alert note">
这个示例不仅配置了全局资源，还为根协调器和代理配置了私有计算资源。使用此配置文件启动 Milvus 集群时，私有资源配置将应用于根协调器和代理，而其余组件将遵循全局资源配置。
</div>

运行以下命令应用新配置：

```
kubectl apply -f milvuscluster.yaml
```


## 接下来的步骤

- 了解如何使用 Milvus Operator 管理以下 Milvus 依赖项：
  - [使用 Milvus Operator 配置对象存储](object_storage_operator.md)
  - [使用 Milvus Operator 配置元数据存储](meta_storage_operator.md)
  - [使用 Milvus Operator 配置消息存储](message_storage_operator.md)