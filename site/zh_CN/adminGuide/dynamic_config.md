---
id: dynamic_config.md
related_key: configure
summary: 了解 Milvus 的系统配置。
title: 在运行时配置 Milvus
---

# 在运行时配置 Milvus

Milvus 允许您在运行时更改部分配置。

## 开始之前

您需要确保：

- 已安装 Birdwatcher。详情请参阅 [安装 Birdwatcher](birdwatcher_install_guides.md)，
- 已安装 etcdctl。详情请参阅 [与 etcd 交互](https://etcd.io/docs/v3.5/dev-guide/interacting_v3/)，或
- 已安装其他 etcd 客户端，如 Python 客户端。

<div class="alert note">

- 本指南中的示例将 `proxy.minPasswordLength` 的值更改为 `8`。您可以将键替换为 [适用配置项](dynamic_config.md#适用配置项) 中列出的适用项。
- 本指南中的示例假定您的 Milvus 根路径为 `by-dev`。所有配置都列在路径 `by-dev/config` 下。Milvus 的根路径因安装方式而异。对于使用 Helm charts 安装的实例，默认根路径为 `by-dev`。如果您不知道根路径，请参阅 [连接到 etcd](birdwatcher_usage_guides.md#连接到-etcd)。

</div>

## 更改配置

在 Milvus 中，默认情况下 `proxy.minPasswordLength` 设置为 `6`。要更改此值，可以按照以下步骤操作：

```shell
$ etcdctl put by-dev/config/proxy/minPasswordLength 8
# 或
$ birdwatcher -olc "#connect --etcd 127.0.0.1:2379 --rootPath=by-dev,set config-etcd --key by-dev/config/proxy/minPasswordLength --value 8"
```

然后，您可以按如下方式检查配置：

```shell
$ etcdctl get by-dev/config/proxy/minPasswordLength
```

## 回滚配置

Milvus 还允许您在更改的值不再适用时回滚配置。

```shell
$ etcdctl del by-dev/config/proxy/minPasswordLength 
# 或 
$ birdwatcher -olc "#connect --etcd 127.0.0.1:2379 --rootPath=by-dev,remove config-etcd --key by-dev/config/proxy/minPasswordLength"
```

然后，您可以按如下方式检查配置：

```shell
$ etcdctl get by-dev/config/proxy/minPasswordLength
```

## 查看配置

除了查看特定配置项的值之外，您还可以列出所有配置项。

```shell
$ etcdctl get --prefix by-dev/config
# 或
$ birdwatcher -olc "#connect --etcd 127.0.0.1:2379 --rootPath=by-dev,show config-etcd"
```

要查看特定节点的配置：

```shell
Offline > connect --etcd ip:port 
Milvus(by-dev) > show session          # 列出所有带有服务器 ID 的节点
Milvus(by-dev) > visit querycoord 1    # 通过服务器 ID 访问节点
QueryCoord-1(ip:port) > configuration  # 列出节点的配置
```

## 适用配置项

目前，您可以在运行时更改以下配置项。

 | 配置项                                                      | 默认值       |
|-------------------------------------------------------------------------|---------------------|
| pulsar.maxMessageSize                                                   | 5242880             |
| common.retentionDuration                                                | 86400               |
| common.entityExpiration                                                 | -1                  |
| common.gracefulTime                                                     | 5000                |
| common.gracefulStopTimeout                                              | 30                  |
| quotaAndLimits.ddl.enabled                                              | FALSE               |
| quotaAndLimits.indexRate.enabled                                        | FALSE               |
| quotaAndLimits.flushRate.enabled                                        | FALSE               |
| quotaAndLimits.compactionRate.enabled                                   | FALSE               |
| quotaAndLimits.dml.enabled                                              | FALSE               |
| quotaAndLimits.dql.enabled                                              | FALSE               |
| quotaAndLimits.limits.collection.maxNum                                 | 64                  |
| quotaAndLimits.limitWriting.forceDeny                                   | FALSE               |
| quotaAndLimits.limitWriting.ttProtection.enabled                        | FALSE               |
| quotaAndLimits.limitWriting.ttProtection.maxTimeTickDelay               | 9223372036854775807 |
| quotaAndLimits.limitWriting.memProtection.enabled                       | TRUE                |
| quotaAndLimits.limitWriting.memProtection.dataNodeMemoryLowWaterLevel   | 0.85                |
| quotaAndLimits.limitWriting.memProtection.dataNodeMemoryHighWaterLevel  | 0.95                |
| quotaAndLimits.limitWriting.memProtection.queryNodeMemoryLowWaterLevel  | 0.85                |
| quotaAndLimits.limitWriting.memProtection.queryNodeMemoryHighWaterLevel | 0.95                |
| quotaAndLimits.limitWriting.diskProtection.enabled                      | TRUE                |
| quotaAndLimits.limitWriting.diskProtection.diskQuota                    | +INF                |
| quotaAndLimits.limitReading.forceDeny                                   | FALSE               |
| quotaAndLimits.limitReading.queueProtection.enabled                     | FALSE               |
| quotaAndLimits.limitReading.queueProtection.nqInQueueThreshold          | 9223372036854775807 |
| quotaAndLimits.limitReading.queueProtection.queueLatencyThreshold       | +INF                |
| quotaAndLimits.limitReading.resultProtection.enabled                    | FALSE               |
| quotaAndLimits.limitReading.resultProtection.maxReadResultRate          | +INF                |
| quotaAndLimits.limitReading.coolOffSpeed                                | 0.9                 |
| autoIndex.enable                                                        | FALSE               |
| autoIndex.params.build                                                  | ""                  |
| autoIndex.params.extra                                                  | ""                  |
| autoIndex.params.search                                                 | ""                  |
| proxy.maxNameLength                                                     | 255                 |
| proxy.maxUsernameLength                                                 | 32                  |
| proxy.minPasswordLength                                                 | 6                   |
| proxy.maxPasswordLength                                                 | 256                 |
| proxy.maxFieldNum                                                       | 64                  |
| proxy.maxShardNum                                                       | 256                 |
| proxy.maxDimension                                                      | 32768               |
| proxy.maxUserNum                                                        | 100                 |
| proxy.maxRoleNum                                                        | 10                  |
| queryNode.enableDisk                                                    | TRUE                |
| dataCoord.segment.diskSegmentMaxSize                                    | 2048                |
| dataCoord.compaction.enableAutoCompaction                               | TRUE                |

## 接下来

- 了解更多关于[系统配置](system_configuration.md)。
- 学习如何配置使用[Milvus Operator](configure_operator.md)、[Helm charts](configure-helm.md)和[Docker](configure-docker.md)安装的Milvus。