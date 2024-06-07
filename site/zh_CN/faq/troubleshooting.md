---
id: troubleshooting.md
summary: 了解在使用 Milvus 时可能遇到的常见问题以及如何解决这些问题。
title: 故障排除
---
# 故障排除
本页面列出了在运行 Milvus 时可能出现的常见问题，以及可能的故障排除提示。本页面的问题分为以下几类：

- [启动问题](#boot_issues)
- [运行时问题](#runtime_issues)
- [API 问题](#api_issues)
- [etcd 崩溃问题](#etcd_crash_issues)


  ## 启动问题

  启动错误通常是致命的。运行以下命令查看错误详情：

  ```
  $ docker logs <your milvus container id>
  ```


  ## 运行时问题

  在运行时发生的错误可能导致服务中断。在继续之前，请检查服务器与客户端之间的兼容性以解决此问题。


  ## API 问题

  这些问题发生在 Milvus 服务器和客户端之间的 API 方法调用期间。它们将同步或异步地返回给客户端。
  

  ## etcd 崩溃问题
  
  ### 1. etcd pod pending

  etcd 集群默认使用 pvc。Kubernetes 集群需要预先配置 StorageClass。

  ### 2. etcd pod crash

  当 etcd pod 崩溃并显示 `Error: bad member ID arg (strconv.ParseUint: parsing "": invalid syntax), expecting ID in Hex` 时，您可以登录到该 pod 并删除 `/bitnami/etcd/data/member_id` 文件。

  ### 3. 多个 pod 持续崩溃，而 `etcd-0` 仍在运行

  如果多个 pod 持续崩溃，而 `etcd-0` 仍在运行，您可以运行以下代码。
  
  ```
  kubectl scale sts <etcd-sts> --replicas=1
  # 删除 etcd-1 和 etcd-2 的 pvc
  kubectl scale sts <etcd-sts> --replicas=3
  ```
  
  ### 4. 所有 pod 均崩溃
  
  当所有 pod 均崩溃时，请尝试复制 `/bitnami/etcd/data/member/snap/db` 文件。使用 `https://github.com/etcd-io/bbolt` 修改数据库数据。

  所有 Milvus 元数据都保存在 `key` 存储桶中。备份此存储桶中的数据并运行以下命令。请注意，`by-dev/meta/session` 文件中的前缀数据不需要备份。
  
  ```
  kubectl kubectl scale sts <etcd-sts> --replicas=0
  # 删除 etcd-0、etcd-1、etcd-2 的 pvc
  kubectl kubectl scale sts <etcd-sts> --replicas=1
  # 恢复备份数据
  ```



<br/>

  如果您需要帮助解决问题，请随时：

  - 加入我们的 [Slack 频道](https://join.slack.com/t/milvusio/shared_invite/enQtNzY1OTQ0NDI3NjMzLWNmYmM1NmNjOTQ5MGI5NDhhYmRhMGU5M2NhNzhhMDMzY2MzNDdlYjM5ODQ5MmE3ODFlYzU3YjJkNmVlNDQ2ZTk) 并寻求 Milvus 团队的支持。
  - 在 GitHub 上 [提交问题](https://github.com/milvus-io/milvus/issues/new/choose)，并提供有关您问题的详细信息。