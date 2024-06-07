---
id: import-data.md
order: 1
title: 导入数据
summary: 本页面演示了导入准备好的数据的步骤。
---

# 导入数据

本页面演示了导入准备好的数据的步骤。

## 开始之前

- 您已经准备好您的数据并将其放入 Milvus 存储桶中。

    如果没有，请首先使用 **RemoteBulkWriter** 准备您的数据，并确保准备好的数据已经传输到与您的 Milvus 实例一起启动的 MinIO 实例上的 Milvus 存储桶中。详情请参考[准备源数据](prepare-source-data.md)。

- 您已经创建了一个具有您用于准备数据的模式的集合。如果没有，请参考[管理集合](manage-collections.md)。

<div class="language-python">

以下代码片段创建了一个具有给定模式的简单集合。有关参数的更多信息，请参考[`create_schema()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_schema.md)和[`create_collection()`](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Collections/create_collection.md)中的 SDK 参考。

</div>

<div class="language-java">

以下代码片段创建了一个具有给定模式的简单集合。有关参数的更多信息，请参考[`createCollection()`](https://milvus.io/api-reference/java/v2.4.x/v1/Collection/createCollection.md)中的 SDK 参考。

</div>

<div class="multipleCode">
  <a href="#python">Python </a>
  <a href="#java">Java</a>
</div>

```python
client = MilvusClient("http://localhost:19530")

schema = MilvusClient.create_schema(
    auto_id=False,
    enable_dynamic_field=True
)

schema.add_field(field_name="id", datatype=DataType.INT64, is_primary=True)
schema.add_field(field_name="vector", datatype=DataType.FLOAT_VECTOR, dim=768)
schema.add_field(field_name="scalar_1", datatype=DataType.VARCHAR, max_length=512)
schema.add_field(field_name="scalar_2", datatype=DataType.INT64)

client.create_collection(
    collection_name="quick_setup",
    schema=schema
)
```
```java
import io.milvus.client.MilvusServiceClient;
import io.milvus.param.ConnectParam;
import io.milvus.grpc.DataType;
import io.milvus.param.collection.CollectionSchemaParam;
import io.milvus.param.collection.CollectionSchemaParam;
import io.milvus.param.collection.FieldType;

final MilvusServiceClient milvusClient = new MilvusServiceClient(
ConnectParam.newBuilder()
    .withUri("localhost:19530")
    .withToken("root:Milvus")
    .build()
);

// 为目标集合定义模式
FieldType id = FieldType.newBuilder()
    .withName("id")
    .withDataType(DataType.Int64)
    .withPrimaryKey(true)
    .withAutoID(false)
    .build();

FieldType vector = FieldType.newBuilder()
    .withName("vector")
    .withDataType(DataType.FloatVector)
    .withDimension(768)
    .build();

FieldType scalar1 = FieldType.newBuilder()
    .withName("scalar_1")
    .withDataType(DataType.VarChar)
    .withMaxLength(512)
    .build();

FieldType scalar2 = FieldType.newBuilder()
    .withName("scalar_2")
    .withDataType(DataType.Int64)
    .build();

CollectionSchemaParam schema = CollectionSchemaParam.newBuilder()
    .withEnableDynamicField(true)
    .addFieldType(id)
    .addFieldType(vector)
    .addFieldType(scalar1)
    .addFieldType(scalar2)
    .build();

// 使用给定的模式创建一个集合
milvusClient.createCollection(CreateCollectionParam.newBuilder()
    .withCollectionName("quick_setup")
    .withSchema(schema)
    .build()
);
```
## 导入数据

要导入准备好的数据，您需要创建一个导入作业，如下所示：

```
export MILVUS_URI="localhost:19530"

curl --request POST "http://${MILVUS_URI}/v2/vectordb/jobs/import/create" \
--header "Content-Type: application/json" \
--data-raw '{
    "files": [
        [
            "/8ca44f28-47f7-40ba-9604-98918afe26d1/1.parquet"
        ],
        [
            "/8ca44f28-47f7-40ba-9604-98918afe26d1/2.parquet"
        ]
    ],
    "collectionName": "quick_setup"
}'
```

请求体包含两个字段：

- `collectionName`

    目标集合的名称。

- `files`

    一个文件路径列表的列表，相对于 Milvus 存储桶在您的 Milvus 实例启动时与 MioIO 实例一起启动的根路径。可能的子列表如下：

    - **JSON 文件**

        如果准备的文件是 JSON 格式，**每个子列表应包含指向单个准备好的 JSON 文件的路径**。

        ```
        [
            "/d1782fa1-6b65-4ff3-b05a-43a436342445/1.json"
        ],
        ```

    - **Parquet 文件**

        如果准备的文件是 Parquet 格式，**每个子列表应包含指向单个准备好的 Parquet 文件的路径**。

        ```
        [
            "/a6fb2d1c-7b1b-427c-a8a3-178944e3b66d/1.parquet"
        ]

可能的返回结果如下：

```
{
    "code": 200,
    "data": {
        "jobId": "448707763884413158"
    }
}
```

## 检查导入进度

一旦获得导入作业 ID，您可以按如下方式检查导入进度：

```
export MILVUS_URI="localhost:19530"

curl --request POST "http://${MILVUS_URI}/v2/vectordb/jobs/import/get_progress" \
--header "Content-Type: application/json" \
--data-raw '{
    "jobId": "449839014328146739"
}'
```

可能的响应如下：

```
{
    "code": 200,
    "data": {
        "collectionName": "quick_setup",
        "completeTime": "2024-05-18T02:57:13Z",
        "details": [
            {
                "completeTime": "2024-05-18T02:57:11Z",
                "fileName": "id:449839014328146740 paths:\"/8ca44f28-47f7-40ba-9604-98918afe26d1/1.parquet\" ",
                "fileSize": 31567874,
                "importedRows": 100000,
                "progress": 100,
                "state": "Completed",
                "totalRows": 100000
            },
            {
                "completeTime": "2024-05-18T02:57:11Z",
                "fileName": "id:449839014328146741 paths:\"/8ca44f28-47f7-40ba-9604-98918afe26d1/2.parquet\" ",
                "fileSize": 31517224,
                "importedRows": 100000,
                "progress": 100,
                "state": "Completed",
                "totalRows": 200000            
            }
        ],
        "fileSize": 63085098,
        "importedRows": 200000,
        "jobId": "449839014328146739",
        "progress": 100,
        "state": "Completed",
        "totalRows": 200000
    }
}
```

## 列出导入作业
你可以按照以下方式列出与特定集合相关的所有导入作业：

```
export MILVUS_URI="localhost:19530"

curl --request POST "http://${MILVUS_URI}/v2/vectordb/jobs/import/list" \
--header "Content-Type: application/json" \
--data-raw '{
    "collectionName": "quick_setup"
}'
```

可能的返回值如下：

```
{
    "code": 200,
    "data": {
        "records": [
            {
                "collectionName": "quick_setup",
                "jobId": "448761313698322011",
                "progress": 50,
                "state": "Importing"
            }
        ]
    }
}
```