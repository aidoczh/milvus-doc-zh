---
id: architecture_overview.md
summary: Milvus提供了一个专门用于相似度搜索和人工智能的快速、可靠和稳定的向量数据库。
title: Milvus架构概述
---

# Milvus架构概述

Milvus基于流行的向量搜索库，包括Faiss、HNSW、DiskANN、SCANN等，专为包含数百万、数十亿甚至数万亿向量的稠密向量数据集设计了相似度搜索功能。在继续之前，请熟悉嵌入式检索的[基本原理](glossary.md)。

Milvus还支持数据分片、流式数据摄入、动态模式、搜索组合向量和标量数据、多向量和混合搜索、稀疏向量等许多高级功能。该平台提供按需性能，并可进行优化以适应任何嵌入式检索场景。我们建议使用Kubernetes部署Milvus以获得最佳的可用性和弹性。

Milvus采用共享存储架构，具有存储和计算分离以及计算节点的水平可伸缩性。遵循数据平面和控制平面分离的原则，Milvus包括[四个层次](four_layers.md)：访问层、协调服务、工作节点和存储。在扩展或灾难恢复时，这些层次彼此独立。

![架构图](../../../../assets/milvus_architecture.png "Milvus架构。")


## 接下来的步骤

- 了解Milvus中的[计算/存储分离](four_layers.md)
- 了解Milvus中的[主要组件](main_components.md)。