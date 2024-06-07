---
id: install_standalone-aptyum.md
label: DEB/RPM
related_key: Install
order: 3
group: install_standalone-docker.md
summary: 学习如何使用 dpkg/yum 在 Linux 系统上安装 Milvus 独立版。
title: 使用 dpkg/yum 安装 Milvus 独立版
---

{{tab}}

# 使用 dpkg/yum 安装 Milvus 独立版

本主题介绍了如何在 Linux 系统上使用软件包管理器 dpkg 或 yum 安装 Milvus 独立版。


## 先决条件

在安装之前，请查看[要求](prerequisite-docker.md)中的硬件和软件要求。

## 安装 Milvus

### 在 Ubuntu 上使用 dpkg 安装 Milvus

```bash
$ wget https://github.com/milvus-io/milvus/releases/download/v2.4.1/milvus_2.4.1-1_amd64.deb
$ sudo apt-get update
$ sudo dpkg -i milvus_2.4.1-1_amd64.deb
$ sudo apt-get -f install
```

### 在 RedHat9 上使用 yum 安装 Milvus

```bash
$ sudo yum install -y https://github.com/milvus-io/milvus/releases/download/v2.4.1/milvus-2.4.1-1.el9.x86_64.rpm
```

## 检查 Milvus 的状态

```bash
$ sudo systemctl restart milvus
$ sudo systemctl status milvus
```

## 连接到 Milvus

请参考[Hello Milvus](https://milvus.io/docs/example_code.md)，然后运行示例代码。

## 卸载 Milvus

### 在 Ubuntu 上卸载 Milvus

```bash
$ sudo dpkg -P milvus
```

### 在 RedHat9 上卸载 Milvus

```bash
$ sudo yum remove -y milvus
```

## 下一步

安装了 Milvus 后，您可以：

- 查看[Hello Milvus](quickstart.md)以使用不同 SDK 运行示例代码，了解 Milvus 的功能。
- 查看[内存索引](index.md)以了解更多关于 CPU 兼容索引类型的信息。

- 学习 Milvus 的基本操作：
  - [管理数据库](manage_databases.md)
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- 探索[Milvus 备份](milvus_backup_overview.md)，这是一个用于 Milvus 数据备份的开源工具。
- 探索[Birdwatcher](birdwatcher_overview.md)，这是一个用于调试 Milvus 和动态配置更新的开源工具。
- 探索[Attu](https://milvus.io/docs/attu.md)，这是一个直观的 Milvus 管理工具。
- [使用 Prometheus 监控 Milvus](monitor.md)
