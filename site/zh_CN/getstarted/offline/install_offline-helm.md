---
id: install_offline-helm.md
summary: 学习如何在 Kubernetes 离线环境下使用 Helm Charts 安装 Milvus。
title: 使用 Helm Charts 在离线环境下安装 Milvus
---

# 使用 Helm Charts 在离线环境下安装 Milvus

本主题介绍了如何在离线环境中使用 Helm Charts 安装 Milvus。

由于图像加载错误，Milvus 的安装可能会失败。您可以在离线环境中安装 Milvus 以避免此问题。

## 下载文件和镜像

要在离线环境中安装 Milvus，您需要首先在在线环境中拉取并保存所有镜像，然后将其传输到目标主机并手动加载。

1. 在本地添加并更新 Milvus Helm 仓库。

```bash
helm repo add milvus https://zilliztech.github.io/milvus-helm/
helm repo update
```

2. 获取 Kubernetes 清单。

- 对于独立的 Milvus：

```bash
helm template my-release --set cluster.enabled=false --set etcd.replicaCount=1 --set minio.mode=standalone --set pulsar.enabled=false milvus/milvus > milvus_manifest.yaml
```

- 对于 Milvus 集群：

```bash
helm template my-release milvus/milvus > milvus_manifest.yaml
```

如果您想要更改多个配置，可以下载一个 [`value.yaml`](https://github.com/milvus-io/milvus-helm/blob/master/charts/milvus/values.yaml) 文件，在其中指定配置，然后基于它生成一个清单。

```bash
wget https://raw.githubusercontent.com/milvus-io/milvus-helm/master/charts/milvus/values.yaml
helm template -f values.yaml my-release milvus/milvus > milvus_manifest.yaml
```

3. 下载需求和脚本文件。

```bash
$ wget https://raw.githubusercontent.com/milvus-io/milvus/master/deployments/offline/requirements.txt
$ wget https://raw.githubusercontent.com/milvus-io/milvus/master/deployments/offline/save_image.py
```

4. 拉取并保存镜像。

```bash
pip3 install -r requirements.txt
python3 save_image.py --manifest milvus_manifest.yaml
```

<div class="alert note">
镜像存储在 <code>/images</code> 文件夹中。
</div>

5. 加载镜像。

```bash
cd images/for image in $(find . -type f -name "*.tar.gz") ; do gunzip -c $image | docker load; done
```

## 在离线环境中安装 Milvus

将镜像传输到目标主机后，运行以下命令在离线环境中安装 Milvus。

```bash
kubectl apply -f milvus_manifest.yaml
```

## 卸载 Milvus

要卸载 Milvus，请运行以下命令。

```bash
kubectl delete -f milvus_manifest.yaml
```

## 下一步

安装完 Milvus 后，您可以：

- 查看 [Hello Milvus](quickstart.md) 以运行带有不同 SDK 的示例代码，了解 Milvus 的功能。

- 学习 Milvus 的基本操作：
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- [使用 Helm Chart 升级 Milvus](upgrade_milvus_cluster-helm.md)。
- [扩展您的 Milvus 集群](scaleout.md)。
- 探索[MilvusDM](migrate_overview.md)，这是一个专为在Milvus中导入和导出数据而设计的开源工具。
- 使用Prometheus监控Milvus，详见[Monitor Milvus with Prometheus](monitor.md)。