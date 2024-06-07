---
id: install-pymilvus.md
label: 安装 PyMilvus
related_key: SDK
summary: 学习如何安装 Milvus 的 Python SDK。
title: 安装 Milvus Python SDK
---

# 安装 Milvus Python SDK

本主题介绍了如何为 Milvus 安装 Python SDK pymilvus。

当前版本的 Milvus 支持 Python、Node.js、GO 和 Java 的 SDK。

## 环境要求

- 需要 Python 3.7 或更高版本。
- 需要安装 Google protobuf。您可以使用命令 `pip3 install protobuf==3.20.0` 进行安装。
- 需要安装 grpcio-tools。您可以使用命令 `pip3 install grpcio-tools` 进行安装。

## 通过 pip 安装 PyMilvus

PyMilvus 可以在 [Python Package Index](https://pypi.org/project/pymilvus/) 中找到。

<div class="alert note">
建议安装与您所安装的 Milvus 服务器版本匹配的 PyMilvus 版本。有关更多信息，请参阅 <a href="release_notes.md">发布说明</a>。
</div>

```
$ python3 -m pip install pymilvus=={{var.milvus_python_sdk_real_version}}
```

## 验证安装

如果 PyMilvus 安装正确，当您运行以下命令时不会引发任何异常。

```
$ python3 -c "from pymilvus import Collection"
```



## 下一步

安装了 PyMilvus 后，您可以：

- 学习 Milvus 的基本操作：
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)

- 探索 [PyMilvus API 参考](/api-reference/pymilvus/v{{var.milvus_python_sdk_version}}/About.md)