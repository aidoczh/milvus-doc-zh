---
id: embed-with-bm25.md
order: 5
summary: BM25是信息检索中使用的排名函数，用于估计文档与给定搜索查询的相关性。
title: BM25
---

# BM25

[BM25](https://en.wikipedia.org/wiki/Okapi_BM25) 是信息检索中使用的排名函数，用于估计文档与给定搜索查询的相关性。它通过包含文档长度归一化和词项频率饱和度来增强基本的词项频率方法。BM25可以通过将文档表示为词项重要性分数的向量来生成稀疏嵌入，从而在稀疏向量空间中实现高效的检索和排名。

Milvus集成了BM25模型，使用__BM25EmbeddingFunction__类。该类处理嵌入的计算，并以与Milvus兼容的格式返回它们，以便进行索引和搜索。在这个过程中，构建一个用于标记化的分析器至关重要。

要使用此功能，请安装必要的依赖项：

```bash
pip install --upgrade pymilvus
pip install "pymilvus[model]"
```

Milvus提供了一个默认分析器，只需要指定文本的语言即可轻松创建一个分词器。

__示例__:

```python
from pymilvus.model.sparse.bm25.tokenizers import build_default_analyzer
from pymilvus.model.sparse import BM25EmbeddingFunction

# 有一些内置的分析器适用于几种语言，现在我们使用 'en' 代表英语。
analyzer = build_default_analyzer(language="en")

corpus = [
    "Artificial intelligence was founded as an academic discipline in 1956.",
    "Alan Turing was the first person to conduct substantial research in AI.",
    "Born in Maida Vale, London, Turing was raised in southern England.",
]

# 分析器可以将文本标记化为标记
tokens = analyzer(corpus[0])
print("tokens:", tokens)
```

__参数__:

- __language__ (_string_)

    要进行标记化的文本的语言。有效选项包括 __en__（英语），__de__（德语），__fr__（法语），__ru__（俄语），__sp__（西班牙语），__it__（意大利语），__pt__（葡萄牙语），__zh__（中文），__jp__（日语），__kr__（韩语）。

预期输出类似于以下内容:

```python
tokens: ['artifici', 'intellig', 'found', 'academ', 'disciplin', '1956']
```

BM25算法通过首先使用内置分析器将文本分解为标记来处理文本，如英语语言标记 __'artifici'__，__'intellig'__ 和 __'academ'__ 所示。然后，它收集这些标记的统计信息，评估它们在文档中的频率和分布。BM25的核心根据其重要性计算每个标记的相关性分数，较稀有的标记获得更高的分数。这个简洁的过程使得文档可以根据与查询的相关性进行有效排名。

要收集语料库的统计信息，使用__fit()__方法：
```python
# 使用分析器实例化 BM25EmbeddingFunction
bm25_ef = BM25EmbeddingFunction(analyzer)

# 在语料库上拟合模型，获取语料库的统计信息
bm25_ef.fit(corpus)
```
接下来，使用__encode_documents()__为文档创建嵌入：

```python
docs = [
    "人工智能领域于1956年确立为学术科目。",
    "艾伦·图灵是在人工智能领域进行重要研究的先驱。",
    "图灵在伦敦梅达维尔出生，成长于英格兰南部地区。",
    "1956年，人工智能作为一个学术领域出现。",
    "图灵原籍伦敦梅达维尔，成长于英格兰南部。"
]

# 为文档创建嵌入
docs_embeddings = bm25_ef.encode_documents(docs)

# 打印嵌入
print("Embeddings:", docs_embeddings)
# 由于输出的嵌入是二维 csr_array 格式，我们将其转换为列表以便更容易操作。
print("Sparse dim:", bm25_ef.dim, list(docs_embeddings)[0].shape)
```

预期输出类似于以下内容：

```python
Embeddings:   (0, 0)        1.0208816705336425
  (0, 1)        1.0208816705336425
  (0, 3)        1.0208816705336425
...
  (4, 16)        0.9606986899563318
  (4, 17)        0.9606986899563318
  (4, 20)        0.9606986899563318
Sparse dim: 21 (1, 21)
```

要为查询创建嵌入，使用__encode_queries()__方法：

```python
queries = ["人工智能是在哪一年成立的", 
           "艾伦·图灵出生在哪里？"]

query_embeddings = bm25_ef.encode_queries(queries)

# 打印嵌入
print("Embeddings:", query_embeddings)
# 由于输出的嵌入是二维 csr_array 格式，我们将其转换为列表以便更容易操作。
print("Sparse dim:", bm25_ef.dim, list(query_embeddings)[0].shape)
```

预期输出类似于以下内容：

```python
Embeddings:   (0, 0)        0.5108256237659907
  (0, 1)        0.5108256237659907
  (0, 2)        0.5108256237659907
  (1, 6)        0.5108256237659907
  (1, 7)        0.11554389108992644
  (1, 14)        0.5108256237659907
Sparse dim: 21 (1, 21)
```

__注意：__

使用__BM25EmbeddingFunction__时，请注意__encoding_queries()__和__encoding_documents()__操作在数学上不能互换。因此，没有实现可用的__bm25_ef(texts)__。