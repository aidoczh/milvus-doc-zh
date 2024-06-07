---
id: integrate_with_cohere.md
summary: 本页面介绍了如何使用 Milvus 作为向量数据库和 Hugging Face 作为嵌入系统来搜索问题的最佳答案。
title: 使用 Milvus 和 Cohere 进行问答
---

# 使用 Milvus 和 Cohere 进行问答

本页面介绍如何利用 Milvus 作为向量数据库和 Cohere 作为嵌入系统，搜索问题的最佳答案。

## 开始之前

本页面的代码片段需要安装 **pymilvus**、**cohere**、**pandas**、**numpy** 和 **tqdm**。在这些软件包中，**pymilvus** 是 Milvus 的客户端。如果系统中没有这些软件包，请运行以下命令进行安装：

```shell
pip install pymilvus cohere pandas numpy tqdm
```

然后需要加载本指南中要使用的模块。

```python
import cohere
import pandas
import numpy as np
from tqdm import tqdm
from pymilvus import connections, FieldSchema, CollectionSchema, DataType, Collection, utility
```

## 参数

这里列出了以下代码片段中使用的参数。其中一些参数需要更改以适应您的环境。每个参数旁边都有一个描述。

```python
FILE = 'https://rajpurkar.github.io/SQuAD-explorer/dataset/train-v2.0.json'  # SQuAD 数据集的 URL
COLLECTION_NAME = 'question_answering_db'  # 集合名称
DIMENSION = 1024  # 嵌入大小，cohere 的默认嵌入大小为 4096（大模型）
COUNT = 5000  # 要嵌入并插入到 Milvus 中的问题数量
BATCH_SIZE = 96 # 用于嵌入和插入的批处理大小
MILVUS_HOST = 'localhost'  # Milvus 服务器 URI
MILVUS_PORT = '19530'
COHERE_API_KEY = 'replace-this-with-the-cohere-api-key'  # 从 Cohere 获取的 API 密钥
```

要了解本页面使用的模型和数据集的更多信息，请参考 [co:here](https://cohere.ai/) 和 [SQuAD](https://rajpurkar.github.io/SQuAD-explorer/)。

## 准备数据集

在这个示例中，我们将使用斯坦福问答数据集（SQuAD）作为回答问题的真实来源。这个数据集以 JSON 文件的形式存在，我们将使用 **pandas** 来加载它。

```python
# 下载数据集
dataset = pandas.read_json(FILE)

# 通过获取所有问题答案对来清理数据集
simplified_records = []
for x in dataset['data']:
    for y in x['paragraphs']:
        for z in y['qas']:
            if len(z['answers']) != 0:
                simplified_records.append({'question': z['question'], 'answer': z['answers'][0]['text']})

# 根据 COUNT 获取记录数量
simplified_records = pandas.DataFrame.from_records(simplified_records)
simplified_records = simplified_records.sample(n=min(COUNT, len(simplified_records)), random_state = 42)

# 检查清理后的数据集长度是否与 count 匹配
print(len(simplified_records))
```

输出应该是数据集中的记录数

```shell
5000
```

## 创建集合
这一部分涉及Milvus和为此用例设置数据库。在Milvus中，我们需要设置一个集合并对其进行索引。

```python
# 连接到Milvus数据库
connections.connect(host=MILVUS_HOST, port=MILVUS_PORT)

# 如果集合已存在，则删除集合
if utility.has_collection(COLLECTION_NAME):
    utility.drop_collection(COLLECTION_NAME)

# 创建包含id、title和embedding的集合。
fields = [
    FieldSchema(name='id', dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name='original_question', dtype=DataType.VARCHAR, max_length=1000),
    FieldSchema(name='answer', dtype=DataType.VARCHAR, max_length=1000),
    FieldSchema(name='original_question_embedding', dtype=DataType.FLOAT_VECTOR, dim=DIMENSION)
]
schema = CollectionSchema(fields=fields)
collection = Collection(name=COLLECTION_NAME, schema=schema)

# 为集合创建IVF_FLAT索引。
index_params = {
    'metric_type':'IP',
    'index_type':"IVF_FLAT",
    'params':{"nlist": 1024}
}
collection.create_index(field_name="original_question_embedding", index_params=index_params)
collection.load()
```

## 插入数据

一旦我们设置好集合，我们需要开始插入数据。这可以分为三个步骤：

- 读取数据，
- 对原始问题进行嵌入，以及
- 将数据插入我们刚在Milvus上创建的集合中。

在这个例子中，数据包括原始问题、原始问题的嵌入以及原始问题的答案。

```python
# 设置co:here客户端。
cohere_client = cohere.Client(COHERE_API_KEY)

# 使用Cohere从问题中提取嵌入。
def embed(texts, input_type):
    res = cohere_client.embed(texts, model='embed-multilingual-v3.0', input_type=input_type)
    return res.embeddings

# 插入每个问题、答案和问题嵌入。
total = pandas.DataFrame()
for batch in tqdm(np.array_split(simplified_records, (COUNT/BATCH_SIZE) + 1)):
    questions = batch['question'].tolist()
    embeddings = embed(questions, "search_document")
    
    data = [
        {
            'original_question': x,
            'answer': batch['answer'].tolist()[i],
            'original_question_embedding': embeddings[i]
        } for i, x in enumerate(questions)
    ]

    collection.insert(data=data)

time.sleep(10)
```

## 提问

一旦所有数据插入Milvus集合，我们可以通过将问题短语嵌入Cohere并在集合中搜索来向系统提问。

<div class="alert note">

在插入数据后立即进行的搜索可能会有点慢，因为搜索未索引的数据是以暴力方式进行的。一旦新数据自动索引，搜索速度将加快。

</div>
```
```python
# 搜索集群以回答问题文本
def search(text, top_k = 5):

    # AUTOINDEX 不需要任何搜索参数
    search_params = {}

    results = collection.search(
        data = embed([text], "search_query"),  # 嵌入问题
        anns_field='original_question_embedding',
        param=search_params,
        limit = top_k,  # 搜索结果限制为 top_k 个
        output_fields=['original_question', 'answer']  # 结果中包括原始问题和答案
    )

    distances = results[0].distances
    entities = [ x.entity.to_dict()['entity'] for x in results[0] ]

    ret = [ {
        "answer": x[1]["answer"],
        "distance": x[0],
        "original_question": x[1]['original_question']
    } for x in zip(distances, entities)]

    return ret

# 提出以下问题
search_questions = ['什么杀死了细菌？', '世界上最大的狗是什么？']

# 按照 [答案, 相似度分数, 原始问题] 的顺序打印结果

ret = [ { "question": x, "candidates": search(x) } for x in search_questions ]
```
```shell
# 输出
#
# [
#     {
#         "question": "什么杀死了细菌?",
#         "candidates": [
#             {
#                 "answer": "农业",
#                 "distance": 0.6261022090911865,
#                 "original_question": "是什么让细菌对抗生素治疗产生抗药性?"
#             },
#             {
#                 "answer": "噬菌体疗法",
#                 "distance": 0.6093736886978149,
#                 "original_question": "用于治疗耐药细菌的方法有哪些?"
#             },
#             {
#                 "answer": "口服避孕药",
#                 "distance": 0.5902313590049744,
#                 "original_question": "在治疗中，抗菌药物与什么相互作用?"
#             },
#             {
#                 "answer": "减缓细菌的繁殖或杀死细菌",
#                 "distance": 0.5874154567718506,
#                 "original_question": "抗生素是如何起作用的?"
#             },
#             {
#                 "answer": "在密集饲养中促进动物生长",
#                 "distance": 0.5667208433151245,
#                 "original_question": "除了用于治疗人类疾病外，抗生素还在哪些地方使用?"
#             }
#         ]
#     },
#     {
#         "question": "最大的狗是什么?",
#         "candidates": [
#             {
#                 "answer": "英国獒",
#                 "distance": 0.7875324487686157,
#                 "original_question": "已知生活过最大的狗是什么品种?"
#             },
#             {
#                 "answer": "森林大象",
#                 "distance": 0.5886962413787842,
#                 "original_question": "国家公园中居住着哪些大型动物?"
#             },
#             {
#                 "answer": "Rico",
#                 "distance": 0.5634892582893372,
#                 "original_question": "能识别超过200种事物的狗的名字是什么?"
#             },
#             {
#                 "answer": "艾迪塔罗德雪橇犬赛",
#                 "distance": 0.546872615814209,
#                 "original_question": "阿拉斯加哪个雪橇犬赛事最著名?"
#             },
#             {
#                 "answer": "家庭的一部分",
#                 "distance": 0.5387814044952393,
#                 "original_question": "今天大多数人如何描述他们的狗?"
#             }
#         ]
#     }
# ]

```