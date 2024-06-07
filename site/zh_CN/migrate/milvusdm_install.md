---
id: milvusdm_install.md
summary: 学习如何安装 Milvus 迁移工具，用于数据迁移。
title: 安装迁移工具
---

# 安装迁移工具

我们支持下载可执行二进制文件，或者从源代码编译 Milvus 迁移工具。

## 下载可执行二进制文件

1. 从 [Milvus 迁移 GitHub 仓库](https://github.com/zilliztech/milvus-migration/tags) 下载最新版本。
2. 解压下载的文件，获取 `milvus-migration` 可执行二进制文件。

## 从源代码编译

或者，下载并编译源代码以获取可执行二进制文件。

1. 克隆 Milvus 迁移仓库：

    ```bash
    # 克隆源代码项目
    git clone https://github.com/zilliztech/milvus-migration.git
    ```
    
2. 进入项目目录：

    ```bash
    cd milvus-migration
    ```
    
3. 编译项目以获取可执行文件：

    ```bash
    # 编译项目以获取可执行文件
    go get & go build
    ```
    
    这将在项目目录中生成 `milvus-migration` 可执行文件。

## 下一步

安装了 Milvus 迁移工具后，您可以从各种来源迁移数据：

- [从 Elasticsearch 迁移](es2m.md)
- [从 Faiss 迁移](f2m.md)
- [从 Milvus 1.x 迁移](m2m.md)