---
id: birdwatcher_usage_guides.md
summary: 学习如何安装 Birdwatch 来调试 Milvus。
title: 使用 Birdwatcher
---

# 使用 Birdwatcher

本指南将指导您如何使用 Birdwatcher 检查 Milvus 的状态并实时配置。

## 启动 Birdwatcher

Birdwatcher 是一个命令行工具，您可以按照以下方式启动它：

```shell
./birdwatcher
```

然后您将看到以下提示：

```shell
Offline >
```

## 连接到 etcd

在执行其他操作之前，您需要使用 Birdwatcher 连接到 etcd。

- 使用默认设置连接

    ```shell
    Offline > connect
    Milvus(by-dev) >
    ```

- 从 Kubernetes pod 中的 Birdwatcher 连接

    如果您选择在 Kubernetes pod 中运行 Birdwatcher，您首先需要获取 etcd 的 IP 地址，操作如下：

    ```shell
    kubectl get pod my-release-etcd-0 -o 'jsonpath={.status.podIP}'
    ```

    然后访问 pod 的 shell。

    ```shell
    kubectl exec --stdin --tty birdwatcher-7f48547ddc-zcbxj -- /bin/sh
    ```

    最后，使用返回的 IP 地址连接到 etcd，操作如下：

    ```shell
    Offline > connect --etcd ${ETCD_IP_ADDR}:2379
    Milvus(by-dev)
    ```

- 使用不同的根路径连接

    如果您的 Milvus 的根路径与 `by-dev` 不同，并且出现有关不正确根路径的错误报告，您可以按以下方式连接到 etcd：

    ```shell
    Offline > connect --rootPath my-release
    Milvus(my-release) >
    ```

    如果您不知道您的 Milvus 的根路径，可以按以下方式连接到 etcd：

    ```shell
    Offline > connect --dry
    using dry mode, ignore rootPath and metaPath
    Etcd(127.0.0.1:2379) > find-milvus
    1 candidates found:
    my-release
    Etcd(127.0.0.1:2379) > use my-release
    Milvus(my-release) >
    ```

## 检查 Milvus 状态

您可以使用 `show` 命令来检查 Milvus 的状态。
```shell
Milvus(my-release) > show -h
用法：
   show [command]

可用命令：
  alias               列出别名元信息
  channel-watch       显示从数据协调器元存储中观察到的通道信息
  checkpoint          列出检查点集合的虚拟通道
  collection-history  显示集合变更历史
  collection-loaded   显示从查询协调器加载的集合信息
  collections         列出来自RootCoord的当前可用集合
  config-etcd         列出由etcd源设置的配置
  configurations      迭代所有在线组件并检查配置
  current-version     
  database            显示来自rootcoord元信息的数据库信息
  index               
  partition           列出提供的集合的分区
  querycoord-channel  显示来自querycoord集群的querynode信息
  querycoord-cluster  显示来自querycoord集群的querynode信息
  querycoord-task     显示来自querycoord的任务信息
  replica             列出来自QueryCoord的当前副本信息
  segment             显示来自数据协调器元存储的段信息
  segment-index       显示段索引信息
  segment-loaded      显示来自querycoordv1元信息的段信息
  segment-loaded-grpc 列出加载的段信息
  session             列出在线Milvus组件

标志：
  -h, --help   显示show的帮助信息

使用" show [command] --help"获取有关命令的更多信息。
```
### 会话列表

您可以按以下方式列出所有 etcd 会话：

```shell
Milvus(by-dev) > show session
Session:datacoord, ServerID: 3, Version: 2.2.11, Address: 10.244.0.8:13333
Session:datanode, ServerID: 6, Version: 2.2.11, Address: 10.244.0.8:21124
Session:indexcoord, ServerID: 4, Version: 2.2.11, Address: 10.244.0.8:31000
Session:indexnode, ServerID: 5, Version: 2.2.11, Address: 10.244.0.8:21121
Session:proxy, ServerID: 8, Version: 2.2.11, Address: 10.244.0.8:19529
Session:querycoord, ServerID: 7, Version: 2.2.11, Address: 10.244.0.8:19531
Session:querynode, ServerID: 2, Version: 2.2.11, Address: 10.244.0.8:21123
Session:rootcoord, ServerID: 1, Version: 2.2.11, Address: 10.244.0.8:53100
```

在命令输出中，您可以找到来自所有 Milvus 组件到 etcd 的会话。

### 检查数据库和集合

您可以列出所有数据库和集合。

- 列出数据库

    在命令输出中，您可以找到有关每个数据库的信息。

    ```shell
    Milvus(by-dev) > show database
    =============================
    ID: 1   Name: default
    TenantID:        State: DatabaseCreated
    --- Total Database(s): 1
    ```

- 列出集合

    在命令输出中，您可以找到有关每个集合的详细信息。

    ```shell
    Milvus(by-dev) > show collections
    ================================================================================
    DBID: 1
    Collection ID: 443407225551410746       Collection Name: medium_articles_2020
    Collection State: CollectionCreated     Create Time: 2023-08-08 09:27:08
    Fields:
    - Field ID: 0   Field Name: RowID       Field Type: Int64
    - Field ID: 1   Field Name: Timestamp   Field Type: Int64
    - Field ID: 100         Field Name: id          Field Type: Int64
            - Primary Key: true, AutoID: false
    - Field ID: 101         Field Name: title       Field Type: VarChar
            - Type Param max_length: 512
    - Field ID: 102         Field Name: title_vector        Field Type: FloatVector
            - Type Param dim: 768
    - Field ID: 103         Field Name: link        Field Type: VarChar
            - Type Param max_length: 512
    - Field ID: 104         Field Name: reading_time        Field Type: Int64
    - Field ID: 105         Field Name: publication         Field Type: VarChar
            - Type Param max_length: 512
    - Field ID: 106         Field Name: claps       Field Type: Int64
    - Field ID: 107         Field Name: responses   Field Type: Int64
    Enable Dynamic Schema: false
    Consistency Level: Bounded
    Start position for channel by-dev-rootcoord-dml_0(by-dev-rootcoord-dml_0_443407225551410746v0): [1 0 28 175 133 76 39 6]
    --- Total collections:  1        Matched collections:  1
    --- Total channel: 1     Healthy collections: 1
    ================================================================================
    ```

- 查看特定集合

    您可以通过指定其 ID 查看特定集合。
```shell
Milvus(by-dev) > 显示集合历史 --id 443407225551410746
===============================================================================
DBID: 1
集合 ID: 443407225551410746       集合名称: medium_articles_2020
集合状态: CollectionCreated     创建时间: 2023-08-08 09:27:08
字段:
- 字段 ID: 0   字段名称: RowID       字段类型: Int64
- 字段 ID: 1   字段名称: Timestamp   字段类型: Int64
- 字段 ID: 100         字段名称: id          字段类型: Int64
            - 主键: true, 自增: false
- 字段 ID: 101         字段名称: title       字段类型: VarChar
            - 类型参数 max_length: 512
- 字段 ID: 102         字段名称: title_vector        字段类型: FloatVector
            - 类型参数 dim: 768
- 字段 ID: 103         字段名称: link        字段类型: VarChar
            - 类型参数 max_length: 512
- 字段 ID: 104         字段名称: reading_time        字段类型: Int64
- 字段 ID: 105         字段名称: publication         字段类型: VarChar
            - 类型参数 max_length: 512
- 字段 ID: 106         字段名称: claps       字段类型: Int64
- 字段 ID: 107         字段名称: responses   字段类型: Int64
启用动态模式: false
一致性级别: Bounded
通道 by-dev-rootcoord-dml_0(by-dev-rootcoord-dml_0_443407225551410746v0) 的起始位置: [1 0 28 175 133 76 39 6]
```

- 查看所有已加载的集合

    您可以让 Birdwatcher 过滤所有已加载的集合。

    ```shell
    Milvus(by-dev) > 显示已加载的集合
    版本: [>= 2.2.0]     集合 ID: 443407225551410746
    复制数: 1        加载状态: 已加载
    --- 已加载的集合数: 1
    ```

- 列出特定集合的所有通道检查点

    您可以让 Birdwatcher 列出特定集合的所有检查点。

    ```shell
    Milvus(by-dev) > 显示检查点 --collection 443407225551410746
    vchannel by-dev-rootcoord-dml_0_443407225551410746v0 定位到 2023-08-08 09:36:09.54 +0000 UTC, 检查点通道: by-dev-rootcoord-dml_0_443407225551410746v0, 来源: 通道检查点
    ```

### 检查索引详情

运行以下命令以详细列出所有索引文件。

```shell
Milvus(by-dev) > 显示索引
*************2.1.x***************
*************2.2.x***************
==================================================================
索引 ID: 443407225551410801    索引名称: _default_idx_102    集合 ID:443407225551410746
创建时间: 2023-08-08 09:27:19.139 +0000      已删除: false
索引类型: HNSW        度量类型: L2
索引参数: 
==================================================================
```

### 列出分区

运行以下命令以列出特定集合中的所有分区。

```shell
Milvus(by-dev) > 显示分区 --collection 443407225551410746
分区 ID: 443407225551410747 名称: _default  状态: PartitionCreated
--- 总数据库数: 1
```
### 检查通道状态

运行以下命令查看通道状态

```shell
Milvus(by-dev) > show channel-watch
=============================
key: by-dev/meta/channelwatch/6/by-dev-rootcoord-dml_0_443407225551410746v0
通道名称: by-dev-rootcoord-dml_0_443407225551410746v0         监控状态: 监控成功
通道监控开始时间: 2023-08-08 09:27:09 +0000, 超时时间: 1970-01-01 00:00:00 +0000
起始位置ID: [1 0 28 175 133 76 39 6], 时间: 1970-01-01 00:00:00 +0000
未刷新的段: []
已刷新的段: []
已删除的段: []
--- 总通道数: 1
```

### 列出所有副本和段

- 列出所有副本

    运行以下命令列出所有副本及其对应的集合。

    ```shell
    Milvus(by-dev) > show replica
    ================================================================================
    副本ID: 443407225685278721 集合ID: 443407225551410746 版本:>=2.2.0
    所有节点:[2]
    ```

- 列出所有段

    运行以下命令列出所有段及其状态

    ```shell
    段ID: 443407225551610865 状态: 已刷新, 行数:5979
    --- 增长中: 0, 封存中: 0, 已刷新: 1
    --- 总段数: 1, 行数: 5979
    ```

    运行以下命令详细列出所有加载的段。对于 Milvus 2.1.x，请使用 `show segment-loaded`。

    ```shell
    Milvus(by-dev) > show segment-loaded-grpc
    ===========
    服务器ID 2
    通道 by-dev-rootcoord-dml_0_443407225551410746v0, 集合: 443407225551410746, 版本 1691486840680656937
    通道的领导视图: by-dev-rootcoord-dml_0_443407225551410746v0
    增长中的段数: 0 , IDs: []
    段ID: 443407225551610865 集合ID: 443407225551410746 通道: by-dev-rootcoord-dml_0_443407225551410746v0
    封存中的段数: 1    
    ```

### 列出配置

您可以让 Birdwatcher 列出每个 Milvus 组件的当前配置。

```shell
Milvus(by-dev) > show configurations
客户端 nil 会话:代理, 服务器ID: 8, 版本: 2.2.11, 地址: 10.244.0.8:19529
组件 rootcoord-1
rootcoord.importtaskexpiration: 900
rootcoord.enableactivestandby: false
rootcoord.importtaskretention: 86400
rootcoord.maxpartitionnum: 4096
rootcoord.dmlchannelnum: 16
rootcoord.minsegmentsizetoenableindex: 1024
rootcoord.port: 53100
rootcoord.address: localhost
rootcoord.maxdatabasenum: 64
组件 datacoord-3
...
querynode.gracefulstoptimeout: 30
querynode.cache.enabled: true
querynode.cache.memorylimit: 2147483648
querynode.scheduler.maxreadconcurrentratio: 2
```

或者，您可以访问每个 Milvus 组件以查找其配置。以下演示了如何列出具有ID 7的 QueryCoord 的配置。
```shell
Milvus(by-dev) > 显示会话
会话:datacoord, 服务器ID: 3, 版本: 2.2.11, 地址: 10.244.0.8:13333
会话:datanode, 服务器ID: 6, 版本: 2.2.11, 地址: 10.244.0.8:21124
会话:indexcoord, 服务器ID: 4, 版本: 2.2.11, 地址: 10.244.0.8:31000
会话:indexnode, 服务器ID: 5, 版本: 2.2.11, 地址: 10.244.0.8:21121
会话:proxy, 服务器ID: 8, 版本: 2.2.11, 地址: 10.244.0.8:19529
会话:querycoord, 服务器ID: 7, 版本: 2.2.11, 地址: 10.244.0.8:19531
会话:querynode, 服务器ID: 2, 版本: 2.2.11, 地址: 10.244.0.8:21123
会话:rootcoord, 服务器ID: 1, 版本: 2.2.11, 地址: 10.244.0.8:53100

Milvus(by-dev) > 访问 querycoord 7
QueryCoord-7(10.244.0.8:19531) > 配置
键: querycoord.enableactivestandby, 值: false
键: querycoord.channeltasktimeout, 值: 60000
键: querycoord.overloadedmemorythresholdpercentage, 值: 90
键: querycoord.distpullinterval, 值: 500
键: querycoord.checkinterval, 值: 10000
键: querycoord.checkhandoffinterval, 值: 5000
键: querycoord.taskexecutioncap, 值: 256
键: querycoord.taskmergecap, 值: 8
键: querycoord.autohandoff, 值: true
键: querycoord.address, 值: localhost
键: querycoord.port, 值: 19531
键: querycoord.memoryusagemaxdifferencepercentage, 值: 30
键: querycoord.refreshtargetsintervalseconds, 值: 300
键: querycoord.balanceintervalseconds, 值: 60
键: querycoord.loadtimeoutseconds, 值: 1800
键: querycoord.globalrowcountfactor, 值: 0.1
键: querycoord.scoreunbalancetolerationfactor, 值: 0.05
键: querycoord.reverseunbalancetolerationfactor, 值: 1.3
键: querycoord.balancer, 值: ScoreBasedBalancer
键: querycoord.autobalance, 值: true
键: querycoord.segmenttasktimeout, 值: 120000
```
## 备份指标

您可以让 Birdwatcher 备份所有组件的指标

```shell
Milvus(my-release) > backup
正在备份 ... 100%(2452/2451)
备份 etcd 的前缀 完成
http://10.244.0.10:9091/metrics
http://10.244.0.10:9091/metrics
http://10.244.0.10:9091/metrics
http://10.244.0.10:9091/metrics
http://10.244.0.10:9091/metrics
http://10.244.0.10:9091/metrics
http://10.244.0.10:9091/metrics
http://10.244.0.10:9091/metrics
备份前缀完成，存储在文件中: bw_etcd_ALL.230810-075211.bak.gz
```

然后您可以在启动 Birdwatcher 的目录中检查该文件。

## 探查集合

您可以让 Birdwatcher 探查已加载集合的状态，使用指定的主键或模拟查询。

### 使用已知主键探查集合

在 `probe` 命令中，您应该使用 `pk` 标志指定主键，并使用 `collection` 标志指定集合 ID。

```shell
Milvus(by-dev) > probe pk --pk 110 --collection 442844725212299747
在段 442844725212299830 上找到 PK 110
字段 id，值: &{long_data:<data:110 > }
字段 title，值: &{string_data:<data:"Human Resources Datafication" > }
字段 title_vector，值: &{dim:768 float_vector:<data:0.022454707 data:0.007861045 data:0.0063843643 data:0.024065714 data:0.013782166 data:0.018483251 data:-0.026526336 ... data:-0.06579628 data:0.00033906146 data:0.030992996 data:-0.028134001 data:-0.01311325 data:0.012471594 > }
字段 article_meta，值: &{json_data:<data:"{\"link\":\"https:\\/\\/towardsdatascience.com\\/human-resources-datafication-d44c8f7cb365\",\"reading_time\":6,\"publication\":\"Towards Data Science\",\"claps\":256,\"responses\":0}" > }
```

### 使用模拟查询探查所有集合

您还可以让 Birdwatcher 使用模拟查询探查所有集合。

```shell
Milvus(by-dev) > probe query
探查集合 442682158191982314
找到向量字段 vector(103)，维度[384]，索引 ID: 442682158191990455
无法生成模拟请求，探查索引类型 IVF_FLAT 尚不支持
探查集合 442844725212086932
找到向量字段 title_vector(102)，维度[768]，索引 ID: 442844725212086936
Shard my-release-rootcoord-dml_1_442844725212086932v0 领导者[298] 探查搜索成功。
探查集合 442844725212299747
找到向量字段 title_vector(102)，维度[768]，索引 ID: 442844725212299751
Shard my-release-rootcoord-dml_4_442844725212299747v0 领导者[298] 探查搜索成功。
探查集合 443294505839900248
找到向量字段 vector(101)，维度[256]，索引 ID: 443294505839900251
Shard my-release-rootcoord-dml_5_443294505839900248v0 领导者[298] 探查搜索成功。
```