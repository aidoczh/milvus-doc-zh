---
id: integrate_with_haystack.md
summary: 本指南演示如何使用 Haystack 和 Milvus 构建一个检索增强生成（RAG）系统。
title: 使用 Milvus 和 Haystack 进行检索增强生成（RAG）
---

# 使用 Milvus 和 Haystack 进行检索增强生成（RAG）

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/rag_with_milvus_and_haystack.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

本指南演示了如何使用 Haystack 和 Milvus 构建一个检索增强生成（RAG）系统。

RAG 系统将检索系统与生成模型结合起来，根据给定的提示生成新文本。该系统首先使用类似 Milvus 的向量相似度搜索引擎从语料库中检索相关文档，然后使用生成模型根据检索到的文档生成新文本。

[Haystack](https://haystack.deepset.ai/) 是由 deepset 开发的用于构建具有大型语言模型（LLMs）的自定义应用程序的开源 Python 框架。[Milvus](https://milvus.io/) 是世界上最先进的开源向量数据库，旨在支持嵌入相似度搜索和人工智能应用程序。

## 先决条件

在运行此笔记本之前，请确保已安装以下依赖项：

```python
$ pip install --upgrade --quiet pymilvus milvus-haystack markdown-it-py mdit_plain
```

<div class="alert note">

如果您正在使用 Google Colab，在启用刚安装的依赖项后，您可能需要**重新启动运行时**。

</div>

我们将使用 OpenAI 的模型。您应准备好[api key](https://platform.openai.com/docs/quickstart) `OPENAI_API_KEY` 作为环境变量。

```python
import os

os.environ["OPENAI_API_KEY"] = "sk-***********"
```

## 准备数据

我们使用关于[列奥纳多·达·芬奇](https://www.gutenberg.org/cache/epub/7785/pg7785.txt)的在线内容作为我们的 RAG 流水线的私有知识存储，这是一个简单 RAG 流水线的良好数据源。

下载并将其保存为本地文本文件。

```python
import os
import urllib.request

url = "https://www.gutenberg.org/cache/epub/7785/pg7785.txt"
file_path = "./davinci.txt"

if not os.path.exists(file_path):
    urllib.request.urlretrieve(url, file_path)
```

## 创建索引流水线

创建一个索引流水线，将文本转换为文档，将其分割为句子，并对其进行嵌入。然后将这些文档写入 Milvus 文档存储。
```python
from haystack import Pipeline
from haystack.components.converters import MarkdownToDocument
from haystack.components.embedders import OpenAIDocumentEmbedder, OpenAITextEmbedder
from haystack.components.preprocessors import DocumentSplitter
from haystack.components.writers import DocumentWriter

from milvus_haystack import MilvusDocumentStore
from milvus_haystack.milvus_embedding_retriever import MilvusEmbeddingRetriever

document_store = MilvusDocumentStore(
    connection_args={"uri": "./milvus.db"},
    drop_old=True,
)
indexing_pipeline = Pipeline()
indexing_pipeline.add_component("converter", MarkdownToDocument())
indexing_pipeline.add_component(
    "splitter", DocumentSplitter(split_by="sentence", split_length=2)
)
indexing_pipeline.add_component("embedder", OpenAIDocumentEmbedder())
indexing_pipeline.add_component("writer", DocumentWriter(document_store))
indexing_pipeline.connect("converter", "splitter")
indexing_pipeline.connect("splitter", "embedder")
indexing_pipeline.connect("embedder", "writer")
indexing_pipeline.run({"converter": {"sources": [file_path]}})

print("文档数量:", document_store.count_documents())
```
```
    将 markdown 文件转换为文档: 100%|█| 1/
    计算嵌入: 100%|█| 9/9 [00:05<00:00, 
    E20240516 10:40:32.945937 5309095 milvus_local.cpp:189] [SERVER][GetCollection][] 集合 HaystackCollection 不存在
    E20240516 10:40:32.946677 5309095 milvus_local.cpp:189] [SERVER][GetCollection][] 集合 HaystackCollection 不存在
    E20240516 10:40:32.946704 5309095 milvus_local.cpp:189] [SERVER][GetCollection][] 集合 HaystackCollection 不存在
    E20240516 10:40:32.946725 5309095 milvus_local.cpp:189] [SERVER][GetCollection][] 集合 HaystackCollection 不存在


    文档数量: 277
```

## 创建检索流水线

创建一个检索流水线，使用向量相似度搜索引擎从 Milvus 文档存储库中检索文档。


```python
question = '绘画《战士》目前存放在哪里?'

retrieval_pipeline = Pipeline()
retrieval_pipeline.add_component("embedder", OpenAITextEmbedder())
retrieval_pipeline.add_component(
    "retriever", MilvusEmbeddingRetriever(document_store=document_store, top_k=3)
)
retrieval_pipeline.connect("embedder", "retriever")

retrieval_results = retrieval_pipeline.run({"embedder": {"text": question}})

for doc in retrieval_results["retriever"]["documents"]:
    print(doc.content)
    print("-" * 10)
```

    ). 这幅油画的构图似乎是在他早先制作的第二幅素描的基础上完成的，而那幅素描早在八年前就已经完成，显然于1516年被带到法国，最终消失了。
    ----------
    
    这幅《洗礼基督》现在收藏在佛罗伦萨的学院美术馆，保存状况不佳，似乎是维罗基奥的一部较早期作品，创作于1480-1482年，当时莱昂纳多大约三十岁。
    
    这幅“战士”的精美素描，现在收藏在大英博物馆的马尔科姆收藏中，创作年代大约也是在这个时期。
    ----------
    "尽管他完成了素描，但最终他用彩色绘制的部分只有前景中描述的“标准之战”事件。这幅壁画的研究之一的众多所谓复制品现在挂在维多利亚和阿尔伯特博物馆的东南楼梯上。
    ----------


## 创建 RAG 流水线

创建一个 RAG 流水线，结合 MilvusEmbeddingRetriever 和 OpenAIGenerator，使用检索到的文档回答问题。

```python
from haystack.utils import Secret
from haystack.components.builders import PromptBuilder
from haystack.components.generators import OpenAIGenerator

prompt_template = """根据提供的上下文回答以下问题。如果上下文中没有答案，请回复“我不知道”。\n
                     查询: {{query}}
                     文档:
                     {% for doc in documents %}
                        {{ doc.content }}
                     {% endfor %}
                     答案:
                  """

rag_pipeline = Pipeline()
rag_pipeline.add_component("text_embedder", OpenAITextEmbedder())
rag_pipeline.add_component(
    "retriever", MilvusEmbeddingRetriever(document_store=document_store, top_k=3)
)
rag_pipeline.add_component("prompt_builder", PromptBuilder(template=prompt_template))
rag_pipeline.add_component(
    "generator",
    OpenAIGenerator(
        api_key=Secret.from_token(os.getenv("OPENAI_API_KEY")),
        generation_kwargs={"temperature": 0},
    ),
)
rag_pipeline.connect("text_embedder.embedding", "retriever.query_embedding")
rag_pipeline.connect("retriever.documents", "prompt_builder.documents")
rag_pipeline.connect("prompt_builder", "generator")

results = rag_pipeline.run(
    {
        "text_embedder": {"text": question},
        "prompt_builder": {"query": question},
    }
)
print("RAG 答案:", results["generator"]["replies"][0])
```
    RAG 答案：绘画《战士》目前存放在大英博物馆的马尔科姆收藏中。