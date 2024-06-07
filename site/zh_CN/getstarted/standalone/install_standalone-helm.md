---
id: install_standalone-helm.md
label: Helm
related_key: Helm
order: 2
group: install_standalone-docker.md
summary: 学习如何在 Kubernetes 上安装独立的 Milvus。
title: 使用 Kubernetes 安装独立的 Milvus
---

{{tab}}

# 使用 Kubernetes 安装独立的 Milvus

本主题描述了如何使用 Kubernetes 安装独立的 Milvus。

## 先决条件

在安装之前，请查看[要求](prerequisite-helm.md)中的硬件和软件要求。

## 使用 minikube 创建 K8s 集群

我们建议使用[minikube](https://minikube.sigs.k8s.io/docs/)在 K8s 上安装 Milvus，这是一个允许您在本地运行 K8s 的工具。

### 1. 安装 minikube

查看[安装 minikube](https://minikube.sigs.k8s.io/docs/start/)获取更多信息。

### 2. 使用 minikube 启动 K8s 集群

安装 minikube 后，运行以下命令启动 K8s 集群。

```
$ minikube start
```

### 3. 检查 K8s 集群状态

运行 `$ kubectl cluster-info` 检查您刚刚创建的 K8s 集群的状态。确保您可以通过 `kubectl` 访问 K8s 集群。如果您尚未在本地安装 `kubectl`，请参阅[在 minikube 中使用 kubectl](https://minikube.sigs.k8s.io/docs/handbook/kubectl/)。

minikube 在安装时依赖于默认的 StorageClass。通过运行以下命令检查依赖关系。其他安装方法需要手动配置 StorageClass。查看[更改默认 StorageClass](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/)获取更多信息。

```
$ kubectl get sc
```

```
NAME                  PROVISIONER                  RECLAIMPOLICY    VOLUMEBIINDINGMODE    ALLOWVOLUMEEXPANSION     AGE
standard (default)    k8s.io/minikube-hostpath     Delete           Immediate             false                    3m36s
```

### 4. 检查默认的存储类

Milvus 依赖于默认的存储类来自动为数据持久性提供卷。运行以下命令检查存储类：

```bash
$ kubectl get sc
```

命令输出应类似于以下内容：

```bash
NAME                   PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path                           Delete          WaitForFirstConsumer   false                  461d
```

## 为 Milvus 安装 Helm Chart

Helm 是一个 K8s 包管理器，可以帮助您快速部署 Milvus。

1. 将 Milvus 添加到 Helm 的存储库中。

```bash
$ helm repo add milvus https://zilliztech.github.io/milvus-helm/
```

<div class="alert note">

Milvus Helm Charts 存储库位于 `https://milvus-io.github.io/milvus-helm/`，已归档，您可以从 `https://zilliztech.github.io/milvus-helm/` 获取进一步更新：
```shell
helm repo add zilliztech https://zilliztech.github.io/milvus-helm
helm repo update
# 升级现有的 Helm 发布
helm upgrade my-release zilliztech/milvus
```
存档库仍可用于版本为 4.0.31 的图表。对于更新的版本，请改用新的库。

</div>

2. 更新本地图表库。

```bash
$ helm repo update
```

## 启动 Milvus

安装 Helm 图表后，您可以在 Kubernetes 上启动 Milvus。在本节中，我们将指导您完成启动 Milvus 的步骤。

您应该通过 Helm 启动 Milvus，指定发布名称、图表以及您希望更改的参数。在本指南中，我们使用 <code>my-release</code> 作为发布名称。要使用不同的发布名称，请将下面命令中的 <code>my-release</code> 替换为您使用的名称。

```bash
$ helm install my-release milvus/milvus --set cluster.enabled=false --set etcd.replicaCount=1 --set minio.mode=standalone --set pulsar.enabled=false
```

<div class="alert note">
查看更多信息，请访问 <a href="https://artifacthub.io/packages/helm/milvus/milvus">Milvus Helm Chart</a> 和 <a href="https://helm.sh/docs/">Helm</a>。
</div>

检查正在运行的 Pod 的状态。

```bash
$ kubectl get pods
```

Milvus 启动后，所有 Pod 的 `READY` 列都显示 `1/1`。

```text
NAME                                               READY   STATUS      RESTARTS   AGE
my-release-etcd-0                                  1/1     Running     0          30s
my-release-milvus-standalone-54c4f88cb9-f84pf      1/1     Running     0          30s
my-release-minio-5564fbbddc-mz7f5                  1/1     Running     0          30s
```

## 连接到 Milvus

验证 Milvus 服务器正在侦听的本地端口。将 Pod 名称替换为您自己的名称。

```bash
$ kubectl get pod my-release-milvus-standalone-54c4f88cb9-f84pf --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
```

```
19530
```

打开新终端并运行以下命令，将本地端口转发到 Milvus 使用的端口。可选择省略指定端口，并使用 `:19530`，让 `kubectl` 为您分配本地端口，以避免管理端口冲突。

```bash
$ kubectl port-forward service/my-release-milvus 27017:19530
```

```
Forwarding from 127.0.0.1:27017 -> 19530
```

默认情况下，由 kubectl 转发的端口仅在 localhost 上侦听。如果希望 Milvus 服务器侦听选定的 IP 或所有地址，请使用 `address` 标志。

```bash
$ kubectl port-forward --address 0.0.0.0 service/my-release-milvus 27017:19530
Forwarding from 0.0.0.0:27017 -> 19530
```

## 卸载 Milvus

运行以下命令以卸载 Milvus。

```bash
$ helm uninstall my-release
```

## 停止 K8s 集群

停止集群和 minikube VM，而不删除您创建的资源。

```bash
$ minikube stop
```

运行 `minikube start` 以重新启动集群。

## 删除 K8s 集群

<div class="alert note">
在删除集群和所有资源之前，运行 <code>$ kubectl logs `pod_name`</code> 以获取 Pod 的 <code>stderr</code> 日志。
</div>
删除集群、minikube虚拟机以及您创建的所有资源，包括持久卷。

```bash
$ minikube delete
```

## 下一步

安装了Milvus之后，您可以：

- 查看[Hello Milvus](quickstart.md)，运行一个示例代码，使用不同的SDK，看看Milvus能做什么。

- 学习Milvus的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- 使用Helm Chart升级Milvus，参见[使用Helm Chart升级Milvus](upgrade_milvus_standalone-helm.md)。
- 探索[Milvus Backup](milvus_backup_overview.md)，这是一个用于Milvus数据备份的开源工具。
- 探索[Birdwatcher](birdwatcher_overview.md)，这是一个用于调试Milvus和动态配置更新的开源工具。
- 探索[Attu](https://milvus.io/docs/attu.md)，这是一个直观的Milvus管理工具。
- 使用Prometheus监控Milvus，参见[使用Prometheus监控Milvus](monitor.md)。