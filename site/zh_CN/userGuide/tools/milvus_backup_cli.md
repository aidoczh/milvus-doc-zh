---
id: milvus_backup_cli.md
summary: 通过命令行学习如何使用 Milvus 备份
title: 使用命令进行数据备份和恢复
---

# 使用命令进行数据备份和恢复

Milvus Backup 提供数据备份和恢复功能，以确保您的 Milvus 数据的安全性。

## 获取 Milvus Backup

您可以选择下载已编译的二进制文件，也可以从源代码构建。

要下载已编译的二进制文件，请访问[发布页面](https://github.com/zilliztech/milvus-backup/releases)，在那里您可以找到所有官方发布版本。请记住，始终使用标记为**Latest**的发布版本中的二进制文件。

要从源代码编译，请执行以下操作：

```shell
git clone git@github.com:zilliztech/milvus-backup.git
go get
go build
```

## 准备配置文件

下载[示例配置文件](https://raw.githubusercontent.com/zilliztech/milvus-backup/master/configs/backup.yaml)，并根据您的需求进行调整。

然后，在下载或构建的 Milvus Backup 二进制文件旁创建一个名为`configs`的文件夹，并将配置文件放入`configs`文件夹内。

您的文件夹结构应类似于以下内容：

<pre>
workspace
├── milvus-backup
└── configs
     └── backup.yaml
</pre>

由于 Milvus Backup 无法将数据备份到本地路径，请确保在调整配置文件时 Minio 设置正确。

<div class="alert note">

默认的 Minio 存储桶名称因安装 Milvus 的方式而异。在更改 Minio 设置时，请参考以下表格。

| field           | Docker Compose | Helm / Milvus Operator |
| --------------- | -------------- | ---------------------- |
| `bucketName`    | a-bucket       | milvus-bucket          |
| `rootPath`      | files          | file                   |

</div>

## 准备数据

如果在默认端口运行空的本地 Milvus 实例，请使用示例 Python 脚本在您的实例中生成一些数据。随意对脚本进行必要的更改以满足您的需求。

获取[脚本](https://raw.githubusercontent.com/zilliztech/milvus-backup/main/example/prepare_data.py)。然后运行脚本生成数据。确保已安装[PyMilvus](https://pypi.org/project/pymilvus/)，官方 Milvus Python SDK。

```shell
python example/prepare_data.py
```

此步骤是可选的。如果跳过此步骤，请确保您的 Milvus 实例中已有一些数据。

## 备份数据

请注意，运行 Milvus Backup 对 Milvus 实例不会通常影响实例的运行。在备份或恢复过程中，您的 Milvus 实例是完全可用的。

{{tab}}

运行以下命令创建备份。

```shell
./milvus-backup create -n <backup_name>
```

执行该命令后，您可以在 Minio 设置中指定的存储桶中检查备份文件。具体来说，您可以使用**Minio 控制台**或**mc**客户端下载这些文件。
要从[Minio控制台](https://min.io/docs/minio/kubernetes/upstream/administration/minio-console.html)下载文件，请登录Minio控制台，找到`minio.address`中指定的存储桶，选择存储桶中的文件，然后点击**下载**按钮进行下载。

如果您更喜欢使用[mc客户端](https://min.io/docs/minio/linux/reference/minio-mc.html#mc-install)，请按照以下步骤操作：

```shell
# 配置Minio主机
mc alias set my_minio https://<minio_endpoint> <accessKey> <secretKey>

# 列出可用的存储桶
mc ls my_minio

# 递归下载存储桶
mc cp --recursive my_minio/<your-bucket-path> <local_dir_path>
```

现在，您可以将备份文件保存在安全的地方以便将来恢复，或者将它们上传到[Zilliz云](https://cloud.zilliz.com)以使用您的数据创建托管的向量数据库。有关详细信息，请参阅[Migrate from Milvus to Zilliz Cloud](https://zilliz.com/doc/migrate_from_milvus-2x)。

## 恢复数据

{{tab}}

您可以运行带有`-s`标志的`restore`命令，通过从备份中恢复数据来创建一个新集合：

```shell
./milvus-backup restore -n my_backup -s _recover
```

`-s`标志允许您为要创建的新集合设置后缀。上述命令将在您的Milvus实例中创建一个名为**hello_milvus_recover**的新集合。

如果您希望在恢复备份的集合时不更改其名称，请在从备份中恢复之前删除该集合。您现在可以通过运行以下命令清除[准备数据](#Prepare-data)中生成的数据。

```shell
python example/clean_data.py
```

然后运行以下命令从备份中恢复数据。

```shell
./milvus-backup restore -n my_backup
```

## 验证恢复的数据

恢复完成后，您可以通过对恢复的集合进行索引来验证恢复的数据，操作如下：

```shell
python example/verify_data.py
```

请注意，上述脚本假定您已经使用`-s`标志运行了`restore`命令，并且后缀设置为`-recover`。请随时根据需要对脚本进行必要更改。