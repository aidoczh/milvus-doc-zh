---
id: install_standalone-operator.md
label: Milvus Operator
order: 1
group: install_standalone-docker.md
summary: 学习如何使用 Milvus Operator 安装独立的 Milvus。
title: 使用 Milvus Operator 安装独立的 Milvus
---

{{tab}}

# 使用 Milvus Operator 安装独立的 Milvus

Milvus Operator 是一个解决方案，可以帮助您在目标 Kubernetes（K8s）集群上部署和管理完整的 Milvus 服务栈。该栈包括所有 Milvus 组件和相关依赖项，如 etcd、Pulsar 和 MinIO。本主题描述了如何使用 Milvus Operator 安装独立的 Milvus。

## 先决条件
在安装之前，请先[检查硬件和软件要求](prerequisite-helm.md)。

## 创建 K8s 集群

如果您已经为生产部署部署了 K8s 集群，可以跳过此步骤，直接进行[部署 Milvus Operator](install_cluster-milvusoperator.md#Deploy-Milvus-Operator)。如果没有，您可以按照以下步骤快速创建一个用于测试的 K8s 集群，然后使用它来部署带有 Milvus Operator 的 Milvus 集群。

{{fragments/create_a_k8s_cluster_using_minikube.md}}

## 部署 Milvus Operator

Milvus Operator 在[Kubernetes自定义资源](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)之上定义了 Milvus 集群的自定义资源。当自定义资源被定义后，您可以以声明方式使用 K8s API，并管理 Milvus 部署栈，以确保其可伸缩性和高可用性。

### 先决条件

- 确保可以通过 `kubectl` 或 `helm` 访问 K8s 集群。
- 确保安装了 StorageClass 依赖项，因为 Milvus 集群依赖默认的 StorageClass 进行数据持久化。当安装 minikube 时，会有一个默认的 StorageClass 依赖项。通过运行命令 `kubectl get sc` 检查依赖项。如果已安装 StorageClass，您将看到以下输出。如果没有，请参阅[更改默认 StorageClass](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/)获取更多信息。

```
NAME                  PROVISIONER                  RECLAIMPOLICY    VOLUMEBIINDINGMODE    ALLOWVOLUMEEXPANSION     AGE
standard (default)    k8s.io/minikube-hostpath     Delete           Immediate             false                    3m36s
```

### 1. 安装 cert-manager

<div class="alert note">
您可以使用 Helm 或 `kubectl` 命令安装 Milvus Operator。如果选择使用 Helm，可以跳过此步骤，直接进行<a href=install_cluster-milvusoperator.md#Install-by-Helm-command>使用 Helm 命令安装</a>。
</div>

Milvus Operator 使用[cert-manager](https://cert-manager.io/docs/installation/supported-releases/)为 webhook 服务器提供证书。运行以下命令安装 cert-manager。

```
$ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```

如果 cert-manager 已安装，您将看到以下输出。
```
自定义资源定义.apiextensions.k8s.io/certificaterequests.cert-manager.io 已创建
自定义资源定义.apiextensions.k8s.io/certificates.cert-manager.io 已创建
自定义资源定义.apiextensions.k8s.io/challenges.acme.cert-manager.io 已创建
自定义资源定义.apiextensions.k8s.io/clusterissuers.cert-manager.io 已创建
自定义资源定义.apiextensions.k8s.io/issuers.cert-manager.io 已创建
自定义资源定义.apiextensions.k8s.io/orders.acme.cert-manager.io 已创建
命名空间/cert-manager 已创建
服务账户/cert-manager-cainjector 已创建
服务账户/cert-manager 已创建
服务账户/cert-manager-webhook 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-cainjector 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-controller-issuers 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-controller-certificates 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-controller-orders 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-controller-challenges 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-view 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-edit 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests 已创建
集群角色.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-cainjector 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-controller-issuers 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-controller-certificates 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-controller-orders 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-controller-challenges 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests 已创建
集群角色绑定.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews 已创建
角色.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection 已创建
角色.rbac.authorization.k8s.io/cert-manager:leaderelection 已创建
角色.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving 已创建
角色绑定.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection 已创建
角色绑定.rbac.authorization.k8s.io/cert-manager:leaderelection 已创建
角色绑定.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving 已创建
服务/cert-manager 已创建
服务/cert-manager-webhook 已创建
部署.apps/cert-manager-cainjector 已创建
部署.apps/cert-manager 已创建
部署.apps/cert-manager-webhook 已创建
变更 Webhook 配置.admissionregistration.k8s.io/cert-manager-webhook 已创建
验证 Webhook 配置.admissionregistration.k8s.io/cert-manager-webhook 已创建
```

<div class="alert note">
需要 cert-manager 版本为 1.13 或更高版本。
</div>

运行 `$ kubectl get pods -n cert-manager` 来检查 cert-manager 是否在运行。如果所有的 pods 都在运行，你会看到以下输出。

```
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-848f547974-gccz8             1/1     Running   0          70s
cert-manager-cainjector-54f4cc6b5-dpj84   1/1     Running   0          70s
cert-manager-webhook-7c9588c76-tqncn      1/1     Running   0          70s
```

### 2. 安装 Milvus Operator

在 K8s 上安装 Milvus Operator 有两种方式：

- 使用 helm chart
- 直接使用原始清单通过 `kubectl` 命令安装

#### 使用 Helm 命令安装

```
helm install milvus-operator \
  -n milvus-operator --create-namespace \
  --wait --wait-for-jobs \
  https://github.com/zilliztech/milvus-operator/releases/download/v{{var.milvus_operator_version}}/milvus-operator-{{var.milvus_operator_version}}.tgz
```

如果 Milvus Operator 安装成功，你会看到以下输出。
```
NAME: milvus-operator
LAST DEPLOYED: Thu Jul  7 13:18:40 2022
NAMESPACE: milvus-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Milvus Operator Is Starting, use `kubectl get -n milvus-operator deploy/milvus-operator` to check if its successfully installed
If Operator not started successfully, check the checker's log with `kubectl -n milvus-operator logs job/milvus-operator-checker`
Full Installation doc can be found in https://github.com/zilliztech/milvus-operator/blob/main/docs/installation/installation.md
Quick start with `kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_minimum.yaml`
More samples can be found in https://github.com/zilliztech/milvus-operator/tree/main/config/samples
CRD Documentation can be found in https://github.com/zilliztech/milvus-operator/tree/main/docs/CRD
```

#### 使用 `kubectl` 命令安装

```
$ kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/deploy/manifests/deployment.yaml
```

如果 Milvus Operator 安装成功，你会看到以下输出。
```
命名空间/milvus-operator 已创建
自定义资源定义.apiextensions.k8s.io/milvusclusters.milvus.io 已创建
服务账户/milvus-operator-controller-manager 已创建
角色.rbac.authorization.k8s.io/milvus-operator-leader-election-role 已创建
集群角色.rbac.authorization.k8s.io/milvus-operator-manager-role 已创建
集群角色.rbac.authorization.k8s.io/milvus-operator-metrics-reader 已创建
集群角色.rbac.authorization.k8s.io/milvus-operator-proxy-role 已创建
角色绑定.rbac.authorization.k8s.io/milvus-operator-leader-election-rolebinding 已创建
集群角色绑定.rbac.authorization.k8s.io/milvus-operator-manager-rolebinding 已创建
集群角色绑定.rbac.authorization.k8s.io/milvus-operator-proxy-rolebinding 已创建
配置映射/milvus-operator-manager-config 已创建
服务/milvus-operator-controller-manager-metrics-service 已创建
服务/milvus-operator-webhook-service 已创建
部署.apps/milvus-operator-controller-manager 已创建
证书.cert-manager.io/milvus-operator-serving-cert 已创建
颁发者.cert-manager.io/milvus-operator-selfsigned-issuer 已创建
变更 Webhook 配置.admissionregistration.k8s.io/milvus-operator-mutating-webhook-configuration 已创建
验证 Webhook 配置.admissionregistration.k8s.io/milvus-operator-validating-webhook-configuration 已创建
```
运行 `$ kubectl get pods -n milvus-operator` 检查 Milvus Operator 是否正在运行。如果 Milvus Operator 正在运行，您将看到以下输出。

```
NAME                               READY   STATUS    RESTARTS   AGE
milvus-operator-5fd77b87dc-msrk4   1/1     Running   0          46s
```

## 安装独立版 Milvus

### 1. 安装 Milvus

当 Milvus Operator 启动后，运行以下命令安装 Milvus。

```
$ kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_default.yaml
```

### 2. 检查独立版 Milvus 的状态

运行以下命令检查您刚安装的 Milvus 的状态。

```
$ kubectl get milvus my-release -o yaml
```

当 Milvus 安装成功后，您可以学习如何[管理集合](manage-collections.md)。

## 连接到 Milvus

验证 Milvus 服务器正在侦听的本地端口。将 pod 名称替换为您自己的名称。

```bash
$ kubectl get pod my-release-milvus-proxy-84f67cdb7f-pg6wf --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
19530
```

打开一个新终端并运行以下命令，将本地端口转发到 Milvus 使用的端口。可选择省略指定端口，使用 `:19530` 让 `kubectl` 为您分配一个本地端口，这样您就无需管理端口冲突。

```bash
$ kubectl port-forward service/my-release-milvus 27017:19530
Forwarding from 127.0.0.1:27017 -> 19530
```

默认情况下，kubectl 的端口转发仅在 localhost 上侦听。如果希望 Milvus 服务器侦听选定的 IP 或所有地址，请使用 `address` 标志。

```bash
$ kubectl port-forward --address 0.0.0.0 service/my-release-milvus 27017:19530
Forwarding from 0.0.0.0:27017 -> 19530
```

## 卸载独立版 Milvus

运行以下命令卸载 Milvus。

```
$ kubectl delete milvus my-release
```

## 卸载 Milvus Operator

在 K8s 上卸载 Milvus Operator 也有两种方法：

### 通过 Helm 命令卸载 Milvus Operator

```
$ helm -n milvus-operator uninstall milvus-operator
```

### 通过 `kubectl` 命令卸载 Milvus Operator

```
$ kubectl delete -f https://raw.githubusercontent.com/zilliztech/milvus-operator/v{{var.milvus_operator_version}}/deploy/manifests/deployment.yaml
```

## 删除 K8s 集群

当您不再需要测试环境中的 K8s 集群时，可以运行 `$ minikube delete` 进行删除。

## 下一步

安装了 Milvus 后，您可以：
- 查看[Hello Milvus](quickstart.md)以运行带有不同 SDK 的示例代码，了解 Milvus 的功能。
- 学习 Milvus 的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)
- [使用 Milvus Operator 升级 Milvus](upgrade_milvus_standalone-operator.md)
- 探索 [Milvus Backup](milvus_backup_overview.md)，这是一个用于 Milvus 数据备份的开源工具。
- 探索 [Birdwatcher](birdwatcher_overview.md)，这是一个用于调试 Milvus 和动态配置更新的开源工具。
- 探索 [Attu](https://milvus.io/docs/attu.md)，这是一个直观的 Milvus 管理的开源 GUI 工具。
- [使用 Prometheus 监控 Milvus](monitor.md)