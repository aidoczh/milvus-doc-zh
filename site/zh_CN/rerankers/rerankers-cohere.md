---
id: rerankers-cohere.md
order: 3
summary: Milvus通过`CohereRerankFunction`类支持Cohere重新排序模型。这一功能可以帮助您有效地评分查询-文档对的相关性。
title: Cohere
---

# Cohere

Milvus通过`CohereRerankFunction`类支持[Cohere](https://docs.cohere.com/docs/rerank-2)重新排序模型。这一功能可以帮助您有效地评分查询-文档对的相关性。

要使用此功能，请安装必要的依赖项：

```bash
pip install --upgrade pymilvus
pip install "pymilvus[model]"
```

然后，实例化`CohereRerankFunction`：

```python
from pymilvus.model.reranker import CohereRerankFunction

# 定义重新排序函数
cohere_rf = CohereRerankFunction(
    model_name="rerank-english-v3.0",  # 指定模型名称。默认为`rerank-english-v2.0`。
    api_key=COHERE_API_KEY # 替换为您的Cohere API密钥
)
```

**参数**

- `model_name` (*字符串*)

    要使用的模型名称。您可以指定任何可用的Cohere重新排序模型名称，例如，`rerank-english-v3.0`，`rerank-multilingual-v3.0`等。如果不指定此参数，将使用`rerank-english-v2.0`。有关可用模型的列表，请参阅[Rerank](https://docs.cohere.com/docs/rerank-2)。

- `api_key` (*字符串*)

    用于访问Cohere API的API密钥。有关如何创建API密钥的信息，请参阅[Cohere dashboard](https://dashboard.cohere.com/api-keys)。

然后，使用以下代码基于查询重新排序文档：

```python
query = "1956年的哪个事件标志着人工智能作为一门学科的正式诞生？"

documents = [
    "1950年，艾伦·图灵发表了他的重要论文《计算机器与智能》，提出了图灵测试作为智能的标准，这是哲学和人工智能发展中的基础概念。",
    "1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡等人创造了术语“人工智能”并阐明了其基本目标。",
    "1951年，英国数学家和计算机科学家艾伦·图灵还开发了第一个旨在下棋的程序，展示了人工智能在游戏策略中的早期示例。",
    "1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克里夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。"
]

results = cohere_rf(
    query=query,
    documents=documents,
    top_k=3,
)

for result in results:
    print(f"索引: {result.index}")
    print(f"得分: {result.score:.6f}")
    print(f"文本: {result.text}\n")
```

预期输出类似于以下内容：
```python
索引: 1
得分: 0.99691266
文本: 1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡和其他人创造了术语“人工智能”，并阐明了其基本目标。

索引: 3
得分: 0.8578872
文本: 1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。

索引: 0
得分: 0.3589146
文本: 1950年，艾伦·图灵发表了他的重要论文《计算机器与智能》，提出了图灵测试作为智能的标准，这是人工智能哲学和发展中的基础概念。
```
