---
id: install_standalone-docker.md
label: Docker
related_key: Docker
summary: 学习如何使用 Docker 在 Milvus 中运行独立实例。
title: 在 Docker 中运行 Milvus
---

# 在 Docker 中运行 Milvus

本页面介绍如何在 Docker 中启动 Milvus 实例。


## 先决条件

- [安装 Docker](https://docs.docker.com/get-docker/)。
- 在安装之前，请查看硬件和软件要求[prerequisite-docker.md](prerequisite-docker.md)。


## 在 Docker 中安装 Milvus

Milvus 提供了一个安装脚本，可以将其安装为 Docker 容器。该脚本可在[Milvus 仓库](https://raw.githubusercontent.com/milvus-io/milvus/master/scripts/standalone_embed.sh)中找到。要在 Docker 中安装 Milvus，只需运行

```shell
# 下载安装脚本
$ curl -sfL https://raw.githubusercontent.com/milvus-io/milvus/master/scripts/standalone_embed.sh

# 启动 Docker 容器
$ bash standalone_embed.sh start
```

运行安装脚本后：

- 名为 milvus 的 Docker 容器已在端口 **19530** 启动。
- 与 Milvus 一起安装了一个嵌入式 etcd，位于同一容器中，并在端口 **2379** 上提供服务。其配置文件映射到当前文件夹中的 **embedEtcd.yaml**。
- Milvus 数据卷映射到当前文件夹中的 **volumes/milvus**。

您可以按以下方式停止和删除此容器

```shell
# 停止 Milvus
$ bash standalone_embed.sh stop

# 删除 Milvus 数据
$ bash standalone_embed.sh stop
```

## 下一步操作

在 Docker 中安装了 Milvus 后，您可以：

- 查看[快速入门](quickstart.md)以了解 Milvus 的功能。

- 学习 Milvus 的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- 使用 Helm Chart [升级 Milvus](upgrade_milvus_cluster-helm.md)。
- [扩展您的 Milvus 集群](scaleout.md)。
- 在云上部署您的 Milvus 集群：
  - [Amazon EC2](aws.md)
  - [Amazon EKS](eks.md)
  - [Google Cloud](gcp.md)
  - [Google Cloud 存储](gcs.md)
  - [Microsoft Azure](azure.md)
  - [Microsoft Azure Blob 存储](abs.md)
- 探索[Milvus Backup](milvus_backup_overview.md)，这是一个用于 Milvus 数据备份的开源工具。
- 探索[Birdwatcher](birdwatcher_overview.md)，这是一个用于调试 Milvus 和动态配置更新的开源工具。
- 探索[Attu](https://milvus.io/docs/attu.md)，这是一个直观的 Milvus 管理工具。
- 使用 Prometheus [监控 Milvus](monitor.md)。