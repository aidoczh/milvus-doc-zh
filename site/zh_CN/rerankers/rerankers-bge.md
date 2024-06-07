---
id: rerankers-bge.md
order: 2
summary: Milvus通过`BGERerankFunction`类支持BGE重新排序模型。这一功能可以有效地评分查询-文档对的相关性。
title: BGE
---

# BGE

Milvus通过`BGERerankFunction`类支持[BGE重新排序模型](https://github.com/FlagOpen/FlagEmbedding/tree/master/FlagEmbedding/reranker)。这一功能可以有效地评分查询-文档对的相关性。

要使用此功能，请安装必要的依赖项：

```bash
pip install --upgrade pymilvus
pip install "pymilvus[model]"
```

然后，实例化`BGERerankFunction`：

```python
from pymilvus.model.reranker import BGERerankFunction

# 定义重新排序函数
bge_rf = BGERerankFunction(
    model_name="BAAI/bge-reranker-v2-m3",  # 指定模型名称。默认为`BAAI/bge-reranker-v2-m3`。
    device="cpu" # 指定要使用的设备，例如'cpu'或'cuda:0'
)
```

**参数**

- `model_name` (*字符串*)

    要使用的模型名称。您可以指定任何可用的BGE重新排序模型名称，例如`BAAI/bge-reranker-base`，`BAAI/bge-reranker-large`等。如果不指定此参数，将使用`BAAI/bge-reranker-v2-m3`。有关可用模型的列表，请参阅[模型列表](https://github.com/FlagOpen/FlagEmbedding/tree/master/FlagEmbedding/llm_reranker#model-list)。

- `device` (*字符串*)

    可选。用于运行模型的设备。如果未指定，模型将在CPU上运行。您可以为CPU指定`cpu`，为第n个GPU设备指定`cuda:n`。

然后，使用以下代码基于查询重新排序文档：

```python
query = "1956年的哪个事件标志着人工智能作为一门学科的正式诞生？"

documents = [
    "1950年，Alan Turing发表了他的重要论文《计算机器与智能》，提出了图灵测试作为智能的标准，这是人工智能哲学和发展中的基本概念。",
    "1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，John McCarthy等人创造了术语“人工智能”并阐明了其基本目标。",
    "1951年，英国数学家和计算机科学家Alan Turing还开发了第一个旨在下棋的程序，展示了人工智能在游戏策略中的早期示例。",
    "1955年，Allen Newell、Herbert A. Simon和Cliff Shaw发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。"
]

results = bge_rf(
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
得分: 0.991162
文本: 1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡和其他人创造了“人工智能”这个术语，并阐明了其基本目标。

索引: 0
得分: 0.032697
文本: 1950年，艾伦·图灵发表了他具有开创性意义的论文《计算机器与智能》，提出了图灵测试作为智能的标准，这是人工智能哲学和发展中的基础概念。

索引: 3
得分: 0.006515
文本: 1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。
```