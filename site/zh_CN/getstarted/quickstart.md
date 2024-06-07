---
id: quickstart.md
summary: Get started with Milvus.
title: Quickstart
---

# 使用 Milvus Lite 快速入门

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/quickstart/quickstart.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

向量是神经网络模型的输出数据格式，能够有效地编码信息，在知识库、语义搜索、检索增强生成（RAG）等人工智能应用中发挥关键作用。

Milvus 是一个开源向量数据库，适用于各种规模的人工智能应用，从在 Jupyter 笔记本中运行演示聊天机器人到构建为数十亿用户提供服务的 Web 规模搜索。在本指南中，我们将指导您如何在几分钟内在本地设置 Milvus，并使用 Python 客户端库生成、存储和搜索向量。

## 安装 Milvus
在本指南中，我们使用 Milvus Lite，这是一个包含在 `pymilvus` 中的 Python 库，可以嵌入到客户端应用程序中。Milvus 还支持在 [Docker](https://milvus.io/docs/install_standalone-docker.md) 和 [Kubernetes](https://milvus.io/docs/install_cluster-milvusoperator.md) 上部署以供生产使用。

在开始之前，请确保本地环境中有 Python 3.7+。安装 `pymilvus`，其中包含 Python 客户端库和 Milvus Lite：

```python
$ pip install -U pymilvus
```

<div class="alert note">

如果您正在使用 Google Colab，在启用刚安装的依赖项后，您可能需要**重新启动运行时**。

</div>

## 设置向量数据库
要创建本地 Milvus 向量数据库，只需实例化一个 `MilvusClient`，并指定一个文件名来存储所有数据，例如 "milvus_demo.db"。

```python
from pymilvus import MilvusClient

client = MilvusClient("milvus_demo.db")
```

## 创建集合
在 Milvus 中，我们需要一个集合来存储向量及其关联的元数据。您可以将其视为传统 SQL 数据库中的表。在创建集合时，您可以定义模式和索引参数来配置向量规格，如维度、索引类型和距离度量。还有一些复杂的概念可用于优化向量搜索性能。现在，让我们专注于基础知识，并尽可能使用默认设置。最少需要设置集合名称和集合的向量字段维度。

```python
client.create_collection(
    collection_name="demo_collection",
    dimension=768,  # 此演示中使用的向量有 768 维
)
```

在上述设置中，
- 主键和向量字段使用它们的默认名称（"id" 和 "vector"）。
- 度量类型（向量距离定义）设置为默认值（[余弦相似度](https://milvus.io/docs/metric.md#Cosine-Similarity)）。
- 主键字段接受整数，不会自动递增（即不使用[自动ID功能](https://milvus.io/docs/schema.md)）。
或者，您可以按照这个[说明](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md)正式定义集合的模式。

## 准备数据
在本指南中，我们使用向量来对文本执行语义搜索。我们需要通过下载嵌入模型为文本生成向量。可以通过使用`pymilvus[model]`库中的实用函数轻松完成此操作。

## 用向量表示文本
首先，安装模型库。该软件包包括诸如PyTorch之类的基本ML工具。如果您的本地环境从未安装过PyTorch，则软件包下载可能需要一些时间。

```python
$ pip install "pymilvus[model]"
```

使用默认模型生成向量嵌入。Milvus希望插入的数据以字典列表的形式组织，其中每个字典表示一个数据记录，称为实体。

```python
from pymilvus import model

# 如果连接到https://huggingface.co/失败，请取消下面路径的注释
# import os
# os.environ['HF_ENDPOINT'] = 'https://hf-mirror.com'

# 这将下载一个小的嵌入模型"paraphrase-albert-small-v2"（~50MB）。
embedding_fn = model.DefaultEmbeddingFunction()

# 要搜索的文本字符串。
docs = [
    "人工智能成立于1956年，是一个学术学科。",
    "艾伦·图灵是进行AI重要研究的第一人。",
    "图灵出生于伦敦梅达维尔，成长在英格兰南部。",
]

vectors = embedding_fn.encode_documents(docs)
# 输出向量具有768维，与我们刚刚创建的集合匹配。
print("维度:", embedding_fn.dim, vectors[0].shape)  # 维度: 768 (768,)

# 每个实体都有id、向量表示、原始文本和我们用来演示后续元数据过滤的主题标签。
data = [
    {"id": i, "vector": vectors[i], "text": docs[i], "subject": "历史"}
    for i in range(len(vectors))
]

print("数据有", len(data), "个实体，每个实体具有字段: ", data[0].keys())
print("向量维度:", len(data[0]["vector"]))
```

    维度: 768 (768,)
    数据有 3 个实体，每个实体具有字段:  dict_keys(['id', 'vector', 'text', 'subject'])
    向量维度: 768

## [或者] 使用随机向量的虚构表示
如果由于网络问题无法下载模型，作为解决方法，您可以使用随机向量来表示文本，仍然完成示例。只是请注意，由于向量是虚构的，搜索结果将不反映语义相似性。
```python
import random

# 要搜索的文本字符串。
docs = [
    "人工智能作为一门学科成立于1956年。",
    "艾伦·图灵是第一个进行大量人工智能研究的人。",
    "图灵出生在伦敦梅达维尔，成长在英格兰南部。",
]
# 使用随机向量（768维）进行虚拟表示。
vectors = [[random.uniform(-1, 1) for _ in range(768)] for _ in docs]
data = [
    {"id": i, "vector": vectors[i], "text": docs[i], "subject": "history"}
    for i in range(len(vectors))
]

print("数据包含", len(data), "个实体，每个实体具有字段：", data[0].keys())
print("向量维度：", len(data[0]["vector"]))
```
数据有3个实体，每个实体都有字段：dict_keys(['id', 'vector', 'text', 'subject'])
向量维度：768


## 插入数据
让我们将数据插入到集合中：


```python
res = client.insert(collection_name="demo_collection", data=data)

print(res)
```

    {'insert_count': 3, 'ids': [0, 1, 2], 'cost': 0}


## 语义搜索
现在我们可以通过将搜索查询文本表示为向量，然后在 Milvus 上进行向量相似度搜索来进行语义搜索。

### 向量搜索
Milvus 可以同时接受一个或多个向量搜索请求。query_vectors 变量的值是一个向量列表，其中每个向量是一个浮点数数组。


```python
query_vectors = embedding_fn.encode_queries(["谁是艾伦·图灵？"])
# 如果您没有嵌入函数，可以使用一个虚拟向量来完成演示：
# query_vectors = [ [ random.uniform(-1, 1) for _ in range(768) ] ]

res = client.search(
    collection_name="demo_collection",  # 目标集合
    data=query_vectors,  # 查询向量
    limit=2,  # 返回的实体数量
    output_fields=["text", "subject"],  # 指定要返回的字段
)

print(res)
```

    data: ["[{'id': 2, 'distance': 0.5859944820404053, 'entity': {'text': 'Born in Maida Vale, London, Turing was raised in southern England.', 'subject': 'history'}}, {'id': 1, 'distance': 0.5118255615234375, 'entity': {'text': 'Alan Turing was the first person to conduct substantial research in AI.', 'subject': 'history'}}]"] , extra_info: {'cost': 0}


输出是一个结果列表，每个结果映射到一个向量搜索查询。每个查询包含一个结果列表，其中每个结果包含实体主键、与查询向量的距离以及具有指定 `output_fields` 的实体详细信息。

## 具有元数据过滤的向量搜索
您还可以在考虑元数据值（在 Milvus 中称为“标量”字段，因为标量指非向量数据）的情况下进行向量搜索。这是通过使用指定某些条件的过滤表达式来完成的。让我们看看如何在以下示例中使用 `subject` 字段进行搜索和过滤。


```python
# 在另一个主题中插入更多文档。
docs = [
    "机器学习已被用于药物设计。",
    "具有 AI 算法的计算合成预测分子性质。",
    "DDR1 参与癌症和纤维化。",
]
vectors = embedding_fn.encode_documents(docs)
data = [
    {"id": 3 + i, "vector": vectors[i], "text": docs[i], "subject": "biology"}
    for i in range(len(vectors))
]

client.insert(collection_name="demo_collection", data=data)

# 这将排除任何与查询向量接近的“history”主题文本。
res = client.search(
    collection_name="demo_collection",
    data=embedding_fn.encode_queries(["告诉我与 AI 相关的信息"]),
    filter="subject == 'biology'",
    limit=2,
    output_fields=["text", "subject"],
)

print(res)
```
```python
数据: ["[{'id': 4, 'distance': 0.27030569314956665, 'entity': {'text': '使用AI算法进行计算合成可预测分子性质。', 'subject': '生物学'}}, {'id': 3, 'distance': 0.16425910592079163, 'entity': {'text': '机器学习已被用于药物设计。', 'subject': '生物学'}}]"] , 额外信息: {'cost': 0}
```

默认情况下，标量字段不被索引。如果您需要在大型数据集中执行元数据过滤搜索，可以考虑使用固定模式，并打开[index](https://milvus.io/docs/scalar_index.md)以提高搜索性能。

除了向量搜索，您还可以执行其他类型的搜索：

### 查询
查询()是一种检索与特定条件匹配的所有实体的操作，比如[过滤表达式](https://milvus.io/docs/boolean.md)或匹配一些id。

例如，检索所有标量字段具有特定值的实体：


```python
res = client.query(
    collection_name="demo_collection",
    filter="subject == 'history'",
    output_fields=["text", "subject"],
)
```

通过主键直接检索实体：


```python
res = client.query(
    collection_name="demo_collection",
    ids=[0, 2],
    output_fields=["vector", "text", "subject"],
)
```

## 删除实体
如果您想清除数据，可以删除指定主键的实体或删除匹配特定过滤表达式的所有实体。


```python
# 通过主键删除实体
res = client.delete(collection_name="demo_collection", ids=[0, 2])

print(res)

# 通过过滤表达式删除实体
res = client.delete(
    collection_name="demo_collection",
    filter="subject == 'biology'",
)

print(res)
```

    [0, 2]
    [3, 4, 5]


## 加载现有数据
由于Milvus Lite的所有数据都存储在本地文件中，您可以通过使用现有文件创建`MilvusClient`将所有数据加载到内存中，即使程序终止后也可以继续。例如，这将从"milvus_demo.db"文件中恢复集合，并继续向其中写入数据。


```python
from pymilvus import MilvusClient

client = MilvusClient("milvus_demo.db")
```

## 删除集合
如果您想删除集合中的所有数据，可以使用以下方法删除集合


```python
# 删除集合
client.drop_collection(collection_name="demo_collection")
```

了解更多
Milvus Lite非常适合开始使用本地python程序。如果您有大规模数据或希望在生产环境中使用Milvus，您可以了解如何在[Docker](https://milvus.io/docs/install_standalone-docker.md)和[Kubernetes](https://milvus.io/docs/install_cluster-milvusoperator.md)上部署Milvus。Milvus的所有部署模式共享相同的API，因此如果要切换到另一个部署模式，客户端代码几乎不需要更改。只需指定Milvus服务器的[URI和Token](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Client/MilvusClient.md)，无论部署在何处：


```python
客户端 = MilvusClient(uri="http://localhost:19530", token="root:Milvus")
```
Milvus 提供 REST 和 gRPC API，同时提供了客户端库，支持多种编程语言，如 [Python](https://milvus.io/docs/install-pymilvus.md)，[Java](https://milvus.io/docs/install-java.md)，[Go](https://milvus.io/docs/install-go.md)，C# 和 [Node.js](https://milvus.io/docs/install-node.md)。