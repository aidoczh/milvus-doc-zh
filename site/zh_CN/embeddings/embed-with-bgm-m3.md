---
id: embed-with-bgm-m3.md
order: 4
summary: BGE-M3以其多语言、多功能和多粒度的能力而闻名。
title: BGE M3
---

# BGE M3

[BGE-M3](https://arxiv.org/abs/2402.03216)以其多语言、多功能和多粒度的能力而闻名。BGE-M3能够支持100多种语言，为多语言和跨语言检索任务设立了新的基准。其在单一框架内执行密集检索、多向量检索和稀疏检索的独特能力，使其成为广泛信息检索（IR）应用的理想选择。

Milvus使用__BGEM3EmbeddingFunction__类与BGE M3模型集成。该类处理嵌入的计算，并以与Milvus兼容的格式返回以进行索引和搜索。要使用此功能，必须安装FlagEmbedding。

要使用此功能，请安装必要的依赖项：

```bash
pip install --upgrade pymilvus
pip install "pymilvus[model]"
```

然后，实例化__BGEM3EmbeddingFunction__：

```python
from pymilvus.model.hybrid import BGEM3EmbeddingFunction

bge_m3_ef = BGEM3EmbeddingFunction(
    model_name='BAAI/bge-m3', # 指定模型名称
    device='cpu', # 指定要使用的设备，例如 'cpu' 或 'cuda:0'
    use_fp16=False # 指定是否使用fp16。如果`device`为`cpu`，则设置为`False`。
)
```

__参数__:

- __model_name__ (_字符串_)

    用于编码的模型名称。默认值为__BAAI/bge-m3__。

- __device__ (_字符串_)

    要使用的设备，使用__cpu__表示CPU，__cuda:n__表示第n个GPU设备。

- __use_fp16__ (_布尔值_)

    是否使用16位浮点精度（fp16）。当__device__为__cpu__时，指定为__False__。

要为文档创建嵌入，使用__encode_documents()__方法：

```python
docs = [
    "人工智能是一门学科，成立于1956年。",
    "艾伦·图灵是进行大量人工智能研究的第一人。",
    "图灵出生于伦敦梅达维尔，成长在英格兰南部。"
]

docs_embeddings = bge_m3_ef.encode_documents(docs)

# 打印嵌入
print("嵌入:", docs_embeddings)
# 打印密集嵌入的维度
print("密集文档维度:", bge_m3_ef.dim["dense"], docs_embeddings["dense"][0].shape)
# 由于稀疏嵌入以2D csr_array格式存在，我们将其转换为列表以便更容易操作。
print("稀疏文档维度:", bge_m3_ef.dim["sparse"], list(docs_embeddings["sparse"])[0].shape)
```

预期输出类似于以下内容：
```python
嵌入向量: {'dense': [array([-0.02505937, -0.00142193,  0.04015467, ..., -0.02094924,
        0.02623661,  0.00324098], dtype=float32), array([ 0.00118463,  0.00649292, -0.00735763, ..., -0.01446293,
        0.04243685, -0.01794822], dtype=float32), array([ 0.00415287, -0.0101492 ,  0.0009811 , ..., -0.02559666,
        0.08084674,  0.00141647], dtype=float32)], 'sparse': <3x250002 sparse array of type '<class 'numpy.float32'>'
        with 43 stored elements in Compressed Sparse Row format>}
密集文档维度: 1024 (1024,)
稀疏文档维度: 250002 (1, 250002)
```
要为查询创建嵌入向量，可以使用 __encode_queries()__ 方法：

```python
queries = ["人工智能是什么时候创建的", 
           "Alan Turing 出生在哪里？"]

query_embeddings = bge_m3_ef.encode_queries(queries)

# 打印嵌入向量
print("嵌入向量:", query_embeddings)
# 打印密集嵌入的维度
print("密集查询维度:", bge_m3_ef.dim["dense"], query_embeddings["dense"][0].shape)
# 由于稀疏嵌入是以 2D csr_array 格式存储的，我们将其转换为列表以便更容易操作。
print("稀疏查询维度:", bge_m3_ef.dim["sparse"], list(query_embeddings["sparse"])[0].shape)
```

预期输出类似于以下内容：

```python
嵌入向量: {'dense': [array([-0.02024024, -0.01514386,  0.02380808, ...,  0.00234648,
       -0.00264978, -0.04317448], dtype=float32), array([ 0.00648045, -0.0081542 , -0.02717067, ..., -0.00380103,
        0.04200587, -0.01274772], dtype=float32)], 'sparse': <2x250002 sparse array of type '<class 'numpy.float32'>'
        with 14 stored elements in Compressed Sparse Row format>}
密集查询维度: 1024 (1024,)
稀疏查询维度: 250002 (1, 250002)
```