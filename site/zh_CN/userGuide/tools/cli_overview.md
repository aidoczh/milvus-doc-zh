---
id: cli_overview.md
summary: Milvus命令行界面（CLI）是一个命令行工具，支持数据库连接、数据操作以及数据的导入和导出。
title: Milvus命令行界面
---

# Milvus命令行界面

Milvus命令行界面（CLI）是一个命令行工具，支持数据库连接、数据操作以及数据的导入和导出。基于[Milvus Python SDK](https://github.com/milvus-io/pymilvus)，它允许通过终端使用交互式命令行提示执行命令。

## 推荐版本

在下表中，您可以找到根据您使用的Milvus版本推荐的PyMilvus和Milvus_CLI版本。

|  Milvus   | PyMilvus | Milvus_CLI |
| :-------: | :------: | :--------: |
|   1.0.x   |  1.0.1   |     x      |
|   1.1.x   |  1.1.2   |     x      |
| 2.0.0-RC1 | 2.0.0rc1 |     x      |
| 2.0.0-RC2 | 2.0.0rc2 |   0.1.3    |
| 2.0.0-RC4 | 2.0.0rc4 |   0.1.4    |
| 2.0.0-RC5 | 2.0.0rc5 |   0.1.5    |
| 2.0.0-RC6 | 2.0.0rc6 |   0.1.6    |
| 2.0.0-RC7 | 2.0.0rc7 |   0.1.7    |
| 2.0.0-RC8 | 2.0.0rc8 |   0.1.8    |
| 2.0.0-RC9 | 2.0.0rc9 |   0.1.9    |
|   2.1.0   |  2.1.0   |   0.3.0    |
|   2.2.x   |  2.2.x   |   0.4.0    |
|   2.3.x   |  2.3.x   |   0.4.2    |

<div class="alert note">Milvus 2.0.0-RC7及更高版本与2.0.0-RC6及更早版本不兼容，因为对存储格式进行了更改。</div>

## 当前版本

Milvus_CLI的当前版本为0.4.2。
要查找您安装的版本并查看是否需要更新，请运行`milvus_cli --version`。