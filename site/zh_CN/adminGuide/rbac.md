---
id: rbac.md
related_key: 启用 RBAC
summary: 学习如何管理用户、角色和权限。
title: 启用 RBAC
---

# 启用 RBAC

通过启用 RBAC，您可以基于用户角色和权限控制对特定 Milvus 资源（例如集合或分区）或权限的访问。目前，此功能仅在 Python 和 Java 中可用。

本主题描述了如何启用 RBAC 并管理[用户和角色](users_and_roles.md)。

<div class="alert note">

本页面上的代码片段使用新的<a href="https://milvus.io/api-reference/pymilvus/v2.4.x/About.md">MilvusClient</a>（Python）与 Milvus 进行交互。其他语言的新 MilvusClient SDK 将在未来更新中发布。

</div>

## 1. 初始化 Milvus 客户端以建立连接

在启用[用户认证](authenticate.md)后，使用包含用户名和密码的 `token` 连接到您的 Milvus 实例。默认情况下，Milvus 使用用户名为 `root`，密码为 `Milvus` 的用户。

```python
from pymilvus import MilvusClient

client = MilvusClient(
    uri='http://localhost:19530', # 替换为您自己的 Milvus 服务器地址
    token='root:Milvus' # 替换为您自己的 Milvus 服务器 token
)
```

## 2. 创建用户

创建一个名为 `user_1`，密码为 `P@ssw0rd` 的用户：

```python
client.create_user(
    user_name='user_1',
    password='P@ssw0rd'
)
```

创建用户后，您可以：

- 更新用户密码。您需要提供原始密码和新密码。

```python
client.update_password(
    user_name='user_1',
    old_password='P@ssw0rd',
    new_password='P@ssw0rd123'
)
```

- 列出所有用户。

```python
client.list_users()

# 输出:
# ['root', 'user_1']
```

- 检查特定用户的角色。

```python
client.describe_user(user_name='user_1')

# 输出:
# {'user_name': 'user_1', 'roles': ()}
```

## 3. 创建角色

以下示例创建一个名为 `roleA` 的角色。

```python
client.create_role(
    role_name="roleA",
)
```

创建角色后，您可以：

- 列出所有角色。

```python
client.list_roles()

# 输出:
# ['admin', 'public', 'roleA']
```

## 4. 授予角色权限

以下示例演示如何将搜索所有集合的权限授予名为 `roleA` 的角色。有关您可以授予的其他类型权限，请参阅[用户和角色](users_and_roles.md)。

在管理角色权限之前，请确保已启用用户认证。否则，可能会出现错误。有关如何启用用户认证的信息，请参阅[认证用户访问](authenticate.md)。

```python
# 授予权限给角色

client.grant_privilege(
    role_name='roleA',
    object_type='User',
    object_name='SelectUser',
    privilege='SelectUser'
)
```

授予角色权限后，您可以：

- 查看授予角色的权限。
```python
client.describe_role(
    role_name='roleA'
)

# 输出:
# {'role': 'roleA',
#  'privileges': [{'object_type': 'User',
#    'object_name': 'SelectUser',
#    'db_name': 'default',
#    'role_name': 'roleA',
#    'privilege': 'SelectUser',
#    'grantor_name': 'root'}]}
```

## 5. 给用户授予角色

将角色授予用户，以便该用户可以继承角色的所有权限。

```python
# 给用户授予角色

client.grant_role(
    user_name='user_1',
    role_name='roleA'
)
```

授予角色后，验证是否已授予：

```python
client.describe_user(
    user_name='user_1'
)

# 输出:
# {'user_name': 'user_1', 'roles': ('roleA',)}
```

## 6. 撤销权限

<div class="alert caution">

执行以下操作时要小心，因为这些操作是不可逆转的。

</div>

- 从角色中移除权限。如果撤销了未授予角色的权限，将会出现错误。

```python
client.revoke_privilege(
    role_name='roleA',
    object_type='User',
    object_name='SelectUser',
    privilege='SelectUser'
)
```

- 从用户中移除角色。如果撤销了未授予用户的角色，将会出现错误。

```python
client.revoke_role(
    user_name='user_1',
    role_name='roleA'
)
```

- 删除角色。

```python
client.drop_role(role_name='roleA')
```

- 删除用户。

```python
client.drop_user(user_name='user_1')
```

## 下一步

- 学习如何管理[用户认证](authenticate.md)。

- 学习如何在Milvus中启用[TLS代理](tls.md)。