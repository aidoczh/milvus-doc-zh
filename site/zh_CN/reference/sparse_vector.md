---
id: sparse_vector.md
summary: 学习如何在Milvus中使用稀疏向量。
title: 稀疏向量
---

# 稀疏向量

稀疏向量使用向量嵌入来表示单词或短语，其中大多数元素为零，只有一个非零元素表示特定单词的存在。稀疏向量模型，如[SPLADEv2](https://arxiv.org/abs/2109.10086)，在跨领域知识搜索、关键词感知和可解释性方面优于密集模型。它们在信息检索、自然语言处理和推荐系统中特别有用，将稀疏向量用于召回，结合大型模型用于排序，可以显著改善检索结果。

在Milvus中，使用稀疏向量与使用密集向量类似。它涉及创建一个带有稀疏向量列的集合，插入数据，创建索引，以及进行相似度搜索和标量查询。

在本教程中，您将学习如何：

- 准备稀疏向量嵌入；
- 创建带有稀疏向量字段的集合；
- 插入具有稀疏向量嵌入的实体；
- 对集合建立索引，并在稀疏向量上执行ANN搜索。

要查看稀疏向量的实际应用，请参考[hello_sparse.py](https://github.com/milvus-io/pymilvus/blob/master/examples/milvus_client/sparse.py)。

<div class="admonition note">
    <p><b>注意</b></p>
        目前，对稀疏向量的支持是2.4.0版本中的一个测试功能，计划在3.0.0版本中正式推出。
</div>

## 准备稀疏向量嵌入

要在Milvus中使用稀疏向量，需准备支持格式之一的向量嵌入：

- __稀疏矩阵__：利用[scipy.sparse](https://docs.scipy.org/doc/scipy/reference/sparse.html#module-scipy.sparse)类系列来表示您的稀疏嵌入。这种方法适用于处理大规模、高维数据。

- __字典列表__：将每个稀疏嵌入表示为一个字典，结构为`{维度索引: 值, ...}`，其中每个键值对表示维度索引及其对应的值。

    示例：

    ```python
    {2: 0.33, 98: 0.72, ...}
    ```

- __元组迭代器列表__：类似于字典列表，但使用元组的可迭代对象`(维度索引, 值)`来指定仅非零维度及其值。

    示例：

    ```python
    [(2, 0.33), (98, 0.72), ...]
    ```

以下示例通过生成一个包含10,000个实体的随机稀疏矩阵，每个实体具有10,000个维度，稀疏度为0.005，来准备稀疏嵌入。
```python
# 准备具有稀疏向量表示的实体
import numpy as np
import random

rng = np.random.default_rng()

num_entities, dim = 10000, 10000

# 生成具有平均每行 25 个非零元素的随机稀疏行
entities = [
    {
        "scalar_field": rng.random(),
        # 要表示单个稀疏向量行，可以使用：
        # - 任何 scipy.sparse 稀疏矩阵类族中的一个，使得 shape[0] == 1
        # - Dict[int, float]
        # - Iterable[Tuple[int, float]]
        "sparse_vector": {
            d: rng.random() for d in random.sample(range(dim), random.randint(20, 30))
        },
    }
    for _ in range(num_entities)
]

# 打印第一个实体以检查表示
print(entities[0])

# 输出:
# {
#     'scalar_field': 0.520821523849214,
#     'sparse_vector': {
#         5263: 0.2639375518635271,
#         3573: 0.34701499565746674,
#         9637: 0.30856525997853057,
#         4399: 0.19771651149001523,
#         6959: 0.31025067641541815,
#         1729: 0.8265339135915016,
#         1220: 0.15303302147479103,
#         7335: 0.9436728846033107,
#         6167: 0.19929870545596562,
#         5891: 0.8214617920371853,
#         2245: 0.7852255053773395,
#         2886: 0.8787982039149889,
#         8966: 0.9000606703940665,
#         4910: 0.3001170013981104,
#         17: 0.00875671667413136,
#         3279: 0.7003425473001098,
#         2622: 0.7571360018373428,
#         4962: 0.3901879090102064,
#         4698: 0.22589525720196246,
#         3290: 0.5510228492587324,
#         6185: 0.4508413201390492
#     }
# }
```

<div class="admonition note">

<p><b>注意事项</b></p>

<p>向量维度必须是 Python <code>int</code> 或 <code>numpy.integer</code> 类型，值必须是 Python <code>float</code> 或 <code>numpy.floating</code> 类型。</p>

</div>

要生成嵌入向量，您还可以使用 PyMilvus 库中内置的 `model` 包，该包提供了一系列嵌入函数。详情请参阅 [嵌入](embeddings.md)。

## 创建具有稀疏向量字段的集合

要创建具有稀疏向量字段的集合，请将稀疏向量字段的 __datatype__ 设置为 __DataType.SPARSE_FLOAT_VECTOR__。与密集向量不同，稀疏向量无需指定维度。

```python
from pymilvus import MilvusClient, DataType

# 创建 MilvusClient 实例
client = MilvusClient(uri="http://localhost:19530")

# 创建具有稀疏向量字段的集合
schema = client.create_schema(
    auto_id=True,
    enable_dynamic_fields=True,
)

schema.add_field(field_name="pk", datatype=DataType.VARCHAR, is_primary=True, max_length=100)
schema.add_field(field_name="scalar_field", datatype=DataType.DOUBLE)
# 对于稀疏向量，无需指定维度
schema.add_field(field_name="sparse_vector", datatype=DataType.SPARSE_FLOAT_VECTOR) # 将 `datatype` 设置为 `SPARSE_FLOAT_VECTOR`

client.create_collection(collection_name="test_sparse_vector", schema=schema)
```

有关常见集合参数的详细信息，请参阅 [create_collection()
](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md)。

## 插入具有稀疏向量嵌入的实体

要插入具有稀疏向量嵌入的实体，只需将实体列表传递给 [`insert()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/insert.md) 方法。

```python
# 插入实体
client.insert(collection_name="test_sparse_vector", data=entities)
```

## 为集合创建索引

在执行相似度搜索之前，为集合创建索引。有关索引类型和参数的更多信息，请参阅 [add_index()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/add_index.md) 和 [create_index()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Management/create_index.md)。

```python
# 为集合创建索引

# 准备索引参数
index_params = client.prepare_index_params()

index_params.add_index(
    field_name="sparse_vector",
    index_name="sparse_inverted_index",
    index_type="SPARSE_INVERTED_INDEX", # 要创建的索引类型。设置为 `SPARSE_INVERTED_INDEX` 或 `SPARSE_WAND`。
    metric_type="IP", # 用于索引的度量类型。目前仅支持 `IP`（内积）。
    params={"drop_ratio_build": 0.2}, # 在索引过程中要删除的小向量值的比例。
)

# 创建索引
client.create_index(collection_name="test_sparse_vector", index_params=index_params)
```
对于稀疏向量的索引构建，请注意以下事项：

- `index_type`：要构建的索引类型。稀疏向量的可能选项包括：

  - `SPARSE_INVERTED_INDEX`：一种倒排索引，将每个维度映射到其非零向量，便于在搜索过程中直接访问相关数据。适用于数据稀疏但维度较高的数据集。

  - `SPARSE_WAND`：利用弱与（Weak-AND，WAND）算法快速绕过不太可能的候选项，将评估重点放在排名潜力较高的候选项上。将维度视为术语，向量视为文档，在大型稀疏数据集中加快搜索速度。

- `metric_type`：稀疏向量仅支持`IP`（内积）距离度量。

- `params.drop_ratio_build`：专门用于稀疏向量的索引参数。它控制在索引过程中排除的小向量值的比例。该参数通过在构建索引时忽略小值，实现了在效率和准确性之间的微调。例如，如果`drop_ratio_build = 0.3`，在索引构建过程中，将收集并排序所有稀疏向量的所有值。这些值中最小的30%不包括在索引中，从而减少了搜索过程中的计算工作量。

欲了解更多信息，请参阅[内存索引](index.md)。

## 执行近似最近邻搜索

在将集合索引化并加载到内存后，使用[`search()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/search.md)方法基于查询检索相关文档。

```python
# 将集合加载到内存中
client.load_collection(collection_name="test_sparse_vector")

# 对稀疏向量执行近似最近邻搜索

# 为演示目的，我们搜索最后插入的向量
query_vector = entities[-1]["sparse_vector"]

search_params = {
    "metric_type": "IP",
    "params": {"drop_ratio_search": 0.2}, # 在搜索过程中要丢弃的小向量值的比例
}

search_res = client.search(
    collection_name="test_sparse_vector",
    data=[query_vector],
    limit=3,
    output_fields=["pk", "scalar_field"],
    search_params=search_params,
)

for hits in search_res:
    for hit in hits:
        print(f"hit: {hit}")
        
# 输出:
# hit: {'id': '448458373272710786', 'distance': 7.220192909240723, 'entity': {'pk': '448458373272710786', 'scalar_field': 0.46767865218233806}}
# hit: {'id': '448458373272708317', 'distance': 1.2287548780441284, 'entity': {'pk': '448458373272708317', 'scalar_field': 0.7315987515699472}}
# hit: {'id': '448458373272702005', 'distance': 0.9848432540893555, 'entity': {'pk': '448458373272702005', 'scalar_field': 0.9871869181562156}}
```

在配置搜索参数时，请注意以下事项：
- `params.drop_ratio_search`：用于稀疏向量的搜索参数。此选项允许通过指定查询向量中要忽略的最小值比例来微调搜索过程。它有助于平衡搜索精度和性能。设置较小的`drop_ratio_search`值，这些小值对最终得分的贡献就越少。通过忽略一些小值，可以在最小影响准确性的情况下提高搜索性能。

## 执行标量查询

除了ANN搜索外，Milvus还支持对稀疏向量进行标量查询。这些查询允许您基于与稀疏向量关联的标量值检索文档。有关参数的更多信息，请参阅[query()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Vector/query.md)。

过滤标量字段大于3的实体：

```python
# 通过指定过滤表达式执行查询
filter_query_res = client.query(
    collection_name="test_sparse_vector",
    filter="scalar_field > 0.999",
)

print(filter_query_res[:2])

# 输出:
# [{'pk': '448458373272701862', 'scalar_field': 0.9994093623822689, 'sparse_vector': {173: 0.35266244411468506, 400: 0.49995484948158264, 480: 0.8757831454277039, 661: 0.9931875467300415, 1040: 0.0965644046664238, 1728: 0.7478245496749878, 2365: 0.4351981580257416, 2923: 0.5505295395851135, 3181: 0.7396837472915649, 3848: 0.4428485333919525, 4701: 0.39119353890419006, 5199: 0.790219783782959, 5798: 0.9623121619224548, 6213: 0.453134149312973, 6341: 0.745091438293457, 6775: 0.27766478061676025, 6875: 0.017947908490896225, 8093: 0.11834774166345596, 8617: 0.2289179265499115, 8991: 0.36600416898727417, 9346: 0.5502803921699524}}, {'pk': '448458373272702421', 'scalar_field': 0.9990218525410719, 'sparse_vector': {448: 0.587817907333374, 1866: 0.0994109958410263, 2438: 0.8672442436218262, 2533: 0.8063794374465942, 2595: 0.02122959867119789, 2828: 0.33827054500579834, 2871: 0.1984412521123886, 2938: 0.09674275666475296, 3154: 0.21552987396717072, 3662: 0.5236313343048096, 3711: 0.6463911533355713, 4029: 0.4041993021965027, 7143: 0.7370485663414001, 7589: 0.37588241696357727, 7776: 0.436136394739151, 7962: 0.06377989053726196, 8385: 0.5808192491531372, 8592: 0.8865005970001221, 8648: 0.05727503448724747, 9071: 0.9450633525848389, 9161: 0.146037295460701, 9358: 0.1903032660484314, 9679: 0.3146636486053467, 9974: 0.8561339378356934, 9991: 0.15841573476791382}}]
```

通过主键过滤实体：
```python
# 满足筛选条件的实体的主键
pks = [ret["pk"] for ret in filter_query_res]

# 通过主键执行查询
pk_query_res = client.query(
    collection_name="test_sparse_vector", filter=f"pk == '{pks[0]}'"
)

print(pk_query_res)

# 输出:
# [{'scalar_field': 0.9994093623822689, 'sparse_vector': {173: 0.35266244411468506, 400: 0.49995484948158264, 480: 0.8757831454277039, 661: 0.9931875467300415, 1040: 0.0965644046664238, 1728: 0.7478245496749878, 2365: 0.4351981580257416, 2923: 0.5505295395851135, 3181: 0.7396837472915649, 3848: 0.4428485333919525, 4701: 0.39119353890419006, 5199: 0.790219783782959, 5798: 0.9623121619224548, 6213: 0.453134149312973, 6341: 0.745091438293457, 6775: 0.27766478061676025, 6875: 0.017947908490896225, 8093: 0.11834774166345596, 8617: 0.2289179265499115, 8991: 0.36600416898727417, 9346: 0.5502803921699524}, 'pk': '448458373272701862'}]
```
## 限制

在 Milvus 中使用稀疏向量时，请考虑以下限制：

- 目前，仅支持使用 __IP__ 距离度量来计算稀疏向量之间的距离。

- 对于稀疏向量字段，仅支持 __SPARSE_INVERTED_INDEX__ 和 __SPARSE_WAND__ 两种索引类型。

- 目前，不支持对稀疏向量进行 [范围搜索](https://milvus.io/docs/single-vector-search.md#Range-search)、[分组搜索](https://milvus.io/docs/single-vector-search.md#Grouping-search) 和 [搜索迭代器](https://milvus.io/docs/with-iterators.md#Search-with-iterator)。

## 常见问题

- __稀疏向量支持哪种距离度量？__

    由于稀疏向量的高维特性，只支持内积（IP）距离度量，因此 L2 距离和余弦距离并不适用。

- __请解释 SPARSE_INVERTED_INDEX 和 SPARSE_WAND 之间的区别，以及如何在它们之间进行选择？__

    __SPARSE_INVERTED_INDEX__ 是传统的倒排索引，而 __SPARSE_WAND__ 使用 [Weak-AND](https://dl.acm.org/doi/10.1145/956863.956944) 算法，在搜索过程中减少完整的内积距离计算次数。通常情况下，__SPARSE_WAND__ 更快，但随着向量密度增加，性能可能会下降。要在它们之间进行选择，请根据您的特定数据集和用例进行实验和基准测试。

- __如何选择 drop_ratio_build 和 drop_ratio_search 参数？__

    选择 __drop_ratio_build__ 和 __drop_ratio_search__ 取决于您的数据特性以及对搜索延迟/吞吐量和准确性的要求。

- __稀疏嵌入支持哪些数据类型？__

    维度部分必须是无符号 32 位整数，值部分可以是非负的 32 位浮点数。

- __稀疏嵌入的维度是否可以是 uint32 空间内的任何离散值？__

    是的，但有一个例外。稀疏嵌入的维度可以是 `[0, uint32 的最大值)` 范围内的任何值。这意味着您不能使用 uint32 的最大值。

- __对增长段进行的搜索是通过索引还是暴力搜索进行的？__

    对增长段的搜索是通过与封存段索引相同类型的索引进行的。对于在构建索引之前的新增长段，将使用暴力搜索。

- __是否可以在单个集合中同时包含稀疏向量和密集向量？__

    是的，通过多向量类型支持，您可以创建同时包含稀疏和密集向量列的集合，并对它们执行混合搜索。

- __插入或搜索稀疏嵌入的要求是什么？__

    稀疏嵌入必须至少有一个非零值，并且向量索引必须是非负的。