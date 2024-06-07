---
id: integrate_with_voyageai.md
title: 使用 Milvus 和 VoyageAI 进行语义搜索
summary: 本页面讨论了如何将向量数据库与 VoyageAI 的嵌入式 API 进行集成。
---

# 使用 Milvus 和 VoyageAI 进行语义搜索

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/semantic_search_with_milvus_and_voyageai.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

本指南展示了如何使用 [VoyageAI 的嵌入式 API](https://docs.voyageai.com/docs/embeddings) 与 Milvus 向量数据库一起进行文本的语义搜索。

## 入门指南
在开始之前，请确保您已准备好 Voyage API 密钥，或者您可以从 [VoyageAI 网站](https://dash.voyageai.com/api-keys) 获取一个。

本示例中使用的数据是书名。您可以从 [这里](https://www.kaggle.com/datasets/jealousleopard/goodreadsbooks) 下载数据集，并将其放在运行以下代码的同一目录中。

首先，安装 Milvus 和 Voyage AI 的包：

```python
$ pip install --upgrade voyageai pymilvus
```

<div class="alert note">

如果您正在使用 Google Colab，在启用刚安装的依赖项后，您可能需要**重新启动运行时**。

</div>

有了这些，我们就可以生成嵌入并使用向量数据库进行语义搜索了。

## 使用 VoyageAI & Milvus 搜索书名

在以下示例中，我们从下载的 CSV 文件中加载书名数据，使用 Voyage AI 嵌入模型生成向量表示，并将它们存储在 Milvus 向量数据库中以进行语义搜索。
```python
import voyageai
from pymilvus import MilvusClient

MODEL_NAME = "voyage-law-2"  # 选择要使用的模型，请查看 https://docs.voyageai.com/docs/embeddings 以获取可用模型
DIMENSION = 1024  # 向量嵌入的维度

# 使用 API 密钥连接到 VoyageAI
voyage_client = voyageai.Client(api_key="<YOUR_VOYAGEAI_API_KEY>")

docs = [
    "人工智能作为一门学科成立于1956年。",
    "艾伦·图灵是第一个进行大量人工智能研究的人。",
    "图灵出生在伦敦的梅达维尔，成长在英格兰南部。"
]

vectors = voyage_client.embed(texts=docs, model=MODEL_NAME, truncation=False).embeddings

# 准备要存储在 Milvus 向量数据库中的数据。
# 我们可以在 Milvus 中存储 id、向量表示、原始文本以及标签，比如在这种情况下是"history"。
data = [
    {"id": i, "vector": vectors[i], "text": docs[i], "subject": "history"}
    for i in range(len(docs))
]

# 连接到 Milvus，所有数据存储在名为 "milvus_voyage_demo.db" 的本地文件中
# 在当前目录。您也可以按照以下说明连接到远程 Milvus 服务器：https://milvus.io/docs/install_standalone-docker.md。
milvus_client = MilvusClient("milvus_voyage_demo.db")
COLLECTION_NAME = "demo_collection"  # Milvus 集合名称
# 创建一个集合来存储向量和文本。
milvus_client.create_collection(collection_name=COLLECTION_NAME, dimension=DIMENSION)

# 将所有数据插入到 Milvus 向量数据库中。
res = milvus_client.insert(collection_name="demo_collection", data=data)

print(res["insert_count"])
```
在 Milvus 向量数据库中的所有数据，我们现在可以通过为查询生成向量嵌入并进行向量搜索来执行语义搜索。

```python
queries = ["人工智能是什么时候创建的？"]

query_vectors = voyage_client.embed(
    texts=queries, model=MODEL_NAME, truncation=False
).embeddings

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

    查询: 人工智能是什么时候创建的？
    [{'id': 0, 'distance': 0.7196218371391296, 'entity': {'text': '人工智能作为一门学科成立于1956年。', 'subject': 'history'}}, {'id': 1, 'distance': 0.6297335028648376, 'entity': {'text': '艾伦·图灵是第一个进行人工智能重要研究的人。', 'subject': 'history'}}]
    
    