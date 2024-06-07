---
id: integrate_with_whyhow.md
summary: 本指南演示了如何使用 whyhow.ai 和 Milvus Lite 进行基于规则的检索。
title: 将 Milvus 与 WhyHow 集成
---

# 将 Milvus 与 WhyHow 集成

本指南演示了如何使用 whyhow.ai 和 Milvus Lite 进行基于规则的检索。

## 概述

WhyHow 是一个平台，为开发人员提供了组织、上下文化和可靠检索非结构化数据以执行复杂 RAG 所需的构建模块。Rule-based Retrieval 是由 WhyHow 开发的一个 Python 包，使人们能够创建和管理具有高级过滤功能的检索增强生成（RAG）应用程序。

## 安装

在开始之前，请安装所有必要的 Python 包以供后续使用。

```shell
pip install --upgrade pymilvus, whyhow_rbr
```

接下来，我们需要初始化 Milvus 客户端，通过使用 Milvus Lite 实现基于规则的检索。

```python
from pymilvus import MilvusClient

# Milvus Lite 本地路径
path="./milvus_demo.db" # 本地 milvus lite 数据库路径的随机名称

# 初始化 ClientMilvus
milvus_client = ClientMilvus(path)
```

您也可以通过 Milvus Cloud 初始化 Milvus 客户端

```python
from pymilvus import MilvusClient

# Milvus Cloud 凭证
YOUR_MILVUS_CLOUD_END_POINT = "YOUR_MILVUS_CLOUD_END_POINT"
YOUR_MILVUS_CLOUD_TOKEN = "YOUR_MILVUS_CLOUD_TOKEN"

# 初始化 ClientMilvus
milvus_client = ClientMilvus(
        milvus_uri=YOUR_MILVUS_CLOUD_END_POINT, 
        milvus_token=YOUR_MILVUS_CLOUD_TOKEN,
)
```

## 创建集合

### 定义必要变量

```python
# 定义集合名称
COLLECTION_NAME="YOUR_COLLECTION_NAME" # 使用您自己的集合名称

# 定义向量维度大小
DIMENSION=1536 # 根据您使用的模型决定
```

### 添加模式

在将任何数据插入 Milvus Lite 数据库之前，我们需要首先定义数据字段，这在这里称为模式。通过创建对象 `CollectionSchema` 并通过 `add_field()` 添加数据字段，我们可以控制数据类型及其特征。在将任何数据插入 Milvus 之前，此步骤是必需的。

```python
schema = milvus_client.create_schema(auto_id=True) # 启用 id 匹配

schema = milvus_client.add_field(schema=schema, field_name="id", datatype=DataType.INT64, is_primary=True)
schema = milvus_client.add_field(schema=schema, field_name="embedding", datatype=DataType.FLOAT_VECTOR, dim=DIMENSION)
```

### 创建索引

对于每个模式，最好有一个索引，以使查询更加高效。要创建索引，我们首先需要一个 `index_params`，然后在这个 `IndexParams` 对象上添加更多索引数据。

```python
# 开始对数据字段进行索引
index_params = milvus_client.prepare_index_params()
index_params = milvus_client.add_index(
    index_params=index_params,  # 传入 index_params 对象
    field_name="embedding",
    index_type="AUTOINDEX",  # 使用自动索引而不是其他复杂的索引方法
    metric_type="COSINE",  # L2、COSINE 或 IP
)
```
这种方法是对官方 Milvus 实现的一个简单封装（[官方文档](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/add_index.md)）。

### 创建集合

在定义所有数据字段并对它们进行索引后，我们现在需要创建数据库集合，以便快速准确地访问我们的数据。需要提到的是，我们将 `enable_dynamic_field` 初始化为 true，这样您就可以自由上传任何数据。代价是数据查询可能效率低下。

```python
# 创建集合
milvus_client.create_collection(
    collection_name=COLLECTION_NAME,
    schema=schema,
    index_params=index_params
)
```

## 上传文档

在创建集合后，我们准备用文档填充它。在 `whyhow_rbr` 中，可以使用 `MilvusClient` 的 `upload_documents` 方法来完成这个任务。它在幕后执行以下步骤：

- **预处理**：读取并拆分提供的 PDF 文件为块
- **嵌入**：使用 OpenAI 模型对所有块进行嵌入
- **插入**：将嵌入和元数据一起上传到 Milvus Lite

```python
# 获取 pdfs
pdfs = ["harry-potter.pdf", "game-of-thrones.pdf"] # 替换为您的 pdfs 路径

# 上传 PDF 文档
milvus_client.upload_documents(
    collection_name=COLLECTION_NAME,
    documents=pdfs
)
```

## 问答

现在我们终于可以进行检索增强生成了。

```python
# 搜索数据并实现 RAG！
res = milvus_client.search(
    question='哈利·波特喜欢吃什么食物？',
    collection_name=COLLECTION_NAME,
    anns_field='embedding',
    output_fields='text'
)
print(res['answer'])
print(res['matches'])
```

### 规则

在前面的示例中，我们考虑了索引中的每个文档。然而，有时仅检索满足一些预定义条件的文档可能更有益（例如 `filename=harry-potter.pdf`）。在 `whyhow_rbr` 通过 Milvus Lite，可以通过调整搜索参数来实现这一点。

规则可以控制以下元数据属性

- `filename` 文件名
- `page_numbers` 对应页码的整数列表（从 0 开始索引）
- `id` 块的唯一标识符（这是最“极端”的过滤器）
- 其他规则基于[布尔表达式](https://milvus.io/docs/boolean.md)
```python
# 规则（在《哈利·波特》这本书的第8页上搜索）：
PARTITION_NAME='harry-potter' # 在书籍中搜索
page_number='page_number == 8'

# 首先创建一个分区来存储这本书，稍后在这个特定分区上进行搜索：
milvus_client.crate_partition(
    collection_name=COLLECTION_NAME,
    partition_name=PARTITION_NAME # 根据您的 PDF 类型进行分隔
)

# 使用规则进行搜索
res = milvus_client.search(
    question='告诉我贪婪算法的相关内容',
    collection_name=COLLECTION_NAME,
    partition_names=PARTITION_NAME,
    filter=page_number, # 根据布尔表达式规则添加任何规则
    anns_field='embedding',
    output_fields='text'
)
print(res['answer'])
print(res['matches'])
```
在这个例子中，我们首先创建一个分区，用来存储与哈利波特相关的 PDF 文件，通过在这个分区内进行搜索，我们可以获取最直接的信息。此外，我们可以应用页面编号作为过滤器，以指定我们希望搜索的确切页面。请记住，过滤器参数需要遵循[布尔规则](https://milvus.io/docs/boolean.md)。

### 清理

最后，在执行所有指令之后，您可以通过调用 `drop_collection()` 来清理数据库。

```python
# 清理
milvus_client.drop_collection(
    collection_name=COLLECTION_NAME
)
```