---
id: integrate_with_ragas.md
summary: 本指南演示了如何使用 Ragas 来评估建立在 Milvus 之上的检索增强生成（RAG）管道。
title: 使用 Ragas 进行评估
---

# 使用 Ragas 进行评估

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/evaluation_with_ragas.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

本指南演示了如何使用 Ragas 来评估建立在 [Milvus](https://milvus.io/) 上的检索增强生成（RAG）管道。

RAG 系统将检索系统与生成模型结合起来，根据给定提示生成新文本。系统首先使用类似 Milvus 的向量相似度搜索引擎从语料库中检索相关文档，然后使用生成模型根据检索到的文档生成新文本。

[Ragas](https://docs.ragas.io/en/latest/index.html#) 是一个框架，可帮助您评估 RAG 管道。有一些现有工具和框架可帮助您构建这些管道，但评估和量化管道性能可能很困难。这就是 Ragas（RAG 评估）发挥作用的地方。


## 先决条件

在运行此笔记本之前，请确保已安装以下依赖项：


```python
$ pip install --upgrade pymilvus openai requests tqdm pandas ragas
```

<div class="alert note">

如果您正在使用 Google Colab，在启用刚安装的依赖项后，您可能需要**重新启动运行时**。

</div>

在本示例中，我们将使用 OpenAI 作为 LLM。您应准备好将 [api key](https://platform.openai.com/docs/quickstart) `OPENAI_API_KEY` 设置为环境变量。


```python
import os

os.environ["OPENAI_API_KEY"] = "sk-***********"
```

## 定义 RAG 管道

我们将定义 RAG 类，该类使用 Milvus 作为向量存储，OpenAI 作为 LLM。
该类包含 `load` 方法，用于将文本数据加载到 Milvus 中，`retrieve` 方法，用于检索与给定问题最相似的文本数据，以及 `answer` 方法，用于根据检索到的知识回答给定问题。


```python
from typing import List
from tqdm import tqdm
from openai import OpenAI
from pymilvus import MilvusClient


class RAG:
    """
    RAG (Retrieval-Augmented Generation) 类，基于 OpenAI 和 Milvus 构建。
    """

    def __init__(self, openai_client: OpenAI, milvus_client: MilvusClient):
        self._prepare_openai(openai_client)
        self._prepare_milvus(milvus_client)

    def _emb_text(self, text: str) -> List[float]:
        return (
            self.openai_client.embeddings.create(input=text, model=self.embedding_model)
            .data[0]
            .embedding
        )

    def _prepare_openai(
        self,
        openai_client: OpenAI,
        embedding_model: str = "text-embedding-3-small",
        llm_model: str = "gpt-3.5-turbo",
    ):
        self.openai_client = openai_client
        self.embedding_model = embedding_model
        self.llm_model = llm_model
        self.SYSTEM_PROMPT = """
Human: You are an AI assistant. You are able to find answers to the questions from the contextual passage snippets provided.
"""
        self.USER_PROMPT = """
使用以下信息（包含在 <context> 标签中）来回答 <question> 标签中的问题。
<context>
{context}
</context>
<question>
{question}
</question>
"""

    def _prepare_milvus(
        self, milvus_client: MilvusClient, collection_name: str = "rag_collection"
    ):
        self.milvus_client = milvus_client
        self.collection_name = collection_name
        if self.milvus_client.has_collection(self.collection_name):
            self.milvus_client.drop_collection(self.collection_name)
        embedding_dim = len(self._emb_text("foo"))
        self.milvus_client.create_collection(
            collection_name=self.collection_name,
            dimension=embedding_dim,
            metric_type="IP",  # 内积距离
            consistency_level="Strong",  # 强一致性级别
        )

    def load(self, texts: List[str]):
        """
        将文本数据加载到 Milvus 中。
        """
        data = []
        for i, line in enumerate(tqdm(texts, desc="Creating embeddings")):
            data.append({"id": i, "vector": self._emb_text(line), "text": line})

        self.milvus_client.insert(collection_name=self.collection_name, data=data)

    def retrieve(self, question: str, top_k: int = 3) -> List[str]:
        """
        检索与给定问题最相似的文本数据。
        """
        search_res = self.milvus_client.search(
            collection_name=self.collection_name,
            data=[self._emb_text(question)],
            limit=top_k,
            search_params={"metric_type": "IP", "params": {}},  # 内积距离
            output_fields=["text"],  # 返回文本字段
        )
        retrieved_texts = [res["entity"]["text"] for res in search_res[0]]
        return retrieved_texts[:top_k]

    def answer(
        self,
        question: str,
        retrieval_top_k: int = 3,
        return_retrieved_text: bool = False,
    ):
        """
        使用检索到的知识回答给定问题。
        """
        retrieved_texts = self.retrieve(question, top_k=retrieval_top_k)
        user_prompt = self.USER_PROMPT.format(
            context="\n".join(retrieved_texts), question=question
        )
        response = self.openai_client.chat.completions.create(
            model=self.llm_model,
            messages=[
                {"role": "system", "content": self.SYSTEM_PROMPT},
                {"role": "user", "content": user_prompt},
            ],
        )
        if not return_retrieved_text:
            return response.choices[0].message.content
        else:
            return response.choices[0].message.content, retrieved_texts
```
让我们使用 OpenAI 和 Milvus 客户端初始化 RAG 类。

```python
openai_client = OpenAI()
milvus_client = MilvusClient("./milvus_demo.db")

my_rag = RAG(openai_client=openai_client, milvus_client=milvus_client)
```

## 运行 RAG 流程并获取结果

我们使用 [Milvus 开发指南](https://github.com/milvus-io/milvus/blob/master/DEVELOPMENT.md) 作为我们 RAG 中的私有知识，这是一个简单 RAG 流程的良好数据源。

下载并加载到 rag 流程中。

```python
import os
import urllib.request

url = "https://raw.githubusercontent.com/milvus-io/milvus/master/DEVELOPMENT.md"
file_path = "./Milvus_DEVELOPMENT.md"

if not os.path.exists(file_path):
    urllib.request.urlretrieve(url, file_path)
with open(file_path, "r") as file:
    file_text = file.read()

# 我们简单地使用“# ”来分隔文件中的内容，这样可以粗略地将 markdown 文件的每个主要部分的内容分开。
text_lines = file_text.split("# ")
my_rag.load(text_lines)  # 将文本数据加载到 RAG 流程中
```

    Creating embeddings: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 47/47 [00:16<00:00,  2.80it/s]


让我们定义一个关于开发指南文档内容的查询问题。然后使用 `answer` 方法获取答案和检索到的上下文文本。

```python
question = "如果我想要构建 Milvus 并从源代码运行，硬件要求是什么规格？"
my_rag.answer(question, return_retrieved_text=True)
```




    ('要构建并从源代码运行 Milvus 的硬件要求规格为 8GB RAM 和 50GB 空闲磁盘空间。',
     ['硬件要求\n\n以下规格（物理或虚拟机资源）建议用于 Milvus 从源代码构建和运行。\n\n```\n- 8GB RAM\n- 50GB 空闲磁盘空间\n```\n\n##',
      '在本地 OS/shell 环境上构建 Milvus\n\n以下详细说明了在 Linux 和 MacOS 上构建所需的硬件和软件要求。\n\n##',
      "软件要求\n\n所有 Linux 发行版都可用于 Milvus 开发。然而，我们的大多数贡献者使用 Ubuntu 或 CentOS 系统，少部分贡献者使用 Mac（x86_64 和 Apple Silicon）。如果您希望 Milvus 在其他发行版上构建和运行，欢迎提出问题并贡献！\n\n以下是 Milvus 可成功构建和运行的经过验证的 OS 类型列表：\n\n- Debian/Ubuntu\n- Amazon Linux\n- MacOS（x86_64）\n- MacOS（Apple Silicon）\n\n##"])



现在让我们准备一些问题及其对应的真实答案。我们从 RAG 流程中获取答案和上下文。
```python
from datasets import Dataset
import pandas as pd

question_list = [
    "如果我想构建 Milvus 并从源代码运行，硬件要求规范是什么？",
    "编写 Knowhere 使用的编程语言是什么？",
    "在运行代码覆盖率之前应该确保什么？",
]
ground_truth_list = [
    "如果您想构建 Milvus 并从源代码运行，推荐的硬件要求规范是：\n\n- 8GB 的 RAM\n- 50GB 的可用磁盘空间。",
    "编写 Knowhere 使用的编程语言是 C++。",
    "在运行代码覆盖率之前，您应该确保您的代码更改已被单元测试覆盖。",
]
contexts_list = []
answer_list = []
for question in tqdm(question_list, desc="Answering questions"):
    answer, contexts = my_rag.answer(question, return_retrieved_text=True)
    contexts_list.append(contexts)
    answer_list.append(answer)

df = pd.DataFrame(
    {
        "question": question_list,
        "contexts": contexts_list,
        "answer": answer_list,
        "ground_truth": ground_truth_list,
    }
)
rag_results = Dataset.from_pandas(df)
df
```
## 使用 Ragas 进行评估

我们使用 Ragas 来评估我们的 RAG 管道结果的性能。

Ragas 提供了一组易于使用的指标。我们将`答案相关性`、`忠实度`、`上下文召回`和`上下文精确度`作为评估我们的 RAG 管道的指标。有关这些指标的更多信息，请参阅[Ragas 指标](https://docs.ragas.io/en/latest/concepts/metrics/index.html)。

```python
from ragas import evaluate
from ragas.metrics import (
    answer_relevancy,
    faithfulness,
    context_recall,
    context_precision,
)

result = evaluate(
    rag_results,
    metrics=[
        answer_relevancy,
        faithfulness,
        context_recall,
        context_precision,
    ],
)

result
```

评估结果如下：

- `答案相关性`: 0.9445
- `忠实度`: 1.0000
- `上下文召回`: 1.0000
- `上下文精确度`: 1.0000