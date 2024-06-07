---
id: integrate_with_sentencetransformers.md
summary: 本页面讨论使用 Milvus 进行电影搜索
title: 使用 Milvus 和 SentenceTransformers 进行电影搜索
---

# 使用 Milvus 和 SentenceTransformers 进行电影搜索

在这个示例中，我们将讨论如何使用 Milvus 和 SentenceTransformers 库进行维基百科文章搜索。我们将搜索的数据集是在[Kaggle](https://www.kaggle.com/datasets/jrobischon/wikipedia-movie-plots)上找到的维基百科电影情节数据集。在这个示例中，我们已经将数据重新托管到了一个公共的 Google Drive 上。

让我们开始吧。

## 安装所需软件包

在这个示例中，我们将使用 `pymilvus` 来连接 Milvus，`sentencetransformers` 来生成向量嵌入，以及 `gdown` 来下载示例数据集。

```shell
pip install pymilvus sentence-transformers gdown
```

## 获取数据

我们将使用 `gdown` 从 Google Drive 上获取 zip 文件，然后使用内置的 `zipfile` 库对其进行解压缩。

```python
import gdown
url = 'https://drive.google.com/uc?id=11ISS45aO2ubNCGaC3Lvd3D7NT8Y7MeO8'
output = './movies.zip'
gdown.download(url, output)

import zipfile

with zipfile.ZipFile("./movies.zip","r") as zip_ref:
    zip_ref.extractall("./movies")
```

## 全局参数

在这里，我们可以找到需要修改的主要参数，以便在您自己的账户上运行。每个参数旁边都有一个描述。

```python
# Milvus 设置参数
COLLECTION_NAME = 'movies_db'  # 集合名称
DIMENSION = 384  # 嵌入维度
COUNT = 1000  # 要插入的向量数量
MILVUS_HOST = 'localhost'
MILVUS_PORT = '19530'

# 推理参数
BATCH_SIZE = 128

# 搜索参数
TOP_K = 3
```

## 设置 Milvus

此时，我们将开始设置 Milvus。步骤如下：

1. 使用提供的 URI 连接到 Milvus 实例。

    ```python
    from pymilvus import connections

    # 连接到 Milvus 数据库
    connections.connect(host=MILVUS_HOST, port=MILVUS_PORT)
    ```

2. 如果集合已经存在，则删除它。

    ```python
    from pymilvus import utility

    # 删除任何具有相同名称的先前集合
    if utility.has_collection(COLLECTION_NAME):
        utility.drop_collection(COLLECTION_NAME)
    ```

3. 创建包含 id、电影标题和情节文本嵌入的集合。

    ```python
    from pymilvus import FieldSchema, CollectionSchema, DataType, Collection


    # 创建包含 id、标题和嵌入的集合。
    fields = [
        FieldSchema(name='id', dtype=DataType.INT64, is_primary=True, auto_id=True),
        FieldSchema(name='title', dtype=DataType.VARCHAR, max_length=200),  # VARCHAR 需要一个最大长度，在这个示例中设置为 200 个字符
        FieldSchema(name='embedding', dtype=DataType.FLOAT_VECTOR, dim=DIMENSION)
    ]
    schema = CollectionSchema(fields=fields)
```python
collection = Collection(name=COLLECTION_NAME, schema=schema)
```

4. 在新创建的集合上创建索引并将其加载到内存中。

```python
# 为集合创建 IVF_FLAT 索引。
index_params = {
    'metric_type':'L2',
    'index_type':"IVF_FLAT",
    'params':{'nlist': 1536}
}
collection.create_index(field_name="embedding", index_params=index_params)
collection.load()
```

完成这些步骤后，集合就准备好进行插入和搜索了。任何添加的数据都将自动建立索引，并可以立即进行搜索。如果数据非常新，搜索可能会较慢，因为在数据仍在索引过程中时将使用暴力搜索。

## 插入数据

在本示例中，我们将使用 SentenceTransformers 的 miniLM 模型来创建情节文本的嵌入。该模型返回 384 维的嵌入。

在接下来的几个步骤中，我们将：

1. 加载数据。
2. 使用 SentenceTransformers 对情节文本数据进行嵌入。
3. 将数据插入到 Milvus 中。

```python
import csv
from sentence_transformers import SentenceTransformer

transformer = SentenceTransformer('all-MiniLM-L6-v2')

# 提取书名
def csv_load(file):
    with open(file, newline='') as f:
        reader = csv.reader(f, delimiter=',')
        for row in reader:
            if '' in (row[1], row[7]):
                continue
            yield (row[1], row[7])


# 使用 OpenAI 对文本进行嵌入
def embed_insert(data):
    embeds = transformer.encode(data[1]) 
    ins = [
            data[0],
            [x for x in embeds]
    ]
    collection.insert(ins)

import time

data_batch = [[],[]]

count = 0

for title, plot in csv_load('./movies/plots.csv'):
    if count <= COUNT:
        data_batch[0].append(title)
        data_batch[1].append(plot)
        if len(data_batch[0]) % BATCH_SIZE == 0:
            embed_insert(data_batch)
            data_batch = [[],[]]
        count += 1
    else:
        break

# 嵌入并插入剩余数据
if len(data_batch[0]) != 0:
    embed_insert(data_batch)

# 调用 flush 方法以索引任何未封装的段。
collection.flush()
```

<div class="alert note">

上述操作相对耗时，因为嵌入需要时间。为了保持时间消耗在可接受范围内，请尝试在[全局参数](#Global-parameters)中将 `COUNT` 设置为适当的值。休息一下，喝杯咖啡吧！

</div>

## 执行搜索

将所有数据插入到 Milvus 后，我们可以开始执行搜索。在本示例中，我们将根据情节搜索电影。由于我们进行的是批量搜索，因此搜索时间将在所有电影搜索之间共享。
```python
# 寻找与以下短语最接近的标题。
search_terms = ['A movie about cars', 'A movie about monsters']

# 根据输入文本搜索数据库
def embed_search(data):
    embeds = transformer.encode(data) 
    return [x for x in embeds]

search_data = embed_search(search_terms)

start = time.time()
res = collection.search(
    data=search_data,  # 嵌入式搜索数值
    anns_field="embedding",  # 在嵌入式中搜索
    param={},
    limit = TOP_K,  # 搜索结果限制为top_k个
    output_fields=['title']  # 在结果中包含标题字段
)
end = time.time()

for hits_i, hits in enumerate(res):
    print('Title:', search_terms[hits_i])
    print('Search Time:', end-start)
    print('Results:')
    for hit in hits:
        print( hit.entity.get('title'), '----', hit.distance)
    print()
```
```shell
标题: 一部关于汽车的电影
搜索时间: 0.08636689186096191
结果:
青春的魅力 ---- 1.0954499244689941
从利德维尔到阿斯彭：落基山的劫案 ---- 1.1019384860992432
胆大包天 ---- 1.1331942081451416

标题: 一部关于怪物的电影
搜索时间: 0.08636689186096191
结果:
郊区居民 ---- 1.0666425228118896
青春的魅力 ---- 1.1072258949279785
无神论女孩 ---- 1.1511223316192627
```