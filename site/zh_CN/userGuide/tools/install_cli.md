---
id: install_cli.md
summary: 学习如何安装 Milvus_CLI。
title: 安装 Milvus_CLI
---

# 安装 Milvus_CLI

本主题介绍了如何安装 Milvus_CLI。


## 从 PyPI 安装

您可以从 [PyPI](https://pypi.org/project/milvus-cli/) 安装 Milvus_CLI。

### 先决条件

- 安装 [Python 3.8.5](https://www.python.org/downloads/release/python-385/) 或更新版本
- 安装 [pip](https://pip.pypa.io/en/stable/installation/)

### 通过 pip 安装

运行以下命令以安装 Milvus_CLI。

```shell
pip install milvus-cli
```

## 使用 Docker 安装

您可以使用 Docker 安装 Milvus_CLI。

### 先决条件

需要 Docker 19.03 或更新版本。

### 基于 Docker 镜像安装

```shell
$ docker run -it zilliz/milvus_cli:latest
```


## 从源代码安装

1. 运行以下命令下载 `milvus_cli` 仓库。

```shell
git clone https://github.com/zilliztech/milvus_cli.git
```

2. 运行以下命令进入 `milvus_cli` 文件夹。

```shell
cd milvus_cli
```

3. 运行以下命令安装 Milvus_CLI。

```shell
python -m pip install --editable .
```

或者，您可以从压缩的 tarball（`.tar.gz` 文件）安装 Milvus_CLI。下载 [tarball](https://github.com/zilliztech/milvus_cli/releases)，然后运行 `python -m pip install milvus_cli-<version>.tar.gz`。

### 从 .exe 文件安装

<div class="alert note"> 此安装方法仅适用于 Windows。 </div>

从 [GitHub](https://github.com/zilliztech/milvus_cli/releases) 下载 .exe 文件并运行它以安装 Milvus_CLI。
如果安装成功，`milvus_cli-<version>.exe` 将如下图所示弹出。

![Milvus_CLI](../../../../assets/milvus_cli_exe.png "Milvus_CLI 安装成功。")
