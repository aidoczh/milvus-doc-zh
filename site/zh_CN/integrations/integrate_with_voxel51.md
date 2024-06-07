---
id: integrate_with_voxel51.md
summary: 本页面讨论了与 voxel51 的集成
title: 使用 Milvus 和 FiftyOne 进行视觉搜索
---

# 使用 Milvus 和 FiftyOne 进行视觉搜索

[FiftyOne](https://docs.voxel51.com/) 是一个用于构建高质量数据集和计算机视觉模型的开源工具。本指南帮助您将 Milvus 的相似度搜索功能集成到 FiftyOne 中，从而能够在自己的数据集上进行视觉搜索。

FiftyOne 提供了一个 API，可以通过编程方式在 Python 中创建 Milvus 集合、上传向量并运行相似度查询，也可以通过应用程序中的点-and-click 进行操作。本页面的演示重点是程序化集成。

## 先决条件

开始之前，请确保您具备以下条件：

- 运行中的 [Milvus 服务器](install_standalone-docker.md)。
- 安装了带有 `pymilvus` 和 `fiftyone` 的 Python 环境。
- 一组要搜索的图像 [数据集](https://docs.voxel51.com/user_guide/dataset_creation/index.html#loading-datasets)。

## 安装要求

在本示例中，我们将使用 `pymilvus` 和 `fiftyone`。您可以通过运行以下命令来安装它们：

```shell
python3 -m pip install pymilvus fiftyone torch torchvision
```

## 基本步骤

使用 Milvus 在 FiftyOne 数据集上创建相似性索引，并使用该索引查询数据的基本工作流程如下：

1. 将一个 [数据集](https://docs.voxel51.com/user_guide/dataset_creation/index.html#loading-datasets) 加载到 FiftyOne 中。
2. 为数据集中的样本或补丁计算向量嵌入，或选择一个模型来生成嵌入。
3. 使用 [`compute_similarity()`](https://docs.voxel51.com/api/fiftyone.brain.html#fiftyone.brain.compute_similarity) 方法通过设置参数 `backend="milvus"` 并指定您选择的 `brain_key` 为数据集中的样本或对象补丁生成 Milvus 相似性索引。
4. 使用这个 Milvus 相似性索引通过 [`sort_by_similarity()`](https://docs.voxel51.com/api/fiftyone.core.collections.html#fiftyone.core.collections.SampleCollection.sort_by_similarity) 查询您的数据。
5. 如果需要，删除该索引。

## 步骤

下面的示例演示了上述工作流程。

### 1. 将数据集加载到 FiftyOne 中，并为样本计算嵌入

以下代码使用 FiftyOne 提供的示例图像集来演示集成。您可以参考[这篇文章](https://docs.voxel51.com/user_guide/dataset_creation/index.html#loading-datasets)准备自己的图像集。

```python
import fiftyone as fo
import fiftyone.brain as fob
import fiftyone.zoo as foz

# 步骤 1：将数据加载到 FiftyOne 中
dataset = foz.load_zoo_dataset("quickstart")

# 步骤 2 和 3：计算嵌入并创建相似性索引
milvus_index = fob.compute_similarity(
    dataset,
    brain_key="milvus_index",
    backend="milvus",
)
```
### 2. 进行视觉相似性搜索

您现在可以使用 Milvus 相似性索引在您的数据集上进行视觉相似性搜索。

```python
# 步骤 4: 查询您的数据
query = dataset.first().id  # 按样本 ID 查询
view = dataset.sort_by_similarity(
    query,
    brain_key="milvus_index",
    k=10,  # 限制为最相似的 10 个样本
)

# 步骤 5（可选）: 清理

# 删除 Milvus 集合
milvus_index.cleanup()

# 从 FiftyOne 中删除运行记录
dataset.delete_brain_run("milvus_index")
```

### 3. 删除索引

如果您不再需要 Milvus 相似性索引，可以使用以下代码进行删除：

```python
# 步骤 5: 删除索引
milvus_index.delete()
```

## 使用 Milvus 后端

默认情况下，调用 [`compute_similarity()`](https://docs.voxel51.com/api/fiftyone.brain.html#fiftyone.brain.compute_similarity) 或 `sort_by_similarity()` 将使用 sklearn 后端。

要使用 Milvus 后端，只需将 [`compute_similarity()`](https://docs.voxel51.com/api/fiftyone.brain.html#fiftyone.brain.compute_similarity) 的可选后端参数设置为 `"milvus"`：

```python
import fiftyone.brain as fob

fob.compute_similarity(..., backend="milvus", ...)
```

或者，您可以通过设置以下环境变量永久配置 FiftyOne 使用 Milvus 后端：

```shell
export FIFTYONE_BRAIN_DEFAULT_SIMILARITY_BACKEND=milvus
```

或者通过设置位于 `~/.fiftyone/brain_config.json` 的 [brain config](https://docs.voxel51.com/user_guide/brain.html#brain-config) 的 `default_similarity_backend` 参数：

```json
{
    "default_similarity_backend": "milvus"
}
```

## 认证

如果您使用自定义的 Milvus 服务器，可以通过多种方式提供您的凭据。

### 环境变量（推荐）

配置 Milvus 凭据的推荐方式是将它们存储在下面显示的环境变量中，FiftyOne 在连接到 Milvus 时会自动访问这些变量。

```python
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_URI=XXXXXX
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_USER=XXXXXX
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_PASSWORD=XXXXXX

# 如果需要也可以使用以下变量
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_SECURE=true
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_TOKEN=XXXXXX
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_DB_NAME=XXXXXX
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_CLIENT_KEY_PATH=XXXXXX
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_CLIENT_PEM_PATH=XXXXXX
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_CA_PEM_PATH=XXXXXX
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_SERVER_PEM_PATH=XXXXXX
export FIFTYONE_BRAIN_SIMILARITY_MILVUS_SERVER_NAME=XXXXXX
```

### FiftyOne Brain 配置

您还可以将您的凭据存储在位于 `~/.fiftyone/brain_config.json` 的 [brain config](https://docs.voxel51.com/user_guide/brain.html#brain-config) 中：
```python
{
    "相似度后端": {
        "Milvus": {
            "URI": "XXXXXX",
            "用户": "XXXXXX",
            "密码": "XXXXXX",

            # 如有必要，还可使用以下参数
            "安全连接": true,
            "令牌": "XXXXXX",
            "数据库名称": "XXXXXX",
            "客户端密钥路径": "XXXXXX",
            "客户端证书路径": "XXXXXX",
            "CA证书路径": "XXXXXX",
            "服务器证书路径": "XXXXXX",
            "服务器名称": "XXXXXX"
        }
    }
}
```
请注意，在您创建文件之前，该文件将不存在。

### 关键字参数

您可以在每次调用像[`compute_similarity()`](https://docs.voxel51.com/api/fiftyone.brain.html#fiftyone.brain.compute_similarity)这样需要连接到 Milvus 的方法时，手动提供您的 Milvus 凭据作为关键字参数：

```python
import fiftyone.brain as fob

milvus_index = fob.compute_similarity(
    ...
    backend="milvus",
    brain_key="milvus_index",
    uri="XXXXXX",
    user="XXXXXX",
    password="XXXXXX",

    # 如果需要，也可以使用以下参数
    secure=True,
    token="XXXXXX",
    db_name="XXXXXX",
    client_key_path="XXXXXX",
    client_pem_path="XXXXXX",
    ca_pem_path="XXXXXX",
    server_pem_path="XXXXXX",
    server_name="XXXXXX",
)
```

请注意，使用此策略时，当以后通过[`load_brain_results()`](https://docs.voxel51.com/api/fiftyone.core.collections.html#fiftyone.core.collections.SampleCollection.load_brain_results)加载索引时，您必须手动提供凭据：

```python
milvus_index = dataset.load_brain_results(
    "milvus_index",
    uri="XXXXXX",
    user="XXXXXX",
    password="XXXXXX",

    # 如果需要，也可以使用以下参数
    secure=True,
    token="XXXXXX",
    db_name="XXXXXX",
    client_key_path="XXXXXX",
    client_pem_path="XXXXXX",
    ca_pem_path="XXXXXX",
    server_pem_path="XXXXXX",
    server_name="XXXXXX",
)
```

### Milvus 配置参数

Milvus 后端支持各种查询参数，可用于自定义相似性查询。这些参数包括：

- **collection_name** (*None*): 要使用或创建的 Milvus 集合的名称。如果未提供，则将创建新集合

- **metric** (*"dotproduct"*): 创建新索引时要使用的嵌入距离度量。支持的值有 (`"dotproduct"`, `"euclidean"`)

- **consistency_level** (*"Session"*): 要使用的一致性级别。支持的值有 (`"Strong"`, `"Session"`, `"Bounded"`, `"Eventually"`)

有关这些参数的详细信息，请参阅[Milvus 认证文档](authenticate.md)和[Milvus 一致性级别文档](consistency.md)。

您可以通过前一节描述的任何策略之一指定这些参数。以下是包含所有可用参数的[brain 配置](https://docs.voxel51.com/user_guide/brain.html#brain-config)示例：

```json
{
    "similarity_backends": {
        "milvus": {
            "collection_name": "your_collection",
            "metric": "dotproduct",
            "consistency_level": "Strong"
        }
    }
}
```

然而，通常这些参数直接传递给[`compute_similarity()`](https://docs.voxel51.com/api/fiftyone.brain.html#fiftyone.brain.compute_similarity)以配置特定的新索引：
```python
milvus_index = fob.compute_similarity(
    ...
    backend="milvus",
    brain_key="milvus_index",
    collection_name="your_collection",
    metric="dotproduct",
    consistency_level="Strong",
)
```
## 管理脑运行

FiftyOne提供了多种方法，您可以使用这些方法来管理脑运行。

例如，您可以调用 [`list_brain_runs()`](https://docs.voxel51.com/api/fiftyone.core.collections.html#fiftyone.core.collections.SampleCollection.list_brain_runs) 来查看数据集上可用的脑键：

```python
import fiftyone.brain as fob

# 列出所有脑运行
dataset.list_brain_runs()

# 仅列出相似性运行
dataset.list_brain_runs(type=fob.Similarity)

# 仅列出特定的相似性运行
dataset.list_brain_runs(
    type=fob.Similarity,
    patches_field="ground_truth",
    supports_prompts=True,
)
```

或者，您可以使用 [`get_brain_info()`](https://docs.voxel51.com/api/fiftyone.core.collections.html#fiftyone.core.collections.SampleCollection.get_brain_info) 来检索有关脑运行配置的信息：

```python
info = dataset.get_brain_info(brain_key)
print(info)
```

使用 [`load_brain_results()`](https://docs.voxel51.com/api/fiftyone.core.collections.html#fiftyone.core.collections.SampleCollection.load_brain_results) 来加载脑运行的 [`SimilarityIndex`](https://docs.voxel51.com/api/fiftyone.brain.similarity.html#fiftyone.brain.similarity.SimilarityIndex) 实例。

您可以使用 [`rename_brain_run()`](https://docs.voxel51.com/api/fiftyone.core.collections.html#fiftyone.core.collections.SampleCollection.rename_brain_run) 来重命名与现有相似性结果运行相关联的脑键：

```python
dataset.rename_brain_run(brain_key, new_brain_key)
```

最后，您可以使用 [`delete_brain_run()`](https://docs.voxel51.com/api/fiftyone.core.collections.html#fiftyone.core.collections.SampleCollection.delete_brain_run) 来删除脑运行：

```python
dataset.delete_brain_run(brain_key)
```

<div class="alert note">

调用 [`delete_brain_run()`](https://docs.voxel51.com/api/fiftyone.core.collections.html#fiftyone.core.collections.SampleCollection.delete_brain_run) 仅会从您的FiftyOne数据集中删除脑运行的记录；它不会删除任何关联的Milvus集合，您可以按以下方式执行：

```python
# 删除Milvus集合
milvus_index = dataset.load_brain_results(brain_key)
milvus_index.cleanup()
```

</div>

对于在FiftyOne数据集上使用Milvus后端进行常见向量搜索工作流程，请参阅[这里的示例](https://docs.voxel51.com/integrations/milvus.html#examples)。