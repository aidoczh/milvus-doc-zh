---
id: rerankers-cross-encoder.md
order: 4
summary: Milvus通过`CrossEncoderRerankFunction`类支持Cross Encoder重新排序模型。该功能可帮助您有效评分查询-文档对的相关性。
title: Cross Encoder
---

# Cross Encoder

Milvus通过`CrossEncoderRerankFunction`类支持[Cross-Encoders](https://github.com/FlagOpen/FlagEmbedding/tree/master/FlagEmbedding/reranker)。该功能可帮助您有效评分查询-文档对的相关性。

要使用此功能，请安装必要的依赖项：

```bash
pip install --upgrade pymilvus
pip install "pymilvus[model]"
```

然后，实例化`CrossEncoderRerankFunction`：

```python
from pymilvus.model.reranker import CrossEncoderRerankFunction

# 定义重新排序函数
ce_rf = CrossEncoderRerankFunction(
    model_name="cross-encoder/ms-marco-MiniLM-L-6-v2",  # 指定模型名称。
    device="cpu" # 指定要使用的设备，例如 'cpu' 或 'cuda:0'
)
```

**参数**：

- `model_name` (*字符串*)

    要使用的模型名称。您可以指定任何可用的Cross-Encoder模型名称，例如 `cross-encoder/ms-marco-TinyBERT-L-2-v2`，`cross-encoder/ms-marco-MiniLM-L-2-v2`等。如果不指定此参数，将使用空字符串。有关可用模型的列表，请参阅[预训练的Cross-Encoders](https://www.sbert.net/docs/pretrained_cross-encoders.html#)。

- `device` (*字符串*)

    用于运行模型的设备。您可以为CPU指定`cpu`，为第n个GPU设备指定`cuda:n`。

然后，使用以下代码根据查询重新排序文档：

```python
query = "What event in 1956 marked the official birth of artificial intelligence as a discipline?"

documents = [
    "In 1950, Alan Turing published his seminal paper, 'Computing Machinery and Intelligence,' proposing the Turing Test as a criterion of intelligence, a foundational concept in the philosophy and development of artificial intelligence.",
    "The Dartmouth Conference in 1956 is considered the birthplace of artificial intelligence as a field; here, John McCarthy and others coined the term 'artificial intelligence' and laid out its basic goals.",
    "In 1951, British mathematician and computer scientist Alan Turing also developed the first program designed to play chess, demonstrating an early example of AI in game strategy.",
    "The invention of the Logic Theorist by Allen Newell, Herbert A. Simon, and Cliff Shaw in 1955 marked the creation of the first true AI program, which was capable of solving logic problems, akin to proving mathematical theorems."
]

results = ce_rf(
    query=query,
    documents=documents,
    top_k=3,
)

for result in results:
    print(f"Index: {result.index}")
    print(f"Score: {result.score:.6f}")
    print(f"Text: {result.text}\n")
```

预期输出类似于以下内容：
```python
索引: 1
得分: 6.250533
文本: 1956年的达特茅斯会议被认为是人工智能领域的诞生地；在这里，约翰·麦卡锡和其他人创造了术语“人工智能”，并阐明了其基本目标。

索引: 0
得分: -2.954602
文本: 1950年，艾伦·图灵发表了他具有开创性的论文《计算机器与智能》，提出了图灵测试作为智能的标准，这是人工智能哲学和发展中的基础概念。

索引: 3
得分: -4.771512
文本: 1955年，艾伦·纽厄尔、赫伯特·A·西蒙和克利夫·肖发明了逻辑理论家，标志着第一个真正的人工智能程序的诞生，该程序能够解决逻辑问题，类似于证明数学定理。
```