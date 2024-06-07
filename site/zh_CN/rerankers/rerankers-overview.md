---
id: rerankers-overview.md
order: 1
summary: PyMilvus 模型库集成了重新排序功能，以优化从初始搜索返回的结果顺序。
title: 概述
---

# 概述

在信息检索和生成式人工智能领域，重新排序器是一种关键工具，用于优化从初始搜索结果返回的顺序。重新排序器与传统的[嵌入模型](embeddings.md)不同，它接受查询和文档作为输入，并直接返回相似性分数，而不是嵌入。这个分数表示输入查询和文档之间的相关性。

重新排序器通常在第一阶段检索之后使用，通常是通过向量近似最近邻（ANN）技术完成的。虽然 ANN 搜索在获取一组潜在相关结果方面效率很高，但它们可能并不总是根据与查询的实际语义接近程度来优先考虑结果。在这里，重新排序器通过使用更深入的上下文分析来优化结果顺序，通常利用像 BERT 或其他基于 Transformer 的先进机器学习模型。通过这样做，重新排序器可以显著提高呈现给用户的最终结果的准确性和相关性。

PyMilvus 模型库集成了重新排序功能，以优化从初始搜索返回的结果顺序。在从 Milvus 检索到最近的嵌入之后，您可以利用这些重新排序工具来优化搜索结果，以增强搜索结果的精度。

| 重新排序功能 | API 或开源 |
| --------------- | ------------------- |
| [bgereranker](https://milvus.io/api-reference/pymilvus/v2.4.x/Rerankers/BGE/BGERerankFunction.md)     | 开源        |
| [cross-encoder](https://milvus.io/api-reference/pymilvus/v2.4.x/Rerankers/CrossEncoder/CrossEncoderRerankFunction.md)   | 开源        |
| [voyageai](https://milvus.io/api-reference/pymilvus/v2.4.x/Rerankers/Voyage/VoyageRerankFunction.md)        | API                 |
| [cohere](https://milvus.io/api-reference/pymilvus/v2.4.x/Rerankers/Cohere/CohereRerankFunction.md)          | API                 |

<div class="alert note">

- 在使用开源重新排序器之前，请确保下载并安装所有必需的依赖项和模型。

- 对于基于 API 的重新排序器，请从提供者那里获取 API 密钥，并将其设置在适当的环境变量或参数中。

</div>

## 示例 1：使用 BGE 重新排序功能根据查询重新排序文档

在此示例中，我们演示了如何使用基于特定查询的[BGE 重新排序器](rerankers-bge.md)重新排序搜索结果。

要使用[PyMilvus 模型](https://github.com/milvus-io/milvus-model)库中的重新排序器，请先安装 PyMilvus 模型库以及包含所有必要重新排序工具的模型子包：

```bash
pip install pymilvus[model]
# 或者对于 zsh，请使用 pip install "pymilvus[model]"。
```

要使用 BGE 重新排序器，首先导入 `BGERerankFunction` 类：
```python
从 pymilvus.model.reranker 模块导入 BGERerankFunction
```
然后，创建一个`BGERerankFunction`实例用于重新排序：

```python
bge_rf = BGERerankFunction(
    model_name="BAAI/bge-reranker-v2-m3",  # 指定模型名称。默认为`BAAI/bge-reranker-v2-m3`。
    device="cpu"  # 指定要使用的设备，例如'cpu'或'cuda:0'
)
```

要根据查询重新排序文档，请使用以下代码：

```python
query = "1956年的哪个事件标志着人工智能作为一门学科的正式诞生？"

documents = [
    "1950年，艾伦·图灵发表了他具有开创性意义的论文《计算机器械与智能》，提出了图灵测试作为智能的标准，这是哲学和人工智能发展中的基本概念。",
    "1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡等人创造了术语“人工智能”并阐明了其基本目标。",
    "1951年，英国数学家兼计算机科学家艾伦·图灵还开发了第一个旨在下棋的程序，展示了人工智能在游戏策略中的早期示例。",
    "1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。"
]

bge_rf(query, documents)
```

预期输出类似于以下内容：

```python
[RerankResult(text="1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡等人创造了术语“人工智能”并阐明了其基本目标。", score=0.9911615761470803, index=1),
 RerankResult(text="1950年，艾伦·图灵发表了他具有开创性意义的论文《计算机器械与智能》，提出了图灵测试作为智能的标准，这是哲学和人工智能发展中的基本概念。", score=0.0326971950177779, index=0),
 RerankResult(text='1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。', score=0.006514905766152258, index=3),
 RerankResult(text='1951年，英国数学家兼计算机科学家艾伦·图灵还开发了第一个旨在下棋的程序，展示了人工智能在游戏策略中的早期示例。', score=0.0042116724917325935, index=2)]
```

## 示例2：使用重新排序器增强搜索结果的相关性

在本指南中，我们将探讨如何利用PyMilvus中的`search()`方法进行相似性搜索，以及如何使用重新排序器增强搜索结果的相关性。我们的演示将使用以下数据集：
```python
entities = [
    {'doc_id': 0, 'doc_vector': [-0.0372721,0.0101959,...,-0.114994], 'doc_text': "1950年，艾伦·图灵发表了开创性论文《计算机器械与智能》，提出了图灵测试作为智能的标准，这是人工智能哲学和发展中的基础概念。"}, 
    {'doc_id': 1, 'doc_vector': [-0.00308882,-0.0219905,...,-0.00795811], 'doc_text': "1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡等人创造了“人工智能”这一术语，并阐明了其基本目标。"}, 
    {'doc_id': 2, 'doc_vector': [0.00945078,0.00397605,...,-0.0286199], 'doc_text': '1951年，英国数学家兼计算机科学家艾伦·图灵还开发了第一个旨在下棋的程序，展示了人工智能在游戏策略中的早期示例。'}, 
    {'doc_id': 3, 'doc_vector': [-0.0391119,-0.00880096,...,-0.0109257], 'doc_text': '1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，它能够解决逻辑问题，类似于证明数学定理。'}
]
```
__数据集组成__：

- `doc_id`：每个文档的唯一标识符。
- `doc_vector`：表示文档的向量嵌入。有关生成嵌入的指导，请参阅[嵌入](embeddings.md)。
- `doc_text`：文档的文本内容。

### 准备工作

在开始相似度搜索之前，您需要与 Milvus 建立连接，创建一个集合，并准备并插入数据到该集合中。以下代码片段展示了这些初步步骤。

```python
from pymilvus import MilvusClient, DataType

client = MilvusClient(
    uri="http://10.102.6.214:19530" # 替换为您自己的 Milvus 服务器地址
)

client.drop_collection('test_collection')

# 定义模式

schema = client.create_schema(auto_id=False, enabel_dynamic_field=True)

schema.add_field(field_name="doc_id", datatype=DataType.INT64, is_primary=True, description="文档标识")
schema.add_field(field_name="doc_vector", datatype=DataType.FLOAT_VECTOR, dim=384, description="文档向量")
schema.add_field(field_name="doc_text", datatype=DataType.VARCHAR, max_length=65535, description="文档文本")

# 定义索引参数

index_params = client.prepare_index_params()

index_params.add_index(field_name="doc_vector", index_type="IVF_FLAT", metric_type="IP", params={"nlist": 128})

# 创建集合

client.create_collection(collection_name="test_collection", schema=schema, index_params=index_params)

# 插入数据到集合

client.insert(collection_name="test_collection", data=entities)

# 输出:
# {'insert_count': 4, 'ids': [0, 1, 2, 3]}
```

### 进行相似度搜索

在数据插入后，使用 `search` 方法执行相似度搜索。

```python
# 基于我们的查询进行搜索结果

res = client.search(
    collection_name="test_collection",
    data=[[-0.045217834, 0.035171617, ..., -0.025117004]], # 替换为您的查询向量
    limit=3,
    output_fields=["doc_id", "doc_text"]
)

for i in res[0]:
    print(f'距离: {i["distance"]}')
    print(f'文档内容: {i["entity"]["doc_text"]}')
```

预期输出类似于以下内容：

```python
距离: 0.7235960960388184
文档内容: 1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡等人创造了“人工智能”一词，并阐明了其基本目标。
距离: 0.6269873976707458
文档内容: 1950年，艾伦·图灵发表了他的重要论文《计算机器与智能》，提出了图灵测试作为智能的标准，这是人工智能哲学和发展中的基础概念。
距离: 0.5340118408203125
文档内容: 1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。
```

### 使用重新排序器增强搜索结果
接下来，通过重新排序步骤提高搜索结果的相关性。在这个示例中，我们使用 PyMilvus 中内置的 `CrossEncoderRerankFunction` 来重新排序结果，以提高准确性。

```python
# 使用 reranker 重新排序搜索结果

from pymilvus.model.reranker import CrossEncoderRerankFunction

ce_rf = CrossEncoderRerankFunction(
    model_name="cross-encoder/ms-marco-MiniLM-L-6-v2",  # 指定模型名称。
    device="cpu"  # 指定要使用的设备，例如 'cpu' 或 'cuda:0'
)

reranked_results = ce_rf(
    query='1956年的哪个事件标志着人工智能作为一门学科的正式诞生？',
    documents=[
        "1950年，Alan Turing 发表了他的开创性论文《计算机器和智能》，提出了图灵测试作为智能的标准，这是哲学和人工智能发展中的基本概念。",
        "1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，John McCarthy 等人创造了术语“人工智能”并阐明了其基本目标。",
        "1951年，英国数学家和计算机科学家 Alan Turing 也开发了第一个旨在下棋的程序，展示了人工智能在游戏策略中的早期示例。",
        "1955年，Allen Newell、Herbert A. Simon 和 Cliff Shaw 发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。"
    ],
    top_k=3
)

# 打印重新排序后的结果
for result in reranked_results:
    print(f'score: {result.score}')
    print(f'doc_text: {result.text}')
```

预期输出类似于以下内容：

```python
score: 6.250532627105713
doc_text: 1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，John McCarthy 等人创造了术语“人工智能”并阐明了其基本目标。
score: -2.9546022415161133
doc_text: 1950年，Alan Turing 发表了他的开创性论文《计算机器和智能》，提出了图灵测试作为智能的标准，这是哲学和人工智能发展中的基本概念。
score: -4.771512031555176
doc_text: 1955年，Allen Newell、Herbert A. Simon 和 Cliff Shaw 发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。
```