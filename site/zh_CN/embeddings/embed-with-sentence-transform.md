---
id: embed-with-sentence-transform.md
order: 3
summary: 本文演示了如何在 Milvus 中使用 Sentence Transformers 将文档和查询编码为密集向量。
title: 句子转换器
---

# 句子转换器

Milvus 通过 __SentenceTransformerEmbeddingFunction__ 类集成了 [Sentence Transformer](https://www.sbert.net/docs/pretrained_models.html#model-overview) 预训练模型。该类提供了使用预训练的 Sentence Transformer 模型对文档和查询进行编码的方法，并返回与 Milvus 索引兼容的密集向量。

要使用此功能，请安装必要的依赖项：

```bash
pip install --upgrade pymilvus
pip install "pymilvus[model]"
```

然后，实例化 __SentenceTransformerEmbeddingFunction__：

```python
from pymilvus import model

sentence_transformer_ef = model.dense.SentenceTransformerEmbeddingFunction(
    model_name='all-MiniLM-L6-v2', # 指定模型名称
    device='cpu' # 指定要使用的设备，例如 'cpu' 或 'cuda:0'
)
```

__参数__：

- __model_name__ (_字符串_)

    要用于编码的 Sentence Transformer 模型的名称。默认值为 __all-MiniLM-L6-v2__。您可以使用 Sentence Transformers 的任何预训练模型。有关可用模型的列表，请参阅[预训练模型](https://www.sbert.net/docs/pretrained_models.html)。

- __device__ (_字符串_)

    要使用的设备，使用 __cpu__ 表示 CPU，__cuda:n__ 表示第 n 个 GPU 设备。

要为文档创建嵌入，使用 __encode_documents()__ 方法：

```python
docs = [
    "人工智能是一门学科，成立于 1956 年。",
    "艾伦·图灵是第一个进行大量人工智能研究的人。",
    "图灵出生于伦敦梅达维尔，在英格兰南部长大。",
]

docs_embeddings = sentence_transformer_ef.encode_documents(docs)

# 打印嵌入
print("嵌入:", docs_embeddings)
# 打印嵌入的维度和形状
print("维度:", sentence_transformer_ef.dim, docs_embeddings[0].shape)
```

预期输出类似于以下内容：

```python
嵌入: [array([-3.09392996e-02, -1.80662833e-02,  1.34775648e-02,  2.77156215e-02,
       -4.86349640e-03, -3.12581174e-02, -3.55921760e-02,  5.76934684e-03,
        2.80773244e-03,  1.35783911e-01,  3.59678417e-02,  6.17732145e-02,
...
       -4.61330153e-02, -4.85207550e-02,  3.13997865e-02,  7.82178566e-02,
       -4.75336798e-02,  5.21207601e-02,  9.04406682e-02, -5.36676683e-02],
      dtype=float32)]
维度: 384 (384,)
```

要为查询创建嵌入，使用 __encode_queries()__ 方法：

```python
queries = ["人工智能是在哪一年成立的", 
           "艾伦·图灵出生在哪里？"]

query_embeddings = sentence_transformer_ef.encode_queries(queries)

# 打印嵌入
print("嵌入:", query_embeddings)
# 打印嵌入的维度和形状
print("维度:", sentence_transformer_ef.dim, query_embeddings[0].shape)
```
```python
嵌入向量: [array([-2.52114702e-02, -5.29330298e-02,  1.14570223e-02,  1.95571519e-02,
       -2.46500354e-02, -2.66519729e-02, -8.48201662e-03,  2.82961670e-02,
       -3.65092754e-02,  7.50745758e-02,  4.28900979e-02,  7.18822703e-02,
...
       -6.76431581e-02, -6.45996556e-02, -4.67132553e-02,  4.78532910e-02,
       -2.31596199e-03,  4.13446948e-02,  1.06935494e-01, -1.08258888e-01],
      dtype=float32)]
维度: 384 (384,)
```