---
id: cli_commands.md
summary: Interact with Milvus using commands.
title: Milvus_CLI 命令参考
---

# Milvus_CLI 命令参考

Milvus 命令行界面（CLI）是一个支持数据库连接、数据操作以及数据导入和导出的命令行工具。

本主题介绍了所有支持的命令及其相应的选项。同时还包括了一些示例供您参考。

## clear

清空屏幕。

<h3 id="clear">语法</h3>

```shell
clear
```

<h3 id="clear">选项</h3>

| 选项   | 全称     | 描述                     |
| :----- | :-------- | :----------------------- |
| --help | n/a       | 显示如何使用该命令的帮助信息。 |

## connect

连接到 Milvus。

<h3 id="connect">语法</h3>

```shell
connect [-uri (text)] [-t (text)]
```

<h3 id="connect">选项</h3>

| 选项   | 全称        | 描述                                                                                           |
| :----- | :----------- | :-------------------------------------------------------------------------------------------- |
| -uri   | --uri        | （可选）uri 名称。默认为 "http://127.0.0.1:19530"。                                                       |
| -t     | --token      | （可选）zilliz 云 apikey 或 `username:password`。默认为 None。                                                         |
| --help | n/a          | 显示如何使用该命令的帮助信息。                                                                        |

<h3 id="connect">示例</h3>

```shell
milvus_cli > connect -uri http://127.0.0.1:19530 
```

## create Database

在 Milvus 中创建数据库

<h3 id="create-database">语法</h3>

```shell
create database -db (text) 
```

### 选项

| 选项   | 全称       | 描述                                     |
| :----- | :--------- | :--------------------------------------- |
| -db    | --database | [必填] milvus 中的数据库名称。            |
| --help | n/a        | 显示如何使用该命令的帮助信息。          |

### 示例

#### 示例 1

以下示例在 milvus 中创建了数据库 <code>testdb</code>。

```shell
milvus_cli > create database -db testdb
```

## use Database

在 Milvus 中使用数据库

<h3 id="use-database">语法</h3>

```shell
use database -db (text) 
```

### 选项

| 选项   | 全称       | 描述                                     |
| :----- | :--------- | :--------------------------------------- |
| -db    | --database | [必填] milvus 中的数据库名称。            |
| --help | n/a        | 显示如何使用该命令的帮助信息。          |

### 示例

#### 示例 1

以下示例在 milvus 中使用了数据库 <code>testdb</code>。

```shell
milvus_cli > use database -db testdb
```

## List Databases

列出 Milvus 中的数据库

<h3 id="list-database">语法</h3>

```shell
list databases 
```
### 示例
#### 示例 1

以下示例列出了 Milvus 中的数据库。

```shell
milvus_cli > list databases
```

## 删除数据库

在 Milvus 中删除数据库

<h3 id="delete-database">语法</h3>

```shell
delete database -db (文本) 
```

### 选项

| 选项   | 全称       | 描述                                             |
| :----- | :--------- | :----------------------------------------------- |
| -db    | --database | [必填] Milvus 中的数据库名称。                   |
| --help | n/a        | 显示如何使用该命令的帮助信息。                 |

### 示例

#### 示例 1

以下示例删除了 Milvus 中的数据库 <code>testdb</code>。

```shell
milvus_cli > delete database -db testdb
```

## 创建用户

在 Milvus 中创建用户

<h3 id="create-user">语法</h3>

```shell
create user -u (文本) -p (文本)
```

### 选项

| 选项   | 全称       | 描述                                             |
| :----- | :--------- | :----------------------------------------------- |
| -p     | --password | Milvus 中的用户密码。默认为 "None"。             |
| -u     | --username | Milvus 中的用户名。默认为 "None"。               |
| --help | n/a        | 显示如何使用该命令的帮助信息。                 |

### 示例

#### 示例 1

以下示例在 Milvus 中创建了用户名为 <code>zilliz</code>，密码为 <code>zilliz</code> 的用户。

```shell
milvus_cli > create user -u zilliz -p zilliz
```

## 创建别名

为集合指定唯一别名。

<div class="alert note">一个集合可以有多个别名。但是，一个别名对应最多一个集合。</div>

<h3 id="create-alias">语法</h3>

```shell
create alias -c (文本) -a (文本) [-A] 
```

<h3 id="create-alias">选项</h3>

| 选项   | 全称              | 描述                                                                                                                                              |
| :----- | :---------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| -c     | --collection-name | 集合的名称。                                                                                                                                      |
| -a     | --alias-name      | 别名。                                                                                                                                            |
| -A     | --alter           | （可选）将别名转移到指定集合的标志。                                                                                                             |
| --help | n/a               | 显示如何使用该命令的帮助信息。                                                                                                                                             |

<h3 id="create-alias">示例</h3>

<h4>示例 1</h4>

以下示例为<code>car</code>集合创建了<code>carAlias1</code>和<code>carAlias2</code>的别名。

```shell
milvus_cli > create alias -c car -a carAlias1
```

<h4>示例 2</h4>

<div class="alert note">示例 2基于示例 1。</div>

以下示例将<code>carAlias1</code>的别名从<code>car</code>集合转移到<code>car2</code>集合。

```shell
milvus_cli > create alias -c car2 -A -a carAlias1
```

## 创建集合

创建一个集合。

<h3 id="create-collection">语法</h3>

```shell
create collection -c (text) -f (text) -p (text) [-a] [-d (text)]
```

<h3 id="create-collection">选项</h3>

| 选项   | 全名                  | 描述                                                                                   |
| :----- | :--------------------- | :------------------------------------------------------------------------------------- |
| -c     | --collection-name      | 集合的名称。                                                                          |
| -f     | --schema-field         | (多个) `<fieldName>:<dataType>:<dimOfVector/desc>`格式的字段模式。                     |
| -p     | --schema-primary-field | 主键字段的名称。                                                                       |
| -a     | --schema-auto-id       | (可选) 用于自动生成ID的标志。                                                         |
| -desc  | --schema-description   | (可选) 集合的描述。                                                                   |
| -level | --consistency-level    | (可选) 一致性级别：Bounded,Session,Strong, Eventual。                                |
| -d     | --is-dynamic           | (可选) 集合模式是否支持动态字段。                                                     |
| -s     | --shards-num           | (可选) 分片数量。                                                                      |
| --help | n/a                    | 显示如何使用该命令的帮助信息。                                                       |

<h3 id="create-collection">示例</h3>

```shell
## 对于数组字段: --schema-field 支持 <fieldName>:<dataType>:<maxCapacity>:<elementDataType>(如果是Varchar，则还需:<maxLength>)

milvus_cli > create collection -c car -f id:INT64:primary_field -f vector:FLOAT_VECTOR:128 -f color:INT64:color -f brand:ARRAY:64:VARCHAR:128 -p id -A -d 'car_collection'
```

## 创建分区

创建一个分区。

<h3 id="creat-partition">语法</h3>

```shell
create partition -c (text) -p (text) [-d (text)]
```

<h3 id="creat-partition">选项</h3>

| 选项   | 全名         | 描述                                  |
| :----- | :------------ | :------------------------------------ |
| -c     | --collection-name | 集合的名称。                  |
| -p     | --partition       | 分区的名称。                          |
| -d     | --description     | （可选）分区的描述。 |
| --help | n/a               | 显示如何使用该命令的帮助信息。         |

<h3 id="creat-partition">示例</h3>

```shell
milvus_cli > create partition -c car -p new_partition -d test_add_partition
```

## 创建索引

为字段创建索引。

<div class="alert note"> 目前，一个集合最多支持一个索引。</div>

<h3 id="creat-index">语法</h3>

```shell
create index
```

<h3 id="creat-index">选项</h3>

| 选项 | 全名 | 描述                          |
| :----- | :-------- | :----------------------------------- |
| --help | n/a       | 显示如何使用该命令的帮助信息。 |

<h3 id="creat-index">示例</h3>

要为字段创建索引并提示输入所需信息：

```shell
milvus_cli > create index

集合名称（car, car2）: car2

要为其创建索引的字段名称（vector）: vector

索引名称: vectorIndex

# 默认为''
索引类型（FLAT, IVF_FLAT, IVF_SQ8, IVF_PQ, RNSG, HNSW, ANNOY, AUTOINDEX, DISKANN, GPU_IVF_FLAT, GPU_IVF_PQ, SCANN, STL_SORT, Trie, ） []: IVF_FLAT  

# 默认为''
索引度量类型（L2, IP, HAMMING, TANIMOTO, COSINE, ） []: 

超时时间 []:
```

## 删除用户

删除用户。

### 语法

```shell
delete user -u (文本)
```

### 选项

| 选项 | 全名  | 描述                          |
| :----- | :-------- | :----------------------------------- |
| -u     | --username | 用户名。                        |
| --help | n/a        | 显示如何使用该命令的帮助信息。 |

### 示例

```shell
milvus_cli > delete user -u zilliz
```

## 删除别名

删除别名。

<h3 id="delete-alias">语法</h3>

```shell
delete alias -a (文本) 
```

<h3 id="delete-alias">选项</h3>

| 选项 | 全名    | 描述                                                                                                                                                                      |
| :----- | :-------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -a     | --alias-name | 别名。                                                                                                                                                                       |
| --help | n/a          | 显示如何使用该命令的帮助信息。                                                                                                                                             |
|        |

## 删除集合

删除集合。

<h3 id="delete-collection">语法</h3>

```shell
delete collection -c (文本) 
```

<h3 id="delete-collection">选项</h3>
| 选项   | 全名               | 描述                                                                                                                                                                           |
| :----- | :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -c     | --collection-name | 要删除的集合名称。                                                                                                                                                              |
| --help | n/a               | 显示如何使用该命令的帮助信息。                                                                                                                                                  |

<h3 id="delete-collection">示例</h3>

```shell
milvus_cli > delete collection -c car
```

## 删除实体

删除实体。

<h3 id="delete-entities">语法</h3>

```
delete entities -c (文本) -p (文本) 
```

<h3 id="delete-entities">选项</h3>

| 选项   | 全名               | 描述                                                                                                                                                                           |
| :----- | :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -c     | --collection-name | 实体所属的集合名称。                                                                                                                                                           |
| -p     | --partition       | （可选）要删除的分区名称。                                                                                                                                                      |
| --help | n/a               | 显示如何使用该命令的帮助信息。                                                                                                                                                  |

<h3 id="delete-entities">示例</h3>

```
milvus_cli > delete entities -c car

用于指定要删除的实体的表达式，例如 "film_id in [ 0, 1 ]": film_id in [ 0, 1 ]

您正在尝试删除集合的实体。此操作无法撤消！

是否要继续？[y/N]: y
```

## 删除分区

删除一个分区。

<h3 id="delete-partition">语法</h3>

```shell
delete partition -c (文本) -p (文本)
```

<h3 id="delete-partition">选项</h3>

| 选项   | 全名               | 描述                                                                                                                                                                           |
| :----- | :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -c     | --collection-name | 要删除分区的集合名称。                                                                                                          |
| -p     | --partition       | 要删除的分区名称。                                                                                                                                         |
| --help | n/a               | 显示如何使用该命令的帮助信息。                                                                                                                                             |

<h3 id="delete-partition">示例</h3>

```shell
milvus_cli > delete partition -c car -p new_partition
```

## 删除索引

删除索引及其对应的索引文件。

<div class="alert note">目前，一个集合最多支持一个索引。</div>

<h3 id="delete-index">语法</h3>

```shell
delete index -c (text) -in (text)
```

<h3 id="delete-index">选项</h3>

| 选项   | 完整名称         | 描述                                                                                                                                                                      |
| :----- | :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -c     | --collection-name | 集合的名称。                                                                                                                                                      |
| -in    | --index-name      | 索引的名称。                                                                                                                                                      |
| --help | n/a               | 显示如何使用该命令的帮助信息。                                                                                                                                             |

<h3 id="delete-index">示例</h3>

```shell
milvus_cli > delete index -c car -in indexName
```

## 显示集合

显示集合的详细信息。

<h3 id="show-collection">语法</h3>

```shell
show collection -c (text)
```

<h3>选项</h3>

| 选项   | 完整名称         | 描述                          |
| :----- | :---------------- | :----------------------------------- |
| -c     | --collection-name | 集合的名称。          |
| --help | n/a               | 显示如何使用该命令的帮助信息。 |

<h3>示例</h3>

```shell
milvus_cli > show collection -c test_collection_insert
```

## 显示分区

显示分区的详细信息。
## 显示分区

### 语法

```shell
show partition -c (文本) -p (文本)
```

### 选项

| 选项   | 完整名称           | 描述                                                         |
| :----- | :---------------- | :----------------------------------------------------------- |
| -c     | --collection-name | 分区所属的集合名称。                                         |
| -p     | --partition       | 分区的名称。                                                 |
| --help | n/a               | 显示如何使用该命令的帮助信息。                               |

### 示例

```shell
milvus_cli > show partition -c test_collection_insert -p _default
```

## 显示索引

显示索引的详细信息。

### 语法

```shell
show index -c (文本) -in (文本)
```

### 选项

| 选项   | 完整名称           | 描述                         |
| :----- | :---------------- | :--------------------------- |
| -c     | --collection-name | 集合的名称。                 |
| -in    | --index-name      | 索引的名称。                 |
| --help | n/a               | 显示如何使用该命令的帮助信息。 |

### 示例

```shell
milvus_cli > show index -c test_collection -in index_name
```

## 退出

关闭命令行窗口。

### 语法

```shell
exit
```

### 选项

| 选项   | 完整名称 | 描述                                 |
| :----- | :------ | :----------------------------------- |
| --help | n/a     | 显示如何使用该命令的帮助信息。       |

## 帮助

显示如何使用命令的帮助信息。

### 语法

```shell
help <command>
```

### 命令

| 命令         | 描述                                                   |
| :----------- | :----------------------------------------------------- |
| clear        | 清除屏幕。                                             |
| connect      | 连接到 Milvus。                                       |
| create       | 创建集合、数据库、分区、用户和索引。                   |
| delete       | 删除集合、数据库、分区、别名、用户或索引。             |
| exit         | 关闭命令行窗口。                                       |
| help         | 显示如何使用命令的帮助信息。                         |
| insert       | 将数据导入分区。                                       |
| list         | 列出集合、数据库、分区、用户或索引。                   |
| load         | 加载一个集合或分区。                                                                                                 |
| query        | 显示符合输入的所有条件的查询结果。                                                                                  |
| release      | 释放一个集合或分区。                                                                                                 |
| search       | 执行向量相似度搜索或混合搜索。                                                                                      |
| show         | 显示连接、数据库、集合、加载进度或索引进度。                                                                        |
| rename       | 重命名集合。                                                                                                        |
| use          | 使用数据库。                                                                                                         |
| version      | 显示 Milvus_CLI 的版本。                                                                                             |

## import

将本地或远程数据导入分区。

<h3 id="import">语法</h3>

```shell
import -c (text)[-p (text)] <file_path>
```

<h3 id="import">选项</h3>

| 选项   | 全名              | 描述                                                                                                                                                                           |
| :----- | :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -c     | --collection-name | 要插入数据的集合名称。                                                                                                                                                    |
| -p     | --partition       | （可选）要插入数据的分区名称。不传递此分区选项表示选择“\_default”分区。                                                                                                      |
| --help | n/a               | 显示如何使用该命令的帮助信息。                                                                                                                                                   |

<h3 id="import">示例 1</h3>
以下示例导入一个本地 CSV 文件。

```shell
milvus_cli > import -c car 'examples/import_csv/vectors.csv'

读取 csv 文件...  [####################################]  100%

列名为 ['vector', 'color', 'brand']

处理了 50001 行。

正在插入...

成功插入。
--------------------------  ------------------
总插入实体数:                   50000
总集合实体数:                  150000
Milvus 时间戳:           428849214449254403
--------------------------  ------------------
```

<h3 id="import">示例 2</h3>
以下示例导入一个远程 CSV 文件。
```shell
milvus_cli > import -c car 'https://raw.githubusercontent.com/milvus-io/milvus_cli/main/examples/import_csv/vectors.csv'

从远程 URL 读取文件。

读取 csv 文件...  [####################################]  100%

列名为 ['vector', 'color', 'brand']

处理了 50001 行。

正在插入...

成功插入。

--------------------------  ------------------
总插入实体数:                   50000
总集合实体数:              150000
Milvus 时间戳:           428849214449254403
--------------------------  ------------------
```
## 用户列表

列出所有用户。

### 语法

```shell
list users
```

### 选项

| 选项   | 全称     | 描述                 |
| ------ | -------- | -------------------- |
| --help | n/a      | 显示命令的帮助信息。 |

## 集合列表

列出所有集合。

<h3 id="list-collections">语法</h3>

```shell
list collections
```

<h3 id="list-collections">选项</h3>

| 选项   | 全称     | 描述                                                                                     |
| :----- | :-------- | :--------------------------------------------------------------------------------------- |
| --help | n/a      | 显示命令的帮助信息。                                                                     |

## 索引列表

列出集合的所有索引。

<div class="alert note"> 目前，一个集合最多支持一个索引。 </div>

<h3 id="list-indexes">语法</h3>

```shell
list indexes -c (text)
```

<h3 id="list-indexes">选项</h3>

| 选项   | 全称             | 描述                   |
| :----- | :---------------- | :-------------------- |
| -c     | --collection-name | 集合的名称。          |
| --help | n/a               | 显示命令的帮助信息。   |

## 分区列表

列出集合的所有分区。

<h3 id="list-partitions">语法</h3>

```shell
list partitions -c (text)
```

<h3 id="list-partitions">选项</h3>

| 选项   | 全称             | 描述                   |
| :----- | :---------------- | :-------------------- |
| -c     | --collection-name | 集合的名称。          |
| --help | n/a               | 显示命令的帮助信息。   |

## 载入

将集合或分区从硬盘加载到内存中。

<h3 id="load">语法</h3>

```shell
load -c (text) [-p (text)]
```

<h3 id="load">选项</h3>

| 选项   | 全称             | 描述                                       |
| :----- | :---------------- | :---------------------------------------- |
| -c     | --collection-name | 分区所属的集合的名称。                    |
| -p     | --partition       | （可选/多个）分区的名称。                  |
| --help | n/a               | 显示命令的帮助信息。                       |

## 查询

显示符合您输入的所有条件的查询结果。

<h3 id="query">语法</h3>

```shell
query
```

<h3 id="query">选项</h3>

| 选项   | 全称     | 描述                 |
| :----- | :-------- | :-------------------- |
| --help | n/a      | 显示命令的帮助信息。 |

<h3 id="query">示例</h3>
<h4 id="query">示例 1</h4>
执行查询并提示输入所需内容：

```shell
milvus_cli > query

集合名称: car

查询表达式: id in [ 428960801420883491, 428960801420883492,
428960801420883493 ]

包含实体的分区名称(如果有多个，请用逗号分隔) []:
default

要返回的字段列表(如果有多个，请用逗号分隔) []: color, brand

超时时间 []:

保证时间戳。这指示 Milvus 查看在提供的时间戳之前执行的所有操作。如果没有提供此类时间戳，则 Milvus 将搜索迄今为止执行的所有操作。 [0]:
优雅时间。仅在有界一致性级别中使用。如果设置了优雅时间，PyMilvus 将使用当前时间戳减去优雅时间作为保证时间戳。如果未设置，默认情况下此选项为 5 秒。 [5]:
```

<h4 id="query">示例 2</h4>

执行查询并提示输入所需内容：

```shell
milvus_cli > query

集合名称: car

查询表达式: id > 428960801420883491

包含实体的分区名称(如果有多个，请用逗号分隔) []:
default

要返回的字段列表(如果有多个，请用逗号分隔) []: id, color, brand

超时时间 []:

保证时间戳。这指示 Milvus 查看在提供的时间戳之前执行的所有操作。如果没有提供此类时间戳，则 Milvus 将搜索迄今为止执行的所有操作。 [0]:
优雅时间。仅在有界一致性级别中使用。如果设置了优雅时间，PyMilvus 将使用当前时间戳减去优雅时间作为保证时间戳。如果未设置，默认情况下此选项为 5 秒。 [5]:
```

## 发布

将集合或分区从 RAM 中释放。

<h3 id="release">语法</h3>

```shell
release -c (文本) [-p (文本)]
```

<h3 id="release">选项</h3>

| 选项 | 全名             | 描述                                               |
| :--- | :--------------- | :-------------------------------------------------- |
| -c   | --collection-name | 分区所属集合的名称。                                |
| -p   | --partition       | (可选/多个) 分区的名称。                            |
| --help | n/a             | 显示如何使用该命令的帮助信息。                      |

## 搜索

执行向量相似性搜索或混合搜索。

<h3 id="search">语法</h3>

```shell
search
```

<h3 id="search">选项</h3>

| 选项 | 全名 | 描述                          |
| :--- | :--- | :--------------------------- |
| --help | n/a | 显示如何使用该命令的帮助信息。 |

<h3 id="search">示例</h3>
<h4 id="search">示例 1</h4>

执行对 csv 文件的搜索并提示输入所需内容：
```shell
milvus_cli > search

请输入集合名称 (car, test_collection): car

搜索数据向量（数据长度为查询数 (nq)，每个向量的维度必须与集合的向量字段相等。您也可以导入一个带有标题行的 CSV 文件）: examples/import_csv/search_vectors.csv

用于搜索的向量字段 (vector): vector

距离度量类型: L2

搜索参数 nprobe 的值: 10

返回的最大记录数，也称为 topk: 2

用于过滤属性的布尔表达式 []: id > 0

要搜索的分区名称（如果有多个，请用逗号分隔） ['_default'] []: _default

超时时间 []:

时间戳保证（它指示 Milvus 查看提供的时间戳之前执行的所有操作。如果没有提供此类时间戳，则 Milvus 将搜索迄今为止执行的所有操作） [0]:
```

### 示例2

要在已索引集合上执行搜索并提示输入所需内容：

```shell
milvus_cli > search

集合名称 (car, test_collection): car

搜索数据的向量（数据长度为查询数（nq），数据中每个向量的维度必须等于集合的向量字段。您也可以导入一个没有标题的 csv 文件）:
    [[0.71, 0.76, 0.17, 0.13, 0.42, 0.07, 0.15, 0.67, 0.58, 0.02, 0.39, 0.47, 0.58, 0.88, 0.73, 0.31, 0.23, 0.57, 0.33, 0.2, 0.03, 0.43, 0.78, 0.49, 0.17, 0.56, 0.76, 0.54, 0.45, 0.46, 0.05, 0.1, 0.43, 0.63, 0.29, 0.44, 0.65, 0.01, 0.35, 0.46, 0.66, 0.7, 0.88, 0.07, 0.49, 0.92, 0.57, 0.5, 0.16, 0.77, 0.98, 0.1, 0.44, 0.88, 0.82, 0.16, 0.67, 0.63, 0.57, 0.55, 0.95, 0.13, 0.64, 0.43, 0.71, 0.81, 0.43, 0.65, 0.76, 0.7, 0.05, 0.24, 0.03, 0.9, 0.46, 0.28, 0.92, 0.25, 0.97, 0.79, 0.73, 0.97, 0.49, 0.28, 0.64, 0.19, 0.23, 0.51, 0.09, 0.1, 0.53, 0.03, 0.23, 0.94, 0.87, 0.14, 0.42, 0.82, 0.91, 0.11, 0.91, 0.37, 0.26, 0.6, 0.89, 0.6, 0.32, 0.11, 0.98, 0.67, 0.12, 0.66, 0.47, 0.02, 0.15, 0.6, 0.64, 0.57, 0.14, 0.81, 0.75, 0.11, 0.49, 0.78, 0.16, 0.63, 0.57, 0.18]]

用于搜索集合的向量字段 (vector): vector

度量类型: L2

搜索参数 nprobe 的值: 10

返回距离的小数位数 [-1]: 5

返回记录的最大数量，也称为 topk: 2

用于过滤属性的布尔表达式 []: id > 0

要搜索的分区名称（如果有多个，请用“,”分隔） ['_default'] []: _default

超时 []:

保证时间戳（它指示 Milvus 查看在提供的时间戳之前执行的所有操作。如果没有提供这样的时间戳，则 Milvus 将搜索到目前为止执行的所有操作） [0]:
```

### 示例3

要在未索引集合上执行搜索并提示输入所需内容：

```shell
milvus_cli > search

集合名称 (car, car2): car

搜索数据的向量（数据长度为查询数（nq），数据中每个向量的维度必须等于集合的向量字段）: examples/import_csv/search_vectors.csv

用于搜索集合的向量字段 (vector): vector

返回距离的小数位数 [-1]: 5

返回记录的最大数量，也称为 topk: 2

用于过滤属性的布尔表达式 []:

要搜索的分区名称（如果有多个，请用“,”分隔） ['_default'] []:

超时 []:

保证时间戳（它指示 Milvus 查看在提供的时间戳之前执行的所有操作。如果没有提供这样的时间戳，则 Milvus 将搜索到目前为止执行的所有操作） [0]:
```

## 列出连接

列出连接。

### 语法

```shell
list connections 
```

### 选项

| 选项 | 全名 | 描述 |
| :----- | :-------- | :---------------------------------------- |
| --help | n/a       | 显示如何使用该命令的帮助信息。      |

## 显示索引进度

显示实体索引的进度。

<h3 id="show-index-progress">语法</h3>

```shell
show index_progress -c (text) [-i (text)]
```

<h3 id="show-index-progress">选项</h3>

| 选项 | 完整名称         | 描述                                             |
| :----- | :---------------- | :------------------------------------------------------ |
| -c     | --collection-name | 实体所属的集合名称。 |
| -i     | --index           | （可选）索引的名称。                       |
| --help | n/a               | 显示如何使用该命令的帮助信息。                    |

## 显示加载进度

显示实体加载的进度。

<h3 id="show-loading-progress">语法</h3>

```shell
show loading_progress -c (text) [-p (text)]
```
<h3 id="show-loading-progress">选项</h3>

|选项|完整名称|描述
|:---|:---|:---|
|-c|--collection-name|实体所属的集合名称。|
|-p|--partition|（可选/多个）加载分区的名称。|
|--help|n/a|显示如何使用该命令的帮助信息。|


## 版本

显示 Milvus_CLI 的版本。

<h3 id="version">语法</h3>

```shell
version
```
<h3 id="version">选项</h3>

|选项|完整名称|描述
|:---|:---|:---|
|--help|n/a|显示如何使用该命令的帮助信息。|

<div class="alert note"> 您也可以在 shell 中检查 Milvus_CLI 的版本，如下例所示。在这种情况下，<code>milvus_cli --version</code> 充当一个命令。</div>

<h3 id="version">示例</h3>

```shell
$ milvus_cli --version
Milvus_CLI v0.4.0
```