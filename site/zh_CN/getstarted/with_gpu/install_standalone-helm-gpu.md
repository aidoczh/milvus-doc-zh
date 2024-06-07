---
id: install_standalone-helm-gpu.md
label: Standalone (Helm)
order: 0
group: install_standalone-helm-gpu.md
related_key: Docker
summary: 在使用 Docker 安装 Milvus 之前，了解必要的准备工作。
title: 使用 GPU 支持安装 Milvus 独立版
---

{{tab}}

# 使用 GPU 支持安装 Milvus 独立版

现在，Milvus 可以利用 GPU 设备构建索引并执行 ANN 搜索，这要归功于 NVIDIA 的贡献。本指南将向您展示如何在您的计算机上安装支持 GPU 的 Milvus。

## 先决条件

在安装支持 GPU 的 Milvus 之前，请确保您具备以下先决条件：

- 您的 GPU 设备的计算能力为 6.0、7.0、7.5、8.0、8.6、9.0。要检查您的 GPU 设备是否符合要求，请查看 NVIDIA 开发者网站上的 [您的 GPU 计算能力](https://developer.nvidia.com/cuda-gpus)。

- 您已在 [支持的 Linux 发行版](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#linux-distributions) 之一上为您的 GPU 设备安装了 NVIDIA 驱动程序，然后按照 [此指南](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) 安装了 NVIDIA 容器工具包。

  对于 Ubuntu 22.04 用户，您可以使用以下命令安装驱动程序和容器工具包：

  ```shell
  $ sudo apt install --no-install-recommends nvidia-headless-545 nvidia-utils-545
  ```

  对于其他操作系统用户，请参考[官方安装指南](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#installing-on-ubuntu-and-debian)。

  您可以通过运行以下命令检查驱动程序是否已正确安装：

  ```shell
  $ modinfo nvidia | grep "^version"
  version:        545.29.06
  ```

  建议您使用版本为 545 及以上的驱动程序。

- 您已安装了 Kubernetes 集群，并且 `kubectl` 命令行工具已配置为与您的集群通信。建议在至少有两个节点且不充当控制平面主机的集群上运行本教程。

## 使用 minikube 创建 K8s 集群

我们建议使用 [minikube](https://minikube.sigs.k8s.io/docs/) 在 K8s 上安装 Milvus，这是一个允许您在本地运行 K8s 的工具。

### 1. 安装 minikube

有关更多信息，请参阅 [安装 minikube](https://minikube.sigs.k8s.io/docs/start/)。

### 2. 使用 minikube 启动 K8s 集群

安装 minikube 后，运行以下命令启动 K8s 集群。

```
$ minikube start --gpus all
```

### 3. 检查 K8s 集群状态

运行 `$ kubectl cluster-info` 来检查您刚刚创建的 K8s 集群的状态。确保您可以通过 `kubectl` 访问 K8s 集群。如果您尚未在本地安装 `kubectl`，请参阅 [在 minikube 中使用 kubectl](https://minikube.sigs.k8s.io/docs/handbook/kubectl/)。
Minikube 在安装时依赖默认的 StorageClass。通过运行以下命令来检查这个依赖。其他安装方法需要手动配置 StorageClass。更多信息请参阅 [更改默认的 StorageClass](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/)。

```bash
$ kubectl get sc
```

```bash
NAME                  PROVISIONER                  RECLAIMPOLICY    VOLUMEBIINDINGMODE    ALLOWVOLUMEEXPANSION     AGE
standard (default)    k8s.io/minikube-hostpath     Delete           Immediate             false                    3m36s
```

## 使用 GPU 工作节点启动 Kubernetes 集群

如果您希望使用启用了 GPU 的工作节点，可以按照以下步骤创建具有 GPU 工作节点的 K8s 集群。我们建议在具有 GPU 工作节点的 Kubernetes 集群上安装 Milvus，并使用默认的存储类进行配置。

### 1. 准备 GPU 工作节点

更多信息请参阅 [准备 GPU 工作节点](https://gitlab.com/nvidia/kubernetes/device-plugin/-/blob/main/README.md#preparing-your-gpu-nodes)。

### 2. 在 Kubernetes 上启用 GPU 支持

更多信息请参阅 [使用 helm 安装 nvidia-device-plugin](https://gitlab.com/nvidia/kubernetes/device-plugin/-/blob/main/README.md#deployment-via-helm)。

设置完成后，运行 `kubectl describe node <gpu-worker-node>` 来查看 GPU 资源。命令输出应类似于以下内容：

```bash
Capacity:
  ...
  nvidia.com/gpu:     4
  ...
Allocatable:
  ...
  nvidia.com/gpu:     4
  ...
```

注意：在此示例中，我们已经设置了一个具有 4 张 GPU 卡的 GPU 工作节点。

### 3. 检查默认的存储类

Milvus 依赖默认的存储类来自动为数据持久性提供卷。运行以下命令来检查存储类：

```bash
$ kubectl get sc
```

命令输出应类似于以下内容：

```bash
NAME                   PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path                           Delete          WaitForFirstConsumer   false                  461d
```

## 安装 Milvus 的 Helm Chart

Helm 是一个 Kubernetes 包管理器，可以帮助您快速部署 Milvus。

1. 将 Milvus 添加到 Helm 的仓库中。

```bash
$ helm repo add milvus https://zilliztech.github.io/milvus-helm/
```

<div class="alert note">

Milvus Helm Charts 仓库位于 `https://milvus-io.github.io/milvus-helm/`，已存档，您可以从 `https://zilliztech.github.io/milvus-helm/` 获取进一步更新，操作如下：

```shell
helm repo add zilliztech https://zilliztech.github.io/milvus-helm
helm repo update
# 升级现有的 helm 发布
helm upgrade my-release zilliztech/milvus
```

存档的仓库仍可用于 4.0.31 版本的图表。对于更新的版本，请使用新仓库。

</div>

2. 更新您的本地图表仓库。
```bash
$ helm 仓库更新
```
## 启动 Milvus

在安装了 Helm chart 之后，您可以在 Kubernetes 上启动 Milvus。在本节中，我们将指导您如何启动支持 GPU 的 Milvus。

您应该通过指定发布名称、图表和您希望更改的参数来使用 Helm 启动 Milvus。在本指南中，我们使用 <code>my-release</code> 作为发布名称。要使用其他发布名称，请将以下命令中的 <code>my-release</code> 替换为您使用的名称。

Milvus 允许您为 Milvus 分配一个或多个 GPU 设备。

- 分配单个 GPU 设备（推荐）

  运行以下命令将一个 GPU 设备分配给 Milvus：

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

- 分配多个 GPU 设备

  运行以下命令将多个 GPU 设备分配给 Milvus：

  运行以下命令将多个 GPU 设备分配给 Milvus：

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

  在上述配置中，indexNode 和 queryNode 共享两个 GPU。要为 indexNode 和 queryNode 分配不同的 GPU，您可以通过在配置文件中设置 `extraEnv` 来相应地修改配置，如下所示：

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

  <div class="alert note">
  查看更多信息，请访问 <a href="https://artifacthub.io/packages/helm/milvus-helm/milvus">Milvus Helm Chart</a> 和 <a href="https://helm.sh/docs/">Helm</a>。
  </div>

  检查正在运行的 Pod 的状态：

  ```bash
  $ kubectl get pods
  ```

Milvus 启动后，所有 Pod 的 `READY` 列将显示为 `1/1`。
```text
名称                                               就绪状态   运行状态      重启次数   年龄
my-release-etcd-0                                  1/1     运行中     0          30秒
my-release-milvus-standalone-54c4f88cb9-f84pf      1/1     运行中     0          30秒
my-release-minio-5564fbbddc-mz7f5                  1/1     运行中     0          30秒
```
## 连接到 Milvus

验证 Milvus 服务器正在侦听的本地端口。将 pod 名称替换为您自己的名称。

```bash
$ kubectl get pod my-release-milvus-standalone-54c4f88cb9-f84pf --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
```

```
19530
```

打开一个新的终端窗口，并运行以下命令将本地端口转发到 Milvus 使用的端口。可选择省略指定端口，使用 `:19530` 让 `kubectl` 为您分配一个本地端口，这样您就无需管理端口冲突。

```bash
$ kubectl port-forward service/my-release-milvus 27017:19530
```

```
Forwarding from 127.0.0.1:27017 -> 19530
```

默认情况下，由 kubectl 转发的端口仅在本地主机上侦听。如果要让 Milvus 服务器侦听选定的 IP 或所有地址，请使用 `address` 标志。

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

停止集群和 minikube 虚拟机，而不删除您创建的资源。

```bash
$ minikube stop
```

运行 `minikube start` 以重新启动集群。

## 删除 K8s 集群

<div class="alert note">
运行 <code>$ kubectl logs `pod_name`</code> 以获取删除集群和所有资源之前的 pod 的 <code>stderr</code> 日志。
</div>

删除集群、minikube 虚拟机以及您创建的所有资源，包括持久卷。

```bash
$ minikube delete
```

## 接下来做什么

安装了 Milvus 后，您可以：

- 查看 [Hello Milvus](quickstart.md) 以使用不同的 SDK 运行示例代码，了解 Milvus 的功能。

- 学习 Milvus 的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- [使用 Helm Chart 升级 Milvus](upgrade_milvus_standalone-helm.md)。
- 探索 [Milvus Backup](milvus_backup_overview.md)，这是一个用于 Milvus 数据备份的开源工具。
- 探索 [Birdwatcher](birdwatcher_overview.md)，这是一个用于调试 Milvus 和动态配置更新的开源工具。
- 探索 [Attu](https://milvus.io/docs/attu.md)，这是一个直观的 Milvus 管理开源 GUI 工具。
- [使用 Prometheus 监控 Milvus](monitor.md)。