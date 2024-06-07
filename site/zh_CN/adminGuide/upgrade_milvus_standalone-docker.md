---
id: upgrade_milvus_standalone-docker.md
label: Docker Compose
order: 1
group: upgrade_milvus_standalone-operator.md
related_key: upgrade Milvus Standalone
summary: 学习如何使用 Docker Compose 升级 Milvus 独立部署版。
title: 使用 Docker Compose 升级 Milvus 独立部署版
---

{{tab}}

# 使用 Docker Compose 升级 Milvus 独立部署版

本主题描述了如何使用 Docker Compose 升级您的 Milvus。

通常情况下，您可以通过[更改其镜像来升级 Milvus](#Upgrade-Milvus-by-changing-its-image)。但是，在从 v2.1.x 升级到 v{{var.milvus_release_version}} 之前，您需要[迁移元数据](#Migrate-the-metadata)。

<div class="alter note">

由于安全原因，Milvus 在 v2.2.5 版本中将其 MinIO 升级到 RELEASE.2023-03-20T20-16-18Z。在从之前使用 Docker Compose 安装的 Milvus 独立部署版进行任何升级之前，您应创建一个单节点单驱动器的 MinIO 部署，并将现有的 MinIO 设置和内容迁移到新部署中。详情请参考[此指南](https://min.io/docs/minio/linux/operations/install-deploy-manage/migrate-fs-gateway.html#id2)。

</div>

## 通过更改其镜像升级 Milvus

通常情况下，您可以按以下步骤升级 Milvus：

1. 在 `docker-compose.yaml` 中更改 Milvus 镜像标签。

    ```yaml
    ...
    standalone:
      container_name: milvus-standalone
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

2. 准备用于元数据迁移的配置文件 `migration.yaml`。

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

    ```
    # 假设您的 docker-compose 与默认的 milvus 网络一起运行，
    # 并且您将 migration.yaml 放在与 docker-compose.yaml 相同的目录中。
    docker run --rm -it --network milvus -v $(pwd)/migration.yaml:/milvus/configs/migration.yaml milvusdb/meta-migration:v2.2.0 /milvus/bin/meta-migration -config=/milvus/configs/migration.yaml
    ```

4. 使用新的 Milvus 镜像重新启动 Milvus 组件。

    ```shell
    // 仅在更新 docker-compose.yaml 中的 milvus 镜像标签后运行以下命令
    docker compose down
    docker compose up -d
    ```

## 下一步操作
- 您可能还想了解如何：
- [扩展 Milvus 集群](scaleout.md)
- 如果您准备在云上部署您的集群：
  - 学习如何使用 Terraform 和 Ansible 在 AWS 上部署 Milvus [在 AWS 上使用 Terraform 和 Ansible 部署 Milvus](aws.md)
  - 学习如何使用 Terraform 在 Amazon EKS 上部署 Milvus [在 Amazon EKS 上使用 Terraform 部署 Milvus](eks.md)
  - 学习如何在 GCP 上使用 Kubernetes 部署 Milvus 集群 [在 GCP 上使用 Kubernetes 部署 Milvus 集群](gcp.md)
  - 学习如何在 Microsoft Azure 上使用 Kubernetes 部署 Milvus [在 Microsoft Azure 上使用 Kubernetes 部署 Milvus](azure.md)