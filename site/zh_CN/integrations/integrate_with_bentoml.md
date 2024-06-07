---
id: integrate_with_bentoml.md
summary: 本指南演示如何在 BentoCloud 上使用开源嵌入模型和大型语言模型与 Milvus 向量数据库构建一个检索增强生成（RAG）应用程序。
title: 使用 Milvus 和 BentoML 进行检索增强生成（RAG）
---

# 使用 Milvus 和 BentoML 进行检索增强生成（RAG）

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/rag_with_milvus_and_bentoml.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>


本指南演示了如何在 BentoCloud 上使用开源嵌入模型和大型语言模型与 Milvus 向量数据库构建一个检索增强生成（RAG）应用程序。

BentoCloud 是一个为快速发展的 AI 团队提供服务的 AI 推理平台，为模型推理提供定制的全托管基础设施。它与 BentoML 配合使用，后者是一个开源的模型服务框架，可促进高性能模型服务的轻松创建和部署。在本演示中，我们使用 Milvus Lite 作为向量数据库，这是 Milvus 的轻量级版本，可以嵌入到您的 Python 应用程序中。

## 开始之前
Milvus Lite 可以通过 PyPI 获得。您可以通过 pip 安装它，适用于 Python 3.7+：

```shell
$ pip install -U pymilvus
```

<div class="alert note">

如果您正在使用 Google Colab，在启用刚安装的依赖项后，您可能需要**重新启动运行时**。

</div>

登录 BentoCloud 后，我们可以在“部署”中与已部署的 BentoCloud 服务进行交互，相关的 END_POINT 和 API 位于 Playground -> Python。

您可以从[这里](https://github.com/ytang07/bento_octo_milvus_RAG/tree/main/data)下载城市数据。

## 使用 BentoML/BentoCloud 提供嵌入

要使用此端点，导入 `bentoml` 并使用 `SyncHTTPClient` 设置一个 HTTP 客户端，指定端点并可选择令牌（如果您在 BentoCloud 上启用了`端点授权`）。或者，您可以使用通过 BentoML 提供的相同模型，使用其[Sentence Transformers Embeddings](https://github.com/bentoml/BentoSentenceTransformers)存储库。

```python
import bentoml

BENTO_EMBEDDING_MODEL_END_POINT = "BENTO_EMBEDDING_MODEL_END_POINT"
BENTO_API_TOKEN = "BENTO_API_TOKEN"

embedding_client = bentoml.SyncHTTPClient(
    BENTO_EMBEDDING_MODEL_END_POINT, token=BENTO_API_TOKEN
)
```

连接到 embedding_client 后，我们需要处理我们的数据。我们提供了几个函数来执行数据拆分和嵌入。

读取文件并将文本预处理为字符串列表。

```python
# 在换行符上进行简单分块
def chunk_text(filename: str) -> list:
    with open(filename, "r") as f:
        text = f.read()
    sentences = text.split("\n")
    return sentences
```

首先，我们需要下载城市数据。
```python
import os
import requests
import urllib.request

# 设置数据源
repo = "ytang07/bento_octo_milvus_RAG"
directory = "data"
save_dir = "./city_data"
api_url = f"https://api.github.com/repos/{repo}/contents/{directory}"

response = requests.get(api_url)
data = response.json()

if not os.path.exists(save_dir):
    os.makedirs(save_dir)

for item in data:
    if item["type"] == "file":
        file_url = item["download_url"]
        file_path = os.path.join(save_dir, item["name"])
        urllib.request.urlretrieve(file_url, file_path)
```
接下来，我们处理我们拥有的每个文件。

```python
# 请在此文件夹下上传您的数据目录
cities = os.listdir("city_data")
# 在字典列表中存储每个城市的分块文本
city_chunks = []
for city in cities:
    chunked = chunk_text(f"city_data/{city}")
    cleaned = []
    for chunk in chunked:
        if len(chunk) > 7:
            cleaned.append(chunk)
    mapped = {"city_name": city.split(".")[0], "chunks": cleaned}
    city_chunks.append(mapped)
```

将字符串列表拆分为嵌入列表，每个包含 25 个文本字符串。

```python
def get_embeddings(texts: list) -> list:
    if len(texts) > 25:
        splits = [texts[x : x + 25] for x in range(0, len(texts), 25)]
        embeddings = []
        for split in splits:
            embedding_split = embedding_client.encode(sentences=split)
            embeddings += embedding_split
        return embeddings
    return embedding_client.encode(
        sentences=texts,
    )
```

现在，我们需要将嵌入和文本块匹配起来。由于嵌入列表和句子列表应该按索引匹配，我们可以通过枚举其中一个列表来将它们匹配起来。

```python
entries = []
for city_dict in city_chunks:
    # 如果 get_embeddings 已经返回一个列表的列表，则不需要嵌入列表
    embedding_list = get_embeddings(city_dict["chunks"])  # 返回一个列表的列表
    # 现在将文本与嵌入和城市名称匹配起来
    for i, embedding in enumerate(embedding_list):
        entry = {
            "embedding": embedding,
            "sentence": city_dict["chunks"][
                i
            ],  # 假设 "chunks" 包含嵌入的相应文本
            "city": city_dict["city_name"],
        }
        entries.append(entry)
    print(entries)
```

## 将数据插入向量数据库以供检索

准备好我们的嵌入和数据后，我们可以将向量连同元数据一起插入 Milvus Lite，以便以后进行向量搜索。本节的第一步是通过连接到 Milvus Lite 来启动客户端。

我们只需导入 `MilvusClient` 模块并初始化一个连接到您的 Milvus Lite 向量数据库的 Milvus Lite 客户端。维度大小取决于嵌入模型的大小，例如 Sentence Transformer 模型 `all-MiniLM-L6-v2` 生成 384 维向量。

```python
from pymilvus import MilvusClient

COLLECTION_NAME = "Bento_Milvus_RAG"  # 为您的集合取一个随机名称
DIMENSION = 384

# 初始化 Milvus Lite 客户端
milvus_client = MilvusClient("milvus_demo.db")
```

或者使用旧的 connections.connect API（不推荐）：

```python
from pymilvus import connections

connections.connect(uri="milvus_demo.db")
```

## 创建您的 Milvus Lite 集合
使用 Milvus Lite 创建集合涉及两个步骤：首先是定义模式，其次是定义索引。在这一部分，我们需要一个模块：DataType 告诉我们字段中将包含什么类型的数据。我们还需要使用两个函数来创建模式并添加字段。create_schema(): 创建集合模式，add_field(): 向集合的模式中添加字段。

```python
from pymilvus import MilvusClient, DataType, Collection

# 创建模式
schema = MilvusClient.create_schema(
    auto_id=True,
    enable_dynamic_field=True,
)

# 添加字段到模式
schema.add_field(field_name="id", datatype=DataType.INT64, is_primary=True)
schema.add_field(field_name="embedding", datatype=DataType.FLOAT_VECTOR, dim=DIMENSION)
```

现在我们已经创建了模式并成功定义了数据字段，我们需要定义索引。在搜索方面，“索引”定义了我们将如何映射数据以进行检索。我们使用默认选择 [AUTOINDEX](https://docs.zilliz.com/docs/autoindex-explained) 来为这个项目中的数据建立索引。

接下来，我们使用先前给定的名称、模式和索引创建集合。最后，我们插入先前处理过的数据。

```python
# 准备索引参数
index_params = milvus_client.prepare_index_params()

# 添加索引
index_params.add_index(
    field_name="embedding",
    index_type="AUTOINDEX",  # 使用 autoindex 而不是其他复杂的索引方法
    metric_type="COSINE",  # L2, COSINE, 或 IP
)

# 创建集合
milvus_client.create_collection(
    collection_name=COLLECTION_NAME, schema=schema, index_params=index_params
)

# 在循环之外，现在一次性更新所有条目
milvus_client.insert(collection_name=COLLECTION_NAME, data=entries)
```

## 为 RAG 设置您的 LLM

要构建一个 RAG 应用程序，我们需要在 BentoCloud 上部署一个 LLM。让我们使用最新的 Llama3 LLM。一旦它运行起来，只需复制该模型服务的端点和令牌，并为其设置一个客户端。

```python
BENTO_LLM_END_POINT = "BENTO_LLM_END_POINT"

llm_client = bentoml.SyncHTTPClient(BENTO_LLM_END_POINT, token=BENTO_API_TOKEN)
```

## LLM 指令

现在，我们使用提示、上下文和问题设置 LLM 指令。以下是一个行为类似 LLM 的函数，它然后以字符串格式从客户端返回输出。

```python
def dorag(question: str, context: str):

    prompt = (
        f"You are a helpful assistant. The user has a question. Answer the user question based only on the context: {context}. \n"
        f"The user question is {question}"
    )

    results = llm_client.generate(
        max_tokens=1024,
        prompt=prompt,
    )

    res = ""
    for result in results:
        res += result

    return res
```

## 一个 RAG 示例
现在我们准备提出一个问题。这个函数简单地接受一个问题，然后执行 RAG 以从背景信息中生成相关上下文。然后，我们将上下文和问题传递给 dorag() 并获取结果。

```python
question = "剑桥在哪个州？"

def ask_a_question(question):
    embeddings = get_embeddings([question])
    res = milvus_client.search(
        collection_name=COLLECTION_NAME,
        data=embeddings,  # 搜索返回的一个 (1) 嵌入作为列表的列表
        anns_field="embedding",  # 在嵌入之间进行搜索
        limit=5,  # 获取前 5 个结果
        output_fields=["sentence"],  # 获取句子/块和城市
    )

    sentences = []
    for hits in res:
        for hit in hits:
            print(hit)
            sentences.append(hit["entity"]["sentence"])
    context = "。".join(sentences)
    return context

context = ask_a_question(question=question)
print(context)
```

实现 RAG

```python
print(dorag(question=question, context=context))
```

对于询问剑桥位于哪个州的示例问题，我们可以打印出来自 BentoML 的完整响应。然而，如果我们花时间解析它，看起来更好，它应该告诉我们剑桥位于马萨诸塞州。