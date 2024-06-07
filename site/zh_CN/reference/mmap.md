---
id: mmap.md
summary: MMap 可以在单个节点中存储更多数据。
title: MMap-启用的数据存储
---

# MMap-启用的数据存储

在 Milvus 中，内存映射文件允许将文件内容直接映射到内存中。这一特性提高了内存利用效率，特别是在可用内存稀缺但完整数据加载不可行的情况下。这种优化机制可以增加数据容量，同时确保性能达到一定限制；然而，当数据量远远超过内存时，搜索和查询性能可能会严重下降，因此请根据需要选择是否启用此功能。

## 配置内存映射

从 Milvus 2.4 开始，您可以灵活调整静态配置文件，以在部署前为整个集群配置默认内存映射设置。此外，您还可以动态调整参数，以微调集群和索引级别的内存映射设置。展望未来，未来的更新将扩展内存映射功能，包括字段级配置。

### 集群部署前：全局配置

在部署集群之前，__集群级别__ 的设置会应用于整个集群的内存映射。这确保所有新对象将自动遵循这些配置。需要注意的是，修改这些设置需要重新启动集群才能生效。

要调整集群的内存映射设置，请编辑 `configs/milvus.yaml` 文件。在该文件中，您可以指定是否默认启用内存映射，并确定用于存储内存映射文件的目录路径。如果未指定路径（`mmapDirPath`），系统将默认将内存映射文件存储在 `{localStorage.path}/mmap` 中。有关更多信息，请参阅[本地存储相关配置](https://milvus.io/docs/configure_localstorage.md#localStoragepath)。

```yaml
# 此参数在 configs/milvus.yaml 中设置
...
queryNode:
  mmap:
    # 为整个集群设置内存映射属性
    mmapEnabled: false | true
    # 设置内存映射目录路径，如果您未指定 mmapDirPath，则内存映射文件将默认存储在 {localStorage.path}/mmap 中。
    mmapDirPath: 任意/有效/路径
....
```

### 集群运行期间：动态配置

在集群运行时，您可以动态调整集合或索引级别的内存映射设置。

在__集合级别__，内存映射应用于集合中所有未索引的原始数据，不包括主键、时间戳和行 ID。这种方法特别适用于大型数据集的全面管理。

要在集合内动态调整内存映射设置，可使用 `set_properties()` 方法。在这里，您可以根据需要在 `mmap.enabled` 之间切换为 `True` 或 `False`。
```python
# 获取现有集合
collection = Collection("test_collection") # 请替换为您的集合名称

# 将内存映射属性设置为 True 或 Flase
collection.set_properties({'mmap.enabled': True})
```
对于__索引级别__的设置，可以将内存映射专门应用于向量索引，而不影响其他数据类型。这一功能对于需要针对向量搜索进行优化性能的集合非常宝贵。

要在集合内启用或禁用索引的内存映射，可以调用`alter_index()`方法，指定目标索引名称为`index_name`，并将`mmap.enabled`设置为`True`或`False`。

```python
collection.alter_index(
    index_name="vector_index", # 替换为您的向量索引名称
    extra_params={"mmap.enabled": True} # 启用索引的内存映射
)
```

## 在不同部署中自定义存储路径

内存映射文件默认存储在`localStorage.path`中的`/mmap`目录中。以下是如何在各种部署方法中自定义此设置：

- 对于使用 Helm Chart 安装的 Milvus：

```bash
# new-values.yaml
extraConfigFiles:
   user.yaml: |+
      queryNode:
         mmap:
           mmapEnabled: true
           mmapDirPath: any/valid/path
        
helm upgrade <milvus-release> --reuse-values -f new-values.yaml milvus/milvus
```

- 对于使用 Milvus Operator 安装的 Milvus：

```bash
# patch.yaml
spec:
  config:
    queryNode:
      mmap:
        mmapEnabled: true
        mmapDirPath: any/valid/path
      
 kubectl patch milvus <milvus-name> --patch-file patch.yaml
```

- 对于使用 Docker 安装的 Milvus：

```bash
# 提供了一个新的安装脚本来启用与内存映射相关的设置。
```

## 限制

- 无法为已加载的集合启用内存映射，请确保在启用内存映射之前已释放该集合。

- 不支持将内存映射用于 DiskANN 或 GPU 类型的索引。

## 常见问题

- __在哪些场景下建议启用内存映射？启用此功能后有哪些权衡？__

    当内存有限或性能要求适中时，建议启用内存映射。启用此功能可以增加数据加载的容量。例如，使用 2 个 CPU 和 8 GB 内存的配置，启用内存映射可以使加载的数据量增加至未启用时的 4 倍。性能影响因情况而异：

    - 内存充足时，预期性能类似于仅使用内存时的性能。

    - 内存不足时，预期性能可能会下降。

- __集合级别和索引级别配置之间有什么关系？__

    集合级别和索引级别不是包容关系，集合级别控制原始数据是否启用内存映射，而索引级别仅适用于向量索引。

- __有没有推荐的适用于内存映射的索引类型？__

    是的，推荐使用 HNSW 启用内存映射。我们之前测试过 HNSW、IVF_FLAT、IVF_PQ/SQ 系列索引，IVF 系列索引的性能严重下降，而对于 HNSW 索引启用内存映射的性能下降仍在预期范围内。
- __内存映射需要什么样的本地存储？__

    高质量的磁盘可以提升性能，NVMe 驱动器是首选选项。

- __标量数据可以进行内存映射吗？__

    内存映射可以应用于标量数据，但不适用于建立在标量字段上的索引。

- __如何确定不同级别内存映射配置的优先级？__

    在 Milvus 中，当明确定义了多个级别的内存映射配置时，索引级别和集合级别的配置具有最高优先级，然后是集群级别的配置。

- __如果我从 Milvus 2.3 升级，并已配置了内存映射目录路径，会发生什么？__

    如果您从 Milvus 2.3 升级，并已配置了内存映射目录路径（`mmapDirPath`），您的配置将被保留，内存映射启用的默认设置（`mmapEnabled`）将为 `true`。重要的是迁移元数据以同步现有内存映射文件的配置。有关更多详细信息，请参阅[Migrate the metadata](https://milvus.io/docs/upgrade_milvus_standalone-docker.md#Migrate-the-metadata)。