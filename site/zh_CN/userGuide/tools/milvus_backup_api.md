---
id: milvus_backup_api.md
summary: 通过 API 学习如何使用 Milvus 备份数据
title: 使用 API 进行数据备份和恢复
---

# 使用 API 进行数据备份和恢复

Milvus Backup 提供数据备份和恢复功能，以确保您的 Milvus 数据的安全性。

## 获取 Milvus Backup

您可以选择下载已编译的二进制文件，也可以从源代码构建。

要下载已编译的二进制文件，请访问 [release](https://github.com/zilliztech/milvus-backup/releases) 页面，在那里您可以找到所有官方发布的版本。请记住，始终使用标记为 **Latest** 的发布版本中的二进制文件。

要从源代码编译，请按照以下步骤操作：

```shell
git clone git@github.com:zilliztech/milvus-backup.git
go get
go build
```

## 准备配置文件

下载 [示例配置文件](https://raw.githubusercontent.com/zilliztech/milvus-backup/master/configs/backup.yaml)，并根据您的需求进行定制。

然后在下载或构建的 Milvus Backup 二进制文件旁创建一个文件夹，命名为 `configs`，并将配置文件放入 `configs` 文件夹中。

您的文件夹结构应类似于以下内容：

<pre>
workspace
├── milvus-backup
└── configs
     └── backup.yaml
</pre>

由于 Milvus Backup 无法将数据备份到本地路径，请确保在定制配置文件时 Minio 设置正确。

<div class="alert note">

默认 Minio 存储桶的名称因安装 Milvus 的方式而异。在更改 Minio 设置时，请参考以下表格。

| field           | Docker Compose | Helm / Milvus Operator |
| --------------- | -------------- | ---------------------- |
| `bucketName`    | a-bucket       | milvus-bucket          |
| `rootPath`      | files          | file                   |          

</div>

## 启动 API 服务器

然后您可以按照以下步骤启动 API 服务器：

```shell
./milvus-backup server
```

API 服务器默认监听端口为 8080。您可以通过使用 `-p` 标志更改端口。要启动监听端口 443 的 API 服务器，请按照以下步骤操作：

```shell
./milvus-backup server -p 443
```

您可以通过 http://localhost:<port>/api/v1/docs/index.html 访问 Swagger UI。

## 准备数据

如果您运行的是监听默认端口 19530 的空本地 Milvus 实例，请使用示例 Python 脚本在您的实例中生成一些数据。随时根据需要对脚本进行必要的更改。

获取 [脚本](https://raw.githubusercontent.com/zilliztech/milvus-backup/main/example/prepare_data.py)。然后运行脚本生成数据。确保已安装官方 Milvus Python SDK [PyMilvus](https://pypi.org/project/pymilvus/)。

```shell
python example/prepare_data.py
```

此步骤是可选的。如果您跳过此步骤，请确保您的 Milvus 实例中已经有一些数据。

## 备份数据

{{tab}}

请注意，对 Milvus 实例运行 Milvus 备份通常不会影响实例的运行。在备份或恢复过程中，您的 Milvus 实例是完全可用的。

运行以下命令创建备份。如有需要，请更改 `collection_names` 和 `backup_name`。

```shell
curl --location --request POST 'http://localhost:8080/api/v1/create' \
--header 'Content-Type: application/json' \
--data-raw '{
  "async": true,
  "backup_name": "my_backup",
  "collection_names": [
    "hello_milvus"
  ]
}'
```

执行该命令后，您可以按照以下方式列出 Minio 设置中指定的存储桶中的备份：

```shell
curl --location --request GET 'http://localhost:8080/api/v1/list' \
--header 'Content-Type: application/json'
```

然后按照以下方式下载备份文件：

```shell
curl --location --request GET 'http://localhost:8080/api/v1/get_backup?backup_id=<test_backup_id>&backup_name=my_backup' \
--header 'Content-Type: application/json'
```

在运行上述命令时，请将 `backup_id` 和 `backup_name` 更改为列表 API 返回的值。

现在，您可以将备份文件保存到安全位置以备将来恢复，或者上传到 [Zilliz Cloud](https://cloud.zilliz.com) 以使用您的数据创建托管向量数据库。有关详细信息，请参阅 [从 Milvus 迁移到 Zilliz Cloud](https://zilliz.com/doc/migrate_from_milvus-2x)。

## 恢复数据

{{tab}}

您可以调用恢复 API 命令，并使用 `collection_suffix` 选项从备份中恢复数据以创建新集合。如有需要，请更改 `collection_names` 和 `backup_name`。

```shell
curl --location --request POST 'http://localhost:8080/api/v1/restore' \
--header 'Content-Type: application/json' \
--data-raw '{
    "async": true,
    "collection_names": [
    "hello_milvus"
  ],
    "collection_suffix": "_recover",
    "backup_name":"my_backup"
}'
```

`collection_suffix` 选项允许您为要创建的新集合设置后缀。上述命令将在您的 Milvus 实例中创建一个名为 **hello_milvus_recover** 的新集合。

如果您希望在不更改名称的情况下恢复备份的集合，请在从备份中恢复之前删除该集合。您现在可以通过运行以下命令清除 [准备数据](#Prepare-data) 生成的数据。

```shell
python example/clean_data.py
```

然后运行以下命令从备份中恢复数据。

```shell
curl --location --request POST 'http://localhost:8080/api/v1/restore' \
--header 'Content-Type: application/json' \
--data-raw '{
    "async": true,
    "collection_names": [
    "hello_milvus"
  ],
    "collection_suffix": "",
    "backup_name":"my_backup"
}'
```

恢复过程的耗时取决于要恢复的数据大小。因此，所有恢复任务都是异步运行的。您可以通过运行以下命令来检查恢复任务的状态：
```shell
curl --location --request GET 'http://localhost:8080/api/v1/get_restore?id=<test_restore_id>' \
--header 'Content-Type: application/json'
```
记得将 `test_restore_id` 更改为由恢复 API 恢复的 ID。

## 验证已恢复的数据

恢复完成后，您可以通过将恢复的集合索引化来验证已恢复的数据，操作如下：

```shell
python example/verify_data.py
```

请注意，上述脚本假定您已经使用 `-s` 标志运行了 `restore` 命令，并且后缀设置为 `-recover`。请随时根据您的需求对脚本进行必要的更改。