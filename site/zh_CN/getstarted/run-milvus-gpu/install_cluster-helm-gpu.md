---
id: install_cluster-helm-gpu.md
label: 集群 (Helm)
related_key: Kubernetes
summary: 学习如何在 Kubernetes 上安装 Milvus 集群。
title: 使用 Helm Chart 运行支持 GPU 的 Milvus
---

# 使用 Helm Chart 运行支持 GPU 的 Milvus

本页面介绍如何使用 Helm Chart 启动支持 GPU 的 Milvus 实例。

## 概述

Helm 使用一种名为 Chart 的打包格式。Chart 是一组文件，用于描述一组相关的 Kubernetes 资源。Milvus 提供了一组 Charts 来帮助您部署 Milvus 的依赖和组件。[Milvus Helm Chart](https://artifacthub.io/packages/helm/milvus-helm/milvus) 是一个解决方案，使用 Helm 软件包管理器在 Kubernetes (K8s) 集群上引导 Milvus 部署。

## 先决条件

- [安装 Helm CLI](https://helm.sh/docs/intro/install/)。
- [创建带有 GPU 工作节点的 K8s 集群](prerequisite-gpu.md#How-can-I-start-a-K8s-cluster-with-GPU-worker-nodes)。
- 安装 [StorageClass](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/)。您可以按如下方式检查已安装的 StorageClass。

    ```bash
    $ kubectl get sc

    NAME                  PROVISIONER                  RECLAIMPOLICY    VOLUMEBIINDINGMODE    ALLOWVOLUMEEXPANSION     AGE
    standard (default)    k8s.io/minikube-hostpath     Delete           Immediate             false 
    ```

- 在安装之前，请检查[硬件和软件要求](prerequisite-gpu.md)。

## 为 Milvus 安装 Helm Chart

Helm 是一个 K8s 软件包管理器，可以帮助您快速部署 Milvus。

1. 添加 Milvus Helm 仓库。

```
$ helm repo add milvus https://zilliztech.github.io/milvus-helm/
```

<div class="alert note">

Milvus Helm Charts 仓库位于 `https://milvus-io.github.io/milvus-helm/`，已存档，您可以从 `https://zilliztech.github.io/milvus-helm/` 获取进一步更新，操作如下：

```shell
helm repo add zilliztech https://zilliztech.github.io/milvus-helm
helm repo update
# 升级现有的 Helm 发布
helm upgrade my-release zilliztech/milvus
```

存档的仓库仍可提供 4.0.31 版本之前的图表。对于更新版本，请改用新仓库。

</div>

2. 在本地更新 Charts。

```
$ helm repo update
```

## 启动 Milvus

安装 Helm Chart 后，您可以在 Kubernetes 上启动 Milvus。在本节中，我们将指导您完成启动支持 GPU 的 Milvus 的步骤。

您应该通过指定发布名称、Chart 和您期望更改的参数来使用 Helm 启动 Milvus。在本指南中，我们使用 <code>my-release</code> 作为发布名称。要使用不同的发布名称，请将以下命令中的 <code>my-release</code> 替换为您正在使用的名称。

Milvus 允许您为 Milvus 指定一个或多个 GPU 设备。

### 1. 指定单个 GPU 设备

支持 GPU 的 Milvus 允许您为 Milvus 指定一个或多个 GPU 设备。

- Milvus 集群

  ```bash
  cat <<EOF > custom-values.yaml
  indexNode:
    resources:  
```bash
      requests:
        nvidia.com/gpu: "1"
      limits:
        nvidia.com/gpu: "1"
  queryNode:
    resources:
      requests:
        nvidia.com/gpu: "1"
      limits:
        nvidia.com/gpu: "1"
  EOF
```

  ```bash
  $ helm install my-release milvus/milvus -f custom-values.yaml
  ```

- Milvus 独立模式

  ```bash
  cat <<EOF > custom-values.yaml
  standalone:
    resources:
      requests:
        nvidia.com/gpu: "1"
      limits:
        nvidia.com/gpu: "1"
  EOF
  ```

  ```bash
  $ helm install my-release milvus/milvus --set cluster.enabled=false --set etcd.replicaCount=1 --set minio.mode=standalone --set pulsar.enabled=false -f custom-values.yaml
  ```

### 2. 分配多个 GPU 设备

除了单个 GPU 设备外，您还可以将多个 GPU 设备分配给 Milvus。

- Milvus 集群模式

  ```bash
  cat <<EOF > custom-values.yaml
  indexNode:
    resources:
      requests:
        nvidia.com/gpu: "2"
      limits:
        nvidia.com/gpu: "2"
  queryNode:
    resources:
      requests:
        nvidia.com/gpu: "2"
      limits:
        nvidia.com/gpu: "2"
  EOF
  ```

  在上述配置中，indexNode 和 queryNode 共享两个 GPU。要将不同的 GPU 分配给 indexNode 和 queryNode，您可以相应地修改配置，通过在配置文件中设置 `extraEnv` 如下所示：

  ```bash
  cat <<EOF > custom-values.yaml
  indexNode:
    resources:
      requests:
        nvidia.com/gpu: "1"
      limits:
        nvidia.com/gpu: "1"
    extraEnv:
      - name: CUDA_VISIBLE_DEVICES
        value: "0"
  queryNode:
    resources:
      requests:
        nvidia.com/gpu: "1"
      limits:
        nvidia.com/gpu: "1"
    extraEnv:
      - name: CUDA_VISIBLE_DEVICES
        value: "1"
  EOF
  ```

  ```bash
  $ helm install my-release milvus/milvus -f custom-values.yaml
  ```

  <div class="alert note">
    <ul>
      <li>发布名称应仅包含字母、数字和破折号。发布名称中不允许使用点。</li>
      <li>默认命令行在使用 Helm 安装 Milvus 时安装 Milvus 的集群版本。在安装 Milvus 独立模式时需要进一步设置。</li>
      <li>根据 Kubernetes 的<a href="https://kubernetes.io/docs/reference/using-api/deprecation-guide/#v1-25">弃用 API 迁移指南</a>，PodDisruptionBudget 的 <b>policy/v1beta1</b> API 版本在 v1.25 之后不再提供服务。建议您将清单文件和 API 客户端迁移到使用 <b>policy/v1</b> API 版本。 <br>对于仍在 Kubernetes v1.25 及更高版本上使用 PodDisruptionBudget 的 <b>policy/v1beta1</b> API 版本的用户，您可以运行以下命令来安装 Milvus：<br>
      <code>helm install my-release milvus/milvus --set pulsar.bookkeeper.pdb.usePolicy=false,pulsar.broker.pdb.usePolicy=false,pulsar.proxy.pdb.usePolicy=false,pulsar.zookeeper.pdb.usePolicy=false</code></li> 
      <li>查看 <a href="https://artifacthub.io/packages/helm/milvus/milvus">Milvus Helm Chart</a> 和 <a href="https://helm.sh/docs/">Helm</a> 获取更多信息。</li>
    </ul>
  </div>

- Milvus 独立部署

  ```bash
  cat <<EOF > custom-values.yaml
  indexNode:
    resources:
      requests:
        nvidia.com/gpu: "2"
      limits:
        nvidia.com/gpu: "2"
  queryNode:
    resources:
      requests:
        nvidia.com/gpu: "2"
      limits:
        nvidia.com/gpu: "2"
  EOF
  ```

  在上述配置中，indexNode 和 queryNode 共享两个 GPU。要为 indexNode 和 queryNode 分配不同的 GPU，可以通过修改配置文件中的 extraEnv 来设置额外的环境变量，如下所示：

  ```bash
  cat <<EOF > custom-values.yaml
  indexNode:
    resources:
      requests:
        nvidia.com/gpu: "1"
      limits:
        nvidia.com/gpu: "1"
    extraEnv:
      - name: CUDA_VISIBLE_DEVICES
        value: "0"
  queryNode:
    resources:
      requests:
        nvidia.com/gpu: "1"
      limits:
        nvidia.com/gpu: "1"
    extraEnv:
      - name: CUDA_VISIBLE_DEVICES
        value: "1"
  EOF
  ```

  ```bash
  $ helm install my-release milvus/milvus --set cluster.enabled=false --set etcd.replicaCount=1 --set minio.mode=standalone --set pulsar.enabled=false -f custom-values.yaml
  ```

### 2. 检查 Milvus 状态

运行以下命令检查 Milvus 状态：

```bash
$ kubectl get pods
```

Milvus 启动后，所有 pod 的 `READY` 列都会显示 `1/1`。

- Milvus 集群

  ```shell
  NAME                                             READY  STATUS   RESTARTS  AGE
  my-release-etcd-0                                1/1    Running   0        3m23s
  my-release-etcd-1                                1/1    Running   0        3m23s
  my-release-etcd-2                                1/1    Running   0        3m23s
  my-release-milvus-datacoord-6fd4bd885c-gkzwx     1/1    Running   0        3m23s
  my-release-milvus-datanode-68cb87dcbd-4khpm      1/1    Running   0        3m23s
  my-release-milvus-indexcoord-5bfcf6bdd8-nmh5l    1/1    Running   0        3m23s
  my-release-milvus-indexnode-5c5f7b5bd9-l8hjg     1/1    Running   0        3m24s
  my-release-milvus-proxy-6bd7f5587-ds2xv          1/1    Running   0        3m24s
  my-release-milvus-querycoord-579cd79455-xht5n    1/1    Running   0        3m24s
  my-release-milvus-querynode-5cd8fff495-k6gtg     1/1    Running   0        3m24s
  my-release-milvus-rootcoord-7fb9488465-dmbbj     1/1    Running   0        3m23s
  my-release-minio-0                               1/1    Running   0        3m23s
  my-release-minio-1                               1/1    Running   0        3m23s
  my-release-minio-2                               1/1    Running   0        3m23s
  my-release-minio-3                               1/1    Running   0        3m23s
  my-release-pulsar-autorecovery-86f5dbdf77-lchpc  1/1    Running   0        3m24s
### Pulsar Bookkeeper

```shell
my-release-pulsar-bookkeeper-0                   1/1    Running   0        3m23s
my-release-pulsar-bookkeeper-1                   1/1    Running   0        98s
```

### Pulsar Broker

```shell
my-release-pulsar-broker-556ff89d4c-2m29m        1/1    Running   0        3m23s
```

### Pulsar Proxy

```shell
my-release-pulsar-proxy-6fbd75db75-nhg4v         1/1    Running   0        3m23s
```

### Pulsar Zookeeper

```shell
my-release-pulsar-zookeeper-0                    1/1    Running   0        3m23s
my-release-pulsar-zookeeper-metadata-98zbr       0/1    Completed 0        3m24s
```

### Milvus Standalone

```shell
NAME                                               READY   STATUS      RESTARTS   AGE
my-release-etcd-0                                  1/1     Running     0          30s
my-release-milvus-standalone-54c4f88cb9-f84pf      1/1     Running     0          30s
my-release-minio-5564fbbddc-mz7f5                  1/1     Running     0          30s
```

### 3. 将本地端口转发到 Milvus

验证 Milvus 服务器正在侦听的本地端口。用您自己的 Pod 名称替换下面的命令。

```bash
$ kubectl get pod my-release-milvus-proxy-6bd7f5587-ds2xv --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
19530
```

然后，运行以下命令将本地端口转发到 Milvus 服务端口。

```bash
$ kubectl port-forward service/my-release-milvus 27017:19530
Forwarding from 127.0.0.1:27017 -> 19530
```

可选地，您可以在上述命令中使用 `:19530` 而不是 `27017:19530`，让 `kubectl` 为您分配一个本地端口，这样您就不必管理端口冲突。

默认情况下，kubectl 的端口转发仅在 `localhost` 上侦听。如果希望 Milvus 在所选或所有 IP 地址上侦听，请使用 `address` 标志。以下命令使端口转发在主机机器上的所有 IP 地址上侦听。

```bash
$ kubectl port-forward --address 0.0.0.0 service/my-release-milvus 27017:19530
Forwarding from 0.0.0.0:27017 -> 19530
```

## 卸载 Milvus

运行以下命令以卸载 Milvus。

```bash
$ helm uninstall my-release
```

## 接下来做什么

安装了 Milvus 后，您可以：

- 查看 [快速入门](quickstart.md) 以了解 Milvus 的功能。

- 学习 Milvus 的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- [使用 Helm Chart 升级 Milvus](upgrade_milvus_cluster-helm.md)。
- [扩展您的 Milvus 集群](scaleout.md)。
- 在云上部署您的 Milvus 集群：
  - [Amazon EC2](aws.md)
  - [Amazon EKS](eks.md)
  - [Google Cloud](gcp.md)
  - [Google Cloud 存储](gcs.md)
  - [Microsoft Azure](azure.md)
  - [Microsoft Azure Blob 存储](abs.md)
- 探索 [Milvus Backup](milvus_backup_overview.md)，这是一个用于 Milvus 数据备份的开源工具。
- 探索[Birdwatcher](birdwatcher_overview.md)，这是一个用于调试Milvus和动态配置更新的开源工具。
- 探索[Attu](https://milvus.io/docs/attu.md)，这是一个直观的Milvus管理工具，提供开源的图形用户界面。
- 使用Prometheus监控Milvus的情况，详见[这里](monitor.md)。