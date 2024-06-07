---
id: integrate_with_jina.md
summary: 本指南演示了如何使用 Jina 嵌入和 Milvus 进行相似性搜索和检索任务。
title: 将 Milvus 与 Jina 集成
---

# 将 Milvus 与 Jina AI 集成

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/milvus_with_Jina.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

本指南演示了如何使用 Jina AI 的嵌入和 Milvus 来进行相似性搜索和检索任务。

## Jina AI 是谁
Jina AI 成立于 2020 年，总部位于柏林，是一家专注于通过其搜索基础技术改变人工智能未来的先导人工智能公司。Jina AI 专注于多模态人工智能，旨在通过其集成的组件套件（包括嵌入、重新排序器、提示操作和核心基础设施）赋能企业和开发人员，利用多模态数据的力量创造价值并节省成本。Jina AI 的尖端嵌入技术拥有一流的性能，采用适用于全面数据表示的 8192 令牌长度模型。这些嵌入提供多语言支持，并与 OpenAI 等领先平台无缝集成，有助于跨语言应用的实现。

## Milvus 和 Jina AI 的嵌入
为了高效存储和搜索这些嵌入以实现速度和规模，需要专门设计用于此目的的基础架构。Milvus 是一个广为人知的先进开源向量数据库，能够处理大规模向量数据。Milvus 能够根据多种指标进行快速准确的向量（嵌入）搜索。其可扩展性使其能够无缝处理海量图像数据，确保即使数据集增长，也能进行高性能搜索操作。

## 示例
Jina 的嵌入已集成到 PyMilvus 模型库中。现在，我们将演示代码示例，展示如何在实际操作中使用 Jina 的嵌入。

在开始之前，我们需要安装 PyMilvus 的模型库。

```python
$ pip install -U pymilvus
$ pip install "pymilvus[model]"
```

<div class="alert note">

如果您正在使用 Google Colab，在启用刚安装的依赖项后，您可能需要**重新启动运行时**。

</div>

## 通用嵌入
Jina AI 的核心嵌入模型擅长理解详细文本，非常适合语义搜索、内容分类，支持高级情感分析、文本摘要和个性化推荐系统。

```python
from pymilvus.model.dense import JinaEmbeddingFunction

jina_api_key = "<YOUR_JINA_API_KEY>"
ef = JinaEmbeddingFunction("jina-embeddings-v2-base-en", jina_api_key)

query = "what is information retrieval?"
doc = "Information retrieval is the process of finding relevant information from a large collection of data or documents."

qvecs = ef.encode_queries([query])
dvecs = ef.encode_documents([doc])
```
## 双语嵌入

Jina AI 的双语模型增强了多语言平台、全球支持和跨语言内容发现。设计用于德语-英语和中文-英语翻译，促进不同语言群体之间的理解，简化跨语言交流。

```python
from pymilvus.model.dense import JinaEmbeddingFunction

jina_api_key = "<YOUR_JINA_API_KEY>"
ef = JinaEmbeddingFunction("jina-embeddings-v2-base-de", jina_api_key)

query = "what is information retrieval?"
doc = "Information Retrieval ist der Prozess, relevante Informationen aus einer großen Sammlung von Daten oder Dokumenten zu finden."

qvecs = ef.encode_queries([query])
dvecs = ef.encode_documents([doc])
```

## 代码嵌入

Jina AI 的代码嵌入模型提供了通过代码和文档进行搜索的功能。它支持英语和30种流行的编程语言，可用于增强代码导航、简化代码审查和自动化文档辅助。

```python
from pymilvus.model.dense import JinaEmbeddingFunction

jina_api_key = "<YOUR_JINA_API_KEY>"
ef = JinaEmbeddingFunction("jina-embeddings-v2-base-code", jina_api_key)

# 案例1: 增强代码导航
# 查询: 功能的文本描述
# 文档: 相关代码片段

query = "function to calculate average in Python."
doc = """
def calculate_average(numbers):
    total = sum(numbers)
    count = len(numbers)
    return total / count
"""

# 案例2: 简化代码审查
# 查询: 编程概念的文本描述
# 文档: 相关代码片段或 PR

query = "pull quest related to Collection"
doc = "fix:[restful v2] parameters of create collection ..."

# 案例3: 自动文档辅助
# 查询: 需要解释的代码片段
# 文档: 相关文档或文档字符串

query = "What is Collection in Milvus"
doc = """
In Milvus, you store your vector embeddings in collections. All vector embeddings within a collection share the same dimensionality and distance metric for measuring similarity.
Milvus collections support dynamic fields (i.e., fields not pre-defined in the schema) and automatic incrementation of primary keys.
"""

qvecs = ef.encode_queries([query])
dvecs = ef.encode_documents([doc])
```

## Jina & Milvus 的语义搜索

借助强大的向量嵌入功能，我们可以结合利用 Jina AI 模型检索的嵌入向量与 Milvus Lite 向量数据库，进行语义搜索。
```python
from pymilvus.model.dense import JinaEmbeddingFunction
from pymilvus import MilvusClient

jina_api_key = "<YOUR_JINA_API_KEY>"
ef = JinaEmbeddingFunction("jina-embeddings-v2-base-en", jina_api_key)
DIMENSION = 768  # jina-embeddings-v2-base-en 的维度大小

doc = [
    "1950年，艾伦·图灵发表了开创性论文《计算机器械与智能》，提出了图灵测试作为智能的标准，这是人工智能哲学和发展中的基础概念。",
    "1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡等人创造了术语“人工智能”并阐明了其基本目标。",
    "1951年，英国数学家兼计算机科学家艾伦·图灵还开发了第一个旨在下棋的程序，展示了人工智能在游戏策略中的早期示例。",
    "1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。",
]

dvecs = ef.encode_documents(doc)

data = [
    {"id": i, "vector": dvecs[i], "text": doc[i], "subject": "history"}
    for i in range(len(dvecs))
]

milvus_client = MilvusClient("./milvus_jina_demo.db")
COLLECTION_NAME = "demo_collection"  # Milvus 集合名称
milvus_client.create_collection(collection_name=COLLECTION_NAME, dimension=DIMENSION)

res = milvus_client.insert(collection_name=COLLECTION_NAME, data=data)

print(res["insert_count"])
```
在 Milvus 向量数据库中的所有数据，我们现在可以通过为查询生成向量嵌入并进行向量搜索来执行语义搜索。

```python
queries = "What event in 1956 marked the official birth of artificial intelligence as a discipline?"
qvecs = ef.encode_queries([queries])

res = milvus_client.search(
    collection_name=COLLECTION_NAME,  # 目标集合
    data=[qvecs[0]],  # 查询向量
    limit=3,  # 返回实体的数量
    output_fields=["text", "subject"],  # 指定要返回的字段
)[0]

for result in res:
    print(result)
```

    {'id': 1, 'distance': 0.8802614808082581, 'entity': {'text': "The Dartmouth Conference in 1956 is considered the birthplace of artificial intelligence as a field; here, John McCarthy and others coined the term 'artificial intelligence' and laid out its basic goals.", 'subject': 'history'}}

## Jina 重新排序器
Jina Ai 还提供重新排序器，以在使用嵌入进行搜索后进一步提高检索质量。

```python
from pymilvus.model.reranker import JinaRerankFunction

jina_api_key = "<YOUR_JINA_API_KEY>"

rf = JinaRerankFunction("jina-reranker-v1-base-en", jina_api_key)

query = "What event in 1956 marked the official birth of artificial intelligence as a discipline?"

documents = [
    "In 1950, Alan Turing published his seminal paper, 'Computing Machinery and Intelligence,' proposing the Turing Test as a criterion of intelligence, a foundational concept in the philosophy and development of artificial intelligence.",
    "The Dartmouth Conference in 1956 is considered the birthplace of artificial intelligence as a field; here, John McCarthy and others coined the term 'artificial intelligence' and laid out its basic goals.",
    "In 1951, British mathematician and computer scientist Alan Turing also developed the first program designed to play chess, demonstrating an early example of AI in game strategy.",
    "The invention of the Logic Theorist by Allen Newell, Herbert A. Simon, and Cliff Shaw in 1955 marked the creation of the first true AI program, which was capable of solving logic problems, akin to proving mathematical theorems.",
]

rf(query, documents)
```




    [RerankResult(text="The Dartmouth Conference in 1956 is considered the birthplace of artificial intelligence as a field; here, John McCarthy and others coined the term 'artificial intelligence' and laid out its basic goals.", score=0.9370958209037781, index=1),
     RerankResult(text='The invention of the Logic Theorist by Allen Newell, Herbert A. Simon, and Cliff Shaw in 1955 marked the creation of the first true AI program, which was capable of solving logic problems, akin to proving mathematical theorems.', score=0.35420963168144226, index=3),
```markdown
- RerankResult(text="1950年，艾伦·图灵发表了开创性论文《计算机器与智能》，提出了图灵测试作为智能的标准，这是人工智能哲学和发展中的基础概念。", score=0.3498658835887909, index=0),
- RerankResult(text='1951年，英国数学家兼计算机科学家艾伦·图灵还开发了第一个旨在下棋的程序，展示了人工智能在游戏策略中的早期示例。', score=0.2728956639766693, index=2)]
```