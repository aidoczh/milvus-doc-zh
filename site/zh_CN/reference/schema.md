---
id: schema.md
summary: 学习如何在Milvus中定义模式。
title: 管理模式
---

# 管理模式

本主题介绍了Milvus中的模式。模式用于定义集合的属性以及其中的字段。

## 字段模式

字段模式是字段的逻辑定义。在定义[集合模式](#Collection-schema)和[管理集合](manage-collections.md)之前，这是您需要定义的第一件事情。

Milvus仅支持一个主键字段在一个集合中。

### 字段模式属性

<table class="properties">
<thead>
<tr>
<th>属性</td>
<th>描述</th>
<th>备注</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>name</code></td>
<td>要在集合中创建的字段的名称</td>
<td>数据类型：字符串。<br/>必填</td>
</tr>
<tr>
<td><code>dtype</code></td>
<td>字段的数据类型</td>
<td>必填</td>
</tr>
<tr>
<td><code>description</code></td>
<td>字段的描述</td>
<td>数据类型：字符串。<br/>可选</td>
</tr>
<tr>
<td><code>is_primary</code></td>
<td>是否将字段设置为主键字段</td>
<td>数据类型：布尔值（<code>true</code>或<code>false</code>）。<br/>主键字段必填</td>
</tr>
<tr>
<td><code>auto_id</code>（主键字段必填）</td>
<td>开关，用于启用或禁用自动ID（主键）分配。</td>
<td><code>True</code>或<code>False</code></td>
</tr>
<tr>
<td><code>max_length</code>（VARCHAR字段必填）</td>
<td>允许插入的字符串的最大长度。</td>
<td>[1, 65,535]</td>
</tr>
<tr>
<td><code>dim</code></td>
<td>向量的维度</td>
<td>数据类型：整数，范围在[1, 32768]之间。<br/>稠密向量字段必填。稀疏向量字段请忽略。</td>
</tr>
<tr>
<td><code>is_partition_key</code></td>
<td>此字段是否为分区键字段。</td>
<td>数据类型：布尔值（<code>true</code>或<code>false</code>）。</td>
</tr>
</tbody>
</table>

### 创建字段模式

为了减少数据插入时的复杂性，Milvus允许您在字段模式创建过程中为每个标量字段指定默认值，不包括主键字段。这意味着，如果在插入数据时将字段留空，将应用您为该字段指定的默认值。

创建常规字段模式：

```python
from pymilvus import FieldSchema
id_field = FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, description="primary id")
age_field = FieldSchema(name="age", dtype=DataType.INT64, description="age")
embedding_field = FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=128, description="vector")

# 以下创建一个字段并将其用作分区键
position_field = FieldSchema(name="position", dtype=DataType.VARCHAR, max_length=256, is_partition_key=True)
```
使用默认字段值创建字段模式：

```python
from pymilvus import FieldSchema

fields = [
  FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),
  # 为字段 `age` 配置默认值 `25`
  FieldSchema(name="age", dtype=DataType.INT64, default_value=25, description="age"),
  embedding_field = FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=128, description="vector")
]
```

### 支持的数据类型

`DataType` 定义了字段包含的数据类型。不同字段支持不同的数据类型。

- 主键字段支持：
  - INT64: numpy.int64
  - VARCHAR: VARCHAR
- 标量字段支持：
  - BOOL: 布尔值（`true` 或 `false`）
  - INT8: numpy.int8
  - INT16: numpy.int16
  - INT32: numpy.int32
  - INT64: numpy.int64
  - FLOAT: numpy.float32
  - DOUBLE: numpy.double
  - VARCHAR: VARCHAR
  - JSON: [JSON](use-json-fields.md)
  - Array: [Array](array_data_type.md)

  JSON 作为一个复合数据类型可用。JSON 字段由键值对组成。每个键是一个字符串，值可以是数字、字符串、布尔值、数组或列表。详情请参阅 [JSON：一种新的数据类型](use-json-fields.md)。

- 向量字段支持：
  - BINARY_VECTOR: 将二进制数据存储为 0 和 1 的序列，用于图像处理和信息检索中的紧凑特征表示。
  - FLOAT_VECTOR: 存储 32 位浮点数，通常用于科学计算和机器学习中表示实数。
  - FLOAT16_VECTOR: 存储 16 位半精度浮点数，用于深度学习和 GPU 计算以实现内存和带宽效率。
  - BFLOAT16_VECTOR: 存储 16 位浮点数，具有减少的精度但与 Float32 相同的指数范围，用于深度学习以减少内存和计算需求而不显著影响准确性。
  - SPARSE_FLOAT_VECTOR: 存储非零元素及其对应索引的列表，用于表示稀疏向量。更多信息，请参阅 [稀疏向量](sparse_vector.md)。

  Milvus 支持在一个集合中使用多个向量字段。更多信息，请参阅 [多向量搜索](multi-vector-search.md)。

## 集合模式

集合模式是集合的逻辑定义。通常在定义[字段模式](#字段模式)之前需要定义集合模式并[管理集合](manage-collections.md)。

### 集合模式属性

<table class="properties">
<thead>
<tr>
<th>属性</td>
<th>描述</th>
<th>备注</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>field</code></td>
<td>要创建的集合中的字段</td>
<td>必需</td>
</tr>
    <tr>
<td><code>description</code></td>
<td>集合的描述</td>
<td>数据类型：字符串。<br/>可选</td>
</tr>
    <tr>
<td><code>partition_key_field</code></td>
<td>设计为分区键的字段名称</td>
<td>数据类型：字符串。<br/>可选</td>
</tr>
    <tr>
<td><code>enable_dynamic_field</code></td>
<td>是否启用动态模式</td>
<td>数据类型：布尔值（<code>true</code> 或 <code>false</code>）。<br/>可选，默认为<code>False</code>。<br/>有关动态模式的详细信息，请参考<a herf="enable-dynamic-field.md">动态模式</a>和管理集合的用户指南。</td>
</tr>
</tbody>
</table>

### 创建集合模式

<div class="alert note">
  在定义集合模式之前定义字段模式。
</div>

```python
from pymilvus import FieldSchema, CollectionSchema
id_field = FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, description="主键")
age_field = FieldSchema(name="age", dtype=DataType.INT64, description="年龄")
embedding_field = FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=128, description="向量")

# 如果需要基于分区键字段实现多租户功能，请在字段上启用分区键
position_field = FieldSchema(name="position", dtype=DataType.VARCHAR, max_length=256, is_partition_key=True)

# 如果需要使用动态字段，请将 enable_dynamic_field 设置为 True
schema = CollectionSchema(fields=[id_field, age_field, embedding_field], auto_id=False, enable_dynamic_field=True, description="集合描述")
```

使用指定的模式创建集合：

```python
from pymilvus import Collection
collection_name1 = "tutorial_1"
collection1 = Collection(name=collection_name1, schema=schema, using='default', shards_num=2)
```
<div class="alert note">

  - 您可以使用<code>shards_num</code>定义分片数。
  - 您可以通过在<code>using</code>中指定别名来定义要在其上创建集合的 Milvus 服务器。
  - 如果需要实现基于分区键的多租户功能，请将字段上的<code>is_partition_key</code>设置为<code>True</code>。
  - 如果需要[启用动态字段](enable-dynamic-field.md)，请在集合模式中将<code>enable_dynamic_field</code>设置为<code>True</code>。

</div>
  
<br/>
您还可以使用<code>Collection.construct_from_dataframe</code>从 DataFrame 自动生成集合模式并创建集合。

```python
import pandas as pd
df = pd.DataFrame({
    "id": [i for i in range(nb)],
    "age": [random.randint(20, 40) for i in range(nb)],
    "embedding": [[random.random() for _ in range(dim)] for _ in range(nb)],
    "position": "test_pos"
})

collection, ins_res = Collection.construct_from_dataframe(
    'my_collection',
    df,
    primary_field='id',
    auto_id=False
    )
```

## 接下来做什么

- 学习如何在[管理集合](manage-collections.md)时准备模式。
- 了解更多关于[动态模式](enable-dynamic-field.md)的信息。
- 在[多租户](multi_tenancy.md)中了解更多关于分区键的信息。