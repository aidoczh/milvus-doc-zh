---
id: install_cluster-milvusoperator.md
label: Milvus Operator
related_key: Kubernetes
summary: 学习如何使用 Milvus Operator 在 Kubernetes 上安装 Milvus 集群
title: 使用 Milvus Operator 安装 Milvus 集群
---

# 在 Kubernetes 中使用 Milvus Operator 运行 Milvus

本页面介绍如何使用 [Milvus Operator](https://github.com/zilliztech/milvus-operator) 在 Kubernetes 中启动 Milvus 实例。

## 概述

Milvus Operator 是一个解决方案，可帮助您在目标 Kubernetes（K8s）集群上部署和管理完整的 Milvus 服务栈。该栈包括所有 Milvus 组件以及相关依赖项，如 etcd、Pulsar 和 MinIO。

## 先决条件

- [创建一个 K8s 集群](prerequisite-helm.md#How-can-I-start-a-K8s-cluster-locally-for-test-purposes)。
- 安装一个 [StorageClass](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/)。您可以按如下方式检查已安装的 StorageClass。

    ```bash
    $ kubectl get sc

    NAME                  PROVISIONER                  RECLAIMPOLICY    VOLUMEBIINDINGMODE    ALLOWVOLUMEEXPANSION     AGE
    standard (default)    k8s.io/minikube-hostpath     Delete           Immediate             false 
    ```

- 在安装之前，请检查[硬件和软件要求](prerequisite-helm.md)。

- 在安装 Milvus 之前，建议使用 [Milvus Sizing Tool](https://milvus.io/tools/sizing) 根据数据大小估算硬件要求。这有助于确保 Milvus 安装的性能和资源分配达到最佳状态。

## 安装 Milvus Operator

Milvus Operator 在 [Kubernetes Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) 上定义了 Milvus 集群的自定义资源。一旦定义了自定义资源，您就可以以声明方式使用 K8s API 来管理 Milvus 部署栈，以确保其可伸缩性和高可用性。

### 1. 安装 cert-manager

Milvus Operator 使用 [cert-manager](https://cert-manager.io/docs/installation/supported-releases/) 为 webhook 服务器提供证书。

<div class="alert note">

- 如果选择使用 Helm 部署 Milvus Operator，可以安全地跳过此步骤。
- Milvus Operator 需要 cert-manager 版本为 1.1.3 或更高。

</div>

运行以下命令安装 cert-manager。

```shell
$ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```

安装过程结束后，您将看到类似以下内容的输出。
```shell
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io 已创建
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io 已创建
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io 已创建
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io 已创建
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io 已创建
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io 已创建
namespace/cert-manager 已创建
serviceaccount/cert-manager-cainjector 已创建
...
service/cert-manager 已创建
service/cert-manager-webhook 已创建
deployment.apps/cert-manager-cainjector 已创建
deployment.apps/cert-manager 已创建
deployment.apps/cert-manager-webhook 已创建
mutatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook 已创建
validatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook 已创建
```
你可以通过以下方式检查 cert-manager 的 Pod 是否正在运行：

```shell
$ kubectl get pods -n cert-manager

NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-848f547974-gccz8             1/1     Running   0          70s
cert-manager-cainjector-54f4cc6b5-dpj84   1/1     Running   0          70s
cert-manager-webhook-7c9588c76-tqncn      1/1     Running   0          70s
```

### 2. 安装 Milvus Operator

您可以通过以下任一方式安装 Milvus Operator：

- [使用 Helm 安装](#Install-with-Helm)
- [使用 kubectl 安装](#Install-with-kubectl)

#### 使用 Helm 安装

运行以下命令以使用 Helm 安装 Milvus Operator。

```shell
$ helm install milvus-operator \
  -n milvus-operator --create-namespace \
  --wait --wait-for-jobs \
  https://github.com/zilliztech/milvus-operator/releases/download/v{{var.milvus_operator_version}}/milvus-operator-{{var.milvus_operator_version}}.tgz
```

安装过程结束后，您将看到类似以下内容的输出。

```shell
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

#### 使用 kubectl 安装

运行以下命令以使用 `kubectl` 安装 Milvus Operator。

```shell
$ kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/deploy/manifests/deployment.yaml
```

安装过程结束后，您将看到类似以下内容的输出。
```shell
namespace/milvus-operator 创建完成
customresourcedefinition.apiextensions.k8s.io/milvusclusters.milvus.io 创建完成
serviceaccount/milvus-operator-controller-manager 创建完成
role.rbac.authorization.k8s.io/milvus-operator-leader-election-role 创建完成
clusterrole.rbac.authorization.k8s.io/milvus-operator-manager-role 创建完成
clusterrole.rbac.authorization.k8s.io/milvus-operator-metrics-reader 创建完成
clusterrole.rbac.authorization.k8s.io/milvus-operator-proxy-role 创建完成
rolebinding.rbac.authorization.k8s.io/milvus-operator-leader-election-rolebinding 创建完成
clusterrolebinding.rbac.authorization.k8s.io/milvus-operator-manager-rolebinding 创建完成
clusterrolebinding.rbac.authorization.k8s.io/milvus-operator-proxy-rolebinding 创建完成
configmap/milvus-operator-manager-config 创建完成
service/milvus-operator-controller-manager-metrics-service 创建完成
service/milvus-operator-webhook-service 创建完成
deployment.apps/milvus-operator-controller-manager 创建完成
certificate.cert-manager.io/milvus-operator-serving-cert 创建完成
issuer.cert-manager.io/milvus-operator-selfsigned-issuer 创建完成
mutatingwebhookconfiguration.admissionregistration.k8s.io/milvus-operator-mutating-webhook-configuration 创建完成
validatingwebhookconfiguration.admissionregistration.k8s.io/milvus-operator-validating-webhook-configuration 创建完成
```
你可以通过以下方式检查 Milvus Operator 的 Pod 是否正在运行：

```shell
$ kubectl get pods -n milvus-operator

NAME                               READY   STATUS    RESTARTS   AGE
milvus-operator-5fd77b87dc-msrk4   1/1     Running   0          46s
```

## 部署 Milvus

### 1. 部署 Milvus 集群

一旦 Milvus Operator 的 Pod 正在运行，您可以按照以下步骤部署 Milvus 集群。

```shell
$ kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_cluster_default.yaml
```

上述命令将使用默认配置在单独的 Pod 中部署 Milvus 集群及其组件和依赖项。如果要自定义这些设置，我们建议您使用 [Milvus Sizing Tool](https://milvus.io/tools/sizing) 根据实际数据大小调整配置，然后下载相应的 YAML 文件。要了解更多配置参数，请参阅 [Milvus 系统配置清单](https://milvus.io/docs/system_configuration.md)。

<div class="alert note">

- 发行名称应仅包含字母、数字和破折号。发行名称中不允许使用点。
- 您还可以在独立模式下部署 Milvus 实例，其中所有组件都包含在单个 Pod 中。要这样做，请将上述命令中的配置文件 URL 更改为 `https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_default.yaml`

</div>

#### 2. 检查 Milvus 集群状态

运行以下命令以检查 Milvus 集群状态

```shell
$ kubectl get milvus my-release -o yaml
```

一旦您的 Milvus 集群准备就绪，上述命令的输出应类似于以下内容。如果 `status.status` 字段仍然为 `Unhealthy`，则您的 Milvus 集群仍在创建中。

```yaml
apiVersion: milvus.io/v1alpha1
kind: MilvusCluster
metadata:
...
status:
  conditions:
  - lastTransitionTime: "2021-11-02T05:59:41Z"
    reason: StorageReady
    status: "True"
    type: StorageReady
  - lastTransitionTime: "2021-11-02T06:06:23Z"
    message: Pulsar is ready
    reason: PulsarReady
    status: "True"
    type: PulsarReady
  - lastTransitionTime: "2021-11-02T05:59:41Z"
    message: Etcd endpoints is healthy
    reason: EtcdReady
    status: "True"
    type: EtcdReady
  - lastTransitionTime: "2021-11-02T06:12:36Z"
    message: All Milvus components are healthy
    reason: MilvusClusterHealthy
    status: "True"
    type: MilvusReady
  endpoint: my-release-milvus.default:19530
  status: Healthy
```

Milvus Operator 创建 Milvus 的依赖项，如 etcd、Pulsar 和 MinIO，然后创建 Milvus 的组件，如代理、协调器和节点。

一旦您的 Milvus 集群准备就绪，Milvus 集群中所有 Pod 的状态应类似于以下内容。
```shell
$ kubectl get pods

名称                                            就绪状态   运行状态      重启次数   年龄
my-release-etcd-0                               1/1     运行中     0          14分钟
my-release-etcd-1                               1/1     运行中     0          14分钟
my-release-etcd-2                               1/1     运行中     0          14分钟
my-release-milvus-datacoord-6c7bb4b488-k9htl    1/1     运行中     0          6分钟
my-release-milvus-datanode-5c686bd65-wxtmf      1/1     运行中     0          6分钟
my-release-milvus-indexcoord-586b9f4987-vb7m4   1/1     运行中     0          6分钟
my-release-milvus-indexnode-5b9787b54-xclbx     1/1     运行中     0          6分钟
my-release-milvus-proxy-84f67cdb7f-pg6wf        1/1     运行中     0          6分钟
my-release-milvus-querycoord-865cc56fb4-w2jmn   1/1     运行中     0          6分钟
my-release-milvus-querynode-5bcb59f6-nhqqw      1/1     运行中     0          6分钟
my-release-milvus-rootcoord-fdcccfc84-9964g     1/1     运行中     0          6分钟
my-release-minio-0                              1/1     运行中     0          14分钟
my-release-minio-1                              1/1     运行中     0          14分钟
my-release-minio-2                              1/1     运行中     0          14分钟
my-release-minio-3                              1/1     运行中     0          14分钟
my-release-pulsar-bookie-0                      1/1     运行中     0          14分钟
my-release-pulsar-bookie-1                      1/1     运行中     0          14分钟
my-release-pulsar-bookie-init-h6tfz             0/1     已完成     0          14分钟
my-release-pulsar-broker-0                      1/1    
```

### 3. 将本地端口转发到 Milvus

运行以下命令以获取 Milvus 集群提供服务的端口号。

```shell
$ kubectl get pod my-release-milvus-proxy-84f67cdb7f-pg6wf --template
='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
19530
```

输出显示 Milvus 实例在默认端口 **19530** 上提供服务。

<div class="alert note">

如果您以独立模式部署了 Milvus，请将 pod 名称从 `my-release-milvus-proxy-xxxxxxxxxx-xxxxx` 更改为 `my-release-milvus-xxxxxxxxxx-xxxxx`。

</div>

然后，运行以下命令将本地端口转发到 Milvus 提供服务的端口。

```shell
$ kubectl port-forward service/my-release-milvus 27017:19530
Forwarding from 127.0.0.1:27017 -> 19530
```

可选择在上述命令中使用 `:19530` 而不是 `27017:19530`，让 `kubectl` 为您分配一个本地端口，这样您就不必管理端口冲突。

默认情况下，kubectl 的端口转发仅监听 `localhost`。如果希望 Milvus 监听选定的或所有 IP 地址，请使用 `address` 标志。以下命令使端口转发在主机上的所有 IP 地址上进行监听。

```shell
$ kubectl port-forward --address 0.0.0.0 service/my-release-milvus 27017:19530
Forwarding from 0.0.0.0:27017 -> 19530
```

## 卸载 Milvus

运行以下命令以卸载 Milvus 集群。

```shell
$ kubectl delete milvus my-release
```

<div class="alert note">

- 使用默认配置删除 Milvus 集群时，诸如 etcd、Pulsar 和 MinIO 等依赖项不会被删除。因此，下次安装相同的 Milvus 集群实例时，这些依赖项将被再次使用。
- 要删除依赖项和私有虚拟云（PVCs）以及 Milvus 集群，请参阅[配置文件](https://github.com/zilliztech/milvus-operator/blob/main/config/samples/milvus_deletion.yaml)。

</div>

## 卸载 Milvus Operator

也有两种方法可以卸载 Milvus Operator。

- [使用 Helm 卸载](#Uninstall-with-Helm)
- [使用 kubectl 卸载](#Uninstall-with-kubectl)

#### 使用 Helm 卸载

```shell
$ helm -n milvus-operator uninstall milvus-operator
```

#### 使用 kubectl 卸载

```shell
$ kubectl delete -f https://raw.githubusercontent.com/zilliztech/milvus-operator/v{{var.milvus_operator_version}}/deploy/manifests/deployment.yaml
```

## 下一步操作

在 Docker 中安装了 Milvus 后，您可以：

- 查看[Hello Milvus](quickstart.md)以了解 Milvus 的功能。

- 学习 Milvus 的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- [使用 Helm Chart 升级 Milvus](upgrade_milvus_cluster-helm.md)。
- [扩展您的 Milvus 集群](scaleout.md)。
- 在云上部署您的 Milvu 集群：
  - [Amazon EC2](aws.md)
  - [Amazon EKS](eks.md)
  - [Google Cloud](gcp.md)
  - [Google Cloud Storage](gcs.md)
  - [Microsoft Azure](azure.md)
  - [Microsoft Azure Blob Storage](abs.md)
- 探索 [Milvus 备份](milvus_backup_overview.md)，这是一个用于 Milvus 数据备份的开源工具。
- 探索 [Birdwatcher](birdwatcher_overview.md)，这是一个用于调试 Milvus 和动态配置更新的开源工具。
- 探索 [Attu](https://milvus.io/docs/attu.md)，这是一个直观的 Milvus 管理工具，提供开源的图形用户界面。
- 使用 Prometheus [监控 Milvus](monitor.md)。