---
id: install_cluster-helm.md
label: Helm
related_key: Kubernetes
summary: 学习如何在 Kubernetes 上使用 Helm 安装 Milvus 集群。
title: 使用 Helm 在 Kubernetes 上安装 Milvus 集群
---

# 使用 Helm 在 Kubernetes 中运行 Milvus

本页面演示了如何使用 [Milvus Helm charts](https://github.com/zilliztech/milvus-helm) 在 Kubernetes 中启动 Milvus 实例。

## 概述

Helm 使用一种名为 charts 的打包格式。Chart 是一组文件，用于描述一组相关的 Kubernetes 资源。Milvus 提供了一组 charts 来帮助您部署 Milvus 的依赖和组件。

## 先决条件

- [安装 Helm CLI](https://helm.sh/docs/intro/install/)。
- [创建一个 K8s 集群](prerequisite-helm.md#How-can-I-start-a-K8s-cluster-locally-for-test-purposes)。
- 安装一个 [StorageClass](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/)。您可以按如下方式检查已安装的 StorageClass。

    ```bash
    $ kubectl get sc

    NAME                  PROVISIONER                  RECLAIMPOLICY    VOLUMEBIINDINGMODE    ALLOWVOLUMEEXPANSION     AGE
    standard (default)    k8s.io/minikube-hostpath     Delete           Immediate             false 
    ```

- 在安装之前，请检查[硬件和软件要求](prerequisite-helm.md)。
- 在安装 Milvus 之前，建议使用 [Milvus Sizing Tool](https://milvus.io/tools/sizing) 根据数据大小估算硬件要求。这有助于确保 Milvus 安装的性能和资源分配最佳。

## 安装 Milvus Helm Chart

在安装 Milvus Helm Charts 之前，您需要添加 Milvus Helm 仓库。

```
$ helm repo add milvus https://zilliztech.github.io/milvus-helm/
```

<div class="alert note">

Milvus Helm Charts 仓库 `https://milvus-io.github.io/milvus-helm/` 已存档，您可以从 `https://zilliztech.github.io/milvus-helm/` 获取进一步更新，具体操作如下：

```shell
helm repo add zilliztech https://zilliztech.github.io/milvus-helm
helm repo update
# 升级现有的 helm 发布
helm upgrade my-release zilliztech/milvus
```

存档的仓库仍可用于 4.0.31 版本的 charts。对于更新版本，请使用新仓库。

</div>

然后按如下方式从仓库获取 Milvus charts：

```
$ helm repo update
```

您可以随时运行此命令以获取最新的 Milvus Helm charts。

## 在线安装

### 1. 部署 Milvus 集群

安装完 Helm chart 后，您可以在 Kubernetes 上启动 Milvus。本节将指导您完成启动 Milvus 的步骤。

```shell
$ helm install my-release milvus/milvus
```

在上述命令中，`my-release` 是发布名称，`milvus/milvus` 是本地安装的 chart 仓库。要使用不同的名称，请将 `my-release` 替换为您认为合适的名称。
上述命令部署了一个 Milvus 集群，包括其组件和依赖项，使用默认配置。要自定义这些设置，我们建议您使用 [Milvus Sizing Tool](https://milvus.io/tools/sizing) 来根据实际数据大小调整配置，然后下载相应的 YAML 文件。要了解更多关于配置参数的信息，请参考 [Milvus System Configurations Checklist](https://milvus.io/docs/system_configuration.md)。

<div class="alert note">
  <ul>
    <li>发布名称应仅包含字母、数字和破折号。发布名称中不允许使用点号。</li>
    <li>默认命令行在使用 Helm 安装 Milvus 时会安装集群版本的 Milvus，而在安装独立版 Milvus 时需要进一步设置。</li>
    <li>根据 Kubernetes 的 <a href="https://kubernetes.io/docs/reference/using-api/deprecation-guide/#v1-25">弃用 API 迁移指南</a>，PodDisruptionBudget 的 <b>policy/v1beta1</b> API 版本在 v1.25 中不再提供服务。建议您将清单文件和 API 客户端迁移到使用 <b>policy/v1</b> API 版本。 <br>对于仍在 Kubernetes v1.25 及更高版本上使用 <b>policy/v1beta1</b> API 版本的 PodDisruptionBudget 的用户，您可以运行以下命令来安装 Milvus：<br>
    <code>helm install my-release milvus/milvus --set pulsar.bookkeeper.pdb.usePolicy=false,pulsar.broker.pdb.usePolicy=false,pulsar.proxy.pdb.usePolicy=false,pulsar.zookeeper.pdb.usePolicy=false</code></li> 
    <li>查看 <a href="https://artifacthub.io/packages/helm/milvus/milvus">Milvus Helm Chart</a> 和 <a href="https://helm.sh/docs/">Helm</a> 获取更多信息。</li>
  </ul>
</div>

### 2. 检查 Milvus 集群状态

运行以下命令来检查 Milvus 集群中所有 Pod 的状态。

```
$ kubectl get pods
```

一旦所有 Pod 都在运行中，上述命令的输出应类似于以下内容：
```
名称                                             就绪状态  状态     重启次数   年龄
my-release-etcd-0                                1/1    运行中   0        3分钟23秒
my-release-etcd-1                                1/1    运行中   0        3分钟23秒
my-release-etcd-2                                1/1    运行中   0        3分钟23秒
my-release-milvus-datacoord-6fd4bd885c-gkzwx     1/1    运行中   0        3分钟23秒
my-release-milvus-datanode-68cb87dcbd-4khpm      1/1    迌行中   0        3分钟23秒
my-release-milvus-indexcoord-5bfcf6bdd8-nmh5l    1/1    运行中   0        3分钟23秒
my-release-milvus-indexnode-5c5f7b5bd9-l8hjg     1/1    运行中   0        3分钟24秒
my-release-milvus-proxy-6bd7f5587-ds2xv          1/1    运行中   0        3分钟24秒
my-release-milvus-querycoord-579cd79455-xht5n    1/1    运行中   0        3分钟24秒
my-release-milvus-querynode-5cd8fff495-k6gtg     1/1    运行中   0        3分钟24秒
my-release-milvus-rootcoord-7fb9488465-dmbbj     1/1    运行中   0        3分钟23秒
my-release-minio-0                               1/1    运行中   0        3分钟23秒
my-release-minio-1                               1/1    运行中   0        3分钟23秒
my-release-minio-2                               1/1    运行中   0        3分钟23秒
my-release-minio-3                               1/1    运行中   0        3分钟23秒
my-release-pulsar-autorecovery-86f5dbdf77-lchpc  1/1    运行中   0        3分钟24秒
my-release-pulsar-bookkeeper-0                   1/1    运行中   0        3分钟23秒
my-release-pulsar-bookkeeper-1                   1/1    运行中   0        98秒
my-release-pulsar-broker-556ff89d4c-2m29m        1/1    运行中   0        3分钟23秒
my-release-pulsar-proxy-6fbd75db75-nhg4v         1/1    运行中   0        3分钟23秒
my-release-pulsar-zookeeper-0                    1/1    运行中   0        3分钟23秒
my-release-pulsar-zookeeper-metadata-98zbr       0/1   已完成   0        3分钟24秒
```
### 3. 将本地端口转发到 Milvus

运行以下命令以获取 Milvus 集群提供服务的端口号。

```bash
$ kubectl get pod my-release-milvus-proxy-6bd7f5587-ds2xv --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
19530
```

输出显示 Milvus 实例在默认端口 **19530** 上提供服务。

<div class="alert note">

如果您是在独立模式下部署了 Milvus，请将 pod 名称从 `my-release-milvus-proxy-xxxxxxxxxx-xxxxx` 更改为 `my-release-milvus-xxxxxxxxxx-xxxxx`。

</div>

然后，运行以下命令将本地端口转发到 Milvus 提供服务的端口。

```bash
$ kubectl port-forward service/my-release-milvus 27017:19530
Forwarding from 127.0.0.1:27017 -> 19530
```

可选地，您可以在上述命令中使用 `:19530` 替代 `27017:19530`，让 `kubectl` 为您分配一个本地端口，这样您就不必管理端口冲突。

默认情况下，kubectl 的端口转发仅在 `localhost` 上监听。如果您希望 Milvus 在选定的 IP 地址或所有 IP 地址上监听，请使用 `address` 标志。以下命令使端口转发在主机机器上的所有 IP 地址上监听。

```bash
$ kubectl port-forward --address 0.0.0.0 service/my-release-milvus 27017:19530
Forwarding from 0.0.0.0:27017 -> 19530
```

## 离线安装

如果您处于网络受限环境中，请按照本节中的步骤启动 Milvus 集群。

### 1. 获取 Milvus 配置清单

运行以下命令以获取 Milvus 配置清单。

```shell
$ helm template my-release milvus/milvus > milvus_manifest.yaml
```

上述命令会渲染 Milvus 集群的图表模板，并将输出保存到名为 `milvus_manifest.yaml` 的清单文件中。使用此清单，您可以在单独的 pod 中安装 Milvus 集群及其组件和依赖项。

<div class="alert note">

- 要在独立模式下安装 Milvus 实例，其中所有 Milvus 组件都包含在单个 pod 中，您应该运行 `helm template my-release --set cluster.enabled=false --set etcd.replicaCount=1 --set minio.mode=standalone --set pulsar.enabled=false milvus/milvus > milvus_manifest.yaml`，以渲染用于独立模式下的 Milvus 实例的图表模板。
- 要更改 Milvus 配置，请下载 [`value.yaml`](https://raw.githubusercontent.com/milvus-io/milvus-helm/master/charts/milvus/values.yaml) 模板，将所需的设置放入其中，并使用 `helm template -f values.yaml my-release milvus/milvus > milvus_manifest.yaml` 来相应地渲染清单。

</div>

### 2. 下载镜像拉取脚本

镜像拉取脚本是用 Python 开发的。您应该下载该脚本以及其在 `requirement.txt` 文件中列出的依赖项。

```shell
$ wget https://raw.githubusercontent.com/milvus-io/milvus/master/deployments/offline/requirements.txt
$ wget https://raw.githubusercontent.com/milvus-io/milvus/master/deployments/offline/save_image.py
```
### 3. 拉取并保存图片

运行以下命令来拉取并保存所需的图片。

```shell
$ pip3 install -r requirements.txt
$ python3 save_image.py --manifest milvus_manifest.yaml
```

这些图片将被拉取到当前目录下名为 `images` 的子文件夹中。

### 4. 载入图片

您现在可以将这些图片加载到网络受限环境中的主机上，方法如下：

```shell
$ for image in $(find . -type f -name "*.tar.gz") ; do gunzip -c $image | docker load; done
```

### 5. 部署 Milvus

```shell
$ kubectl apply -f milvus_manifest.yaml
```

到目前为止，您可以按照在线安装的步骤 [2](#2-Check-Milvus-cluster-status) 和 [3](#3-Forward-a-local-port-to-Milvus) 来检查集群状态并将本地端口转发到 Milvus。

## 升级正在运行的 Milvus 集群

运行以下命令来卸载您的 Milvus 集群。

```shell
$ helm uninstall my-release
```

## 卸载 Milvus

运行以下命令来卸载 Milvus。

```bash
$ helm uninstall my-release
```

## 接下来做什么

在 Docker 中安装了 Milvus 后，您可以：

- 查看 [Hello Milvus](quickstart.md) 以了解 Milvus 的功能。

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
- 探索 [Milvus 备份](milvus_backup_overview.md)，这是一个用于 Milvus 数据备份的开源工具。
- 探索 [Birdwatcher](birdwatcher_overview.md)，这是一个用于调试 Milvus 和动态配置更新的开源工具。
- 探索 [Attu](https://milvus.io/docs/attu.md)，这是一个直观的 Milvus 管理的开源 GUI 工具。
- [使用 Prometheus 监控 Milvus](monitor.md)。