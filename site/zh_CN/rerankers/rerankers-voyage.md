---
id: rerankers-voyage.md
order: 5
summary: Milvus通过`VoyageRerankFunction`类支持Voyage重新排序模型。该功能可帮助您有效地评分查询-文档对的相关性。
title: Voyage
---

# Voyage

Milvus通过`VoyageRerankFunction`类支持[Voyage重新排序模型](https://github.com/FlagOpen/FlagEmbedding/tree/master/FlagEmbedding/reranker)。该功能可帮助您有效地评分查询-文档对的相关性。

要使用此功能，请安装必要的依赖项：

```bash
pip install --upgrade pymilvus
pip install "pymilvus[model]"
```

然后，实例化`VoyageRerankFunction`：

```python
from pymilvus.model.reranker import VoyageRerankFunction

# 定义重新排序函数
voyage_rf = VoyageRerankFunction(
    model_name="rerank-lite-1",  # 指定模型名称，默认为`rerank-lite-1`。
    api_key=VOYAGE_API_KEY # 用您的Voyage API密钥替换
)
```

**参数**：

- `model_name` (*字符串*)

    要用于编码的Voyage模型的名称。如果不指定此参数，将使用`rerank-lite-1`。有关可用模型的列表，请参阅[Rerankers](https://docs.voyageai.com/docs/reranker)。

- `api_key` (*字符串*)

    用于访问Voyage API的API密钥。有关如何创建API密钥的信息，请参阅[API密钥和Python客户端](https://docs.voyageai.com/docs/api-key-and-installation)。

然后，使用以下代码基于查询重新排序文档：

```python
query = "1956年的哪个事件标志着人工智能作为一门学科的正式诞生？"

documents = [
    "1950年，艾伦·图灵发表了他的重要论文《计算机器与智能》，提出了图灵测试作为智能的标准，这是哲学和人工智能发展中的基本概念。",
    "1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡等人创造了术语“人工智能”并阐明了其基本目标。",
    "1951年，英国数学家兼计算机科学家艾伦·图灵还开发了第一个旨在下棋的程序，展示了人工智能在游戏策略中的早期示例。",
    "1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。"
]

results = voyage_rf(
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
得分: 0.898438
文本: 1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡和其他人创造了术语“人工智能”，并阐明了其基本目标。

索引: 3
得分: 0.718750
文本: 1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。

索引: 0
得分: 0.679688
文本: 1950年，艾伦·图灵发表了开创性论文《计算机器与智能》，提出了图灵测试作为智能的标准，这是人工智能哲学和发展中的基础概念。
```