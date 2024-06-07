---
id: operational_faq.md
summary: 在 Milvus 运维中常见问题的解答。
title: 运维常见问题解答
---

# 运维常见问题解答

<!-- TOC -->


<!-- /TOC -->

#### 如果无法从 Docker Hub 拉取 Milvus Docker 镜像怎么办？

如果无法从 Docker Hub 拉取 Milvus Docker 镜像，可以尝试添加其他镜像源。

中国大陆用户可以在 **/etc.docker/daemon.json** 文件的 registry-mirrors 数组中添加 URL "https://registry.docker-cn.com"。

```
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

#### Docker 是安装和运行 Milvus 的唯一方式吗？

Docker 是部署 Milvus 的高效方式之一，但不是唯一方式。您也可以从源代码部署 Milvus。这需要 Ubuntu（18.04 或更高版本）或 CentOS（7 或更高版本）。更多信息请参见 [从源代码构建 Milvus](https://github.com/milvus-io/milvus#build-milvus-from-source-code)。

#### 影响召回率的主要因素是什么？

召回率主要受索引类型和搜索参数影响。

对于 FLAT 索引，Milvus 在集合内进行详尽扫描，返回率为 100%。

对于 IVF 索引，nprobe 参数确定了在集合内搜索的范围。增加 nprobe 可以增加搜索的向量比例和召回率，但会降低查询性能。

对于 HNSW 索引，ef 参数确定了图搜索的广度。增加 ef 可以增加在图上搜索的点数和召回率，但会降低查询性能。

更多信息，请参见 [向量索引](https://www.zilliz.com/blog/Accelerating-Similarity-Search-on-Really-Big-Data-with-Vector-Indexing)。

#### 为什么对配置文件的更改没有生效？

Milvus 不支持在运行时修改配置文件。您必须重新启动 Milvus Docker，使配置文件更改生效。

#### 如何知道 Milvus 是否成功启动？

如果使用 Docker Compose 启动 Milvus，请运行 `docker ps` 查看运行的 Docker 容器数量，以及检查 Milvus 服务是否正确启动。

对于独立部署的 Milvus，您应该至少看到三个正在运行的 Docker 容器，一个是 Milvus 服务，另外两个是 etcd 管理和存储服务。更多信息请参见 [安装独立部署的 Milvus](install_standalone-docker.md)。

#### 为什么日志文件中的时间与系统时间不一致？

时间差异通常是由于主机机器未使用协调世界时（UTC）造成的。

Docker 镜像内的日志文件默认使用 UTC。如果您的主机机器未使用 UTC，可能会出现这个问题。

#### 如何知道我的 CPU 是否支持 Milvus？

{{fragments/cpu_support.md}}

#### 为什么 Milvus 在启动过程中返回 `illegal instruction`？
Milvus 需要您的 CPU 支持 SIMD 指令集：SSE4.2、AVX、AVX2 或 AVX512。CPU 必须至少支持其中一种，以确保 Milvus 正常运行。在启动过程中返回 `illegal instruction` 错误表明您的 CPU 不支持上述四种指令集中的任何一种。

请参阅 [CPU’s support for SIMD Instruction Set](prerequisite-docker.md)。

#### 我可以在 Windows 上安装 Milvus 吗？

可以。您可以通过从源代码编译或使用二进制包在 Windows 上安装 Milvus。

请参阅 [Run Milvus on Windows](https://milvus.io/blog/2021-11-19-run-milvus-2.0-on-windows.md) 了解如何在 Windows 上安装 Milvus。

#### 在 Windows 上安装 pymilvus 时出现错误。我该怎么办？

不建议在 Windows 上安装 PyMilvus。但如果您必须在 Windows 上安装 PyMilvus 但遇到错误，请尝试在 [Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html) 环境中安装。有关如何在 Conda 环境中安装 PyMilvus 的更多信息，请参阅 [Install Milvus SDK](install-pymilvus.md)。

#### 在没有连接到互联网时，我可以部署 Milvus 吗？

可以。您可以在离线环境中安装 Milvus。有关更多信息，请参阅 [Install Milvus Offline](install_offline-helm.md)。

#### 我在哪里可以找到 Milvus 生成的日志？

Milvus 日志默认打印到标准输出（stdout）和标准错误（stderr），但我们强烈建议在生产环境中将日志重定向到持久卷。要这样做，请更新 **milvus.yaml** 中的 `log.file.rootPath`。如果您使用 `milvus-helm` 图表部署 Milvus，还需要通过 `--set log.persistence.enabled=true` 启用日志持久化。

如果您没有更改配置，使用 kubectl logs <pod-name> 或 docker logs CONTAINER 也可以帮助您找到日志。

#### 在向段插入数据之前，我可以为其创建索引吗？

可以。但我们建议在为每个段创建索引之前，将数据分批插入，每批不应超过 256 MB。

#### 我可以在多个 Milvus 实例之间共享一个 etcd 实例吗？

可以。您可以在多个 Milvus 实例之间共享一个 etcd 实例。要这样做，您需要在每个 Milvus 实例的配置文件中将 `etcd.rootPath` 更改为不同的值，然后再启动它们。

#### 我可以在多个 Milvus 实例之间共享一个 Pulsar 实例吗？

可以。您可以在多个 Milvus 实例之间共享一个 Pulsar 实例。要这样做，您可以：

- 如果您的 Pulsar 实例启用了多租户模式，请考虑为每个 Milvus 实例分配一个单独的租户或命名空间。为此，您需要在每个 Milvus 实例的配置文件中将 `pulsar.tenant` 或 `pulsar.namespace` 更改为唯一值，然后再启动它们。
- 如果您不打算在您的 Pulsar 实例上启用多租户功能，请在启动之前将 Milvus 实例的配置文件中的 `msgChannel.chanNamePrefix.cluster` 更改为每个实例的唯一值。

#### 我可以在多个 Milvus 实例之间共享一个 MinIO 实例吗？

是的，您可以在多个 Milvus 实例之间共享一个 MinIO 实例。为此，您需要在每个 Milvus 实例的配置文件中将 `minio.rootPath` 更改为一个唯一值，然后再启动它们。

#### 仍有疑问吗？

您可以：

- 查看 [Milvus](https://github.com/milvus-io/milvus/issues) 的 GitHub 页面。随时提出问题，分享想法，并帮助他人。
- 加入我们的 [Milvus 论坛](https://discuss.milvus.io/) 或 [Slack 频道](https://join.slack.com/t/milvusio/shared_invite/enQtNzY1OTQ0NDI3NjMzLWNmYmM1NmNjOTQ5MGI5NDhhYmRhMGU5M2NhNzhhMDMzY2MzNDdlYjM5ODQ5MmE3ODFlYzU3YjJkNmVlNDQ2ZTk)，以获取支持并与我们的开源社区互动。