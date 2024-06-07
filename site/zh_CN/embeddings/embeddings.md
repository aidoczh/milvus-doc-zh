---
id: embeddings.md
order: 1
summary: 学习如何为您的数据生成嵌入向量。
title: 概述
---

# 概述

嵌入（Embedding）是一个机器学习概念，用于将数据映射到高维空间，其中具有相似语义的数据被放置在一起。通常是来自 BERT 或其他 Transformer 家族的深度神经网络，嵌入模型可以有效地用一系列称为向量的数字表示文本、图像和其他数据类型的语义。这些模型的一个关键特征是高维空间中向量之间的数学距离可以指示原始文本或图像的语义相似性。这一特性解锁了许多信息检索应用，例如谷歌和必应等网络搜索引擎，电子商务网站上的产品搜索和推荐，以及最近流行的生成式人工智能中的检索增强生成（RAG）范式。

嵌入有两个主要类别，每个类别产生不同类型的向量：

- __密集嵌入（Dense embedding）__：大多数嵌入模型将信息表示为数百到数千维的浮点向量。输出被称为“密集”向量，因为大多数维度具有非零值。例如，流行的开源嵌入模型 BAAI/bge-base-en-v1.5 输出 768 个浮点数的向量（768 维浮点向量）。

- __稀疏嵌入（Sparse embedding）__：相反，稀疏嵌入的输出向量大多数维度为零，即“稀疏”向量。这些向量通常具有更高的维度（数万甚至更多），这取决于标记词汇的大小。稀疏向量可以通过深度神经网络或对文本语料库进行的统计分析生成。由于它们的可解释性和观察到的更好的跨领域泛化能力，开发人员越来越多地采用稀疏嵌入作为密集嵌入的补充。

Milvus 是一个专为向量数据管理、存储和检索而设计的向量数据库。通过集成主流的嵌入和[重新排序](https://milvus.io/docs/rerankers-overview.md)模型，您可以轻松地将原始文本转换为可搜索的向量，或使用强大的模型重新排序结果，以实现更准确的 RAG 结果。这种集成简化了文本转换，并消除了额外嵌入或重新排序组件的需求，从而简化了 RAG 的开发和验证过程。

要查看嵌入的实际操作，请参考[使用 PyMilvus 模型生成文本嵌入](https://github.com/milvus-io/bootcamp/blob/master/bootcamp/model/embedding_functions.ipynb)。

|  嵌入函数                                                                   |  类型   |  API 或开源 |
| ------------------------------------------------------------------------------------- | ------- | -------------------- |
|  [openai](https://milvus.io/api-reference/pymilvus/v2.4.x/EmbeddingModels/OpenAIEmbeddingFunction/OpenAIEmbeddingFunction.md)                            |  Dense  |  API                 |
|  [sentence-transformer](https://milvus.io/api-reference/pymilvus/v2.4.x/EmbeddingModels/SentenceTransformerEmbeddingFunction/SentenceTransformerEmbeddingFunction.md) |  Dense  |  开源        |
|  [bm25](https://milvus.io/api-reference/pymilvus/v2.4.x/EmbeddingModels/BM25EmbeddingFunction/BM25EmbeddingFunction.md)                                |  稀疏 |  开源        |
|  [Splade](https://milvus.io/api-reference/pymilvus/v2.4.x/EmbeddingModels/SpladeEmbeddingFunction/SpladeEmbeddingFunction.md)                            |  稀疏 |  开源        |
|  [bge-m3](https://milvus.io/api-reference/pymilvus/v2.4.x/EmbeddingModels/BGEM3EmbeddingFunction/BGEM3EmbeddingFunction.md)                             |  混合 |  开源        |
|  [voyageai](https://milvus.io/api-reference/pymilvus/v2.4.x/EmbeddingModels/VoyageEmbeddingFunction/VoyageEmbeddingFunction.md)                            |  Dense  |  API                 |

## 示例 1：使用默认嵌入函数生成密集向量

要在 Milvus 中使用嵌入函数，首先安装 PyMilvus 客户端库，其中包含 `model` 子包，该子包包装了用于生成嵌入的所有实用工具。

```python
pip install pymilvus[model]
# 或者对于 zsh，请使用 pip install "pymilvus[model]"。
```

`model` 子包支持各种嵌入模型，从[OpenAI](https://milvus.io/docs/embed-with-openai.md)、[Sentence Transformers](https://milvus.io/docs/embed-with-sentence-transform.md)、[BGE M3](https://milvus.io/docs/embed-with-bgm-m3.md)、[BM25](https://milvus.io/docs/embed-with-bm25.md)到[SPLADE](https://milvus.io/docs/embed-with-splade.md)预训练模型。为简单起见，此示例使用 `DefaultEmbeddingFunction`，它是 __all-MiniLM-L6-v2__ 句子转换模型，该模型约为 70MB，在首次使用时将被下载：

```python
from pymilvus import model

# 这将下载 "all-MiniLM-L6-v2"，一个轻量级模型。
ef = model.DefaultEmbeddingFunction()

# 要生成嵌入的数据
docs = [
    "人工智能创立于1956年作为一门学科。",
    "艾伦·图灵是第一个进行大量人工智能研究的人。",
    "图灵出生于伦敦梅达维尔，在英格兰南部长大。",
]

embeddings = ef.encode_documents(docs)

# 打印嵌入
print("嵌入:", embeddings)
# 打印嵌入的维度和形状
print("维度:", ef.dim, embeddings[0].shape)
```

预期输出类似于以下内容：
```python
嵌入向量: [array([-3.09392996e-02, -1.80662833e-02,  1.34775648e-02,  2.77156215e-02,
       -4.86349640e-03, -3.12581174e-02, -3.55921760e-02,  5.76934684e-03,
        2.80773244e-03,  1.35783911e-01,  3.59678417e-02,  6.17732145e-02,
...
       -4.61330153e-02, -4.85207550e-02,  3.13997865e-02,  7.82178566e-02,
       -4.75336798e-02,  5.21207601e-02,  9.04406682e-02, -5.36676683e-02],
      dtype=float32)]
维度: 384 (384,)
```
## 示例 2：使用 BGE M3 模型一次生成密集向量和稀疏向量

在这个示例中，我们使用 [BGE M3](https://milvus.io/docs/embed-with-bgm-m3.md) 混合模型将文本嵌入到密集向量和稀疏向量中，并使用它们来检索相关文档。总体步骤如下：

1. 使用 BGE-M3 模型将文本嵌入为密集向量和稀疏向量；

2. 设置一个 Milvus 集合来存储密集向量和稀疏向量；

3. 将数据插入到 Milvus 中；

4. 搜索并检查结果。

首先，我们需要安装必要的依赖项。

```python
from pymilvus.model.hybrid import BGEM3EmbeddingFunction
from pymilvus import (
    utility,
    FieldSchema, CollectionSchema, DataType,
    Collection, AnnSearchRequest, RRFRanker, connections,
)
```

使用 BGE M3 对文档和查询进行编码以进行嵌入检索。

```python
# 1. 准备一个小语料库进行搜索
docs = [
    "人工智能是一门学科，成立于1956年。",
    "艾伦·图灵是第一个进行大量人工智能研究的人。",
    "图灵出生于伦敦梅达维尔，成长在英格兰南部。",
]
query = "谁开始了人工智能研究？"

# BGE-M3 模型可以将文本嵌入为密集向量和稀疏向量。
# 它包含在 pymilvus 的可选 `model` 模块中，要安装它，
# 只需运行 "pip install pymilvus[model]".

bge_m3_ef = BGEM3EmbeddingFunction(use_fp16=False, device="cpu")

docs_embeddings = bge_m3_ef(docs)
query_embeddings = bge_m3_ef([query])
```

## 示例 3：使用 BM25 模型生成稀疏向量

BM25 是一种使用词出现频率来确定查询和文档之间相关性的著名方法。在这个示例中，我们将展示如何使用 `BM25EmbeddingFunction` 为查询和文档生成稀疏嵌入。

首先，导入 __BM25EmbeddingFunction__ 类。

```xml
from pymilvus.model.sparse import BM25EmbeddingFunction
```

在 BM25 中，计算文档中的统计数据以获得 IDF（逆文档频率）是很重要的，它可以代表文档中的模式。IDF 是一个衡量词提供信息量的指标，即它在所有文档中是常见的还是罕见的。
```python
# 1. 准备一个小语料库进行搜索
docs = [
    "人工智能作为一门学科于1956年建立。",
    "艾伦·图灵是第一个进行人工智能实质性研究的人。",
    "图灵出生在伦敦梅达维尔，成长在英格兰南部。",
]
query = "图灵出生在哪里？"
bm25_ef = BM25EmbeddingFunction()

# 2. 将语料库拟合以获得文档上的BM25模型参数。
bm25_ef.fit(docs)

# 3. 将拟合的参数存储到磁盘以加快未来处理速度。
bm25_ef.save("bm25_params.json")

# 4. 加载保存的参数
new_bm25_ef = BM25EmbeddingFunction()
new_bm25_ef.load("bm25_params.json")

docs_embeddings = new_bm25_ef.encode_documents(docs)
query_embeddings = new_bm25_ef.encode_queries([query])
print("Dim:", new_bm25_ef.dim, list(docs_embeddings)[0].shape)
```
期望的输出类似于以下内容：

```python
Dim: 21 (1, 21)
```