---
id: monitor_overview.md
title: 概述
related_key: 监控, 警报
summary: 了解 Prometheus 和 Grafana 在 Milvus 中如何用于监控和警报服务。
---

# Milvus 监控框架概述

本主题解释了 Milvus 如何使用 Prometheus 监控指标，并使用 Grafana 可视化指标并创建警报。

## Milvus 中的 Prometheus
[Prometheus](https://prometheus.io/docs/introduction/overview/) 是用于 Kubernetes 实现的开源监控和警报工具包。它将指标作为时间序列数据进行收集和存储。这意味着指标在记录时会带有时间戳，并可选择性地附带称为标签的键-值对。

目前 Milvus 使用 Prometheus 的以下组件：
- Prometheus 端点用于从由出口器设置的端点拉取数据。
- Prometheus 运营商用于有效管理 Prometheus 监控实例。
- Kube-prometheus 用于提供易于操作的端到端 Kubernetes 集群监控。

### 指标名称

在 Prometheus 中，有效的指标名称包含三个元素：命名空间、子系统和名称。这三个元素用 "_" 连接。

由 Prometheus 监控的 Milvus 指标的命名空间是 "milvus"。根据指标所属的角色，其子系统应为以下八个角色之一："rootcoord"、"proxy"、"querycoord"、"querynode"、"indexcoord"、"indexnode"、"datacoord"、"datanode"。

例如，用于计算查询的向量总数的 Milvus 指标命名为 `milvus_proxy_search_vectors_count`。

### 指标类型

Prometheus 支持四种类型的指标：

- 计数器：一种累积指标类型，其值只能增加或在重新启动时重置为零。
- 仪表：一种指标类型，其值可以增加或减少。
- 直方图：一种基于可配置桶计数的指标类型。一个常见示例是请求持续时间。
- 摘要：一种类似于直方图的指标类型，它在滑动时间窗口上计算可配置的分位数。

### 指标标签

Prometheus 通过标记区分具有相同指标名称的样本。标签是指标的某个属性。具有相同名称的指标必须对 `variable_labels` 字段具有相同值。以下表格列出了 Milvus 指标常见标签的名称和含义。

| 标签名称 | 定义 | 值 |
|---|---|---|
| "node_id" | 角色的唯一标识符。 | Milvus 生成的全局唯一 ID。 |
| "status" | 处理的操作或请求的状态。 | "abandon"、"success" 或 "fail"。 |
| “query_type” | 读取请求的类型。 | "search" 或 "query"。 |
| "msg_type" | 消息的类型。 | "insert"、"delete"、"search" 或 "query"。 |
| "segment_state" | 段的状态。 | "Sealed"、"Growing"、"Flushed"、"Flushing"、"Dropped" 或 "Importing"。 |
| "cache_state" | 缓存对象的状态。 | "hit" 或 "miss"。 |
| "cache_name" | 缓存对象的名称。此标签与标签“cache_state”一起使用。 | 例如 "CollectionID", "Schema", 等。 |
| “channel_name" | 消息存储中的物理主题（Pulsar 或 Kafka）。 | 例如 "by-dev-rootcoord-dml_0", "by-dev-rootcoord-dml_255", 等。 |
| "function_name" | 处理特定请求的函数名称。 | 例如 "CreateCollection", "CreatePartition", "CreateIndex", 等。 |
| "user_name" | 用于身份验证的用户名。 | 任意您喜欢的用户名。 |
| "index_task_status" | 元存储中索引任务的状态。 | "unissued", "in-progress", "failed", "finished", 或 "recycled"。 |

## Milvus 中的 Grafana
[Grafana](https://grafana.com/docs/grafana/latest/introduction/) 是一个开源的可视化堆栈，可以连接所有数据源。通过提取指标，它帮助用户理解、分析和监控海量数据。

Milvus 使用 Grafana 的可定制仪表板进行指标可视化。

## 接下来做什么
在了解了监控和警报的基本工作流程之后，学习：
- [部署监控服务](monitor.md)
- [可视化 Milvus 指标](visualize.md)
- [创建警报](alert.md)