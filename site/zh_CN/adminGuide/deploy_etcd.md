---
id: deploy_etcd.md
title: 使用 Docker Compose 或 Helm 配置元数据存储
related_key: S3, 存储
summary: 学习如何使用 Docker Compose/Helm 为 Milvus 配置元数据存储。
---

# 使用 Docker Compose 或 Helm 配置元数据存储

Milvus 使用 etcd 来存储元数据。本主题介绍如何使用 Docker Compose 或 Helm 配置 etcd。

## 使用 Docker Compose 配置 etcd

### 1. 配置 etcd

要使用 Docker Compose 配置 etcd，请在 milvus/configs 路径下的 `milvus.yaml` 文件中为 `etcd` 部分提供您的值。

```
etcd:
  endpoints:
    - localhost:2379
  rootPath: by-dev # 数据存储在 etcd 中的根路径
  metaSubPath: meta # metaRootPath = rootPath + '/' + metaSubPath
  kvSubPath: kv # kvRootPath = rootPath + '/' + kvSubPath
  log:
    # path 可以是以下之一：
    #  - "default" 代表 os.Stderr，
    #  - "stderr" 代表 os.Stderr，
    #  - "stdout" 代表 os.Stdout，
    #  - 文件路径，用于追加服务器日志。
    # 请在内嵌 Milvus 中进行调整：/tmp/milvus/logs/etcd.log
    path: stdout
    level: info # 仅支持 debug、info、warn、error、panic 或 fatal。默认为 'info'。
  use:
    # 请在内嵌 Milvus 中进行调整：true
    embed: false # 是否启用内嵌 Etcd（一个进程内的 EtcdServer）。
  data:
    # 仅适用于内嵌 Etcd。
    # 请在内嵌 Milvus 中进行调整：/tmp/milvus/etcdData/
    dir: default.etcd
```

查看 [etcd 相关配置](configure_etcd.md) 以获取更多信息。

### 2. 启动 Milvus

运行以下命令以启动使用 etcd 配置的 Milvus。

```
docker compose up
```

<div class="alert note">配置仅在 Milvus 启动后生效。查看 <a href=https://milvus.io/docs/install_standalone-docker.md#Start-Milvus>启动 Milvus</a> 以获取更多信息。</div>

## 在 K8s 上配置 etcd

对于在 K8s 上的 Milvus 集群，您可以在启动 Milvus 的同一命令中配置 etcd。或者，您可以在启动 Milvus 之前在 [milvus-helm](https://github.com/milvus-io/milvus-helm) 仓库的 /charts/milvus 路径下使用 <code>values.yml</code> 文件配置 etcd。

以下表格列出了在 YAML 文件中配置 etcd 的键。
| 键名             | 描述                          | 值                                 |
| --------------------- | ------------------------------------ | ------------------------------------ |
| <code>etcd.enabled</code>           | 启用或禁用 etcd。          | <code>true</code>/<code>false</code> |
| <code>externalEtcd.enabled</code>   | 启用或禁用外部 etcd。 | <code>true</code>/<code>false</code> |
| <code>externalEtcd.endpoints</code> | 访问 etcd 的端点。       |                                      |



### 使用 YAML 文件

1. 在 <code>values.yaml</code> 文件中使用您的值配置 <code>etcd</code> 部分。

```yaml
etcd:
  enabled: false
```
2. 在 <code>values.yaml</code> 文件中使用你的数值配置 <code>externaletcd</code> 部分。

```yaml
externalEtcd:
  enabled: true
  ## 外部 etcd 的端点
  endpoints:
    - <your_etcd_IP>:2379
```

3. 配置上述部分并保存 <code>values.yaml</code> 文件后，运行以下命令安装使用 etcd 配置的 Milvus。

```shell
helm install <your_release_name> milvus/milvus -f values.yaml
```

### 使用命令

要安装 Milvus 并配置 etcd，请使用以下命令并填入你的数值。

```shell
helm install <your_release_name> milvus/milvus --set cluster.enabled=true --set etcd.enabled=false --set externaletcd.enabled=true --set externalEtcd.endpoints={<your_etcd_IP>:2379}
```

## 下一步

学习如何使用 Docker Compose 或 Helm 配置其他 Milvus 依赖项：

- [使用 Docker Compose 或 Helm 配置对象存储](deploy_s3.md)
- [使用 Docker Compose 或 Helm 配置消息存储](deploy_pulsar.md)