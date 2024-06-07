---
id: manage_databases.md
title: 数据库管理
---

# 数据库管理

与传统数据库引擎类似，您可以在 Milvus 中创建数据库并为特定用户分配权限来管理这些数据库。这样，这些用户就有权利管理数据库中的集合。一个 Milvus 集群支持最多 64 个数据库。

<div class="alert note">

本页面的代码片段使用 <a href="https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/Connections/connect.md">PyMilvus ORM 模块</a> 与 Milvus 进行交互。带有新 <a href="https://milvus.io/api-reference/pymilvus/v2.4.x/About.md">MilvusClient SDK</a> 的代码片段即将推出。

</div>

## 创建数据库

<div class="language-python">

使用 [connect()](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/Connections/connect.md) 连接到 Milvus 服务器，并使用 [create_database()](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/db/create_database.md) 创建新数据库：

</div>

<div class="language-java">

使用 [MilvusClient](https://milvus.io/api-reference/java/v2.4.x/v1/Connections/MilvusClient.md) 连接到 Milvus 服务器，并使用 [createDatabase()](https://milvus.io/api-reference/java/v2.4.x/v1/Database/createDatabase.md) 创建新数据库：

</div>

<div class="language-javascript">

使用 [MilvusClient](https://milvus.io/api-reference/node/v2.4.x/Client/MilvusClient.md) 连接到 Milvus 服务器，并使用 [createDatabase()](https://milvus.io/api-reference/node/v2.4.x/Database/createDatabase.md) 创建新数据库：

</div>

<div class="multipleCode">
    <a href="#python"">Python </a>
    <a href="#java"">Java</a>
    <a href="#javascript"">Node.js</a>
</div>

```python
from pymilvus import connections, db

conn = connections.connect(host="127.0.0.1", port=19530)

database = db.create_database("book")
```

```java
import io.milvus.client.MilvusServiceClient;
import io.milvus.param.ConnectParam;
import io.milvus.param.collection.CreateDatabaseParam;

// 1. 连接到 Milvus 服务器
ConnectParam connectParam = ConnectParam.newBuilder()
    .withUri(CLUSTER_ENDPOINT)
    .withToken(TOKEN)
    .build();

MilvusServiceClient client = new MilvusServiceClient(connectParam);

// 3. 创建新数据库
CreateDatabaseParam createDatabaseParam = CreateDatabaseParam.newBuilder()
    .withDatabaseName("my_database")
    .build();

R<RpcStatus> response = client.createDatabase(createDatabaseParam);
```

```javascript
const address = "http://localhost:19530"

// 1. 设置 Milvus 客户端
client = new MilvusClient({address}); 

// 3. 创建数据库
res = await client.createDatabase({
    db_name: "my_db"
})

console.log(res)

// {
//   error_code: 'Success',
//   reason: '',
//   code: 0,
//   retriable: false,
//   detail: ''
// }
```

上述代码片段连接到默认数据库并创建了一个名为 `my_database` 的新数据库。

## 使用数据库
一个 Milvus 集群默认配备一个名为 'default' 的数据库。除非另有指定，否则集合将在默认数据库中创建。

要更改默认数据库，请按照以下步骤操作：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
db.using_database("book")
```

```java
// 没有等效的方法可用。
```

```javascript
// 4. 激活另一个数据库
res = await client.useDatabase({
    db_name: "my_db"
})

console.log(res)
```

您还可以在连接到 Milvus 集群时设置要使用的数据库，方法如下：

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
conn = connections.connect(
    host="127.0.0.1",
    port="19530",
    db_name="my_database"
)
```

```java
ConnectParam connectParam = ConnectParam.newBuilder()
    .withDatabaseName("my_database")
    .withUri(CLUSTER_ENDPOINT)
    .withToken(TOKEN)
    .build();

MilvusServiceClient client = new MilvusServiceClient(connectParam);
```

```javascript
const address = "http://localhost:19530";
const db_name = "my_database";

// 1. 设置一个 Milvus 客户端
client = new MilvusClient({address, db_name}); 
```

## 列出数据库

<div class="language-python">

要查找 Milvus 集群中所有现有的数据库，请使用 [list_database()](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/db/list_database.md) 方法：

</div>

<div class="language-java">

要查找 Milvus 集群中所有现有的数据库，请使用 [listDatabases()](https://milvus.io/api-reference/java/v2.4.x/v1/Database/listDatabases.md) 方法：

</div>

<div class="language-javascript">

要查找 Milvus 集群中所有现有的数据库，请使用 [listDatabases()](https://milvus.io/api-reference/node/v2.4.x/Database/listDatabases.md) 方法：

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
db.list_database()

# 输出
['default', 'my_database']
```

```java
import io.milvus.grpc.ListDatabasesResponse;
import io.milvus.param.R;

// 2. 列出所有数据库
R<ListDatabasesResponse> listDatabasesResponse = client.listDatabases();
System.out.println(listDatabasesResponse.getData());

// status {
// }
// db_names: "default"
// db_names: "my_database"
// created_timestamp: 1716794498117757990
// created_timestamp: 1716797196479639477
```

```javascript
res = await client.listDatabases()

console.log(res.db_names)

// [ 'default', 'my_db' ]
```

## 删除数据库

要删除数据库，您必须首先删除其所有集合。否则，删除将失败。

<div class="language-python">

要删除数据库，请使用 [drop_database()](https://milvus.io/api-reference/pymilvus/v2.4.x/ORM/db/drop_database.md) 方法：

</div>

<div class="language-java">
要删除数据库，请使用 [dropDatabase()](https://milvus.io/api-reference/java/v2.4.x/v1/Database/dropDatabase.md) 方法：

</div>

<div class="language-javascript">

要删除数据库，请使用 [dropDatabase()](https://milvus.io/api-reference/node/v2.4.x/Database/dropDatabase.md) 方法：

</div>

<div class="multipleCode">
    <a href="#python">Python </a>
    <a href="#java">Java</a>
    <a href="#javascript">Node.js</a>
</div>

```python
db.drop_database("book")

db.list_database()

# 输出
['default']
```

```java
import io.milvus.param.collection.DropDatabaseParam;

DropDatabaseParam dropDatabaseParam = DropDatabaseParam.newBuilder()
    .withDatabaseName("my_database")
    .build();

response = client.dropDatabase(dropDatabaseParam);
```

```javascript
res = await client.dropDatabase({
    db_name: "my_db"
})
```

## 使用 RBAC 管理数据库

RBAC 还涵盖数据库操作，并确保向前兼容性。在权限 API（Grant / Revoke / List Grant）中，**database** 一词具有以下含义：

- 如果 Milvus 连接和权限 API 调用均未指定 `db_name`，则 **database** 指默认数据库。
- 如果 Milvus 连接指定了 `db_name`，但后续权限 API 调用未指定，则 **database** 指 Milvus 连接中指定名称的数据库。
- 如果在 Milvus 连接上进行了权限 API 调用，无论是否指定了 `db_name`，**database** 均指权限 API 调用中指定名称的数据库。

以下代码片段在下面列出的块之间共享。

```
from pymilvus import connections, Role

_HOST = '127.0.0.1'
_PORT = '19530'
_ROOT = "root"
_ROOT_PASSWORD = "Milvus"
_ROLE_NAME = "test_role"
_PRIVILEGE_INSERT = "Insert"


def connect_to_milvus(db_name="default"):
    print(f"connect to milvus\n")
    connections.connect(host=_HOST, port=_PORT, user=_ROOT, password=_ROOT_PASSWORD, db_name=db_name)
```

- 如果 Milvus 连接和权限 API 调用均未指定 `db_name`，**database** 指默认数据库。

```
connect_to_milvus()
role = Role(_ROLE_NAME)
role.create()

connect_to_milvus()
role.grant("Collection", "*", _PRIVILEGE_INSERT)
print(role.list_grants())
print(role.list_grant("Collection", "*"))
role.revoke("Global", "*", _PRIVILEGE_INSERT)
```

- 如果 Milvus 连接指定了 `db_name`，但后续权限 API 调用未指定，**database** 指 Milvus 连接中指定的数据库名称。

```python
# 注意：请确保已创建 'foo' 数据库
connect_to_milvus(db_name="foo")
# 此角色将具有 foo 数据库下所有集合的插入权限，但不包括其他数据库下集合的插入权限
role.grant("Collection", "*", _PRIVILEGE_INSERT)
print(role.list_grants())
print(role.list_grant("Collection", "*"))
role.revoke("Global", "*", _PRIVILEGE_INSERT)
```
- 如果在 Milvus 连接上进行了权限 API 调用，无论是否指定了 `db_name`，**数据库** 都指的是在权限 API 调用中指定名称的数据库。

```python
# 注意：请确保已创建 'foo' 数据库
db_name = "foo"
connect_to_milvus()
role.grant("Collection", "*", _PRIVILEGE_INSERT, db_name=db_name)
print(role.list_grants(db_name=db_name))
print(role.list_grant("Collection", "*", db_name=db_name))
role.revoke("Global", "*", _PRIVILEGE_INSERT, db_name=db_name)
```

## 接下来的步骤

[启用 RBAC](rbac.md)

[多租户](multi_tenancy.md)
