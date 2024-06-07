---
id: authenticate.md
summary: 学习如何在Milvus中管理用户认证。
title: 用户访问认证
---

# 用户访问认证

本指南介绍如何在Milvus中管理用户认证，包括启用认证、以用户身份连接以及修改用户凭据。

<div class="alert note">

- TLS和用户认证是两种不同的安全方法。如果在Milvus系统中同时启用了用户认证和TLS，您必须提供用户名、密码和证书文件路径。有关如何启用TLS的信息，请参考[传输加密](tls.md)。

- 本页上的代码片段使用新的<a href="https://milvus.io/api-reference/pymilvus/v2.4.x/About.md">MilvusClient</a>（Python）与Milvus进行交互。将来会发布其他语言的新MilvusClient SDK。

</div>

## 启用用户认证

<div class="filter">
<a href="#docker">Docker Compose</a> <a href="#helm">Helm</a>
</div>

<div class="table-wrapper filter-docker" markdown="block">

在<a href="configure-docker.md">配置Milvus</a>时，将<code>milvus.yaml</code>中的<code>common.security.authorizationEnabled</code>设置为<code>true</code>以启用认证。

</div>

<div class="table-wrapper filter-helm" markdown="block">

从Milvus Helm Chart 4.0.0开始，您可以通过修改`values.yaml`来启用用户认证，如下所示：

<pre>
  <code>
extraConfigFiles:
  user.yaml: |+
    common:
      security:
        authorizationEnabled: true
  </code>
</pre>

</div>

## 使用认证连接到Milvus

启用认证后，您需要使用用户名和密码连接到Milvus。在Milvus初始化时，默认会创建`root`用户，密码为`Milvus`。以下是使用默认`root`用户连接到启用认证的Milvus的示例：

```python
# 使用默认的`root`用户连接到Milvus

from pymilvus import MilvusClient

client = MilvusClient(
    uri='http://localhost:19530', # 替换为您自己的Milvus服务器地址
    token="root:Milvus"
) 
```

<div class="alert note">
如果在启用认证的情况下连接到Milvus时未提供有效的令牌，将收到gRPC错误。
</div>

## 创建新用户

连接为默认的`root`用户后，您可以按以下方式创建和认证新用户：

```python
# 创建用户
client.create_user(
    user_name="user_1",
    password="P@ssw0rd",
)

# 验证用户已创建

client.describe_user("user_1")

# 输出
# {'user_name': 'user_1', 'roles': ()}
```

有关创建用户的更多信息，请参考[create_user()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Authentication/create_user.md)。

## 使用新用户连接到Milvus
```python
# 使用新创建的用户连接到 Milvus

client = MilvusClient(
    uri="http://localhost:19530",
    token="user_1:P@ssw0rd"
)
```
## 更新用户密码

使用以下代码更改现有用户的密码：

```python
# 更新密码

client.update_password(
    user_name="user_1",
    old_password="P@ssw0rd",
    new_password="P@ssw0rd123"
)
```

有关更新用户密码的更多信息，请参考 [update_password()](https://milvus.io/api-reference/pymilvus/v2.4.x/MilvusClient/Authentication/update_password.md)。

如果您忘记了旧密码，Milvus 提供了一个配置项，允许您指定某些用户为超级用户。这样在重置密码时就不需要提供旧密码了。

默认情况下，Milvus 配置文件中的 `common.security.superUsers` 字段为空，这意味着所有用户在重置密码时必须提供旧密码。但是，您可以指定特定用户为超级用户，这些用户在重置密码时无需提供旧密码。在下面的代码片段中，`root` 和 `foo` 被指定为超级用户。

您应该在管理 Milvus 实例运行的 Milvus 配置文件中添加以下配置项。

```yaml
common:
    security:
        superUsers: root, foo
```

## 删除用户

要删除用户，请使用 `drop_user()` 方法。

```python
client.drop_user(user_name="user_1")
```

<div class="alert note">
要删除用户，您不能是要被删除的用户。否则，将会引发错误。
</div>

## 列出所有用户

列出所有用户。

```python
# 列出所有用户

client.list_users()
```

## 限制

1. 用户名不能为空，长度不能超过32个字符。必须以字母开头，只能包含下划线、字母或数字。
2. 密码必须至少包含6个字符，长度不能超过256个字符。

## 下一步
- 您可能还想了解如何：
  - [扩展 Milvus 集群](scaleout.md)
- 如果您准备在云上部署您的集群：
  - 学习如何 [使用 Terraform 和 Ansible 在 AWS 上部署 Milvus](aws.md)
  - 学习如何 [使用 Terraform 在 Amazon EKS 上部署 Milvus](eks.md)
  - 学习如何 [使用 Kubernetes 在 GCP 上部署 Milvus 集群](gcp.md)
  - 学习如何 [使用 Kubernetes 在 Microsoft Azure 上部署 Milvus](azure.md)