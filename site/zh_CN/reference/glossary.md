---
id: glossary.md
title: 术语表
---

# 术语表

## AutoID

AutoID 是主字段的一个属性，用于确定是否为主字段启用自动递增。AutoID 的值是基于时间戳定义的。更多信息，请参考 [create_schema](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md)。

## AutoIndex

Milvus 根据经验数据自动确定特定字段的最合适的索引类型和参数。这在您不需要控制特定索引参数的情况下非常理想。更多信息，请参考 [add_index](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/add_index.md)。

## Attu

[Attu](https://github.com/zilliztech/attu) 是 Milvus 的一款全功能管理工具，极大地降低了管理系统的复杂性和成本。

## Birdwatcher

[Birdwatcher](birdwatcher_overview.md) 是 Milvus 的一款调试工具，连接到 etcd，允许您实时监视 Milvus 服务器的状态并进行调整。它还支持 etcd 文件备份，帮助开发人员进行故障排除。

## Bulk Writer

[Bulk Writer](https://milvus.io/api-reference/pymilvus/v2.4.x/DataImport/LocalBulkWriter/LocalBulkWriter.md) 是 Milvus SDK（例如 PyMilvus、Java SDK）提供的数据处理工具，旨在将原始数据集转换为与 Milvus 兼容的格式，以便进行高效导入。

## Bulk Insert

[Bulk Insert](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/utility/do_bulk_insert.md) 是一种 API，通过允许一次性导入多个文件来增强写入性能，优化大型数据集的操作。

## Cardinal

Cardinal 是由 Zilliz Cloud 开发的尖端向量搜索算法，提供无与伦比的搜索质量和性能。凭借其创新设计和广泛优化，Cardinal 在自适应处理各种生产场景方面比 Knowhere 表现出色数倍甚至数量级的优势，例如处理不同的 K 大小、高过滤、不同的数据分布等。

## Channel

Milvus 利用两种类型的通道，[PChannel](https://milvus.io/docs/glossary.md#PChannel) 和 [VChannel](https://milvus.io/docs/glossary.md#VChannel)。每个 PChannel 对应于日志存储的主题，而每个 VChannel 对应于集合中的一个分片。

## Collection

在 Milvus 中，集合相当于关系数据库管理系统（RDBMS）中的表。集合是用于存储和管理实体的主要逻辑对象。更多信息，请参考 [管理集合](manage-collections.md)。

## 依赖（Dependency）
依赖是指一个程序依赖于其他程序才能正常工作。Milvus 的依赖包括 etcd（用于存储元数据）、MinIO 或 S3（对象存储）以及 Pulsar（用于管理快照日志）。更多信息请参考[管理依赖](https://milvus.io/docs/manage_dependencies.md#Manage-Dependencies)。

## 动态模式

动态模式允许您向集合中插入具有新字段的实体，而无需修改现有模式。这意味着您可以在不了解集合的完整模式的情况下插入数据，并且可以包含尚未定义的字段。您可以在创建集合时启用动态字段，从而启用这种无模式的能力。更多信息请参考[启用动态字段](enable-dynamic-field.md)。

## 嵌入

Milvus 提供了与流行的嵌入提供商配合使用的内置嵌入函数。在 Milvus 中创建集合之前，您可以使用这些函数为您的数据集生成嵌入，简化数据准备和向量搜索的过程。要查看嵌入的创建过程，请参考[使用 PyMilvus 的模型生成文本嵌入](https://github.com/milvus-io/bootcamp/blob/master/bootcamp/model/embedding_functions.ipynb)。

## 实体

实体由代表现实世界对象的一组字段组成。Milvus 中的每个实体都由唯一的主键表示。

您可以自定义主键。如果您不手动配置，Milvus 会自动为实体分配主键。如果选择自定义主键，请注意，目前 Milvus 不支持主键去重。因此，同一集合中可能存在重复的主键。更多信息请参考[插入实体](insert-update-delete.md#Insert-entities)。

## 字段

在 Milvus 集合中，字段相当于关系数据库管理系统（RDBMS）中的表列。字段可以是结构化数据的标量字段（例如数字、字符串），也可以是嵌入向量的向量字段。

## 过滤器

Milvus 支持通过使用谓词进行标量过滤，允许您在查询和搜索中定义[过滤条件](https://milvus.io/docs/boolean.md)以细化结果。

## 过滤搜索

过滤搜索将标量过滤器应用于向量搜索，允许您根据特定条件细化搜索结果。更多信息请参考[过滤搜索](single-vector-search.md#Filtered-search)。

## 混合搜索

[混合搜索](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/Collection/hybrid_search.md)是自 Milvus 2.4.0 版本起支持的多向量搜索 API。您可以搜索多个向量字段并将它们融合。结合标量字段过滤的向量搜索称为“过滤搜索”。更多信息请参考[多向量搜索](multi-vector-search.md)。

## 索引
向量索引是从原始数据派生出的重新组织的数据结构，可以极大加速向量相似性搜索的过程。Milvus支持广泛的索引类型，适用于向量字段和标量字段。更多信息请参考[向量索引类型](https://milvus.io/docs/index.md)。

## Kafka-Milvus 连接器

[Kafka-Milvus 连接器](https://github.com/zilliztech/kafka-connect-milvus) 指的是用于 Milvus 的 Kafka sink 连接器。它允许您将向量数据从 Kafka 流式传输到 Milvus。

## Knowhere

[Knowhere](https://milvus.io/docs/knowhere.md#Knowhere) 是 Milvus 的核心向量执行引擎，集成了几个向量相似性搜索库，包括 Faiss、Hnswlib 和 Annoy。Knowhere 还设计为支持异构计算。它控制在哪种硬件（CPU 或 GPU）上执行索引构建和搜索请求。这就是 Knowhere 得名的原因 - 知道在哪里执行操作。

## 日志代理

[日志代理](https://milvus.io/docs/four_layers.md#Log-broker) 是一个支持回放的发布-订阅系统。它负责流式数据持久化、可靠异步查询的执行、事件通知以及查询结果的返回。在工作节点从系统故障中恢复时，它还确保增量数据的完整性。

## 日志快照

日志快照是二进制日志，是记录和处理 Milvus 中数据更新和更改的较小单元。来自段的数据被持久化在多个二进制日志中。Milvus 中有三种类型的二进制日志：InsertBinlog、DeleteBinlog 和 DDLBinlog。更多信息请参考[元数据存储](https://milvus.io/docs/four_layers.md#Meta-storage)。

## 日志订阅者

日志订阅者订阅日志序列以更新本地数据，并以只读副本的形式提供服务。

## 消息存储

消息存储是 Milvus 的日志存储引擎。Milvus 支持 Kafka 或 Pulsa 作为消息存储。更多信息请参考[配置消息存储](https://milvus.io/docs/message_storage_operator.md#Configure-Message-Storage-with-Milvus-Operator)。

## 相似度度量类型

相似度度量类型用于衡量向量之间的相似性。目前，Milvus 支持欧氏距离（L2）、内积（IP）、余弦相似度（COSINE）和二进制度量类型。您可以根据场景选择最合适的相似度度量类型。更多信息请参考[相似度度量](https://milvus.io/docs/metric.md)。

## Mmap
内存映射文件通过将文件内容直接映射到内存中，实现了高效的数据处理。当内存有限且无法加载所有数据时，这种技术尤其有用。这种技术可以提升数据容量并保持性能到一定程度。然而，如果数据远远超过内存容量，搜索和查询速度可能会显著降低。更多信息，请参考[MMap-enabled Data Storage](https://milvus.io/docs/mmap.md)。

## Milvus 备份

[Milvus 备份](https://milvus.io/docs/milvus_backup_overview.md#Milvus-Backup) 是一个用于创建数据副本的工具，可在数据丢失事件后用于恢复原始数据。

## Milvus CDC

[Milvus CDC](https://milvus.io/docs/milvus-cdc-overview.md)（变更数据捕获）是一个用户友好的工具，可以捕获和同步 Milvus 实例中的增量数据。它通过在源实例和目标实例之间无缝传输数据，确保业务数据的可靠性，从而实现轻松的增量备份和灾难恢复。

## Milvus CLI

[Milvus 命令行界面](https://milvus.io/docs/cli_overview.md)（CLI）是一个支持数据库连接、数据操作以及数据导入和导出的命令行工具。基于[Milvus Python SDK](https://github.com/milvus-io/pymilvus)，它允许通过终端使用交互式命令行提示执行命令。

## Milvus 迁移

[Milvus 迁移](https://github.com/zilliztech/milvus-migration/) 是一个旨在简化从各种数据源迁移数据到 Milvus 2.x 的开源工具。

## Milvus 集群

在 Milvus 的[集群部署](https://milvus.io/docs/install_cluster-milvusoperator.md)中，一组节点提供服务，实现高可用性和易扩展性。

## Milvus 独立部署

在 Milvus 的[独立部署](https://milvus.io/docs/install_standalone-docker.md)中，所有操作，包括数据插入、索引构建和向量相似度搜索，均在一个单一进程中完成。

## 多向量

自 2.4.0 版本起，Milvus 支持在一个集合中支持多个向量字段。更多信息，请参考[多向量搜索](multi-vector-search.md)。

## 分区

分区是集合的一个划分。Milvus 支持将集合数据在物理存储上划分为多个部分。这个过程称为分区，每个分区可以包含多个段。更多信息，请参考[管理分区](https://milvus.io/docs/manage-partitions.md#Manage-Partitions)。
分区键属性是一个字段，它基于其分区键值将实体分隔成不同的分区。这种分组确保具有相同键值的实体被存储在一起，这可以通过允许系统在按分区键字段过滤的查询中跳过无关分区来加快搜索操作的速度。更多信息，请参考[使用分区键](https://milvus.io/docs/use-partition-key.md#Use-Partition-Key)。

## PChannel

PChannel代表物理通道。每个PChannel对应于日志存储的一个主题。默认情况下，当Milvus集群启动时，将分配一组16个PChannels来存储记录数据插入、删除和更新的日志。更多信息，请参考[消息通道相关配置](https://milvus.io/docs/configure_messagechannel.md#Message-Channel-related-Configurations)。

## PyMilvus

PyMilvus是Milvus的Python SDK。其源代码是开源的，托管在[GitHub](https://github.com/milvus-io/pymilvus)上。您可以灵活选择MilvusClient（新版本Python SDK）或原始的ORM模块与Milvus进行通信。

## 查询

[查询](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/query.md)是一个API，它使用指定的布尔表达式作为过滤器进行标量过滤。更多信息，请参考[获取和标量查询](https://milvus.io/docs/get-and-scalar-query.md#Use-Basic-Operators)。

## 范围搜索

范围搜索允许您找到与搜索向量距离在指定范围内的向量。更多信息，请参考[范围搜索](https://milvus.io/docs/single-vector-search.md#Range-search)。

## 模式

模式是定义数据类型和数据属性的元信息。每个集合都有自己的集合模式，定义了集合的所有字段、自动ID（主键）分配启用性和集合描述。字段模式也包含在集合模式中，定义了字段的名称、数据类型和其他属性。更多信息，请参考[管理模式](https://milvus.io/docs/schema.md#Manage-Schema)。

## 搜索

[搜索](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/search.md)是一个API，执行一个操作进行向量相似度搜索，执行时需要向量数据。更多信息，请参考[单向量搜索](https://milvus.io/docs/single-vector-search.md)。

## 段

段是自动创建的存储插入数据的数据文件。一个集合可能包含多个段，每个段可以容纳大量实体。在向量相似度搜索期间，Milvus检查每个段以编制搜索结果。
有两种类型的段：增长型和封存型。增长型段会持续收集新数据，直到达到特定阈值或时间限制，之后会变为封存型。一旦封存，段将不再接受新数据，并被转移到对象存储中。同时，新进数据会被路由到一个新的增长型段。从增长型段转变为封存型段的转变是通过达到预定义的实体限制或超过增长状态下允许的最大持续时间来触发的。更多信息请参考[设计细节](https://milvus.io/docs/replica.md#Design-Details)。

## Spark-Milvus 连接器

[Spark-Milvus 连接器](https://github.com/zilliztech/spark-milvus) 提供了 Apache Spark 和 Milvus 之间的无缝集成，结合了 Apache Spark 的数据处理和机器学习（ML）功能与 Milvus 的向量数据存储和搜索能力。

## 分片

Milvus通过使用基于主键哈希的分片将写操作分布到多个节点，从而提高数据写入性能。这利用了集群的并行计算能力。

*分区通过指定分区名称来减少读取负载，而分片则将写入负载分散在多个服务器之间。*

## 稀疏向量

稀疏向量使用向量嵌入来表示单词或短语，其中大多数元素为零，只有一个非零元素表示特定单词的存在。稀疏向量模型，如 SPLADEv2，在领域外知识搜索、关键词感知和可解释性方面优于密集模型。更多信息请参考[稀疏向量](https://milvus.io/docs/sparse_vector.md#Sparse-Vector)。

## 非结构化数据

非结构化数据，包括图像、视频、音频和自然语言，是指不遵循预定义模型或组织方式的信息。这种数据类型约占世界数据的80%，可以使用各种人工智能（AI）和机器学习（ML）模型转换为向量。

## VChannel

[VChannel](https://milvus.io/docs/data_processing.md#Data-insertion) 代表逻辑通道。每个 VChannel 表示集合中的一个分片。每个集合将被分配一组 VChannel 用于记录数据插入、删除和更新。VChannel 在逻辑上是分离的，但在物理上共享资源。

## 向量

嵌入向量是对非结构化数据（如电子邮件、物联网传感器数据、Instagram 照片、蛋白质结构等）的特征抽象。从数学角度来看，嵌入向量是一组浮点数或二进制数的数组。现代嵌入技术用于将非结构化数据转换为嵌入向量。Milvus 从 2.4.0 版本开始支持密集向量和稀疏向量。

## Zilliz 云

[Zilliz 云](https://zilliz.com/) 上的全托管 Milvus，具有更多企业功能和高度优化的性能。