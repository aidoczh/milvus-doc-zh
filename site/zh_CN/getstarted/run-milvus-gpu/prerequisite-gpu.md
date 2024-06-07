---
id: prerequisite-gpu.md
label: GPU requirements
related_key: GPU
summary: 在安装 Milvus 与 GPU 支持之前，了解必要的准备工作。
title: 硬件和软件要求
---

# 硬件和软件要求

本页面列出了设置支持 GPU 的 Milvus 所需的硬件和软件要求。

## 计算能力

您的 GPU 设备的计算能力必须是以下之一：6.0、7.0、7.5、8.0、8.6、9.0。

要检查您的 GPU 设备是否符合要求，请访问 NVIDIA 开发者网站上的 [Your GPU Compute Capability](https://developer.nvidia.com/cuda-gpus) 页面。

## NVIDIA 驱动程序

您的 GPU 设备的 NVIDIA 驱动程序必须是 [支持的 Linux 发行版](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#linux-distributions) 之一，并且必须按照 [此指南](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) 安装了 NVIDIA 容器工具包。

对于 Ubuntu 22.04 用户，您可以使用以下命令安装驱动程序和容器工具包：

```shell
$ sudo apt install --no-install-recommends nvidia-headless-545 nvidia-utils-545
```

对于其他操作系统用户，请参考 [官方安装指南](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#installing-on-ubuntu-and-debian)。

您可以通过运行以下命令来检查驱动程序是否已正确安装：

```shell
$ modinfo nvidia | grep "^version"
version:        545.29.06
```

建议您使用版本号为 545 及以上的驱动程序。

## 软件要求

建议在 Linux 平台上运行 Kubernetes 集群。

- kubectl 是 Kubernetes 的命令行工具。请使用与您的集群相差一个次要版本的 kubectl 版本。使用最新版本的 kubectl 有助于避免意外问题。
- 在本地运行 Kubernetes 集群时需要 minikube。minikube 需要 Docker 作为依赖项。在使用 Helm 安装 Milvus 之前，请确保安装了 Docker。有关更多信息，请参阅 [获取 Docker](https://docs.docker.com/get-docker)。

| 操作系统        | 软件                                                         | 备注                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Linux 平台       | <ul><li>Kubernetes 1.16 或更高版本</li><li>kubectl</li><li>Helm 3.0.0 或更高版本</li><li>minikube（用于 Milvus 独立运行）</li><li>Docker 19.03 或更高版本（用于 Milvus 独立运行）</li></ul> | 有关更多信息，请参阅 [Helm 文档](https://helm.sh/docs/)。 |

## 常见问题解答

### 如何在本地启动 K8s 集群进行测试？
你可以使用像[minikube](https://minikube.sigs.k8s.io/docs/)、[kind](https://kind.sigs.k8s.io/)和[Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)这样的工具，在本地快速搭建一个 Kubernetes 集群。以下步骤以 minikube 为例。

1. 下载 minikube

   前往[开始使用](https://minikube.sigs.k8s.io/docs/start/)页面，检查是否符合**你需要的条件**部分列出的条件，点击描述您目标平台的按钮，并复制下载和安装二进制文件的命令。

2. 使用 minikube 启动一个 K8s 集群

   ```shell
   $ minikube start
   ```

3. 检查 K8s 集群的状态

   您可以使用以下命令检查已安装的 K8s 集群的状态。

   ```shell
   $ kubectl cluster-info
   ```

<div class="alert note">

确保可以通过 `kubectl` 访问 K8s 集群。如果您尚未在本地安装 `kubectl`，请参阅[在 minikube 中使用 kubectl](https://minikube.sigs.k8s.io/docs/handbook/kubectl/)。

</div>

### 如何使用 GPU 工作节点启动 K8s 集群？

如果您希望使用启用 GPU 的工作节点，可以按照以下步骤创建具有 GPU 工作节点的 K8s 集群。我们建议在具有 GPU 工作节点的 K8s 集群上安装 Milvus，并使用默认的存储类进行配置。

1. 准备 GPU 工作节点

   要使用启用 GPU 的工作节点，请按照[准备您的 GPU 节点](https://gitlab.com/nvidia/kubernetes/device-plugin/-/blob/main/README.md#preparing-your-gpu-nodes)中的步骤进行操作。

2. 在 K8s 上启用 GPU 支持

   通过以下步骤使用 Helm 部署 **nvidia-device-plugin**。

   部署完成后，使用以下命令查看 GPU 资源。将 `<gpu-worker-node>` 替换为实际节点名称。

   ```shell
   $ kubectl describe node <gpu-worker-node>
   
   Capacity:
   ...
   nvidia.com/gpu:     4
   ...
   Allocatable:
   ...
   nvidia.com/gpu:     4
   ...
   ```