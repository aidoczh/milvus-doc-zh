---
id: install_standalone-docker-compose.md
label: Docker Compose
related_key: Docker Compose
summary: 学习如何使用 Docker Compose 在独立模式下安装 Milvus。
title: 使用 Docker Compose 运行 Milvus
---

# 使用 Docker Compose 运行 Milvus

本页面介绍如何使用 Docker Compose 在 Docker 中启动 Milvus 实例。

## 先决条件

- [安装 Docker](https://docs.docker.com/get-docker/)。
- 在安装之前，请查看硬件和软件要求[prerequisite-docker.md](prerequisite-docker.md)。

## 安装 Milvus

Milvus 在其存储库中提供了一个 Docker Compose 配置文件。要使用 Docker Compose 安装 Milvus，只需运行

```shell
# 下载配置文件
$ wget https://github.com/milvus-io/milvus/releases/download/v{{var.milvus_release_version}}/milvus-standalone-docker-compose.yml -O docker-compose.yml

# 启动 Milvus
$ sudo docker compose up -d

Creating milvus-etcd  ... done
Creating milvus-minio ... done
Creating milvus-standalone ... done
```

<div class="alert note">

如果无法运行上述命令，请检查系统是否安装了 Docker Compose V1。如果是这种情况，请根据[此页面](https://docs.docker.com/compose/)上的说明迁移到 Docker Compose V2。

</div>

启动 Milvus 后，

- 名为 **milvus-standalone**、**milvus-minio** 和 **milvus-etcd** 的容器已启动。
  - **milvus-etcd** 容器不向主机公开任何端口，并将其数据映射到当前文件夹中的 **volumes/etcd**。
  - **milvus-minio** 容器在本地提供端口 **9090** 和 **9091**，使用默认身份验证凭据，并将其数据映射到当前文件夹中的 **volumes/minio**。
  - **milvus-standalone** 容器在本地提供端口 **19530**，使用默认设置，并将其数据映射到当前文件夹中的 **volumes/milvus**。

您可以使用以下命令检查容器是否正在运行：

```shell
$ sudo docker compose ps

      Name                     Command                  State                            Ports
--------------------------------------------------------------------------------------------------------------------
milvus-etcd         etcd -advertise-client-url ...   Up             2379/tcp, 2380/tcp
milvus-minio        /usr/bin/docker-entrypoint ...   Up (healthy)   9000/tcp
milvus-standalone   /tini -- milvus run standalone   Up             0.0.0.0:19530->19530/tcp, 0.0.0.0:9091->9091/tcp
```

您可以按以下方式停止和删除此容器

```shell
# 停止 Milvus
$ sudo docker compose down

# 删除服务数据
$ sudo rm -rf volumes
```

## 接下来做什么

在 Docker 中安装了 Milvus 后，您可以：

- 查看[快速入门](quickstart.md)以了解 Milvus 的功能。

- 学习 Milvus 的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
- [单向量搜索](single-vector-search.md)
- [多向量搜索](multi-vector-search.md)

- [使用 Helm Chart 升级 Milvus](upgrade_milvus_cluster-helm.md)。
- [扩展你的 Milvus 集群](scaleout.md)。
- 在云上部署你的 Milvus 集群：
  - [Amazon EC2](aws.md)
  - [Amazon EKS](eks.md)
  - [Google Cloud](gcp.md)
  - [Google Cloud 存储](gcs.md)
  - [Microsoft Azure](azure.md)
  - [Microsoft Azure Blob 存储](abs.md)
- 探索 [Milvus 备份](milvus_backup_overview.md)，这是一个用于 Milvus 数据备份的开源工具。
- 探索 [Birdwatcher](birdwatcher_overview.md)，这是一个用于调试 Milvus 和动态配置更新的开源工具。
- 探索 [Attu](https://milvus.io/docs/attu.md)，这是一个直观的 Milvus 管理的开源 GUI 工具。
- [使用 Prometheus 监控 Milvus](monitor.md)。