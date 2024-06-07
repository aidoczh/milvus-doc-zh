---
id: prerequisite-helm.md
label: 在 Kubernetes 上安装
related_key: Kubernetes
summary: 在使用 Helm 安装 Milvus 之前学习必要的准备工作。
title: 需求
---

# 需求

本页面列出了使 Milvus 运行所需的硬件和软件需求。

## 硬件需求

| 组件                | 需求                                                         | 推荐配置     | 备注                                                         |
| ------------------- | ------------------------------------------------------------ |--------------| ------------------------------------------------------------ |
| CPU                 | <ul><li>Intel 第二代 Core CPU 或更高</li><li>Apple Silicon</li></ul>|<ul><li>独立部署：4 核或更多</li><li>集群部署：8 核或更多</li></ul>|  |
| CPU 指令集          | <ul><li>SSE4.2</li><li>AVX</li><li>AVX2</li><li>AVX-512</li></ul> |<ul><li>SSE4.2</li><li>AVX</li><li>AVX2</li><li>AVX-512</li></ul> |  Milvus 中的向量相似度搜索和索引构建需要 CPU 支持单指令，多数据（SIMD）扩展集。确保 CPU 至少支持所列 SIMD 扩展中的一种。请参阅 [支持 AVX 指令集的 CPU](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX) 了解更多信息。                           |
| RAM                 | <ul><li>独立部署：8G</li><li>集群部署：32G</li></ul>         |<ul><li>独立部署：16G</li><li>集群部署：128G</li></ul>        | RAM 大小取决于数据量。                                      |
| 硬盘               | SATA 3.0 SSD 或 云存储                                      |NVMe SSD 或更高 | 硬盘大小取决于数据量。                                      |

## 软件需求

建议在 Linux 平台上运行 Kubernetes 集群。

kubectl 是 Kubernetes 的命令行工具。使用与您的集群相差一个次要版本的 kubectl 版本。使用最新版本的 kubectl 有助于避免意外问题。

在本地运行 Kubernetes 集群时需要 minikube。minikube 需要 Docker 作为依赖项。在使用 Helm 安装 Milvus 之前，请确保安装了 Docker。请参阅 <a href="https://docs.docker.com/get-docker">获取 Docker</a> 了解更多信息。

| 操作系统         | 软件                                                         | 备注                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Linux 平台       | <ul><li>Kubernetes 1.16 或更高</li><li>kubectl</li><li>Helm 3.0.0 或更高</li><li>minikube（用于 Milvus 独立部署）</li><li>Docker 19.03 或更高（用于 Milvus 独立部署）</li></ul> | 请参阅 [Helm 文档](https://helm.sh/docs/) 了解更多信息。 |

| 软件     | 版本                         | 备注 |
| -------- | ----------------------------- | ---- |
| etcd     | 3.5.0                         |  查看[额外的磁盘要求](#Additional-disk-requirements)。 |
| MinIO    |  RELEASE.2023-03-20T20-16-18Z | |
| Pulsar   | 2.8.2                         | |

### 额外的磁盘要求

磁盘性能对于 etcd 至关重要。强烈建议使用本地 NVMe SSD。较慢的磁盘响应可能导致频繁的集群选举，最终会降低 etcd 服务的性能。

要测试磁盘是否符合要求，请使用[fio](https://github.com/axboe/fio)。

```bash
mkdir test-data
fio --rw=write --ioengine=sync --fdatasync=1 --directory=test-data --size=2200m --bs=2300 --name=mytest
```

理想情况下，您的磁盘应该达到每秒超过500次的 IOPS，并且第99百分位的 fsync 延迟应该低于10毫秒。阅读 etcd [文档](https://etcd.io/docs/v3.5/op-guide/hardware/#disks) 以获取更详细的要求。

## 常见问题

### 如何在本地启动 K8s 集群进行测试？

您可以使用像[minikube](https://minikube.sigs.k8s.io/docs/)、[kind](https://kind.sigs.k8s.io/)和[Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)这样的工具，快速在本地设置一个 Kubernetes 集群。以下步骤以 minikube 为例。

1. 下载 minikube

   前往[入门指南](https://minikube.sigs.k8s.io/docs/start/)页面，检查是否满足**您需要什么**部分列出的条件，点击描述您目标平台的按钮，并复制下载和安装二进制文件的命令。

2. 使用 minikube 启动 K8s 集群

   ```shell
   $ minikube start
   ```

3. 检查 K8s 集群的状态

   您可以使用以下命令检查已安装的 K8s 集群的状态。

   ```shell
   $ kubectl cluster-info
   ```

<div class="alert note">

确保可以通过 `kubectl` 访问 K8s 集群。如果尚未在本地安装 `kubectl`，请参阅[在 minikube 中使用 kubectl](https://minikube.sigs.k8s.io/docs/handbook/kubectl/)。

</div>

## 接下来的步骤

- 如果您的硬件和软件符合要求，您可以：
  - [使用 Milvus Operator 在 Kubernetes 中运行 Milvus](install_cluster-milvusoperator.md)
  - [使用 Helm 在 Kubernetes 中运行 Milvus](install_cluster-helm.md)

- 请查看[System Configuration](system_configuration.md)以了解在安装 Milvus 时可以设置的参数。