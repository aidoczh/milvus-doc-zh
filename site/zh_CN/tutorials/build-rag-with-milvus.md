---
id: build-rag-with-milvus.md
summary: 使用 Milvus 构建 RAG
title: 使用 Milvus 构建 RAG
---

# 使用 Milvus 构建 RAG

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/quickstart/build_RAG_with_milvus.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

在本教程中，我们将展示如何使用 Milvus 构建一个 RAG（检索增强生成）流水线。

RAG 系统将检索系统与生成模型结合起来，根据给定的提示生成新文本。系统首先使用类似 Milvus 的向量相似度搜索引擎从语料库中检索相关文档，然后使用生成模型根据检索到的文档生成新文本。

## 准备工作
### 依赖和环境

```python
$ pip install --upgrade pymilvus openai requests tqdm
```

<div class="alert note">

如果您正在使用 Google Colab，为了启用刚安装的依赖项，您可能需要**重新启动运行时**。

</div>

在本示例中，我们将使用 OpenAI 作为 LLM。您应该准备好[api key](https://platform.openai.com/docs/quickstart) `OPENAI_API_KEY` 作为环境变量。

```python
import os

os.environ["OPENAI_API_KEY"] = "sk-***********"
```

### 准备数据

我们使用[Milvus 开发指南](https://github.com/milvus-io/milvus/blob/master/DEVELOPMENT.md)作为我们的 RAG 中的私有知识，这是一个简单 RAG 流水线的良好数据源。

下载并将其保存为本地文本文件。

```python
import json
import urllib.request

url = "https://raw.githubusercontent.com/milvus-io/milvus/master/DEVELOPMENT.md"
file_path = "./Milvus_DEVELOPMENT.md"

if not os.path.exists(file_path):
    urllib.request.urlretrieve(url, file_path)
```

我们简单地使用“# ”来分隔文件中的内容，这样可以粗略地将 markdown 文件的每个主要部分的内容分开。

```python
with open(file_path, "r") as file:
    file_text = file.read()

text_lines = file_text.split("# ")
```

### 准备嵌入模型

我们初始化 OpenAI 客户端以准备嵌入模型。

```python
from openai import OpenAI

openai_client = OpenAI()
```

定义一个使用 OpenAI 客户端生成文本嵌入的函数。我们以[text-embedding-3-small](https://platform.openai.com/docs/guides/embeddings)模型为例。

```python
def emb_text(text):
    return (
        openai_client.embeddings.create(input=text, model="text-embedding-3-small")
        .data[0]
        .embedding
    )
```

生成一个测试嵌入并打印其维度和前几个元素。

```python
test_embedding = emb_text("This is a test")
embedding_dim = len(test_embedding)
print(embedding_dim)
print(test_embedding[:10])
```

    1536
## 在 Milvus 中加载数据

### 创建集合

```python
from pymilvus import MilvusClient

milvus_client = MilvusClient("./milvus_demo.db")

collection_name = "my_rag_collection"
```

检查集合是否已存在，如果存在则删除。

```python
if milvus_client.has_collection(collection_name):
    milvus_client.drop_collection(collection_name)
```

使用指定参数创建一个新的集合。

如果我们没有指定任何字段信息，Milvus 将自动为主键创建一个默认的 `id` 字段，以及一个 `vector` 字段来存储向量数据。一个保留的 JSON 字段用于存储非模式定义字段及其值。

```python
milvus_client.create_collection(
    collection_name=collection_name,
    dimension=embedding_dim,
    metric_type="IP",  # 内积距离
    consistency_level="Strong",  # 强一致性级别
)
```

### 插入数据

遍历文本行，创建嵌入向量，然后将数据插入 Milvus。

这里有一个新字段 `text`，它是集合模式中未定义的字段。它将自动添加到保留的 JSON 动态字段中，可以在高级别上将其视为正常字段。

```python
from tqdm import tqdm

data = []

for i, line in enumerate(tqdm(text_lines, desc="Creating embeddings")):
    data.append({"id": i, "vector": emb_text(line), "text": line})

milvus_client.insert(collection_name=collection_name, data=data)
```

    Creating embeddings: 100%|█| 47/47 [00:16<00:00,  





    {'insert_count': 47,
     'ids': [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46],
     'cost': 0}



## 构建 RAG

### 检索查询数据

让我们定义一个关于开发指南文档内容的查询问题。

```python
question = "如果我想构建 Milvus 并从源代码运行，硬件要求规格是什么？"
```

在集合中搜索问题并检索语义前三匹配项。

```python
search_res = milvus_client.search(
    collection_name=collection_name,
    data=[
        emb_text(question)
    ],  # 使用 `emb_text` 函数将问题转换为嵌入向量
    limit=3,  # 返回前 3 个结果
    search_params={"metric_type": "IP", "params": {}},  # 内积距离
    output_fields=["text"],  # 返回文本字段
)
```

让我们看一下查询的搜索结果

```python
retrieved_lines_with_distances = [
    (res["entity"]["text"], res["distance"]) for res in search_res[0]
]
print(json.dumps(retrieved_lines_with_distances, indent=4))
```

    [
        [
### 使用LLM获取一个RAG响应

将检索到的文档转换为字符串格式。

```python
context = "\n".join(
    [line_with_distance[0] for line_with_distance in retrieved_lines_with_distances]
)
```

为语言模型定义系统提示和用户提示。这个提示是由Milvus中检索到的文档组装而成。

```python
SYSTEM_PROMPT = """
Human: 你是一个AI助手。你可以从提供的上下文段落中找到问题的答案。
"""
USER_PROMPT = f"""
使用以下信息片段（用<context>标记）来回答<question>标记中的问题。
<context>
{context}
</context>
<question>
{question}
</question>
"""
```

使用OpenAI ChatGPT根据提示生成响应。

```python
response = openai_client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": USER_PROMPT},
    ],
)
print(response.choices[0].message.content)
```

Milvus构建和运行的硬件要求规格如下：

- 8GB的RAM
- 50GB的可用磁盘空间