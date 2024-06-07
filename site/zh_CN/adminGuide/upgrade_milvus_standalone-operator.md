---
id: upgrade_milvus_standalone-operator.md
label: Milvus Operator
order: 0
group: upgrade_milvus_standalone-operator.md
related_key: upgrade Milvus Standalone
summary: 学习如何使用 Milvus Operator 升级 Milvus 独立版。
title: 使用 Milvus Operator 升级 Milvus 独立版
---

{{tab}}

# 使用 Milvus Operator 升级 Milvus 独立版

本指南介绍了如何使用 Milvus Operator 升级您的 Milvus 独立版。

## 升级您的 Milvus Operator

运行以下命令将您的 Milvus Operator 版本升级至 v{{var.milvus_operator_version}}。

```
helm repo add zilliztech-milvus-operator https://zilliztech.github.io/milvus-operator/
helm repo update zilliztech-milvus-operator
helm -n milvus-operator upgrade milvus-operator zilliztech-milvus-operator/milvus-operator
```

一旦将您的 Milvus Operator 升级至最新版本，您将有以下选择：

- 要将 Milvus 从 v2.2.3 或更高版本升级至 {{var.milvus_release_version}}，您可以[执行滚动升级](#Conduct-a-rolling-upgrade)。
- 要将 Milvus 从 v2.2.3 之前的次要版本升级至 {{var.milvus_release_version}}，建议您通过[更改其镜像版本升级 Milvus](#Upgrade-Milvus-by-changing-its-image)。
- 要将 Milvus 从 v2.1.x 升级至 {{var.milvus_release_version}}，您需要在实际升级之前[迁移元数据](#Migrate-the-metadata)。

## 执行滚动升级

自 Milvus 2.2.3 版本开始，您可以配置 Milvus 协调器以工作在主备模式，并为它们启用滚动升级功能，以便在协调器升级期间 Milvus 能够响应传入请求。在之前的版本中，升级期间需要删除然后创建协调器，这可能会引入一定的服务停机时间。

基于 Kubernetes 提供的滚动更新功能，Milvus Operator 根据依赖关系强制执行部署的有序更新。此外，Milvus 实现了一种机制，确保其组件在升级期间与依赖于它们的组件保持兼容，大大减少潜在的服务停机时间。

滚动升级功能默认处于禁用状态。您需要通过配置文件显式启用它。

```yaml
apiVersion: milvus.io/v1beta1
kind: Milvus
metadata:
  name: my-release
spec:
  components:
    enableRollingUpdate: true
    imageUpdateMode: rollingUpgrade # 默认值，可省略
    image: milvusdb/milvus:v{{var.milvus_release_tag}}
```

在上述配置文件中，将 `spec.components.enableRollingUpdate` 设置为 `true`，将 `spec.components.image` 设置为所需的 Milvus 版本。

默认情况下，Milvus 以有序方式执行协调器的滚动升级，逐个替换协调器 pod 图像。为缩短升级时间，考虑将 `spec.components.imageUpdateMode` 设置为 `all`，以便 Milvus 同时替换所有 pod 图像。
```yaml
apiVersion: milvus.io/v1beta1
kind: Milvus
metadata:
  name: my-release
spec:
  components:
    enableRollingUpdate: true
    imageUpdateMode: all
    image: milvusdb/milvus:v{{var.milvus_release_tag}}
```
你可以将 `spec.components.imageUpdateMode` 设置为 `rollingDowngrade`，这样 Milvus 将用较低版本替换协调器 pod 的镜像。

```yaml
apiVersion: milvus.io/v1beta1
kind: Milvus
metadata:
  name: my-release
spec:
  components:
    enableRollingUpdate: true
    imageUpdateMode: rollingDowngrade
    image: milvusdb/milvus:<some-older-version>
```

然后将配置保存为一个 YAML 文件（例如 `milvusupgrade.yml`），并按以下方式将此配置文件应用于您的 Milvus 实例：

```shell
kubectl apply -f milvusupgrade.yml
```

## 通过更改镜像升级 Milvus

在正常情况下，您可以通过更改镜像将 Milvus 简单地升级到最新版本。但是，请注意，以这种方式升级 Milvus 时会有一定的停机时间。

按照以下方式组成一个配置文件，并将其保存为 **milvusupgrade.yaml**：

```yaml
apiVersion: milvus.io/v1beta1
kind: Milvus
metadata:
    name: my-release
labels:
    app: milvus
spec:
  # 省略其他字段 ...
  components:
   image: milvusdb/milvus:v{{var.milvus_release_tag}}
```

然后运行以下命令执行升级：

```shell
kubectl apply -f milvusupgrade.yaml
```

## 迁移元数据

自 Milvus 2.2.0 版本开始，元数据与之前版本不兼容。以下示例代码片段假定从 Milvus 2.1.4 升级到 Milvus v{{var.milvus_release_version}}。

### 1. 创建用于元数据迁移的 `.yaml` 文件

创建一个元数据迁移文件。以下是一个示例。您需要在配置文件中指定 `name`、`sourceVersion` 和 `targetVersion`。以下示例将 `name` 设置为 `my-release-upgrade`，`sourceVersion` 设置为 `v2.1.4`，`targetVersion` 设置为 `v{{var.milvus_release_version}}`。这意味着您的 Milvus 实例将从 v2.1.4 升级到 v{{var.milvus_release_version}}。

```
apiVersion: milvus.io/v1beta1
kind: MilvusUpgrade
metadata:
  name: my-release-upgrade
spec:
  milvus:
    namespace: default
    name: my-release
  sourceVersion: "v2.1.4"
  targetVersion: "v{{var.milvus_release_version}}"
  # 以下是一些省略的默认值:
  # targetImage: "milvusdb/milvus:v{{var.milvus_release_tag}}"
  # toolImage: "milvusdb/meta-migration:v2.2.0"
  # operation: upgrade
  # rollbackIfFailed: true
  # backupPVC: ""
  # maxRetry: 3
```

### 2. 应用新配置

运行以下命令应用新配置。

```
$ kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvusupgrade.yaml
```



### 3. 检查元数据迁移的状态

运行以下命令检查元数据迁移的状态。

```
kubectl describe milvus release-name
```

输出中的 `ready` 状态表示元数据迁移成功。

或者，您也可以运行 `kubectl get pod` 来检查所有的 pod。如果所有的 pod 都是 `ready` 状态，那么元数据迁移就成功了。



### 4. 删除 `my-release-upgrade`
升级成功后，在 YAML 文件中删除 `my-release-upgrade`。