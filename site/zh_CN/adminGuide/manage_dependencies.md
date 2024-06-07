---
id: manage_dependencies.md
title: 管理依赖
---

# 管理依赖

Milvus利用第三方依赖项来进行对象、元数据和消息存储。有两种主要方式来配置第三方依赖。

- 对象存储：Milvus支持使用MinIO或S3进行对象存储。
  - [使用Docker Compose/Helm配置对象存储](deploy_s3.md)
  - [使用Milvus Operator配置对象存储](object_storage_operator.md)
- 元数据存储：Milvus使用etcd进行元数据存储。
  - [使用Docker Compose/Helm配置元数据存储](deploy_etcd.md)
  - [使用Milvus Operator配置元数据存储](meta_storage_operator.md)
- 消息存储：Milvus支持使用Pulsar、Kafka或RocksMQ进行消息存储。
  - [使用Docker Compose/Helm配置消息存储](deploy_pulsar.md)
  - [使用Milvus Operator配置消息存储](message_storage_operator.md)
