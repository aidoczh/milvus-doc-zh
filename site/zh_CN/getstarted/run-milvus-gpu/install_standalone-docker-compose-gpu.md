---
id: install_standalone-docker-compose-gpu.md
label: 独立部署 (Docker Compose)
related_key: Kubernetes
summary: 学习如何在 Kubernetes 上安装 Milvus 集群。
title: 使用 Docker Compose 运行支持 GPU 的 Milvus
---

# 使用 Docker Compose 运行支持 GPU 的 Milvus

本页面介绍如何使用 Docker Compose 启动支持 GPU 的 Milvus 实例。

## 先决条件

- [安装 Docker](https://docs.docker.com/get-docker/)。
- 在安装之前，请查看硬件和软件要求，请参考[此处](prerequisite-gpu.md)。

## 安装 Milvus

要使用 Docker Compose 安装支持 GPU 的 Milvus，请按照以下步骤操作。

### 1. 下载并配置 YAML 文件

下载 [`milvus-standalone-docker-compose-gpu.yml`](https://github.com/milvus-io/milvus/releases/download/v{{var.milvus_release_version}}/milvus-standalone-docker-compose-gpu.yml) 文件，并手动保存为 docker-compose.yml，或使用以下命令。

```shell
$ wget https://github.com/milvus-io/milvus/releases/download/v{{var.milvus_release_version}}/milvus-standalone-docker-compose-gpu.yml -O docker-compose.yml
```

您需要对 YAML 文件中独立服务的环境变量进行一些更改，如下所示：

- 要为 Milvus 指定特定的 GPU 设备，请找到 `standalone` 服务定义中的 `deploy.resources.reservations.devices[0].devices_ids` 字段，并将其值替换为所需 GPU 的 ID。您可以使用随 NVIDIA GPU 显示驱动程序一起提供的 `nvidia-smi` 工具来确定 GPU 设备的 ID。Milvus 支持多个 GPU 设备。

- 设置用于 GPU 索引的内存池大小，其中 `initialSize` 表示内存池的初始大小，`maximumSize` 表示其最大大小。这两个值应该是以 MB 为单位的整数。Milvus 使用这些字段为每个进程分配显示内存。

为 Milvus 指定单个 GPU 设备：

```yaml
...
standalone:
  gpu:
    initMemSize: 0
    maxMemSize: 1024
  ...
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            capabilities: ["gpu"]
            device_ids: ["0"]
...
```

为 Milvus 指定多个 GPU 设备：

```yaml
...
standalone:
  gpu:
    initMemSize: 0
    maxMemSize: 1024
  ...
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            capabilities: ["gpu"]
            device_ids: ['0', '1']
...
```

### 2. 启动 Milvus

在包含 docker-compose.yml 文件的目录中，通过以下命令启动 Milvus：

```shell
$ sudo docker compose up -d

Creating milvus-etcd  ... done
Creating milvus-minio ... done
Creating milvus-standalone ... done
```

<div class="alert note">

如果无法运行上述命令，请检查您的系统是否安装了 Docker Compose V1。如果是这种情况，建议您迁移到 Docker Compose V2，具体原因请参考[此页面](https://docs.docker.com/compose/)。

</div>

启动 Milvus 后，
- 名为**milvus-standalone**、**milvus-minio**和**milvus-etcd**的容器已经启动。
  - **milvus-etcd**容器不向主机暴露任何端口，将其数据映射到当前文件夹中的**volumes/etcd**。
  - **milvus-minio**容器在本地使用默认身份验证凭据提供端口**9090**和**9091**，将其数据映射到当前文件夹中的**volumes/minio**。
  - **milvus-standalone**容器在本地使用默认设置提供端口**19530**，将其数据映射到当前文件夹中的**volumes/milvus**。

您可以使用以下命令检查容器是否正在运行：

```shell
$ sudo docker compose ps

      Name                     Command                  State                            Ports
--------------------------------------------------------------------------------------------------------------------
milvus-etcd         etcd -advertise-client-url ...   Up             2379/tcp, 2380/tcp
milvus-minio        /usr/bin/docker-entrypoint ...   Up (healthy)   9000/tcp
milvus-standalone   /tini -- milvus run standalone   Up             0.0.0.0:19530->19530/tcp, 0.0.0.0:9091->9091/tcp
```

如果您在docker-compose.yml中为Milvus分配了多个GPU设备，您可以指定哪个GPU设备对Milvus可见或可用。

使GPU设备`0`对Milvus可见：

```shell
$ CUDA_VISIBLE_DEVICES=0 ./milvus run standalone
```

使GPU设备`0`和`1`对Milvus可见：

```shell
$ CUDA_VISIBLE_DEVICES=0,1 ./milvus run standalone
```

您可以按以下方式停止和删除此容器。

```shell
# 停止 Milvus
$ sudo docker compose down

# 删除服务数据
$ sudo rm -rf volumes
```

## 下一步

在Docker中安装了Milvus后，您可以：

- 查看[快速入门](quickstart.md)以了解Milvus的功能。

- 学习Milvus的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- [使用Helm Chart升级Milvus](upgrade_milvus_cluster-helm.md)。
- [扩展您的Milvus集群](scaleout.md)。
- 在云上部署您的Milvus集群：
  - [Amazon EC2](aws.md)
  - [Amazon EKS](eks.md)
  - [Google Cloud](gcp.md)
  - [Google Cloud存储](gcs.md)
  - [Microsoft Azure](azure.md)
  - [Microsoft Azure Blob存储](abs.md)
- 探索[Milvus备份](milvus_backup_overview.md)，这是一个用于Milvus数据备份的开源工具。
- 探索[Birdwatcher](birdwatcher_overview.md)，这是一个用于调试Milvus和动态配置更新的开源工具。
- 探索[Attu](https://milvus.io/docs/attu.md)，这是一个直观的Milvus管理的开源GUI工具。
- [使用Prometheus监控Milvus](monitor.md)。