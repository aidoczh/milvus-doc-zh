---
id: architecture.md
title: 架构
---

# 架构

作为一款云原生向量数据库，Milvus 通过设计将存储和计算分离。为了增强弹性和灵活性，Milvus 中的所有组件都是无状态的。

- [Milvus 架构概述](architecture_overview.md): Milvus 采用了共享存储架构，具有存储/计算分离和计算节点可扩展性的特点。

- [存储/计算分离](four_layers.md): Milvus 包括四个层，这些层在可扩展性和灾难恢复方面是相互独立的。

- [主要组件](main_components.md): Milvus 独立版包括三个组件，而 Milvus 集群版包括八个微服务组件和三个第三方依赖。

- [数据处理](data_processing.md): 对 Milvus 中数据插入、索引构建和数据查询实现的详细描述。