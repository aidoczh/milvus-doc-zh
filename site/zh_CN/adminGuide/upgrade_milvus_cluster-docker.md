---
id: upgrade_milvus_cluster-docker.md
summary: 学习如何使用 Docker Compose 升级 Milvus 集群。
title: 使用 Docker Compose 升级 Milvus 集群
---

{{tab}}

# 使用 Docker Compose 升级 Milvus 集群

本主题描述了如何使用 Docker Compose 升级您的 Milvus。

在正常情况下，您可以通过更改其镜像来[升级 Milvus](#Upgrade-Milvus-by-changing-its-image)。但是，在从 v2.1.x 升级到 v{{var.milvus_release_version}} 之前，您需要[迁移元数据](#Migrate-the-metadata)。

## 通过更改其镜像升级 Milvus

在正常情况下，您可以按照以下步骤升级 Milvus：

1. 更改 `docker-compose.yaml` 中的 Milvus 镜像标签。

    请注意，您需要为代理、所有协调器和所有工作节点更改镜像标签。

    ```yaml
    ...
    rootcoord:
      container_name: milvus-rootcoord
      image: milvusdb/milvus:v{{var.milvus_release_version}}
    ...
    proxy:
      container_name: milvus-proxy
      image: milvusdb/milvus:v{{var.milvus_release_version}}
    ...
    querycoord:
      container_name: milvus-querycoord
      image: milvusdb/milvus:v{{var.milvus_release_version}}  
    ...
    querynode:
      container_name: milvus-querynode
      image: milvusdb/milvus:v{{var.milvus_release_version}}
    ...
    indexcoord:
      container_name: milvus-indexcoord
      image: milvusdb/milvus:v{{var.milvus_release_version}}
    ...
    indexnode:
      container_name: milvus-indexnode
      image: milvusdb/milvus:v{{var.milvus_release_version}} 
    ...
    datacoord:
      container_name: milvus-datacoord
      image: milvusdb/milvus:v{{var.milvus_release_version}}   
    ...
    datanode:
      container_name: milvus-datanode
      image: milvusdb/milvus:v{{var.milvus_release_version}}
    ```

2. 运行以下命令执行升级。

    ```shell
    docker compose down
    docker compose up -d
    ```

## 迁移元数据

1. 停止所有 Milvus 组件。

    ```
    docker stop <milvus-component-docker-container-name>
    ```

2. 为元数据迁移准备配置文件 `migrate.yaml`。

    ```yaml
    # migration.yaml
    cmd:
      # Option: run/backup/rollback
      type: run
      runWithBackup: true
    config:
      sourceVersion: 2.1.4   # 指定您的 Milvus 版本
      targetVersion: {{var.milvus_release_version}}
      backupFilePath: /tmp/migration.bak
    metastore:
      type: etcd
    etcd:
      endpoints:
        - milvus-etcd:2379  # 使用 etcd 容器名称
      rootPath: by-dev # 数据存储在 etcd 中的根路径
      metaSubPath: meta
      kvSubPath: kv
    ```

3. 运行迁移容器。

    ```bash
    # 假设您的 docker-compose 与默认的 milvus 网络一起运行，
    # 并且您将 migration.yaml 放在与 docker-compose.yaml 相同的目录中。
    
    docker run --rm -it --network milvus -v $(pwd)/migration.yaml:/milvus/configs/migration.yaml milvus/meta-migration:v2.2.0 /milvus/bin/meta-migration -config=/milvus/configs/migration.yaml
	```

4. 使用新的 Milvus 镜像再次启动 Milvus 组件。

```
#在 docker-compose.yaml 中更新 milvus 镜像标签
docker compose down
docker compose up -d
```

## 接下来做什么
- 您可能还想了解如何：
  - [扩展 Milvus 集群](scaleout.md)
- 如果您准备在云上部署您的集群：
  - 学习如何使用 Terraform 和 Ansible 在 AWS 上部署 Milvus：[在 AWS 上使用 Terraform 和 Ansible 部署 Milvus](aws.md)
  - 学习如何使用 Terraform 在 Amazon EKS 上部署 Milvus：[在 Amazon EKS 上使用 Terraform 部署 Milvus](eks.md)
  - 学习如何在 GCP 上使用 Kubernetes 部署 Milvus 集群：[在 GCP 上使用 Kubernetes 部署 Milvus](gcp.md)
  - 学习如何在 Microsoft Azure 上使用 Kubernetes 部署 Milvus：[在 Microsoft Azure 上使用 Kubernetes 部署 Milvus](azure.md)
```