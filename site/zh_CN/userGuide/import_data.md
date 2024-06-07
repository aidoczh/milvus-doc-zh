---
id: import_data.md
related_key: bulk load
summary: 学习如何在Milvus中进行批量数据导入。
title: 导入数据
---

# 导入数据

本主题描述了如何通过批量加载在Milvus中导入数据。

通常，将大批实体插入Milvus的常规方法通常会导致跨客户端、代理、Pulsar和数据节点的大量网络传输。为了避免这种情况，Milvus 2.1支持通过批量加载从文件加载数据。您可以仅通过几行代码将大量数据导入集合，并为整个实体批量赋予原子性。

您还可以使用[MilvusDM](migrate_overview.md)将数据迁移至Milvus，这是一个专门设计用于与Milvus导入和导出数据的开源工具。

<div class="alert note">

本页上的代码片段使用新的<a href="https://milvus.io/api-reference/pymilvus/v2.4.x/About.md">MilvusClient</a>（Python）与Milvus进行交互。未来更新将发布其他语言的新MilvusClient SDK。

</div>

## 准备数据文件

您可以按行或列准备数据文件。

- 基于行的数据文件

基于行的数据文件是一个包含多行的JSON文件。根键必须是"rows"。文件名可以任意指定。

```json
{
  "rows":[
    {"book_id": 101, "word_count": 13, "book_intro": [1.1, 1.2]},
    {"book_id": 102, "word_count": 25, "book_intro": [2.1, 2.2]},
    {"book_id": 103, "word_count": 7, "book_intro": [3.1, 3.2]},
    {"book_id": 104, "word_count": 12, "book_intro": [4.1, 4.2]},
    {"book_id": 105, "word_count": 34, "book_intro": [5.1, 5.2]},
  ]
}
```

- 基于列的数据文件

基于列的数据文件可以是一个包含多列的JSON文件，几个包含单列的Numpy文件，每个文件包含一个列，或一个包含多列和一些Numpy文件的JSON文件。

   - 包含多列的JSON文件
    ```json
    {
            "book_id": [101, 102, 103, 104, 105],
            "word_count": [13, 25, 7, 12, 34],
            "book_intro": [
                    [1.1, 1.2],
                    [2.1, 2.2],
                    [3.1, 3.2],
                    [4.1, 4.2],
                    [5.1, 5.2]
            ]
    }
    ```

  - Numpy文件

    ```python
    import numpy
    numpy.save('book_id.npy', numpy.array([101, 102, 103, 104, 105]))
    numpy.save('word_count.npy', numpy.array([13, 25, 7, 12, 34]))
    arr = numpy.array([[1.1, 1.2],
                [2.1, 2.2],
                [3.1, 3.2],
                [4.1, 4.2],
                [5.1, 5.2]])
    numpy.save('book_intro.npy', arr)
    ```

  - 一个JSON文件包含多列和一些Numpy文件。

    ```json
    {
            "book_id": [101, 102, 103, 104, 105],
            "word_count": [13, 25, 7, 12, 34]
    }
    ```

    ```python
    {
        "book_id": [101, 102, 103, 104, 105],
        "word_count": [13, 25, 7, 12, 34]
    }
    ```

## 上传数据文件

将数据文件上传到对象存储。
你可以将数据文件上传到 MinIO 或本地存储（仅在 Milvus Standalone 中可用）。

- 上传到 MinIO

将数据文件上传到由配置文件 `milvus.yml` 中的 [`minio.bucketName`](configure_minio.md#miniobucketName) 定义的存储桶中。

- 上传到本地存储

将数据文件复制到由配置文件 `milvus.yml` 中的 [`localStorage.path`](configure_localstorage.md#localStoragepath) 定义的目录中。

## 将数据插入到 Milvus

将数据导入到集合中。

- 对于基于行的文件

```python
from pymilvus import utility
tasks = utility.bulk_load(
    collection_name="book",
    is_row_based=True,
    files=["row_based_1.json", "row_based_2.json"]
)
```

- 对于基于列的文件

```python
from pymilvus import utility
tasks = utility.bulk_load(
    collection_name="book",
    is_row_based=False,
    files=["columns.json", "book_intro.npy"]
)
```

<table class="language-python">
<thead>
<tr>
<th>参数</th>
<th>描述</th>
</tr>
</thead>
<tbody>
    <tr>
<td><code>collection_name</code></td>
<td>要加载数据的集合名称。</td>
</tr>
    <tr>
<td><code>is_row_based</code></td>
<td>布尔值，指示文件是否基于行。</td>
</tr>
    <tr>
<td><code>files</code></td>
<td>要加载到 Milvus 中的文件名列表。</td>
</tr>
<tr>
<td><code>partition_name</code>（可选）</td>
<td>要插入数据的分区名称。</td>
</tr>
</tbody>
</table>

## 检查导入任务状态

检查导入任务的状态。

```python
state = utility.get_bulk_load_state(tasks[0])
print(state.state_name())
print(state.ids())
print(state.infos())
```
状态代码及其对应的描述。

| 状态代码 | 状态                   | 描述                                                    |
| ---------- | ----------------------- | -------------------------------------------------------------- |
| 0          | BulkLoadPending         | 任务在待处理列表中                                        |
| 1          | BulkLoadFailed          | 任务失败，使用 `state.infos["failed_reason"]` 获取失败原因 |
| 2          | BulkLoadStarted         | 任务已分派到数据节点，即将执行          |
| 3          | BulkLoadDownloaded      | 数据文件已从 MinIO 下载到本地              |
| 4          | BulkLoadParsed          | 数据文件已验证并解析                       |
| 5          | BulkLoadPersisted       | 已生成并持久化新段                 |
| 6          | BulkLoadCompleted       | 任务已完成                                                 |

## 限制

|特性|最大限制|
|---|---|
|待处理任务列表的最大大小|32|
|数据文件的最大大小|4GB|

## 接下来做什么

- 了解更多 Milvus 的基本操作：
  - [索引向量字段](index-vector-fields.md)
  - [单向量搜索](single-vector-search.md)
  - [多向量搜索](multi-vector-search.md)
- 浏览 Milvus SDK 的 API 参考文档：
  - [PyMilvus API 参考文档](/api-reference/pymilvus/v{{var.milvus_python_sdk_version}}/tutorial.html)
  - [Node.js API 参考文档](/api-reference/node/v{{var.milvus_node_sdk_version}}/tutorial.html)