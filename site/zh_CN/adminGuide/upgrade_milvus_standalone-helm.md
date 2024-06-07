---
id: upgrade_milvus_standalone-helm.md
label: Helm
order: 1
group: upgrade_milvus_standalone-operator.md
related_key: upgrade Milvus Standalone
summary: 学习如何使用 Helm Chart 升级 Milvus 独立版。
title: 使用 Helm Chart 升级 Milvus 独立版
---

{{tab}}

# 使用 Helm Chart 升级 Milvus 独立版

本指南介绍了如何使用 Milvus Helm charts 升级您的 Milvus 独立版。

## 检查 Milvus 版本

运行以下命令以检查新的 Milvus 版本。

```
$ helm repo update
$ helm search repo zilliztech/milvus --versions
```

<div class="alert note">

Milvus Helm Charts 仓库位于 `https://milvus-io.github.io/milvus-helm/` 已存档，您可以从 `https://zilliztech.github.io/milvus-helm/` 获取进一步更新，操作如下：

```shell
helm repo add zilliztech https://zilliztech.github.io/milvus-helm
helm repo update
# 升级现有的 helm 发布
helm upgrade my-release zilliztech/milvus
```

存档的仓库仍可用于 4.0.31 版本的图表。对于更新版本，请改用新的仓库。

</div>

```                                       
名称                      图表版本        应用程序版本            描述                                       
zilliztech/milvus       4.1.33          2.4.4                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.32          2.4.3                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.31          2.4.1                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.30          2.4.1                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.29          2.4.0                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.24          2.3.11                  Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.23          2.3.10                  Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.22          2.3.10                  Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.21          2.3.10                  Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.20          2.3.10                  Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.18          2.3.10                  Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.18          2.3.9                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.17          2.3.8                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.16          2.3.7                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.15          2.3.5                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.14          2.3.6                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.13          2.3.5                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.12          2.3.5                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.11          2.3.4                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.10          2.3.3                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.9           2.3.3                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.8           2.3.2                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.7           2.3.2                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.6           2.3.1                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.5           2.3.1                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.4           2.3.1                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.3           2.3.1                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.2           2.3.1                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.1           2.3.0                   Milvus 是一个开源的向量数据库，构建于...
zilliztech/milvus       4.1.0           2.3.0                   Milvus 是一个开源的向量数据库，构建于...
```
您可以按照以下方式为您的 Milvus 选择升级路径：

<div style="display: none;">- [执行滚动升级](#执行滚动升级) 从 Milvus v2.2.3 及更高版本升级到 v{{var.milvus_release_version}}。</div>

- [使用 Helm 升级 Milvus](#使用-Helm-升级-Milvus) 从 v2.2.3 之前的次要版本升级到 v{{var.milvus_release_version}}。

- [迁移元数据](#迁移元数据) 在从 Milvus v2.1.x 升级到 v{{var.milvus_release_version}} 之前。

<div style="display:none;">

## 执行滚动升级

自 Milvus 2.2.3 版本开始，您可以配置 Milvus 协调器以工作在主备模式，并为其启用滚动升级功能，以便在协调器升级期间 Milvus 能够响应传入请求。在之前的版本中，升级过程中需要删除然后重新创建协调器，这可能会导致服务的某些停机时间。

滚动升级要求协调器以主备模式工作。您可以使用我们提供的 [脚本](https://raw.githubusercontent.com/milvus-io/milvus/master/deployments/upgrade/rollingUpdate.sh) 来配置协调器以主备模式工作并启动滚动升级。

基于 Kubernetes 提供的滚动更新功能，上述脚本根据其依赖关系强制执行部署的有序更新。此外，Milvus 实现了一种机制，确保其组件在升级期间与依赖于它们的组件保持兼容，显著减少潜在的服务停机时间。

该脚本仅适用于使用 Helm 安装的 Milvus 的升级。下表列出了脚本中可用的命令标志。

| 参数         | 描述                                                     | 默认值                           | 是否必需                |
| ------------ | ----------------------------------------------------------| -------------------------------- | ----------------------- |
| `i`          | Milvus 实例名称                                           | `None`                           | 是                      |
| `n`          | Milvus 安装的命名空间                                     | `default`                        | 否                      |
| `t`          | 目标 Milvus 版本                                          | `None`                           | 是                      |
| `w`          | 新 Milvus 镜像标签                                       | `milvusdb/milvus:v2.2.3`         | 是                      |
| `o`          | 操作                                                     | `update`                         | 否                      |

一旦确保您的 Milvus 实例中的所有部署处于正常状态，您可以运行以下命令将 Milvus 实例升级到 {{var.milvus_release_version}}。
```shell
sh rollingUpdate.sh -n default -i my-release -o update -t {{var.milvus_release_version}} -w 'milvusdb/milvus:v{{var.milvus_release_tag}}'
```

<div class="alert note">

1. 脚本**不适用**于使用**RocksMQ**安装的 Milvus 实例。
2. 脚本硬编码了部署的升级顺序，无法更改。
3. 脚本使用 `kubectl patch` 更新部署，并使用 `kubectl rollout status` 监视其状态。
4. 脚本使用 `kubectl patch` 更新部署的 `app.kubernetes.io/version` 标签，该标签的值为在命令中 `-t` 标志之后指定的值。

</div>
    
</div>

## 使用 Helm 升级 Milvus

要将 Milvus 从 v2.2.3 之前的次要版本升级到最新版本，请运行以下命令：

```shell
helm repo update
helm upgrade my-release milvus/milvus --reuse-values --version={{var.milvus_helm_chart_version}} # 在此处使用 Helm 图表版本
```

在上述命令中使用 Helm 图表版本。有关如何获取 Helm 图表版本的详细信息，请参阅[检查 Milvus 版本](#Check-the-Milvus-version)。

## 迁移元数据

自 Milvus 2.2.0 起，元数据与之前版本不兼容。以下示例代码片段假定从 Milvus 2.1.4 升级到 Milvus 2.2.0。

### 1. 检查 Milvus 版本

运行 `$ helm list` 来检查您的 Milvus 应用程序版本。您可以看到 `APP VERSION` 是 2.1.4。

```
NAME             NAMESPACEREVISIONUPDATED                                STATUS  CHART        APP VERSION     
my-release      default  1       2022-11-21 15:41:25.51539 +0800 CST    deployedmilvus-3.2.182.1.4
```

### 2. 检查运行中的 Pod

运行 `$ kubectl get pods` 来检查正在运行的 Pod。您可以看到以下输出。

```
NAME                                            READY   STATUS    RESTARTS   AGE
my-release-etcd-0                               1/1     Running   0          84s
my-release-milvus-standalone-75c599fffc-6rwlj   1/1     Running   0          84s
my-release-minio-744dd9586f-qngzv               1/1     Running   0          84s
```

### 3. 检查镜像标签

检查 Pod `my-release-milvus-proxy-6c548f787f-scspp` 的镜像标签。您可以看到您的 Milvus 集群的版本是 v2.1.4。

```shell
$ kubectl get pods my-release-milvus-proxy-6c548f787f-scspp -o=jsonpath='{$.spec.containers[0].image}'
# milvusdb/milvus:v2.1.4
```

### 4. 迁移元数据

Milvus 2.2 中的一个重大变化是段索引的元数据结构。因此，在将 Milvus 从 v2.1.x 升级到 v2.2.0 时，您需要使用 Helm 迁移元数据。这里有[一个脚本](https://github.com/milvus-io/milvus/blob/master/deployments/migrate-meta/migrate.sh)供您安全地迁移您的元数据。

此脚本仅适用于安装在 K8s 集群上的 Milvus。如果在过程中出现错误，请首先使用回滚操作回滚到先前的版本。

以下表列出了您可以执行的元数据迁移操作。

| 参数         | 描述                             | 默认值          | 必需的       |
| ------------ | ---------------------------------------------------------------- | ---------------------------- | ----------------------- |
| `i`          | Milvus 实例名称。                                         | `None`                         | True                    |
| `n`          | Milvus 安装的命名空间。                                    | `default`                      | False                   |
| `s`          | Milvus 的源版本。                                         | `None`                         | True                    |
| `t`          | Milvus 的目标版本。                                       | `None`                         | True                    |
| `r`          | Milvus 元数据的根路径。                                   | `by-dev`                       | False                   |
| `w`          | 新的 Milvus 镜像标签。                                    | `milvusdb/milvus:v2.2.0`       | False                   |
| `m`          | 元数据迁移镜像标签。                                     | `milvusdb/meta-migration:v2.2.0`       | False                   |
| `o`          | 元数据迁移操作。                                         | `migrate`                      | False                   |
| `d`          | 迁移完成后是否删除迁移 pod。                              | `false`                        | False                   |
| `c`          | 用于元数据迁移 pvc 的存储类。                             | `default storage class`          | False                   |
| `e`          | Milvus 使用的 etcd 终端点。                               | `etcd svc installed with milvus` | False                   |

#### 1. 迁移元数据

1. 下载 [迁移脚本](https://github.com/milvus-io/milvus/blob/master/deployments/migrate-meta/migrate.sh)。
2. 停止 Milvus 组件。Milvus etcd 中的任何活动会话都可能导致迁移失败。
3. 为 Milvus 元数据创建备份。
4. 迁移 Milvus 元数据。
5. 使用新镜像启动 Milvus 组件。

#### 2. 从 v2.1.x 升级 Milvus 到 {{var.milvus_release_version}}

以下命令假定您将 Milvus 从 v2.1.4 升级到 {{var.milvus_release_version}}。请根据您的需求更改版本。

1. 指定 Milvus 实例名称、源 Milvus 版本和目标 Milvus 版本。

    ```
    ./migrate.sh -i my-release -s 2.1.4 -t {{var.milvus_release_version}}
    ```

2. 如果您的 Milvus 未安装在默认的 K8s 命名空间中，请使用 `-n` 指定命名空间。

    ```
    ./migrate.sh -i my-release -n milvus -s 2.1.4 -t {{var.milvus_release_version}}
    ```

3. 如果您的 Milvus 是使用自定义 `rootpath` 安装的，请使用 `-r` 指定根路径。

    ```
    ./migrate.sh -i my-release -n milvus -s 2.1.4 -t {{var.milvus_release_version}} -r by-dev
    ```

4. 如果您的 Milvus 是使用自定义 `image` 安装的，请使用 `-w` 指定镜像标签。

    ```
    s./migrate.sh -i my-release -n milvus -s 2.1.4 -t {{var.milvus_release_version}} -r by-dev -w milvusdb/milvus:v{{var.milvus_release_tag}}
    ```

5. 如果您希望在迁移完成后自动删除迁移 pod，请设置 `-d true`。

    ```shell
    ./migrate.sh -i my-release -n milvus -s 2.1.4 -t {{var.milvus_release_version}} -w milvusdb/milvus:v{{var.milvus_release_tag}} -d true
    ```

6. 如果迁移失败，可以执行回滚操作后再次尝试迁移。

    ```shell
    ./migrate.sh -i my-release -n milvus -s 2.1.4 -t {{var.milvus_release_version}} -r by-dev -o rollback -w milvusdb/milvus:v2.1.1
    ./migrate.sh -i my-release -n milvus -s 2.1.4 -t {{var.milvus_release_version}} -r by-dev -o migrate -w milvusdb/milvus:v{{var.milvus_release_tag}}

