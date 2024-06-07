---
id: integrate_with_llamaindex.md
summary: 本指南演示如何使用 LlamaIndex 和 Milvus 构建检索增强生成（RAG）系统。
title: 使用 Milvus 和 LlamaIndex 进行检索增强生成（RAG）
---

# 使用 Milvus 和 LlamaIndex 进行检索增强生成（RAG）

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/rag_with_milvus_and_llamaindex.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

本指南演示了如何使用 LlamaIndex 和 Milvus 构建检索增强生成（RAG）系统。

RAG 系统将检索系统与生成模型结合起来，根据给定提示生成新文本。系统首先使用类似 Milvus 的向量相似度搜索引擎从语料库中检索相关文档，然后使用生成模型根据检索到的文档生成新文本。

[LlamaIndex](https://www.llamaindex.ai/) 是一个简单灵活的数据框架，用于将自定义数据源连接到大型语言模型（LLMs）。[Milvus](https://milvus.io/) 是全球最先进的开源向量数据库，旨在支持嵌入相似度搜索和人工智能应用。

在这个笔记本中，我们将展示如何使用 MilvusVectorStore 进行快速演示。

## 开始之前

### 安装依赖项
本页面上的代码片段需要 pymilvus 和 llamaindex 依赖项。您可以使用以下命令安装它们：

```python
$ pip install pymilvus>=2.4.2
```

```python
$ pip install llama-index-vector-stores-milvus
```

```python
$ pip install llama-index
```

<div class="alert note">

如果您使用 Google Colab，在启用刚安装的依赖项后，您可能需要**重新启动运行时**。

</div>

### 设置 OpenAI

让我们首先添加 openai api 密钥。这将允许我们访问 chatgpt。

```python
import openai

openai.api_key = "sk-***********"
```

### 准备数据

您可以使用以下命令下载示例数据：

```python
$ mkdir -p 'data/paul_graham/'
$ wget 'https://raw.githubusercontent.com/run-llama/llama_index/main/docs/docs/examples/data/paul_graham/paul_graham_essay.txt' -O 'data/paul_graham/paul_graham_essay.txt'
```

## 入门指南

### 生成我们的数据
作为第一个示例，让我们从`data/paul_graham/`文件夹中的文件生成一个文档。在这个文件夹中，有一篇来自 Paul Graham 的单篇文章，标题为`What I Worked On`。我们将使用 SimpleDirectoryReader 来生成文档。

```python
from llama_index.core import SimpleDirectoryReader

# 加载文档
documents = SimpleDirectoryReader("./data/paul_graham/").load_data()

print("文档 ID:", documents[0].doc_id)
```

    文档 ID: 11c3a6fe-799e-4e40-8122-2339936c2722

### 在数据上创建索引
现在我们已经有了一个文档，我们可以创建一个索引并插入这个文档。对于索引，我们将使用一个 GPTMilvusIndex。GPTMilvusIndex 接受一些参数：

- `uri (str, optional)`: 连接的 URI，如果使用 Milvus 或 Zilliz Cloud 服务，则形式为 "https://address:port"，如果使用本地的轻量级 Milvus，则为 "path/to/local/milvus.db"。默认为 "./milvus_llamaindex.db"。
- `token (str, optional)`: 登录的令牌。如果不使用 rbac，则为空，如果使用 rbac，则可能是 "username:password"。默认为 ""。
- `collection_name (str, optional)`: 数据将被存储的集合的名称。默认为 "llamalection"。
- `dim (int, optional)`: 嵌入的维度。如果未提供，将在第一次插入时创建集合。默认为 None。
- `embedding_field (str, optional)`: 集合中嵌入字段的名称，默认为 DEFAULT_EMBEDDING_KEY。
- `doc_id_field (str, optional)`: 集合中文档 ID 字段的名称，默认为 DEFAULT_DOC_ID_KEY。
- `similarity_metric (str, optional)`: 要使用的相似度度量，目前支持 IP 和 L2。默认为 "IP"。
- `consistency_level (str, optional)`: 为新创建的集合使用的一致性级别。默认为 "Strong"。
- `overwrite (bool, optional)`: 是否覆盖同名的现有集合。默认为 False。
- `text_key (str, optional)`: 在传递的集合中存储文本的键。在使用自己的集合时使用。默认为 None。
- `index_config (dict, optional)`: 用于构建 Milvus 索引的配置。默认为 None。
- `search_config (dict, optional)`: 用于搜索 Milvus 索引的配置。请注意，这必须与 index_config 指定的索引类型兼容。默认为 None。

<div class="alert note">

请注意，**Milvus Lite** 需要 `pymilvus>=2.4.2`。

</div>


```python
# 在文档上创建一个索引
from llama_index.core import VectorStoreIndex, StorageContext
from llama_index.vector_stores.milvus import MilvusVectorStore


vector_store = MilvusVectorStore(uri="./milvus_demo.db", dim=1536, overwrite=True)
storage_context = StorageContext.from_defaults(vector_store=vector_store)
index = VectorStoreIndex.from_documents(documents, storage_context=storage_context)
```

### 查询数据
现在我们的文档存储在索引中，我们可以针对索引提出问题。索引将使用其自身存储的数据作为 ChatGPT 的知识库。

```python
import textwrap


query_engine = index.as_query_engine()
response = query_engine.query("作者学到了什么？")
print(textwrap.fill(str(response), 100))
```

    作者学到了关于在像 IBM 1401 这样的早期计算机上使用 Fortran 进行编程，早期计算技术的限制，向微型计算机的过渡以及对早期计算技术的兴奋。
拥有像 TRS-80 这样的个人电脑。此外，作者探索了不同的学术路径，最初计划学习哲学，但最终由于对哲学课程缺乏兴趣而转向人工智能。后来，作者追求艺术教育，就读于 RISD 和佛罗伦萨的 Accademia di Belli Arti，在那里他们接触到了不同的艺术教学方法。

```python
response = query_engine.query("What was a hard moment for the author?")
print(textwrap.fill(str(response), 100))
```

处理与管理 Hacker News 相关的压力和挑战是作者的一个困难时刻。

这个下一个测试显示覆盖会删除先前的数据。

```python
from llama_index.core import Document

vector_store = MilvusVectorStore(uri="./milvus_demo.db", dim=1536, overwrite=True)
storage_context = StorageContext.from_defaults(vector_store=vector_store)
index = VectorStoreIndex.from_documents(
    [Document(text="The number that is being searched for is ten.")],
    storage_context,
)
query_engine = index.as_query_engine()
res = query_engine.query("Who is the author?")
print("Res:", res)
```

Res: 作者是创建所讨论内容或作品的个人。

下一个测试显示向已存在的索引添加额外数据。

```python
del index, vector_store, storage_context, query_engine

vector_store = MilvusVectorStore(uri="./milvus_demo.db", overwrite=False)
storage_context = StorageContext.from_defaults(vector_store=vector_store)
index = VectorStoreIndex.from_documents(documents, storage_context=storage_context)
query_engine = index.as_query_engine()
res = query_engine.query("What is the number?")
print("Res:", res)
```

Res: 数字是十。

```python
res = query_engine.query("Who is the author?")
print("Res:", res)
```

Res: Paul Graham