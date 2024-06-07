---
id: integrate_with_memgpt.md
summary: MemGPT使构建和部署有状态的LLM代理变得简单。通过Milvus集成，您可以构建与外部数据源（RAG）连接的代理。
title: MemGPT与Milvus集成
---

# MemGPT与Milvus集成

[MemGPT](https://memgpt.readme.io/docs/index)使构建和部署有状态的LLM代理变得简单。通过Milvus集成，您可以构建与外部数据源（RAG）连接的代理。

在这个示例中，我们将使用MemGPT与存储在Milvus中的自定义数据源进行对话。

## 配置

要启用Milvus后端，请确保安装所需的依赖项：

```shell
$ pip install 'pymemgpt[milvus]'
```

您可以通过以下命令配置Milvus连接：

```shell
$ memgpt configure
```

```shell
...
? 选择用于存档数据的存储后端: milvus
? 输入Milvus连接URI（默认：~/.memgpt/milvus.db）: ~/.memgpt/milvus.db
```

您只需将URI设置为本地文件路径，例如 `~/.memgpt/milvus.db`，这将通过Milvus Lite自动调用本地Milvus服务实例。

如果您有大规模的数据，例如超过一百万个文档，我们建议在[docker或kubenetes](https://milvus.io/docs/quickstart.md)上设置一个性能更高的Milvus服务器。
在这种情况下，您的URI应该是服务器URI，例如 `http://localhost:19530`。

## 创建外部数据源

要将外部数据输入到MemGPT聊天机器人中，我们首先需要创建一个数据源。

要下载MemGPT研究论文，我们将使用`curl`（您也可以直接从浏览器下载PDF）：

```shell
# 我们将文件保存为"memgpt_research_paper.pdf"
$ curl -L -o memgpt_research_paper.pdf https://arxiv.org/pdf/2310.08560.pdf
```

现在我们已经下载了论文，我们可以使用`memgpt load`创建一个MemGPT数据源：

```shell
$ memgpt load directory --name memgpt_research_paper --input-files=memgpt_research_paper.pdf
```

```text
Loading files: 100%|███████████████████████████████████| 1/1 [00:00<00:00,  3.94file/s]
从memgpt_research_paper加载了74个段落和13个文档
```

## 将数据源附加到MemGPT代理

现在我们已经创建了这个数据源，我们随时可以将其附加到MemGPT聊天机器人上。

让我们使用`memgpt_doc`人设创建一个新的聊天机器人（但您可以使用任何您想要的人设）：

```shell
# 提醒：`memgpt run --persona memgpt_doc`将使用`memgpt_doc`人设创建一个新的MemGPT代理
$ memgpt run --persona memgpt_doc
```

一旦我们与代理对话，我们可以将数据源“附加”到代理的存档内存中：
```text
? 你想选择现有的代理吗？不

🧬 创建新代理...
->  🤖 使用人设配置文件：'sam_pov'
->  🧑 使用人类配置文件：'basic'
🎉 创建了新代理 'PoliteButterfly'（id=d26e1981-ff36-4095-97a0-61a1601dfb5d）

按 Enter 键开始（将请求第一条 MemGPT 消息）

💭 有趣，我遇到了一个首次用户。是时候介绍自己并了解用户的需求了。我想知道今天 Chad 来这里是为了什么。
🤖 你好，Chad！我是 MemGPT。我今天可以如何帮助你？

> 输入您的消息：/attach
? 选择数据源 memgpt_research_paper
100%|███████████████████████████████████| 1/1 [00:00<00:00,  4.81it/s]
```
## 测试我们的新聊天机器人

现在数据已经加载到聊天机器人的内存中，我们可以开始询问相关问题：

```text
> 输入您的消息：我加载到您的档案内存中的论文名为 MemGPT。你能告诉我更多关于它的信息吗？

💭 看起来 Chad 想了解更多关于存储在我的档案内存中名为 'MemGPT' 的论文。现在是时候在档案内存中搜索详细信息和背景数据，以提供对该论文的全面理解了。
⚡🧠 [function] 使用档案内存搜索进行搜索
💭 根据找到的结果，MemGPT 是一个专为执行扩展对话和文档分析等任务而设计的人工智能系统。它有效地管理不同层次的存储，以在大型语言模型（LLM）的有限容量内提供扩展上下文。另外，根据我的研究发现，MemGPT 在深度记忆检索（DMR）和文档问答任务中表现出色。让我们为 Chad 准备一个全面的回答。
🤖 当然，Chad。MemGPT，或称 MemoryGPT，是一个专为执行扩展对话和文档分析等任务而设计的系统，这些任务传统上受到现代大型语言模型（LLMs）有限上下文窗口的限制。它采用了受传统操作系统中分层内存系统启发的技术，称为虚拟上下文管理。这种技术通过智能地管理不同存储层，类似于在物理内存和磁盘之间进行分页，有效地在LLM的有限上下文窗口内提供扩展上下文。值得注意的是，MemGPT在深度记忆检索方面优于固定上下文基线，并且能够精确分析大型文档。
```