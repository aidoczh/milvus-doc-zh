---
id: manage-cdc-tasks.md
order: 3
summary: 一个捕获数据更改（CDC）任务可以实现从源 Milvus 实例到目标 Milvus 实例的数据同步。
title: 管理 CDC 任务
---

# 管理 CDC 任务

一个捕获数据更改（CDC）任务可以实现从源 Milvus 实例到目标 Milvus 实例的数据同步。它监视源实例的操作日志，并实时复制数据更改，如插入、删除和索引操作到目标实例。这有助于实现 Milvus 部署之间的实时灾难恢复或主动-主动负载均衡。

本指南涵盖了如何管理 CDC 任务，包括通过 HTTP 请求进行创建、暂停、恢复、检索详情、列出和删除。

## 创建任务

创建 CDC 任务允许将源 Milvus 中的数据更改操作同步到目标 Milvus。

要创建 CDC 任务：

```bash
curl -X POST http://localhost:8444/cdc \
-H "Content-Type: application/json" \
-d '{
  "request_type": "create",
  "request_data": {
    "milvus_connect_param": {
      "host": "localhost",
      "port": 19530,
      "username": "root",
      "password": "Milvus",
      "enable_tls": false,
      "connect_timeout": 10
    },
    "collection_infos": [
      {
        "name": "*"
      }
    ],
    "rpc_channel_info": {
      "name": "by-dev-replicate-msg"
    }
  }
}'
```

将 __localhost__ 替换为目标 Milvus 服务器的 IP 地址。

__参数__：

- __milvus_connect_param__：目标 Milvus 的连接参数。

    - __host__：Milvus 服务器的主机名或 IP 地址。

    - __port__：Milvus 服务器监听的端口号。

    - __username__：用于与 Milvus 服务器进行身份验证的用户名。

    - __password__：用于与 Milvus 服务器进行身份验证的密码。

    - __enable_tls__：是否使用 TLS/SSL 加密进行连接。

    - __connect_timeout__：建立连接的超时时间，单位为秒。

- __collection_infos__：要同步的集合。目前仅支持星号（__*__），因为 Milvus-CDC 在集群级别进行同步，而不是单个集合。

- __rpc_channel_info__：用于同步的 RPC 通道名称，由源 Milvus 配置中的 __common.chanNamePrefix.cluster__ 和 __common.chanNamePrefix.replicateMsg__ 的值连接而成，中间用连字符（__-__）分隔。

预期响应：

```json
{
  "code": 200,
  "data": {
    "task_id":"xxxx"
  }
}
```

## 列出任务

要列出所有已创建的 CDC 任务：

```bash
curl -X POST -H "Content-Type: application/json" -d '{
  "request_type": "list"
}' http://localhost:8444/cdc
```

将 __localhost__ 替换为目标 Milvus 服务器的 IP 地址。

预期响应：
```json
{
  "code": 200,
  "data": {
    "tasks": [
      {
        "task_id": "xxxxx",
        "milvus_connect_param": {
          "host": "localhost",
          "port": 19530,
          "connect_timeout": 10
        },
        "collection_infos": [
          {
            "name": "*"
          }
        ],
        "state": "Running"
      }
    ]
  }
}
```
## 暂停任务

要暂停 CDC 任务：

```bash
curl -X POST -H "Content-Type: application/json" -d '{
  "request_type":"pause",
  "request_data": {
    "task_id": "xxxx"
  }
}' http://localhost:8444/cdc
```

将 __localhost__ 替换为目标 Milvus 服务器的 IP 地址。

__参数__：

- __task_id__: 要暂停的 CDC 任务的 ID。

预期响应：

```bash
{
  "code": 200,
  "data": {}
}
```

## 恢复任务

要恢复暂停的 CDC 任务：

```bash
curl -X POST -H "Content-Type: application/json" -d '{
  "request_type":"resume",
  "request_data": {
    "task_id": "xxxx"
  }
}' http://localhost:8444/cdc
```

将 __localhost__ 替换为目标 Milvus 服务器的 IP 地址。

__参数__：

- __task_id__: 要恢复的 CDC 任务的 ID。

预期响应：

```bash
{
  "code": 200,
  "data": {}
}
```

## 获取任务详情

要获取特定 CDC 任务的详情：

```bash
curl -X POST -H "Content-Type: application/json" -d '{
  "request_type":"get",
  "request_data": {
    "task_id": "xxxx"
  }
}' http://localhost:8444/cdc
```

将 __localhost__ 替换为目标 Milvus 服务器的 IP 地址。

__参数__：

- __task_id__: 要查询的 CDC 任务的 ID。

预期响应：

```bash
{
  "code": 200,
  "data": {
    "Task": {
      "collection_infos": [
        {
          "name": "*"
        }
      ],
      "milvus_connect_param": {
        "connect_timeout": 10,
        "enable_tls": true,
        "host": "localhost",
        "port": 19530
      },
      "state": "Running",
      "task_id": "xxxx"
    }
  }
}
```

## 删除任务

要删除 CDC 任务：

```bash
curl -X POST -H "Content-Type: application/json" -d '{
  "request_type":"delete",
  "request_data": {
    "task_id": "30d1e325df604ebb99e14c2a335a1421"
  }
}' http://localhost:8444/cdc
```

将 __localhost__ 替换为目标 Milvus 服务器的 IP 地址。

__参数__：

- __task_id__: 要删除的 CDC 任务的 ID。

预期响应：

```json
{
  "code": 200,
  "data": {}
}
```