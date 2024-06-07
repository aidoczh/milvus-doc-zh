---
id: embed-with-voyage.md
order: 7
summary: 本文介绍如何使用 VoyageEmbeddingFunction 来使用 Voyage 模型对文档和查询进行编码。
title: Voyage
---

# Voyage

Milvus 通过 VoyageEmbeddingFunction 类与 Voyage 的模型集成。该类提供了使用 Voyage 模型对文档和查询进行编码的方法，并返回与 Milvus 索引兼容的密集向量作为嵌入。要使用此功能，请从[Voyage](https://docs.voyageai.com/docs/api-key-and-installation)获取 API 密钥，方法是在其平台上创建一个帐户。

要使用此功能，首先安装必要的依赖项：

```bash
pip install --upgrade pymilvus
pip install "pymilvus[model]"
```

然后，实例化 `VoyageEmbeddingFunction`：

```python
from pymilvus.model.dense import VoyageEmbeddingFunction

voyage_ef = VoyageEmbeddingFunction(
    model_name="voyage-lite-02-instruct", # 默认为 `voyage-2`
    api_key=VOYAGE_API_KEY # 提供您的 Voyage API 密钥
)
```

__参数__：

- `model_name`（字符串）
  要用于编码的 Voyage 模型的名称。您可以指定任何可用的 Voyage 模型名称，例如 `voyage-law-2`，`voyage-code-2` 等。如果不指定此参数，将使用 `voyage-2`。有关可用模型的列表，请参阅[Voyage 官方文档](https://docs.voyageai.com/docs/embeddings)。
- `api_key`（字符串）
  用于访问 Voyage API 的 API 密钥。有关如何创建 API 密钥的信息，请参阅[API 密钥和 Python 客户端](https://docs.voyageai.com/docs/api-key-and-installation)。

要为文档创建嵌入，使用 `encode_documents()` 方法：

```python
docs = [
    "Artificial intelligence was founded as an academic discipline in 1956.",
    "Alan Turing was the first person to conduct substantial research in AI.",
    "Born in Maida Vale, London, Turing was raised in southern England.",
]

docs_embeddings = voyage_ef.encode_documents(docs)

# 打印嵌入
print("嵌入:", docs_embeddings)
# 打印嵌入的维度和形状
print("维度:", voyage_ef.dim, docs_embeddings[0].shape)
```

预期输出类似于以下内容：

```python
嵌入: [array([ 0.02582654, -0.00907086, -0.04604037, ..., -0.01227521,
        0.04420955, -0.00038829]), array([ 0.03844212, -0.01597065, -0.03728884, ..., -0.02118733,
        0.03349845,  0.0065346 ]), array([ 0.05143557, -0.01096631, -0.02690451, ..., -0.02416254,
        0.07658645,  0.03064499])]
维度: 1024 (1024,)
```

要为查询创建嵌入，使用 `encode_queries()` 方法：

```python
queries = ["When was artificial intelligence founded", 
           "Where was Alan Turing born?"]

query_embeddings = voyage_ef.encode_queries(queries)

print("嵌入:", query_embeddings)
print("维度", voyage_ef.dim, query_embeddings[0].shape)
```

预期输出类似于以下内容：
```python
嵌入向量: [array([ 0.01733501, -0.0230672 , -0.05208827, ..., -0.00957995,
        0.04493361,  0.01485138]), array([ 0.05937521, -0.00729363, -0.02184347, ..., -0.02107683,
        0.05706626,  0.0263358 ])]
维度 1024 (1024,)
```