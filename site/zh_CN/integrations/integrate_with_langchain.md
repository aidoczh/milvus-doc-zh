---
id: integrate_with_langchain.md
summary: 本指南演示了如何使用 LangChain 和 Milvus 构建检索增强生成（RAG）系统。
title: 使用 Milvus 和 LangChain 进行检索增强生成（RAG）
---

# 使用 Milvus 和 LangChain 进行检索增强生成（RAG）

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/rag_with_milvus_and_langchain.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

本指南演示了如何使用 LangChain 和 Milvus 构建检索增强生成（RAG）系统。

RAG 系统将检索系统与生成模型结合起来，根据给定的提示生成新文本。该系统首先使用类似 Milvus 的向量相似度搜索引擎从语料库中检索相关文档，然后使用生成模型根据检索到的文档生成新文本。

[LangChain](https://www.langchain.com/) 是一个用于开发由大型语言模型（LLMs）驱动的应用程序的框架。[Milvus](https://milvus.io/) 是世界上最先进的开源向量数据库，旨在支持嵌入式相似度搜索和人工智能应用程序。


## 先决条件

在运行此笔记本之前，请确保已安装以下依赖项：

```python
$ pip install --upgrade --quiet  langchain langchain-core langchain-community langchain-text-splitters langchain-milvus langchain-openai bs4
```

<div class="alert note">

如果您正在使用 Google Colab，请启用刚刚安装的依赖项，您可能需要**重新启动运行时**。

</div>

我们将使用 OpenAI 的模型。您应准备好 [api key](https://platform.openai.com/docs/quickstart) `OPENAI_API_KEY` 作为环境变量。


```python
import os

os.environ["OPENAI_API_KEY"] = "sk-***********"
```


## 准备数据

我们使用 Langchain 的 WebBaseLoader 从网络源加载文档，并使用 RecursiveCharacterTextSplitter 将其拆分为块。


```python
import bs4
from langchain_community.document_loaders import WebBaseLoader
from langchain_text-splitters import RecursiveCharacterTextSplitter

# 创建一个 WebBaseLoader 实例，从网络源加载文档
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            class_=("post-content", "post-title", "post-header")
        )
    ),
)
# 使用加载器从网络源加载文档
documents = loader.load()
# 初始化一个 RecursiveCharacterTextSplitter 用于将文本拆分为块
text_splitter = RecursiveCharacterTextSplitter(chunk_size=2000, chunk_overlap=200)

# 使用 text_splitter 将文档拆分为块
docs = text_splitter.split_documents(documents)

# 让我们看一下第一个文档
docs[1]
```
## 用 Milvus 向量存储构建 RAG 链

我们将使用 Milvus 向量存储初始化文档，将文档加载到 Milvus 向量存储中，并在后台构建索引。

```python
from langchain_milvus import Milvus
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()

vectorstore = Milvus.from_documents(
    documents=docs,
    embedding=embeddings,
    connection_args={
        "uri": "./milvus_demo.db",
    },
    drop_old=True,  # 如果存在，删除旧的 Milvus 集合
)
```

    /Users/zilliz/miniforge3/envs/py39/lib/python3.9/site-packages/scipy/__init__.py:155: UserWarning: 此版本的 SciPy 需要 NumPy 版本 >=1.18.5 且 <1.26.0（检测到版本 1.26.4）
```python
warnings.warn(f"NumPy 版本 >={np_minversion} 且 <{np_maxversion}"
```

使用一个测试查询问题在 Milvus 向量存储中搜索文档。我们将获得前 3 个文档。


```python
query = "一个 AI Agent 的自我反思是什么？"
vectorstore.similarity_search(query, k=3)
```




    [Document(page_content='自我反思#\n自我反思是一个至关重要的方面，它使得自主代理能够通过改进过去的行动决策和纠正以往的错误来不断提高。在试错不可避免的现实任务中，自我反思发挥着至关重要的作用。\nReAct（Yao 等人，2023）通过将行动空间扩展为任务特定的离散行动和语言空间的组合，将推理和行动集成到 LLM 中。前者使 LLM 能够与环境交互（例如使用维基百科搜索 API），而后者促使 LLM 生成自然语言中的推理痕迹。\nReAct 提示模板包含了 LLM 思考的明确步骤，大致格式如下：\n思考：...\n行动：...\n观察：...\n...（重复多次）', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/', 'pk': 449757808726900738}),
     Document(page_content='图 2. 知识密集型任务（例如 HotpotQA、FEVER）和决策任务（例如 AlfWorld Env、WebShop）的推理轨迹示例（来源：Yao 等人，2023）。\n在知识密集型任务和决策任务的两项实验中，ReAct 比删除了“思考：…”步骤的仅行动基线表现更好。\nReflexion（Shinn & Labash，2023）是一个框架，为代理提供动态记忆和自我反思能力，以提高推理能力。Reflexion 具有标准的强化学习设置，其中奖励模型提供简单的二进制奖励，行动空间遵循 ReAct 中的设置，其中任务特定的行动空间通过语言进行增强，以启用复杂的推理步骤。在每个动作 $a_t$ 后，代理计算一个启发式 $h_t$，并根据自我反思结果可能决定重置环境以开始新的试验。\n\n图 3. Reflexion 框架示意图（来源：Shinn & Labash，2023）\n启发式函数确定轨迹何时低效或包含幻觉并应停止。低效规划指的是花费过长时间而没有成功的轨迹。幻觉被定义为遇到一系列连续相同的动作，导致环境中出现相同观察的情况。\n通过向 LLM 展示两个示例，每个示例是一对（失败的轨迹，用于指导未来计划变更的理想反思）。然后将反思添加到代理的工作记忆中，最多三个，用作查询 LLM 的上下文。', metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/', 'pk': 449757808726900739}),
```
文档（页面内容='图1. 基于LLM的自主代理系统概览。\n组件一：规划#\n一个复杂的任务通常涉及许多步骤。代理需要知道这些步骤并提前规划。\n任务分解#\n思维链（CoT; Wei等人，2022）已成为增强模型在复杂任务上性能的标准提示技术。模型被指示“逐步思考”，利用更多的测试时间计算将困难任务分解为更小、更简单的步骤。思维链将大任务转化为多个可管理的任务，并揭示模型思考过程的解释。\n思维树（Yao等人，2023）通过探索每个步骤的多个推理可能性扩展了思维链。它首先将问题分解为多个思考步骤，并在每个步骤生成多个思考，形成树状结构。搜索过程可以是BFS（广度优先搜索）或DFS（深度优先搜索），每个状态由分类器（通过提示）或多数投票评估。\n任务分解可以通过以下方式进行：（1）LLM使用简单提示，如“XYZ的步骤。\\n1。”，“实现XYZ的子目标是什么？”，（2）使用特定于任务的说明；例如，“写一个故事大纲。”用于写小说，或（3）通过人类输入。\n另一种非常不同的方法，LLM+P（Liu等人，2023），涉及依赖外部经典规划器进行长期规划。该方法利用规划领域定义语言（PDDL）作为描述规划问题的中间接口。在这个过程中，LLM（1）将问题转化为“问题PDDL”，然后（2）请求经典规划器基于现有的“领域PDDL”生成PDDL计划，最后（3）将PDDL计划转化回自然语言。基本上，规划步骤被外部工具外包，假设领域特定的PDDL和适当的规划器是可用的，这在某些机器人设置中很常见，但在许多其他领域并非如此。\n自我反思#'，元数据={'来源': 'https://lilianweng.github.io/posts/2023-06-23-agent/', '主键': 449757808726900737}））
```
```python
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI

# 初始化 OpenAI 语言模型以生成响应
llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0)

# 定义用于生成 AI 响应的提示模板
PROMPT_TEMPLATE = """
Human: You are an AI assistant, and provides answers to questions by using fact based and statistical information when possible.
Use the following pieces of information to provide a concise answer to the question enclosed in <question> tags.
If you don't know the answer, just say that you don't know, don't try to make up an answer.
<context>
{context}
</context>

<question>
{question}
</question>

The response should be specific and use statistics or numbers when possible.

Assistant:"""

# 使用定义的模板和输入变量创建 PromptTemplate 实例
prompt = PromptTemplate(
    template=PROMPT_TEMPLATE, input_variables=["context", "question"]
)
# 将向量存储转换为检索器
retriever = vectorstore.as_retriever()


# 定义一个函数来格式化检索到的文档
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)
```
使用 LCEL（LangChain 表达语言）构建一个 RAG 链。

```python
# 为 AI 响应生成定义 RAG（检索增强生成）链
rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# rag_chain.get_graph().print_ascii()

# 使用特定问题调用 RAG 链并检索响应
res = rag_chain.invoke(query)
res
```




    "AI 代理的自我反思涉及将记忆合成为随时间推移指导代理未来行为的更高级推断的过程。它包括最近性、重要性和相关性因素，以确定显著的高级问题并优化可信度。这种机制通过改进过去的行动决策和纠正先前的错误帮助代理者迭代地改进。"

本教程构建了一个由 Milvus 提供支持的基本 RAG 链。有关更高级的 RAG 技术，请参考[高级 rag 训练营](https://github.com/milvus-io/bootcamp/tree/master/bootcamp/RAG/advanced_rag)。