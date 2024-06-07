---
id: release_notes.md
summary: Milvus 发布说明
title: 发布说明
---

# 发布说明

了解 Milvus 的新功能！本页面总结了每个版本发布后的新功能、改进、已知问题和 bug 修复情况。您可以在此部分找到 v2.4.0 之后每个发布版本的发布说明。我们建议您定期访问此页面以了解更新内容。

## v2.4.4

发布日期：2024年5月31日

| Milvus 版本 | Python SDK 版本 | Java SDK 版本    | Node.js SDK 版本 |
|----------------|--------------------| --------------------| --------------------|
| 2.4.4          | 2.4.3              | 2.4.1               | 2.4.2               |

Milvus v2.4.4 包含了几个关键的 bug 修复和改进，旨在提高性能和稳定性。值得注意的是，我们**解决了一个关键问题，即批量插入统计日志被错误地垃圾回收**，可能影响数据完整性。**我们强烈建议所有 v2.4 用户升级到此版本以从这些修复中受益。**

**如果您正在使用批量插入，请尽快升级到 v2.4.4 以确保数据完整性。**

### 关键 bug 修复

- 填充统计日志 ID 并验证其正确性（[#33478](https://github.com/milvus-io/milvus/pull/33478)）

### 改进

- 升级 ARM SVE 的位集合（[#33440](https://github.com/milvus-io/milvus/pull/33440)）
- 启用 Milvus 与 GCC-13 的编译（[#33441](https://github.com/milvus-io/milvus/pull/33441)）

### Bug 修复

- 当授予所有权限时显示空集合（[#33454](https://github.com/milvus-io/milvus/pull/33454)）
- 确保 CMake 下载并安装到当前平台，而不仅仅是 x86_64（[#33439](https://github.com/milvus-io/milvus/pull/33439)）

## v2.4.3

发布日期：2024年5月29日

| Milvus 版本 | Python SDK 版本 | Java SDK 版本    | Node.js SDK 版本 |
|----------------|--------------------| --------------------| --------------------|
| 2.4.3          | 2.4.3              | 2.4.1               | 2.4.2               |

Milvus 版本 2.4.3 引入了一系列功能、改进和 bug 修复，以提升性能和可靠性。值得注意的增强功能包括支持稀疏浮点向量批量插入和优化的布隆过滤器加速。改进涵盖了各个领域，从动态配置更新到内存使用优化。Bug 修复解决了像 panic 场景这样的关键问题，并确保系统运行更加顺畅。此版本强调了 Milvus 不断改进功能、优化性能并提供强大用户体验的承诺。

### 功能

- 支持 binlog/json/parquet 的稀疏浮点向量批量插入（[#32649](https://github.com/milvus-io/milvus/pull/32649)）

### 改进

- 基于 RPC 实现 Datacoord/node 监视通道（[#32036](https://github.com/milvus-io/milvus/pull/32036)）
- 优化了布隆过滤器以加速删除过滤（[#32642](https://github.com/milvus-io/milvus/pull/32642), [#33329](https://github.com/milvus-io/milvus/pull/33329), [#33284](https://github.com/milvus-io/milvus/pull/33284)）
- 如果标量索引没有原始数据，则通过 mmap 加载原始数据（[#33317](https://github.com/milvus-io/milvus/pull/33317)）
- 将 milvus 配置同步到 milvus.yaml（[#33322](https://github.com/milvus-io/milvus/pull/33322), [#32920](https://github.com/milvus-io/milvus/pull/32920), [#32857](https://github.com/milvus-io/milvus/pull/32857), [#32946](https://github.com/milvus-io/milvus/pull/32946)）
- 更新了 knowhere 版本（[#33310](https://github.com/milvus-io/milvus/pull/33310), [#32931](https://github.com/milvus-io/milvus/pull/32931), [#33043](https://github.com/milvus-io/milvus/pull/33043)）
- 在 QueryCoord 中启用动态更新平衡器策略（[#33272](https://github.com/milvus-io/milvus/pull/33272)）
- 在写缓冲区中使用预构建的日志记录器以减少日志记录器分配（[#33304](https://github.com/milvus-io/milvus/pull/33304)）
- 改进了参数检查（[#32777](https://github.com/milvus-io/milvus/pull/32777), [#33271](https://github.com/milvus-io/milvus/pull/33271), [#33218](https://github.com/milvus-io/milvus/pull/33218)）
- 添加了一个参数以忽略检查点中不正确的消息 ID（[#33249](https://github.com/milvus-io/milvus/pull/33249)）
- 添加了控制插件初始化失败处理的配置（[#32680](https://github.com/milvus-io/milvus/pull/32680)）
- 为 knowhere 添加了得分计算一致性配置（[#32997](https://github.com/milvus-io/milvus/pull/32997)）
- 引入了一个配置选项以控制公共角色权限的初始化（[#33174](https://github.com/milvus-io/milvus/pull/33174)）
- 在读取字段时优化了内存使用（[#33196](https://github.com/milvus-io/milvus/pull/33196)）
- 优化了 Channel Manager v2 的实现（[#33172](https://github.com/milvus-io/milvus/pull/33172), [#33121](https://github.com/milvus-io/milvus/pull/33121), [#33014](https://github.com/milvus-io/milvus/pull/33014)）
- 添加了跟踪内存中数据大小的功能以用于 binlog（[#33025](https://github.com/milvus-io/milvus/pull/33025)）
- 为段索引文件大小添加了指标（[#32979](https://github.com/milvus-io/milvus/pull/32979), [#33305](https://github.com/milvus-io/milvus/pull/33305)）
- 用 DeletePartialMatch 替换 Delete 以删除指标（[#33029](https://github.com/milvus-io/milvus/pull/33029)）
- 根据段类型获取相关数据大小（[#33017](https://github.com/milvus-io/milvus/pull/33017)）
- 在元数据存储中清除通道节点信息（[#32988](https://github.com/milvus-io/milvus/pull/32988)）
- 从 datanode broker 中移除 rootcoord（[#32818](https://github.com/milvus-io/milvus/pull/32818)）
- 启用批量上传（[#32788](https://github.com/milvus-io/milvus/pull/32788)）
- 当使用分区键时，将默认分区数更改为 16 ([#32950](https://github.com/milvus-io/milvus/pull/32950))
- 改进了对非常大的 top-k 查询的减少性能 ([#32871](https://github.com/milvus-io/milvus/pull/32871))
- 利用 TestLocations 能力加速写入和压缩 ([#32948](https://github.com/milvus-io/milvus/pull/32948))
- 优化计划解析器池，避免不必要的回收 ([#32869](https://github.com/milvus-io/milvus/pull/32869))
- 改进了加载速度 ([#32898](https://github.com/milvus-io/milvus/pull/32898))
- 为 restv2 使用集合默认一致性级别 ([#32956](https://github.com/milvus-io/milvus/pull/32956))
- 为 rest API 添加了成本响应 ([#32620](https://github.com/milvus-io/milvus/pull/32620))
- 启用了通道独占平衡策略 ([#32911](https://github.com/milvus-io/milvus/pull/32911))
- 在代理中暴露了 describedatabase API ([#32732](https://github.com/milvus-io/milvus/pull/32732))
- 在通过集合获取 RG 时利用 coll2replica 映射 ([#32892](https://github.com/milvus-io/milvus/pull/32892))
- 为搜索和查询添加了更多的追踪信息 ([#32734](https://github.com/milvus-io/milvus/pull/32734))
- 支持 opentelemetry 追踪的动态配置 ([#32169](https://github.com/milvus-io/milvus/pull/32169))
- 在更新 leaderview 时避免对通道结果进行迭代 ([#32887](https://github.com/milvus-io/milvus/pull/32887))
- 优化了 parquet 的向量偏移处理 ([#32822](https://github.com/milvus-io/milvus/pull/32822))
- 通过集合改进了 datacoord 段过滤 ([#32831](https://github.com/milvus-io/milvus/pull/32831))
- 调整了日志级别和频率 ([#33042](https://github.com/milvus-io/milvus/pull/33042), [#32838](https://github.com/milvus-io/milvus/pull/32838), [#33337](https://github.com/milvus-io/milvus/pull/33337))
- 在暂停平衡后启用停止平衡 ([#32812](https://github.com/milvus-io/milvus/pull/32812))
- 在领导者位置更改时更新 shard leader 缓存 ([#32470](https://github.com/milvus-io/milvus/pull/32470))
- 移除了已弃用的 API 和字段 ([#32808](https://github.com/milvus-io/milvus/pull/32808), [#32704](https://github.com/milvus-io/milvus/pull/32704))
- 添加了 metautil.channel 以将字符串比较转换为整数 ([#32749](https://github.com/milvus-io/milvus/pull/32749))
- 为 payload writer 错误消息和日志添加了类型信息，并在 querynode 发现新集合时记录 ([#32522](https://github.com/milvus-io/milvus/pull/32522))
- 在创建带有分区键的集合时检查分区数 ([#32670](https://github.com/milvus-io/milvus/pull/32670))
- 如果监视失败，则移除旧的 l0 段 ([#32725](https://github.com/milvus-io/milvus/pull/32725))
- 改进了请求类型的打印 ([#33319](https://github.com/milvus-io/milvus/pull/33319))
- 在获取类型之前检查数组字段数据是否为空 ([#33311](https://github.com/milvus-io/milvus/pull/33311))
- 启动时返回错误，删除/添加节点操作失败 ([#33258](https://github.com/milvus-io/milvus/pull/33258))
- 允许数据节点的服务器 ID 被更新 ([#31597](https://github.com/milvus-io/milvus/pull/31597))
- 在集合释放中统一查询节点的指标清理 ([#32805](https://github.com/milvus-io/milvus/pull/32805))
- 修复标量自动索引配置不正确的版本 ([#32795](https://github.com/milvus-io/milvus/pull/32795))
- 优化创建/修改索引的索引参数检查 ([#32712](https://github.com/milvus-io/milvus/pull/32712))
- 移除多余的副本恢复 ([#32985](https://github.com/milvus-io/milvus/pull/32985))
- 启用通道元数据表写入超过 200k 段 ([#33300](https://github.com/milvus-io/milvus/pull/33300))

### Bug 修复

- 修复速率限制拦截器中数据库不存在时的恐慌错误 ([#33308](https://github.com/milvus-io/milvus/pull/33308))
- 修复因参数不正确导致的配额中心指标收集失败 ([#33399](https://github.com/milvus-io/milvus/pull/33399))
- 修复 processactivestandby 返回错误时的恐慌 ([#33372](https://github.com/milvus-io/milvus/pull/33372))
- 修复 restful v2 中 nq > 1 时搜索结果截断的问题 ([#33363](https://github.com/milvus-io/milvus/pull/33363))
- 在 restful v2 中为角色操作添加数据库名称字段 ([#33291](https://github.com/milvus-io/milvus/pull/33291))
- 修复全局速率限制不起作用的问题 ([#33336](https://github.com/milvus-io/milvus/pull/33336))
- 修复由于构建索引失败导致的恐慌 ([#33314](https://github.com/milvus-io/milvus/pull/33314))
- 在 segcore 中为稀疏向量添加验证以确保合法性 ([#33312](https://github.com/milvus-io/milvus/pull/33312))
- 在任务完成后从 syncmgr 中移除任务 ([#33303](https://github.com/milvus-io/milvus/pull/33303))
- 修复数据导入过程中分区键过滤失败的问题 ([#33277](https://github.com/milvus-io/milvus/pull/33277))
- 修复使用 noop 导出器时无法生成 traceID 的问题 ([#33208](https://github.com/milvus-io/milvus/pull/33208))
- 改进查询结果的检索 ([#33179](https://github.com/milvus-io/milvus/pull/33179))
- 标记通道检查点已删除以防止检查点滞后指标泄漏 ([#33201](https://github.com/milvus-io/milvus/pull/33201))
- 修复查询节点在停止过程中卡住的问题 ([#33154](https://github.com/milvus-io/milvus/pull/33154))
- 修复刷新响应中缺少的段 ([#33061](https://github.com/milvus-io/milvus/pull/33061))
- 使提交操作具有幂等性 ([#33053](https://github.com/milvus-io/milvus/pull/33053))
- 在流式读取器中为每个批次分配新切片 ([#33360](https://github.com/milvus-io/milvus/pull/33360))
- 在 QueryCoord 重新启动后从资源组中清除离线节点 ([#33233](https://github.com/milvus-io/milvus/pull/33233))
- 在 completedCompactor 中移除 l0 压缩器 ([#33216](https://github.com/milvus-io/milvus/pull/33216))
- 在初始化限制器时重置配额值 ([#33152](https://github.com/milvus-io/milvus/pull/33152))
- 修复了超出 etcd 限制的问题 ([#33041](https://github.com/milvus-io/milvus/pull/33041))
- 解决了由于字段过多导致的 etcd 事务限制超出的问题 ([#33040](https://github.com/milvus-io/milvus/pull/33040))
- 移除了在 GetNumRowsOfPartition 中的 RLock 重入 ([#33045](https://github.com/milvus-io/milvus/pull/33045))
- 在 SyncAll 之前启动了 LeaderCacheObserver ([#33035](https://github.com/milvus-io/milvus/pull/33035))
- 启用了已释放备用通道的平衡 ([#32986](https://github.com/milvus-io/milvus/pull/32986))
- 在服务器初始化之前初始化了访问日志记录器 ([#32976](https://github.com/milvus-io/milvus/pull/32976))
- 使压缩器能够清除空段 ([#32821](https://github.com/milvus-io/milvus/pull/32821))
- 在 l0 压缩中填充了 deltalog 条目编号和时间范围 ([#33004](https://github.com/milvus-io/milvus/pull/33004))
- 修复了由于分片领导者缓存数据竞争而导致代理崩溃的问题 ([#32971](https://github.com/milvus-io/milvus/pull/32971))
- 更正了加载索引指标的时间单位 ([#32935](https://github.com/milvus-io/milvus/pull/32935))
- 修复了在停止查询节点时段无法成功释放的问题 ([#32929](https://github.com/milvus-io/milvus/pull/32929))
- 修复了索引资源估算的问题 ([#32842](https://github.com/milvus-io/milvus/pull/32842))
- 将通道检查点设置为增量位置 ([#32878](https://github.com/milvus-io/milvus/pull/32878))
- 在返回 future 之前锁定了 syncmgr 键 ([#32865](https://github.com/milvus-io/milvus/pull/32865))
- 确保倒排索引只有一个段 ([#32858](https://github.com/milvus-io/milvus/pull/32858))
- 修复了压缩触发器选择两个相同段的问题 ([#32800](https://github.com/milvus-io/milvus/pull/32800))
- 修复了无法在 binlog 导入中指定分区名称的问题 ([#32730](https://github.com/milvus-io/milvus/pull/32730), [#33027](https://github.com/milvus-io/milvus/pull/33027))
- 使 parquet 导入中的动态列变为可选项 ([#32738](https://github.com/milvus-io/milvus/pull/32738))
- 插入数据时跳过自动 ID 检查 ([#32775](https://github.com/milvus-io/milvus/pull/32775))
- 验证插入字段数据的行数与模式匹配 ([#32770](https://github.com/milvus-io/milvus/pull/32770))
- 为 CTraceContext ID 添加了 Wrapper 和 Keepalive ([#32746](https://github.com/milvus-io/milvus/pull/32746))
- 修复了数据坐标元对象中找不到数据库名称的问题 ([#33412](https://github.com/milvus-io/milvus/pull/33412))
- 同步了删除的分区的已删除段 ([#33332](https://github.com/milvus-io/milvus/pull/33332))
- 修复了由于参数不正确导致的 quotaCenter 指标收集失败 ([#33399](https://github.com/milvus-io/milvus/pull/33399))

## v2.4.1

发布日期：2024年5月6日
| Milvus 版本 | Python SDK 版本 | Java SDK 版本 | Node.js SDK 版本 |
|--------------|-----------------|-----------------|-----------------|
| 2.4.1        | 2.4.1           | 2.4.0           | 2.4.2           |

Milvus 版本 2.4.1 带来了许多改进和 bug 修复，旨在提升软件的性能、可观察性和稳定性。这些改进包括声明式资源组 API、增强的批量插入功能，支持 Float16/BFloat16 向量数据类型，优化了减少对象存储的列表操作的垃圾回收（GC）机制，以及其他与性能优化相关的更改。此外，bug 修复解决了诸如编译错误、换行符上失败的模糊匹配、RESTful 接口的参数数据类型不正确以及启用动态字段时 BulkInsert 在 numpy 文件上引发错误的问题。

### 破坏性更改

- 停止支持使用空过滤表达式进行删除。([#32472](https://github.com/milvus-io/milvus/pull/32472))

### 特性

- 在批量插入中添加了对 Float16/BFloat16 向量数据类型的支持 ([#32157](https://github.com/milvus-io/milvus/pull/32157))
- 增强了稀疏浮点向量以支持暴力迭代搜索和范围搜索 ([#32635](https://github.com/milvus-io/milvus/pull/32635))

### 改进

- 添加了声明式资源组 API ([#31930](https://github.com/milvus-io/milvus/pull/31930), [#32297](https://github.com/milvus-io/milvus/pull/32297), [#32536](https://github.com/milvus-io/milvus/pull/32536), [#32666](https://github.com/milvus-io/milvus/pull/32666))
- 重写了 QueryCoord 中的 collection observer，使其基于任务驱动 ([#32441](https://github.com/milvus-io/milvus/pull/32441))
- 重构了 DataNode 的 SyncManager 中使用的数据结构，以减少内存使用量并防止错误 ([#32673](https://github.com/milvus-io/milvus/pull/32673))
- 修订了垃圾回收的实现，以最小化与对象存储相关的列表操作 ([#31740](https://github.com/milvus-io/milvus/pull/31740))
- 在集合数量较高时降低了 CPU 使用率 ([#32245](https://github.com/milvus-io/milvus/pull/32245))
- 通过代码自动生成 milvus.yaml 文件中的相关配置项，增强了 milvus.yaml 的管理 ([#31832](https://github.com/milvus-io/milvus/pull/31832), [#32357](https://github.com/milvus-io/milvus/pull/32357))
- 通过执行本地减少后检索数据，增强了 Query 的性能 ([#32346](https://github.com/milvus-io/milvus/pull/32346))
- 为 etcd 客户端创建添加了 WithBlock 选项 ([#32641](https://github.com/milvus-io/milvus/pull/32641))
- 如果客户端提供了 client_request_id，则将其指定为 TraceID ([#32264](https://github.com/milvus-io/milvus/pull/32264))
- 为删除和批量插入操作的指标添加了 db 标签 ([#32611](https://github.com/milvus-io/milvus/pull/32611))
- 添加了通过配置跳过对 AutoID 和 PartitionKey 列进行验证的逻辑 ([#32592](https://github.com/milvus-io/milvus/pull/32592))
- 优化了与认证相关的错误信息 ([#32253](https://github.com/milvus-io/milvus/pull/32253))
- 优化了 DataCoord 中 AllocSegmentID 的错误日志 ([#32351](https://github.com/milvus-io/milvus/pull/32351), [#32335](https://github.com/milvus-io/milvus/pull/32335))
- 移除了重复的指标 ([#32380](https://github.com/milvus-io/milvus/pull/32380), [#32308](https://github.com/milvus-io/milvus/pull/32308))，清理了未使用的指标 ([#32404](https://github.com/milvus-io/milvus/pull/32404), [#32515](https://github.com/milvus-io/milvus/pull/32515))
- 添加了配置选项，用于控制是否强制激活 partitionKey 功能 ([#32433](https://github.com/milvus-io/milvus/pull/32433))
- 添加了配置选项，用于控制单个请求中可插入的最大数据量 ([#32433](https://github.com/milvus-io/milvus/pull/32433))
- 在段级别并行化 applyDelete 操作，加速 Delegator 处理 Delete 消息的过程 ([#32291](https://github.com/milvus-io/milvus/pull/32291))
- 在 QueryCoord 中使用索引 ([#32232](https://github.com/milvus-io/milvus/pull/32232), [#32505](https://github.com/milvus-io/milvus/pull/32505), [#32533](https://github.com/milvus-io/milvus/pull/32533), [#32595](https://github.com/milvus-io/milvus/pull/32595))，并添加缓存 ([#32580](https://github.com/milvus-io/milvus/pull/32580))，加速频繁的过滤操作。
- 重写了数据结构 ([#32273](https://github.com/milvus-io/milvus/pull/32273))，重构了代码 ([#32389](https://github.com/milvus-io/milvus/pull/32389))，加速了 DataCoord 中的常见操作。
- 从 conan 中移除了 openblas ([#32002](https://github.com/milvus-io/milvus/pull/32002))

### Bug 修复

- 修复了在 rockylinux8 中构建 milvus 的问题 ([#32619](https://github.com/milvus-io/milvus/pull/32619))
- 修复了 ARM 上 SVE 的编译错误 ([#32463](https://github.com/milvus-io/milvus/pull/32463), [#32270](https://github.com/milvus-io/milvus/pull/32270))
- 修复了基于 ARM 的 GPU 映像上的崩溃问题 ([#31980](https://github.com/milvus-io/milvus/pull/31980))
- 修复了正则表达式查询无法处理带有换行符的文本的问题 ([#32569](https://github.com/milvus-io/milvus/pull/32569))
- 修复了由于 GetShardLeaders 返回空节点列表导致搜索结果为空的问题 ([#32685](https://github.com/milvus-io/milvus/pull/32685))
- 修复了在 numpy 文件中遇到动态字段时 BulkInsert 报错的问题 ([#32596](https://github.com/milvus-io/milvus/pull/32596))
- 修复了与 RESTFulV2 接口相关的 bug，包括一个重要的修复，允许请求中的数字参数接受数字输入而不是字符串类型（[#32485](https://github.com/milvus-io/milvus/pull/32485)，[#32355](https://github.com/milvus-io/milvus/pull/32355)）
- 通过移除速率限制器中的配置事件监视，修复了代理中的内存泄漏问题（[#32313](https://github.com/milvus-io/milvus/pull/32313)）
- 修复了速率限制器在未指定 partitionName 时错误地报告分区找不到的问题（[#32647](https://github.com/milvus-io/milvus/pull/32647)）
- 在错误类型中添加了对 Collection 处于恢复状态和未加载的情况的检测。([#32447](https://github.com/milvus-io/milvus/pull/32447))
- 修正了负查询实体数指标（[#32361](https://github.com/milvus-io/milvus/pull/32361))


## v2.4.0

发布日期：2024年4月17日

| Milvus 版本 | Python SDK 版本 | Node.js SDK 版本 |
|----------------|--------------------| --------------------|
| 2.4.0          | 2.4.0              | 2.4.0               |

我们很高兴地宣布 Milvus 2.4.0 的正式发布。在基于 2.4.0-rc.1 版本的稳固基础上，我们专注于解决用户报告的关键 bug，同时保留现有功能。此外，Milvus 2.4.0 引入了一系列旨在增强系统性能、通过各种指标提高可观察性以及简化代码库以提高简易性的优化。

### 改进

- 支持 MinIO 的 TLS 连接（[#31396](https://github.com/milvus-io/milvus/pull/31396)，[#31618](https://github.com/milvus-io/milvus/pull/31618)）
- 标量字段的 AutoIndex 支持（[#31593](https://github.com/milvus-io/milvus/pull/31593)）
- 一致的执行路径重构以支持混合搜索与常规搜索（[#31742](https://github.com/milvus-io/milvus/pull/31742)，[#32178](https://github.com/milvus-io/milvus/pull/32178)）
- 通过位图和位图视图重构加速过滤（[#31592](https://github.com/milvus-io/milvus/pull/31592)，[#31754](https://github.com/milvus-io/milvus/pull/31754)，[#32139](https://github.com/milvus-io/milvus/pull/32139)）
- 导入任务现在支持等待数据索引完成（[#31733](https://github.com/milvus-io/milvus/pull/31733)）
- 增强了导入兼容性（[#32121](https://github.com/milvus-io/milvus/pull/32121)）、任务调度（[#31475](https://github.com/milvus-io/milvus/pull/31475)）以及导入文件大小和数量的限制（[#31542](https://github.com/milvus-io/milvus/pull/31542)）
- 代码简化工作，包括用于类型检查的接口标准化（[#31945](https://github.com/milvus-io/milvus/pull/31945)，[#31857](https://github.com/milvus-io/milvus/pull/31857)），移除废弃代码和指标（[#32079](https://github.com/milvus-io/milvus/pull/32079)，[#32134](https://github.com/milvus-io/milvus/pull/32134)，[#31535](https://github.com/milvus-io/milvus/pull/31535)，[#32211](https://github.com/milvus-io/milvus/pull/32211)，[#31935](https://github.com/milvus-io/milvus/pull/31935)），以及常量名称的规范化（[#31515](https://github.com/milvus-io/milvus/pull/31515)）
- 为QueryCoord当前目标通道检查点滞后延迟新增指标（[#31420](https://github.com/milvus-io/milvus/pull/31420)）
- 为常见指标新增数据库标签（[#32024](https://github.com/milvus-io/milvus/pull/32024)）
- 新增关于已删除、已索引和已加载实体数量的指标，包括collectionName和dbName等标签（[#31861](https://github.com/milvus-io/milvus/pull/31861)）
- 改进了不匹配向量类型的错误处理（[#31766](https://github.com/milvus-io/milvus/pull/31766)）
- 当索引无法构建时，支持抛出错误而不是崩溃（[#31845](https://github.com/milvus-io/milvus/pull/31845)）
- 在删除数据库时支持使数据库元数据缓存失效（[#32092](https://github.com/milvus-io/milvus/pull/32092)）
- 为通道分发进行接口重构（[#31814](https://github.com/milvus-io/milvus/pull/31814)）和领导者视图管理（[#32127](https://github.com/milvus-io/milvus/pull/32127)）
- 重构通道分发管理器接口（[#31814](https://github.com/milvus-io/milvus/pull/31814)）和重构领导者视图管理器接口（[#32127](https://github.com/milvus-io/milvus/pull/32127)）
- 批处理（[#31632](https://github.com/milvus-io/milvus/pull/31632)），添加映射信息（[#32234](https://github.com/milvus-io/milvus/pull/32234)，[#32249](https://github.com/milvus-io/milvus/pull/32249)），并避免使用锁（[#31787](https://github.com/milvus-io/milvus/pull/31787）以加速频繁调用的操作

### 破坏性更改

- 停止对二进制向量进行分组搜索（[#31735](https://github.com/milvus-io/milvus/pull/31735)）
- 停止使用混合搜索进行分组搜索（[#31812](https://github.com/milvus-io/milvus/pull/31812)）
- 停止在二进制向量上使用HNSW索引（[#31883](https://github.com/milvus-io/milvus/pull/31883)）

### Bug 修复
- 优化了查询和插入的数据类型和数值检查，以防止崩溃（[#31478](https://github.com/milvus-io/milvus/pull/31478), [#31653](https://github.com/milvus-io/milvus/pull/31653), [#31698](https://github.com/milvus-io/milvus/pull/31698), [#31842](https://github.com/milvus-io/milvus/pull/31842), [#32042](https://github.com/milvus-io/milvus/pull/32042), [#32251](https://github.com/milvus-io/milvus/pull/32251), [#32204](https://github.com/milvus-io/milvus/pull/32204))
- 修复了 RESTful API 的 bug（[#32160](https://github.com/milvus-io/milvus/pull/32160)）
- 改进了倒排索引资源使用情况的预测（[#31641](https://github.com/milvus-io/milvus/pull/31641)）
- 解决了启用授权时与 etcd 的连接问题（[#31668](https://github.com/milvus-io/milvus/pull/31668)）
- 对 nats 服务器进行了安全更新（[#32023](https://github.com/milvus-io/milvus/pull/32023)）
- 将倒排索引文件存储到 QueryNode 的本地存储路径而不是 /tmp 目录中（[#32210](https://github.com/milvus-io/milvus/pull/32210)）
- 解决了 collectionInfo 中的 datacoord 内存泄漏问题（[#32243](https://github.com/milvus-io/milvus/pull/32243)）
- 修复了与 fp16/bf16 相关的 bug，可能导致系统崩溃（[#31677](https://github.com/milvus-io/milvus/pull/31677), [#31841](https://github.com/milvus-io/milvus/pull/31841), [#32196](https://github.com/milvus-io/milvus/pull/32196)）
- 解决了分组搜索返回结果不足的问题（[#32151](https://github.com/milvus-io/milvus/pull/32151)）
- 调整了使用迭代器进行搜索，以更有效地处理 Reduce 步骤中的偏移量，并确保在启用 "reduceStopForBest" 时获得充分的结果（[#32088](https://github.com/milvus-io/milvus/pull/32088))

## v2.4.0-rc.1
发布日期：2024年3月20日

| Milvus 版本 | Python SDK 版本 |
|----------------|--------------------|
| 2.4.0-rc.1     | 2.4.0              |

此版本引入了几个基于场景的功能：

- **新 GPU 索引 - CAGRA**：感谢 NVIDIA 的贡献，这个新的 GPU 索引提供了 10 倍的性能提升，特别适用于批量搜索。详情请参考 [GPU 索引](gpu_index.md)。
  
- **多向量** 和 **混合搜索**：此功能使得可以存储来自多个模型的向量嵌入，并进行多向量搜索。详情请参考 [多向量搜索](multi-vector-search.md)。
  
- **稀疏向量**：适用于关键词解释和分析，现在支持在您的集合中处理稀疏向量。详情请参考 [稀疏向量](sparse_vector.md)。
  
- **分组搜索**：分类聚合增强了文档级别的检索增强生成（RAG）应用程序的召回。详情请参考 [分组搜索](https://milvus.io/docs/single-vector-search.md#Grouping-search)。
- **倒排索引** 和 **模糊匹配**：这些功能提高了标量字段的关键字检索。详情请参阅 [索引标量字段](index-scalar-fields.md) 和 [过滤搜索](single-vector-search.md#filtered-search)。

### 新功能

#### GPU 索引 - CAGRA

我们要向 NVIDIA 团队表示诚挚的感谢，感谢他们为 CAGRA 做出的宝贵贡献，这是一种最先进的基于 GPU 的图形索引，可在线使用。

与以往的 GPU 索引不同，CAGRA 在小批量查询方面表现出压倒性的优势，这正是 CPU 索引传统上擅长的领域。此外，CAGRA 在大批量查询和索引构建速度方面的性能，这些领域是 GPU 索引已经表现出色的地方，真正是无与伦比的。

示例代码可在 [example_gpu_cagra.py](https://github.com/milvus-io/pymilvus/blob/2.4/examples/example_gpu_cagra.py) 中找到。

#### 稀疏向量（Beta）

在这个版本中，我们引入了一种称为稀疏向量的新型向量字段。稀疏向量与其密集对应物不同，因为它们往往具有数量级更高的维度，只有少数维度是非零的。由于其基于术语的特性，这一特性提供了更好的可解释性，并且在某些领域可能更有效。像 SPLADEv2/BGE-M3 这样的学习稀疏模型已被证明对于常见的第一阶段排名任务非常有用。Milvus 中这一新功能的主要用例是允许对由神经模型（如 SPLADEv2/BGE-M3）和统计模型（如 BM25 算法）生成的稀疏向量进行高效的近似语义最近邻搜索。Milvus 现在支持对稀疏向量进行有效和高性能的存储、索引和搜索（MIPS，最大内积搜索）。

示例代码可在 [hello_sparse.py](https://github.com/milvus-io/pymilvus/blob/2.4/examples/hello_sparse.py) 中找到。

#### 多嵌入 & 混合搜索

多向量支持是需要多模型数据处理或混合密集和稀疏向量的应用程序的基石。有了多向量支持，现在您可以：

- 存储为来自多个模型的非结构化文本、图像或音频样本生成的向量嵌入。
- 进行包括每个实体多个向量的 ANN 搜索。
- 通过为不同嵌入模型分配权重来定制搜索策略。
- 尝试各种嵌入模型，找到最佳的模型组合。
多向量支持允许在集合中存储、索引和应用多个不同类型的向量字段，如 FLOAT_VECTOR 和 SPARSE_FLOAT_VECTOR 的重新排序策略。目前，有两种重新排序策略可用：**倒数秩融合 (RRF)** 和 **平均加权评分**。这两种策略将来自不同向量字段的搜索结果合并为一个统一的结果集。第一种策略优先考虑在不同向量字段的搜索结果中一贯出现的实体，而另一种策略则为每个向量字段的搜索结果分配权重，以确定它们在最终结果集中的重要性。

示例代码可在 [hybrid_search.py](https://github.com/milvus-io/pymilvus/blob/2.4/examples/hybrid_search.py) 中找到。

#### 倒排索引和模糊匹配

在之前的 Milvus 版本中，基于内存的二进制搜索索引和 Marisa Trie 索引用于标量字段索引。然而，这些方法占用内存较多。Milvus 的最新版本现在采用基于 Tantivy 的倒排索引，可应用于所有数值和字符串数据类型。这种新索引显著提高了标量查询性能，将字符串中关键字的查询速度提高了十倍。此外，倒排索引消耗的内存更少，这要归功于数据压缩和内部索引结构的内存映射存储 (MMap) 机制的额外优化。

此版本还支持使用前缀、中缀和后缀进行标量过滤的模糊匹配。

示例代码可在 [inverted_index_example.py](https://github.com/milvus-io/pymilvus/blob/2.4/examples/inverted_index_example.py) 和 [fuzzy_match.py](https://github.com/milvus-io/pymilvus/blob/2.4/examples/fuzzy_match.py) 中找到。

#### 分组搜索

现在可以通过特定标量字段中的值对搜索结果进行聚合。这有助于 RAG 应用程序实现文档级召回。考虑一个文档集合，每个文档分为各种段落。每个段落由一个向量嵌入表示，并属于一个文档。为了找到最相关的文档而不是散落的段落，您可以在 search() 操作中包含 group_by_field 参数，以按文档 ID 对结果进行分组。

示例代码可在 [example_group_by.py](https://github.com/milvus-io/pymilvus/blob/2.4/examples/example_group_by.py) 中找到。

#### Float16 和 BFloat- 向量数据类型

机器学习和神经网络通常使用半精度数据类型，如 Float16 和 BFloat- 虽然这些数据类型可以提高查询效率并减少内存使用量，但却会降低准确性。通过此版本，Milvus 现在支持这些数据类型用于向量字段。

示例代码可在 [float16_example.py](https://github.com/milvus-io/pymilvus/blob/2.4/examples/float16_example.py) 和 [bfloat16_example.py](https://github.com/milvus-io/pymilvus/blob/2.4/examples/bfloat16_example.py) 中找到。
### 升级后的架构

#### L0 段

此版本包括一个名为 L0 段的新段，旨在记录已删除的数据。该段定期压缩存储的已删除记录，并将其拆分为密封段，减少了对小删除所需的数据刷新次数，同时占用较小的存储空间。通过这种机制，Milvus完全将数据压缩与数据刷新分离，提升了删除和更新操作的性能。

#### 重构的 BulkInsert

此版本还引入了改进的批量插入逻辑。这使您可以在单个批量插入请求中导入多个文件。通过重构版本，批量插入的性能和稳定性都得到了显著提升。用户体验也得到了增强，例如优化的速率限制和更友好的错误消息。此外，您可以通过 Milvus 的 RESTful API 轻松访问批量插入端点。

#### 内存映射存储

Milvus 使用内存映射存储（MMap）来优化其内存使用情况。这种机制不是直接将文件内容加载到内存中，而是将文件内容映射到内存中。这种方法会带来一定的性能降级。通过在具有 2 个 CPU 和 8 GB RAM 的主机上为 HNSW 索引集合启用 MMap，您可以在不到 10% 的性能降级下加载多达 4 倍的数据。

此外，此版本还允许动态和细粒度地控制 MMap，无需重新启动 Milvus。

有关详细信息，请参阅[MMap 存储](mmap.md)。

### 其他

#### Milvus-CDC

Milvus-CDC 是一个易于使用的辅助工具，用于捕获和同步 Milvus 实例之间的增量数据，实现轻松的增量备份和灾难恢复。在此版本中，Milvus-CDC 的稳定性得到了改善，其变更数据捕获（CDC）功能现已正式推出。

要了解有关 Milvus-CDC 的更多信息，请参阅[GitHub 仓库](https://github.com/zilliztech/milvus-cdc)和[Milvus-CDC 概述](milvus-cdc-overview.md)。

#### 精炼的 MilvusClient 接口

MilvusClient 是 ORM 模块的易于使用的替代方案。它采用纯函数式方法简化与服务器的交互。每个 MilvusClient 不再维护连接池，而是与服务器建立一个 gRPC 连接。
MilvusClient 模块已实现了 ORM 模块的大部分功能。
要了解有关 MilvusClient 模块的更多信息，请访问[pymilvus](https://github.com/milvus-io/pymilvus)和[参考文档](/api-reference/pymilvus/v2.4.x/About.md)。