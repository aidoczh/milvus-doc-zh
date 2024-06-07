---
id: prerequisite-docker.md
label: Docker要求
related_key: Docker
summary: 在使用Docker Compose安装Milvus之前，了解必要的准备工作。
title: 要求
---

# 要求

在安装Milvus实例之前，请检查您的硬件和软件是否符合以下要求。

## 硬件要求

| 组件                 | 要求                                                         |推荐配置| 备注                                                         |
| ------------------- | ------------------------------------------------------------ |--------------| ------------------------------------------------------------ |
| CPU                 | <ul><li>Intel第二代酷睿CPU或更高</li><li>Apple Silicon</li></ul>  |<ul><li>独立部署：4核或更多</li><li>集群部署：8核或更多</li></ul>|  |
| CPU指令集 | <ul><li>SSE4.2</li><li>AVX</li><li>AVX2</li><li>AVX-512</li></ul> |<ul><li>SSE4.2</li><li>AVX</li><li>AVX2</li><li>AVX-512</li></ul> |  Milvus内的向量相似度搜索和索引构建需要CPU支持单指令，多数据（SIMD）扩展集。确保CPU至少支持所列SIMD扩展中的一种。有关更多信息，请参阅[支持AVX指令集的CPU](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX)。                           |
| RAM                 | <ul><li>独立部署：8G</li><li>集群部署：32G</li></ul>       |<ul><li>独立部署：16G</li><li>集群部署：128G</li></ul>        | RAM的大小取决于数据量。                  |
| 硬盘          | SATA 3.0固态硬盘或更高                                       | NVMe固态硬盘或更高 | 硬盘的大小取决于数据量。           |

## 软件要求

| 操作系统           | 软件                                                     | 备注                                                         |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| macOS 10.14或更高       | Docker Desktop                                               | 设置Docker虚拟机（VM）使用至少2个虚拟CPU（vCPUs）和8 GB的初始内存。否则，安装可能会失败。<br/>有关更多信息，请参阅[在Mac上安装Docker Desktop](https://docs.docker.com/desktop/mac/install/)。 |
| Linux平台            | <ul><li>Docker 19.03或更高</li><li>Docker Compose 1.25.1或更高</li></ul> | 有关更多信息，请参阅[安装Docker Engine](https://docs.docker.com/engine/install/)和[安装Docker Compose](https://docs.docker.com/compose/install/)。 |
| 启用 WSL 2 的 Windows | Docker Desktop | 我们建议将源代码和其他数据绑定到 Linux 容器中，存储在 Linux 文件系统中，而不是 Windows 文件系统中。<br/>更多信息请参阅 [在启用 WSL 2 后在 Windows 上安装 Docker Desktop](https://docs.docker.com/desktop/windows/install/#wsl-2-backend)。 |

| 软件 | 版本 | 备注 |
| ---- | ---- | ---- |
| etcd | 3.5.0 | 请参阅 [额外的磁盘需求](#Additional-disk-requirements)。 |
| MinIO | RELEASE.2023-03-20T20-16-18Z | |
| Pulsar | 2.8.2 | |

### 额外的磁盘需求

磁盘性能对于 etcd 至关重要。强烈建议使用本地 NVMe SSD。较慢的磁盘响应可能导致频繁的集群选举，最终会降低 etcd 服务的性能。

要测试您的磁盘是否符合要求，请使用 [fio](https://github.com/axboe/fio)。

```bash
mkdir test-data
fio --rw=write --ioengine=sync --fdatasync=1 --directory=test-data --size=2200m --bs=2300 --name=mytest
```

理想情况下，您的磁盘应该达到每秒超过 500 次 IOPS，且第 99 百分位的 fsync 延迟应低于 10 毫秒。阅读 etcd 的[文档](https://etcd.io/docs/v3.5/op-guide/hardware/#disks)以获取更详细的要求。

## 接下来做什么

如果您的硬件和软件符合上述要求，您可以

- [在 Docker 中运行 Milvus](install_standalone-docker.md)
- [使用 Docker Compose 运行 Milvus](install_standalone-docker-compose.md)