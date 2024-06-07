---
id: resource_group.md
related_key: 管理资源组
summary: 学习如何管理资源组。
title: 管理资源组
---

# 管理资源组

在 Milvus 中，您可以使用资源组将某些查询节点与其他节点物理隔离开来。本指南将指导您如何创建和管理自定义资源组，以及在它们之间转移节点。

## 什么是资源组

资源组可以容纳 Milvus 集群中的若干个或所有查询节点。您可以根据自己的需求决定如何在资源组之间分配查询节点。例如，在多集合场景中，您可以为每个资源组分配适当数量的查询节点，并将集合加载到不同的资源组中，以便每个集合内的操作在物理上独立于其他集合。

请注意，Milvus 实例在启动时会维护一个默认资源组来容纳所有查询节点，并将其命名为 **__default_resource_group**。您可以将一些节点从默认资源组移动到您创建的资源组中。

## 管理资源组

<div class="alert note">

本页上的所有代码示例均使用 PyMilvus {{var.milvus_python_sdk_real_version}}。在运行这些示例之前，请升级您的 PyMilvus 安装。

</div>

1. 创建资源组。

    要创建资源组，请在连接到 Milvus 实例后运行以下代码。以下代码片段假定 `default` 是您 Milvus 连接的别名。

    ```Python
    import pymilvus

    # 资源组名称应为 1 到 255 个字符的字符串，以字母或下划线 (_) 开头，只包含数字、字母和下划线 (_)
    name = "rg"

    try:
        utility.create_resource_group(name, using='default')
        print(f"成功创建资源组 {name}。")
    except Exception:
        print("创建资源组失败。")

    # 成功创建资源组 rg.
    ```

2. 列出资源组。

    创建资源组后，您可以在资源组列表中看到它。

    要查看 Milvus 实例中资源组的列表，请执行以下操作：

    ```Python
    rgs = utility.list_resource_groups(using='default')
    print(f"资源组列表: {rgs}")

    # 资源组列表: ['__default_resource_group', 'rg']
    ```

3. 描述资源组。

    您可以让 Milvus 描述感兴趣的资源组，如下所示：

```Python
    info = utility.describe_resource_group(name, using="default")
    print(f"资源组描述: {info}")
    
    # 资源组描述: 
    #        <name:"rg">,           // 字符串, rg 名称
    #        <capacity:1>,            // 整数, 已转移至此 rg 的节点数
    #        <num_available_node:0>,  // 整数, 可用节点数，一些节点可能已关闭
    #        <num_loaded_replica:{}>, // map[string]int, from collection_name to loaded replica of each collecion in this rg
    #        <num_outgoing_node:{}>,  // map[string]int, from collection_name to outgoging accessed node num by replica loaded in this rg 
    #        <num_incoming_node:{}>.  // map[string]int, from collection_name to incoming accessed node num by replica loaded in other rg
```

4. 在资源组之间转移节点。

您可能注意到所描述的资源组尚未具有任何查询节点。按照以下步骤，将一些节点从默认资源组移动到您创建的资源组中：

```Python
source = '__default_resource_group'
target = 'rg'
num_nodes = 1

try:
    utility.transfer_node(source, target, num_nodes, using="default")
    print(f"成功从 {source} 移动 {num_node} 个节点到 {target}。")
except Exception:
    print("移动节点时出现问题。")

# 成功从 __default_resource_group 移动 1 个节点到 rg.
```

5. 将集合和分区加载到资源组中。

一旦资源组中有查询节点，您就可以将集合加载到该资源组中。以下代码片段假定已经存在名为 `demo` 的集合。

```Python
from pymilvus import Collection

collection = Collection('demo')

# Milvus将集合加载到默认资源组中。
collection.load(replica_number=2)

# 或者，您可以要求Milvus将集合加载到所需的资源组中。
# 确保查询节点数大于等于副本数
resource_groups = ['rg']
collection.load(replica_number=2, _resource_groups=resource_groups) 
```

    此外，您还可以将一个分区加载到资源组中，并将其副本分布在多个资源组中。以下代码假定已经存在名为 `Books` 的集合，并且该集合有一个名为 `Novels` 的分区。

```Python
collection = Collection("Books")

# 使用集合的加载方法加载其分区之一
collection.load(["Novels"], replica_number=2, _resource_groups=resource_groups)

# 或者，您可以直接使用分区的加载方法
partition = Partition(collection, "Novels")
partition.load(replica_number=2, _resource_groups=resource_groups)
```

    请注意，`_resource_groups` 是一个可选参数，如果不指定，Milvus将副本加载到默认资源组中的查询节点上。
    
    要求Milvus将集合的每个副本加载到单独的资源组中，请确保资源组的数量等于副本的数量。

6. 在资源组之间转移副本。

    Milvus使用[副本](replica.md)来实现分布在多个查询节点上的[段](glossary.md#Segment)之间的负载均衡。您可以按照以下步骤将集合的某些副本从一个资源组移动到另一个资源组：
```Python
source = '__default_resource_group'
target = 'rg'
collection_name = 'c'
num_replicas = 1

try:
    utility.transfer_replica(source, target, collection_name, num_replicas, using="default")
    print(f"成功将 {collection_name} 的 {num_replicas} 个副本从 {source} 移动到 {target}。")
except Exception:
    print("移动副本时出现问题。")

# 成功将 1 个副本的 c 从 __default_resource_group 移动到 rg.
```

7. 删除资源组。

    只有在资源组中没有查询节点时，才能随时删除资源组。在本指南中，资源组 `rg` 现在有一个查询节点。在删除此资源组之前，您需要将其移动到另一个资源组。

    ```Python
    source = 'rg'
    target = '__default_resource_group'
    num_nodes = 1
    
    try:
        utility.transfer_node(source, target, num_nodes, using="default")
        utility.drop_resource_group(source, using="default")
        print(f"成功删除 {source}。")
    except Exception:
        print(f"删除 {source} 时出现问题。")
    ```

# 接下来做什么

要部署多租户 Milvus 实例，请阅读以下内容：

- [启用 RBAC](rbac.md)
- [用户和角色](users_and_roles.md)
```