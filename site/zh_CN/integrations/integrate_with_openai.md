---
id: integrate_with_openai.md
title: 使用 Milvus 和 OpenAI 进行语义搜索
summary: 本页面讨论了如何将向量数据库与 OpenAI 的嵌入式 API 进行集成。
---

# 使用 Milvus 和 OpenAI 进行语义搜索

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/semantic_search_with_milvus_and_openai.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

本指南展示了如何将 [OpenAI 的嵌入式 API](https://platform.openai.com/docs/guides/embeddings) 与 Milvus 向量数据库结合使用，进行文本的语义搜索。

## 入门指南
在开始之前，请确保您已准备好 OpenAI API 密钥，或者您可以从 [OpenAI 网站](https://openai.com/index/openai-api/) 获取一个。

本示例中使用的数据是书名。您可以从 [这里](https://www.kaggle.com/datasets/jealousleopard/goodreadsbooks) 下载数据集，并将其放在运行以下代码的同一目录中。

首先，安装 Milvus 和 OpenAI 的包：

```shell
pip install --upgrade openai pymilvus
```

<div class="alert note">

如果您正在使用 Google Colab，在启用刚安装的依赖项后，您可能需要**重新启动运行时**。

</div>

有了这些，我们就可以生成嵌入并使用向量数据库进行语义搜索了。

## 使用 OpenAI 和 Milvus 搜索书名

在以下示例中，我们从下载的 CSV 文件中加载书名数据，使用 OpenAI 嵌入模型生成向量表示，并将它们存储在 Milvus 向量数据库中，以进行语义搜索。
```python
从 openai 模块导入 OpenAI
从 pymilvus 模块导入 MilvusClient

MODEL_NAME = "text-embedding-3-small"  # 要使用的模型，请查看 https://platform.openai.com/docs/guides/embeddings 获取可用模型
DIMENSION = 1536  # 向量嵌入的维度

# 使用 API 密钥连接到 OpenAI
openai_client = OpenAI(api_key="<YOUR_OPENAI_API_KEY>")

docs = [
    "人工智能作为一门学科成立于1956年。",
    "艾伦·图灵是第一个进行大量人工智能研究的人。",
    "图灵出生于伦敦梅达维尔，在英格兰南部长大。"
]

vectors = [
    vec.embedding
    for vec in openai_client.embeddings.create(input=docs, model=MODEL_NAME).data
]

# 准备要存储在 Milvus 向量数据库中的数据。
# 我们可以在 Milvus 中存储 id、向量表示、原始文本和标签，比如在这种情况下是"history"。
data = [
    {"id": i, "vector": vectors[i], "text": docs[i], "subject": "history"}
    for i in range(len(docs))
]

# 连接到 Milvus，所有数据存储在名为 "milvus_openai_demo.db" 的本地文件中
# 在当前目录。您也可以按照以下说明连接到远程 Milvus 服务器：https://milvus.io/docs/install_standalone-docker.md。
milvus_client = MilvusClient("milvus_openai_demo.db")
COLLECTION_NAME = "demo_collection"  # Milvus 集合名称
# 创建一个集合来存储向量和文本。
milvus_client.create_collection(collection_name=COLLECTION_NAME, dimension=DIMENSION)

# 将所有数据插入 Milvus 向量数据库。
res = milvus_client.insert(collection_name="demo_collection", data=data)

print(res["insert_count"])
```
在 Milvus 向量数据库中的所有数据后，我们现在可以通过为查询生成向量嵌入并进行向量搜索来执行语义搜索。

```python
queries = ["人工智能是在什么时候创建的？"]

query_vectors = [
    vec.embedding
    for vec in openai_client.embeddings.create(input=queries, model=MODEL_NAME).data
]

res = milvus_client.search(
    collection_name=COLLECTION_NAME,  # 目标集合
    data=query_vectors,  # 查询向量
    limit=2,  # 返回实体的数量
    output_fields=["text", "subject"],  # 指定要返回的字段
)

for q in queries:
    print("查询:", q)
    for result in res:
        print(result)
    print("\n")
```

你应该会看到以下内容作为输出：

```python
[
    {
        "id": 0,
        "distance": -0.772376537322998,
        "entity": {
            "text": "人工智能作为一门学科是在1956年创建的。",
            "subject": "历史",
        },
    },
    {
        "id": 1,
        "distance": -0.58596271276474,
        "entity": {
            "text": "艾伦·图灵是第一个进行人工智能重要研究的人。",
            "subject": "历史",
        },
    },
]
```