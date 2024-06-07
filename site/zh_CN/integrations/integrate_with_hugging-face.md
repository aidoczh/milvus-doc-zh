---
id: integrate_with_hugging-face.md
summary: 本教程展示如何使用 Hugging Face 作为数据加载器和嵌入生成器，以进行数据处理，并使用 Milvus 作为语义搜索的向量数据库，构建问答系统。
title: 使用 Milvus 和 Hugging Face 进行问答
---

# 使用 Milvus 和 Hugging Face 进行问答

<a href="https://colab.research.google.com/github/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/qa_with_milvus_and_hf.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

基于语义搜索的问答系统通过在问题-答案对数据集中找到与给定查询问题最相似的问题来工作。一旦确定了最相似的问题，数据集中相应的答案被视为查询的答案。这种方法依赖于语义相似度度量来确定问题之间的相似性，并检索相关答案。

本教程展示了如何使用 [Hugging Face](https://huggingface.co) 作为数据加载器和嵌入生成器，以进行数据处理，并使用 [Milvus](https://milvus.io) 作为语义搜索的向量数据库来构建问答系统。

## 开始之前

您需要确保已安装所有必需的依赖项：

- `pymilvus`：与由 Milvus 或 Zilliz Cloud 提供支持的向量数据库服务一起使用的 Python 包。
- `datasets`、`transformers`：Hugging Face 包，用于管理数据和利用模型。
- `torch`：一个强大的库，提供高效的张量计算和深度学习工具。

```python
$ pip install --upgrade pymilvus transformers datasets torch
```

<div class="alert note">

如果您正在使用 Google Colab，在启用刚安装的依赖项时，您可能需要**重新启动运行时**。

</div>

## 准备数据

在本节中，我们将从 Hugging Face Datasets 中加载示例问题-答案对。作为演示，我们仅从 [SQuAD](https://huggingface.co/datasets/rajpurkar/squad) 的验证集中获取部分数据。

```python
from datasets import load_dataset

DATASET = "squad"  # 来自 HuggingFace Datasets 的数据集名称
INSERT_RATIO = 0.001  # 要插入的示例数据集比例

data = load_dataset(DATASET, split="validation")
# 生成一个固定的子集。要生成一个随机子集，请删除 seed。
data = data.train_test_split(test_size=INSERT_RATIO, seed=42)["test"]
# 清理数据集中的数据结构。
data = data.map(
    lambda val: {"answer": val["answers"]["text"][0]},
    remove_columns=["id", "answers", "context"],
)
```
# 查看示例数据的摘要
```
print(data)
```
```

    Dataset({
        features: ['title', 'question', 'answer'],
        num_rows: 11
    })
```

为了为问题生成嵌入向量，您可以从Hugging Face Models中选择一个文本嵌入模型。在本教程中，我们将使用一个小型的句子嵌入模型 [all-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2) 作为示例。

```python
from transformers import AutoTokenizer, AutoModel
import torch

MODEL = (
    "sentence-transformers/all-MiniLM-L6-v2"  # 模型名称来自 HuggingFace Models
)
INFERENCE_BATCH_SIZE = 64  # 模型推断的批处理大小

# 从 HuggingFace Hub 加载分词器和模型
tokenizer = AutoTokenizer.from_pretrained(MODEL)
model = AutoModel.from_pretrained(MODEL)


def encode_text(batch):
    # 对句子进行分词
    encoded_input = tokenizer(
        batch["question"], padding=True, truncation=True, return_tensors="pt"
    )

    # 计算标记嵌入
    with torch.no_grad():
        model_output = model(**encoded_input)

    # 执行池化
    token_embeddings = model_output[0]
    attention_mask = encoded_input["attention_mask"]
    input_mask_expanded = (
        attention_mask.unsqueeze(-1).expand(token_embeddings.size()).float()
    )
    sentence_embeddings = torch.sum(
        token_embeddings * input_mask_expanded, 1
    ) / torch.clamp(input_mask_expanded.sum(1), min=1e-9)

    # 规范化嵌入
    batch["question_embedding"] = torch.nn.functional.normalize(
        sentence_embeddings, p=2, dim=1
    )
    return batch


data = data.map(encode_text, batched=True, batch_size=INFERENCE_BATCH_SIZE)
data_list = data.to_list()
```

## 插入数据

现在我们已经准备好带有问题嵌入向量的问题-答案对。下一步是将它们插入向量数据库。

首先，我们需要连接到 Milvus 服务并创建一个 Milvus 集合。本节将以 [Milvus Lite](https://milvus.io/docs/milvus_lite.md) 为例。如果您想使用其他类型的 Milvus 或 [Zilliz Cloud](https://zilliz.com)，请确保已启动服务并连接到您自己的 URI 和凭据。您也可以更改参数以自定义您的集合。

```python
from pymilvus import MilvusClient


MILVUS_URI = "./huggingface_milvus_test.db"  # 连接 URI
COLLECTION_NAME = "huggingface_test"  # 集合名称
DIMENSION = 384  # 嵌入维度取决于模型

milvus_client = MilvusClient(MILVUS_URI)
milvus_client.create_collection(
    collection_name=COLLECTION_NAME,
    dimension=DIMENSION,
    auto_id=True,  # 启用自动 ID
    enable_dynamic_field=True,  # 启用动态字段
    vector_field_name="question_embedding",  # 将向量字段名称映射到数据集中的嵌入列
    consistency_level="Strong",  # 启用最新数据的搜索
)
```

将所有数据插入到集合中：

```python
milvus_client.insert(collection_name=COLLECTION_NAME, data=data_list)
```
```
'ids': [450072488481390592, 450072488481390593, 450072488481390594, 450072488481390595, 450072488481390596, 450072488481390597, 450072488481390598, 450072488481390599, 450072488481390600, 450072488481390601, 450072488481390602],
'cost': 0}
```


## 提问

当所有数据都插入到 Milvus 中后，我们可以提出问题，看看最接近的答案是什么。


```python
questions = {
    "question": [
        "LGM 是什么意思？",
        "马萨诸塞州何时首次要求孩子在学校接受教育？",
    ]
}

# 生成问题的嵌入向量
question_embeddings = [v.tolist() for v in encode_text(questions)["question_embedding"]]

# 在 Milvus 中进行搜索
search_results = milvus_client.search(
    collection_name=COLLECTION_NAME,
    data=question_embeddings,
    limit=3,  # 输出多少个搜索结果
    output_fields=["answer", "question"],  # 在搜索结果中包含这些字段
)

# 打印结果
for q, res in zip(questions["question"], search_results):
    print("问题:", q)
    for r in res:
        print(
            {
                "答案": r["entity"]["answer"],
                "得分": r["distance"],
                "原始问题": r["entity"]["question"],
            }
        )
    print("\n")
```
```
    问题: LGM 是什么意思？
    {'答案': '末次冰期最冷时期', '得分': 0.956273078918457, '原始问题': 'LGM 代表什么？'}
    {'答案': '协调对禁运的响应', '得分': 0.2120140939950943, '原始问题': '为什么创建这个短期组织？'}
    {'答案': '“组合问题中的可约性”', '得分': 0.1945795714855194, '原始问题': '理查德·卡普（Richard Karp）在 1972 年撰写的论文 ushered in a new era of understanding between intractability and NP-complete problems 是什么？'}
```
```
    问题: 马萨诸塞州何时首次要求孩子在学校接受教育？
​    {'答案': '1852', '得分': 0.9709997177124023, '原始问题': '马萨诸塞州首次要求孩子在学校接受教育是在哪一年？'}
​    {'答案': '几所地区性学院和大学', '得分': 0.34164726734161377, '原始问题': '1890 年，大学决定与谁合作？'}
​    {'答案': '1962', '得分': 0.1931006908416748, '原始问题': 'stromules 是在哪一年被发现的？'}
```
