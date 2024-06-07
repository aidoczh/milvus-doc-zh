---
id: install-go.md
label: 安装 GO SDK
related_key: SDK
summary: 学习如何安装 Milvus 的 GO SDK。
title: 安装 Milvus Go SDK
---

# 安装 Milvus Go SDK

本主题介绍了如何为 Milvus 安装 GO SDK。

当前版本的 Milvus 支持 Python、Node.js、GO 和 Java 的 SDK。

## 要求

需要 GO (1.17 或更高版本)。

## 安装 Milvus GO SDK

通过 `go get` 安装 Milvus GO SDK 及其依赖。

```bash
$ go get -u github.com/milvus-io/milvus-sdk-go/v2
```

## 下一步操作

安装了 Milvus GO SDK 后，您可以：

- 学习 Milvus 的基本操作：
  - [管理集合](manage-collections.md)
  - [管理分区](manage-partitions.md)
  - [插入、更新和删除](insert-update-delete.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)
