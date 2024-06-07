---
id: configure_access_logs.md
title: 配置访问日志
---

# 配置访问日志

Milvus 中的访问日志功能允许服务器管理员记录和分析用户访问行为，帮助了解查询成功率和失败原因等方面。

本指南提供了有关在 Milvus 中配置访问日志的详细说明。

访问日志的配置取决于 Milvus 的安装方法：

- **Helm 安装**：在 `values.yaml` 中进行配置。更多信息，请参阅[使用 Helm Charts 配置 Milvus](configure-helm.md)。
- **Docker 安装**：在 `milvus.yaml` 中进行配置。更多信息，请参阅[使用 Docker Compose 配置 Milvus](configure-docker.md)。
- **Operator 安装**：修改配置文件中的 `spec.components`。更多信息，请参阅[使用 Milvus Operator 配置 Milvus](configure_operator.md)。

## 配置选项

根据您的需求，可以从三个配置选项中选择：

- **基本配置**：用于一般目的。
- **本地访问日志文件配置**：用于本地存储日志。
- **配置将本地访问日志上传至 MinIO**：用于云存储和备份。

### 基本配置

基本配置包括启用访问日志并定义日志文件名或使用 stdout。

```yaml
proxy:
  accessLog:
    enable: true
    # 如果 `filename` 为空，则日志将打印到 stdout。
    filename: ""
    # 其他格式化配置...
```

- `proxy.accessLog.enable`：是否启用访问日志功能。默认为 **false**。
- `proxy.accessLog.filename`：访问日志文件的名称。如果将此参数留空，访问日志将打印到 stdout。

### 本地访问日志文件配置

使用参数配置本地存储访问日志文件，包括本地文件路径、文件大小和轮换间隔：

```yaml
proxy:
  accessLog:
    enable: true
    filename: "access_log.txt" # 访问日志文件的名称
    localPath: "/var/logs/milvus" # 存储访问日志文件的本地文件路径
    maxSize: 500 # 每个单独访问日志文件的最大大小。单位：MB
    rotatedTime: 24 # 日志轮换的时间间隔。单位：秒
    maxBackups: 7 # 可保留的已封存访问日志文件的最大数量
    # 其他格式化配置...
```

这些参数在 `filename` 不为空时指定。

- `proxy.accessLog.localPath`：存储访问日志文件的本地文件路径。
- `proxy.accessLog.maxSize`：单个访问日志文件允许的最大大小（MB）。如果日志文件大小达到此限制，将触发轮换过程。此过程封存当前访问日志文件，创建新的日志文件，并清除原始日志文件的内容。
- `proxy.accessLog.rotatedTime`: 旋转单个访问日志文件允许的最长时间间隔（以秒为单位）。达到指定时间间隔后，将触发旋转过程，创建新的访问日志文件并封存先前的文件。
- `proxy.accessLog.maxBackups`: 可保留的已封存访问日志文件的最大数量。如果已封存访问日志文件的数量超过此限制，最旧的文件将被删除。

### 配置上传本地访问日志文件到 MinIO

启用并配置设置以将本地访问日志文件上传到 MinIO：

```yaml
proxy:
  accessLog:
    enable: true
    filename: "access_log.txt"
    localPath: "/var/logs/milvus"
    maxSize: 500
    rotatedTime: 24 
    maxBackups: 7
    minioEnable: true
    remotePath: "/milvus/logs/access_logs"
    remoteMaxTime: 0
    # 其他格式配置...
```

在配置 MinIO 参数时，请确保已设置 `maxSize` 或 `rotatedTime`。否则可能导致无法成功将本地访问日志文件上传到 MinIO。

- `proxy.accessLog.minioEnable`: 是否将本地访问日志文件上传到 MinIO。默认为 **false**。
- `proxy.accessLog.remotePath`: 用于上传访问日志文件的对象存储路径。
- `proxy.accessLog.remoteMaxTime`: 允许上传访问日志文件的时间间隔。如果日志文件的上传时间超过此间隔，文件将被删除。将值设置为 0 可禁用此功能。

## 格式化器配置

所有方法使用的默认日志格式为 `base` 格式，不需要特定方法关联。但是，如果您希望为特定方法自定义日志输出，可以定义自定义日志格式并将其应用于相关方法。

```yaml
proxy:
  accessLog:
    enable: true
    filename: "access_log.txt"
    localPath: "/var/logs/milvus"
    # 为具有格式和适用方法的访问日志定义自定义格式化器
    formatters:
      # `base` 格式化器默认适用于所有方法
      # `base` 格式化器不需要特定方法关联
      base: 
        # 格式字符串；空字符串表示无日志输出
        format: "[$time_now] [ACCESS] <$user_name: $user_addr> $method_name-$method_status-$error_code [traceID: $trace_id] [timeCost: $time_cost]"
      # 特定方法的自定义格式化器（例如，Query，Search）
      query: 
        format: "[$time_now] [ACCESS] <$user_name: $user_addr> $method_status-$method_name [traceID: $trace_id] [timeCost: $time_cost] [database: $database_name] [collection: $collection_name] [partitions: $partition_name] [expr: $method_expr]"
        # 指定此自定义格式化器适用的方法
        methods: ["Query", "Search"]
```

- `proxy.accessLog.<formatter_name>.format`: 定义具有动态指标的日志格式。有关更多信息，请参阅 [支持的指标](#reference-supported-metrics)。
- `proxy.accessLog.<formatter_name>.methods`: 列出使用此格式化程序的 Milvus 操作。要获取方法名称，请参阅[Milvus 方法](https://github.com/milvus-io/milvus-proto/blob/master/proto/milvus.proto)中的 **MilvusService**。

## 参考文献：支持的指标

| 指标名称           | 描述                                                                      |
|--------------------|---------------------------------------------------------------------------|
| `$method_name`     | 方法名称                                                                  |
| `$method_status`   | 访问状态：**OK** 或 **Fail**                                              |
| `$method_expr`     | 用于查询、搜索或删除操作的表达式                                          |
| `$trace_id`        | 与访问相关的 TraceID                                                      |
| `$user_addr`       | 用户的 IP 地址                                                            |
| `$user_name`       | 用户名称                                                                  |
| `$response_size`   | 响应数据的大小                                                            |
| `$error_code`      | Milvus 特定的错误代码                                                     |
| `$error_msg`       | 详细的错误消息                                                            |
| `$database_name`   | 目标 Milvus 数据库的名称                                                  |
| `$collection_name` | 目标 Milvus 集合的名称                                                    |
| `$partition_name`  | 目标 Milvus 分区的名称或名称                                              |
| `$time_cost`       | 完成访问所花费的时间                                                      |
| `$time_now`        | 访问日志打印的时间（通常等同于 `$time_end`）                              |
| `$time_start`      | 访问开始的时间                                                            |
| `$time_end`        | 访问结束的时间                                                            |
| `$sdk_version`     | 用户使用的 Milvus SDK 版本                                                 |