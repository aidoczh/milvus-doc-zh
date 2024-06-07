---
id: configure-docker.md
label: Docker Compose
related_key: configure
summary: 使用 Docker Compose 配置 Milvus。
title: 使用 Docker Compose 配置 Milvus
---

# 使用 Docker Compose 配置 Milvus

本主题描述如何使用 Docker Compose 配置 Milvus 组件及其第三方依赖项。

<div class="alert note">
在当前版本中，所有参数仅在 Milvus 重新启动后生效。
</div>

## 下载配置文件

[下载](https://raw.githubusercontent.com/milvus-io/milvus/v{{var.milvus_release_tag}}/configs/milvus.yaml) `milvus.yaml` 文件，或使用以下命令下载。

```
$ wget https://raw.githubusercontent.com/milvus-io/milvus/v{{var.milvus_release_tag}}/configs/milvus.yaml
```

## 修改配置文件

通过调整 `milvus.yaml` 中的相应参数，配置您的 Milvus 实例以适应您的应用场景。

查看以下链接，了解有关每个参数的更多信息。

按以下方式排序：

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
            <li><a href="configure_messagechannel.md">Message channel</a></li>
            <li><a href="configure_common.md">Common</a></li>
            <li><a href="configure_knowhere.md">Knowhere</a></li>
            <li><a href="configure_quota_limits.md">Quota and Limits</a></li>
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


## 下载安装文件

下载 Milvus 的安装文件 [standalone](https://github.com/milvus-io/milvus/releases/download/v{{var.milvus_release_tag}}/milvus-standalone-docker-compose.yml)，并将其保存为 `docker-compose.yml`。

您也可以直接运行以下命令。

```
# 对于 Milvus standalone
$ wget https://github.com/milvus-io/milvus/releases/download/v{{var.milvus_release_tag}}/milvus-standalone-docker-compose.yml -O docker-compose.yml
```

## 修改安装文件

在 `docker-compose.yml` 文件中，在每个 `milvus-standalone` 下添加一个 `volumes` 部分。

将本地路径映射到对应的 Docker 容器路径，以配置文件 `/milvus/configs/milvus.yaml` 为例，在所有 `volumes` 部分下。

```yaml
...
  standalone:
    container_name: milvus-standalone
    image: milvusdb/milvus:v2.2.13
    command: ["milvus", "run", "standalone"]
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    volumes:
      - /local/path/to/your/milvus.yaml:/milvus/configs/milvus.yaml   # 将本地路径映射到容器路径
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/milvus:/var/lib/milvus
    ports:
      - "19530:19530"
      - "9091:9091"
    depends_on:
      - "etcd"
      - "minio"
...
```

<div class="alert note">
数据根据默认配置存储在 <code>/volumes</code> 文件夹中，该配置位于 <code>docker-compose.yml</code> 中。要更改存储数据的文件夹，请编辑 <code>docker-compose.yml</code> 或运行 <code>$ export DOCKER_VOLUME_DIRECTORY=</code>。

## 启动 Milvus

完成修改配置文件和安装文件后，您可以启动 Milvus。

```
$ sudo docker compose up -d
```

## 下一步

- 学习如何使用 Docker Compose 或 Helm 管理以下 Milvus 依赖项：
  - [使用 Docker Compose 或 Helm 配置对象存储](deploy_s3.md)
  - [使用 Docker Compose 或 Helm 配置元数据存储](deploy_etcd.md)
  - [使用 Docker Compose 或 Helm 配置消息存储](deploy_pulsar.md)