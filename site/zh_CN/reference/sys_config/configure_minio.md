---
id: configure_minio.md
related_key: configure
group: system_configuration.md
summary: 学习如何为 Milvus 配置 MinIO。
title: 与 MinIO 相关的配置
---

# 与 MinIO 相关的配置

本主题介绍了 Milvus 的与 MinIO 相关的配置。

Milvus 支持将 MinIO 和 Amazon S3 作为数据持久化的存储引擎，用于插入日志文件和索引文件。而 MinIO 是 S3 兼容性的事实标准，您可以直接在 MinIO 部分配置 S3 参数。

在本节中，您可以配置 MinIO 或 S3 的地址、相关访问密钥等。

<div class="alert note">
要在多个 Milvus 实例之间共享一个 MinIO 实例，您需要为每个 Milvus 实例更改 <code>minio.bucketName</code> 或 <code>minio.rootPath</code> 为唯一值。详情请参考 <a href="operational_faq.md#Can-I-share-a-MinIO-instance-among-multiple-Milvus-instances">操作常见问题</a>。
</div>

## `minio.address`

<table id="minio.address">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>MinIO 或 S3 服务的 IP 地址。</li>
        <li>环境变量：<code>MINIO_ADDRESS</code></li>
        <li><code>minio.address</code> 和 <code>minio.port</code> 一起生成对 MinIO 或 S3 服务的有效访问。</li>
        <li>当启动 Milvus 时，MinIO 优先从环境变量 <code>MINIO_ADDRESS</code> 获取有效的 IP 地址。</li>
        <li>当 MinIO 或 S3 在与 Milvus 相同的网络上运行时，将应用默认值。</li>
        <li>Milvus 2.0 不支持对 MinIO 或 S3 服务的安全访问。未来版本将支持对 MinIO 的安全访问。</li>
      </td>
      <td>localhost</td>
    </tr>
  </tbody>
</table>


## `minio.port`

<table id="minio.port">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>MinIO 或 S3 服务的端口。</li>
        <li>环境变量：<code>MINIO_ADDRESS</code></li>
        <li><code>minio.address</code> 和 <code>minio.port</code> 一起生成对 MinIO 或 S3 服务的有效访问。</li>
        <li>当启动 Milvus 时，MinIO 优先从环境变量 <code>MINIO_ADDRESS</code> 获取有效的端口。</li>
      </td>
      <td>9000</td>
    </tr>
  </tbody>
</table>

## `minio.accessKeyID`

<table id="minio.accessKeyID">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>MinIO 或 S3 分配给用户以进行授权访问的访问密钥 ID。</li>
        <li>环境变量：<code>MINIO_ACCESS_KEY_ID</code> 或 <code>minio.accessKeyID</code></li>

## `minio.accessKeyID` 和 `minio.secretAccessKey`

<table id="minio.accessKeyID">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>用于身份验证以访问 MinIO 或 S3 服务的身份验证密钥对。</li>
        <li>此配置必须与环境变量 <code>MINIO_ACCESS_KEY_ID</code> 设置为相同值，这对于启动 MinIO 或 S3 是必要的。</li>
        <li>默认值适用于使用默认的 <b>docker-compose.yml</b> 文件启动的 MinIO 或 S3 服务。</li>
      </td>
      <td>minioadmin</td>
    </tr>
  </tbody>
</table>

## `minio.secretAccessKey`

<table id="minio.secretAccessKey">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>用于加密签名字符串和验证服务器上签名字符串的秘钥。必须严格保密，只能由 MinIO 或 S3 服务器和用户访问。</li>
        <li>环境变量：<code>MINIO_SECRET_ACCESS_KEY</code> 或 <code>minio.secretAccessKey</code></li>
        <li>`minio.accessKeyID` 和 `minio.secretAccessKey` 一起用于身份验证以访问 MinIO 或 S3 服务。</li>
        <li>此配置必须与环境变量 <code>MINIO_SECRET_ACCESS_KEY</code> 设置为相同值，这对于启动 MinIO 或 S3 是必要的。</li>
        <li>默认值适用于使用默认的 <b>docker-compose.yml</b> 文件启动的 MinIO 或 S3 服务。</li>
      </td>
      <td>minioadmin</td>
    </tr>
  </tbody>
</table>

## `minio.useSSL`

<table id="minio.useSSL">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>用于控制是否通过 SSL 访问 MinIO 或 S3 服务的开关值。</li>
      </td>
      <td>false</td>
    </tr>
  </tbody>
</table>

## `minio.bucketName`

<table id="minio.bucketName">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Milvus 在 MinIO 或 S3 中存储数据的存储桶名称。</li>
        <li>Milvus 2.0.0 不支持在多个存储桶中存储数据。</li>
        <li>如果存储桶不存在，将创建具有此名称的存储桶。如果存储桶已存在且可访问，则将直接使用。否则将出现错误。</li>
        <li>要在多个 Milvus 实例之间共享一个 MinIO 实例，请考虑在启动它们之前为每个 Milvus 实例更改此值。有关详细信息，请参阅 <a href="operational_faq.md#Can-I-share-a-MinIO-instance-among-multiple-Milvus-instances">操作常见问题解答</a>。</li>
        <li>如果使用 Docker 在本地启动 MinIO 服务，则数据将存储在本地 Docker 中。请确保有足够的存储空间。</li>
      </td>
      <td></td>
    </tr>
  </tbody>
</table>
## `minio.rootPath`

<table id="minio.rootPath">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>MinIO或S3中Milvus存储数据的键的根前缀。</li>
        <li>建议在首次启动Milvus之前更改此参数。</li>
        <li>要在多个Milvus实例之间共享一个MinIO实例，请考虑在启动它们之前为每个Milvus实例更改此值。有关详细信息，请参见<a href="operational_faq.md#Can-I-share-a-MinIO-instance-among-multiple-Milvus-instances">操作常见问题</a>。</li>
        <li>如果etcd服务已存在，请为Milvus设置一个易于识别的根键前缀。</li>
        <li>更改已运行的Milvus实例的此参数可能导致无法读取旧数据。</li>
      </td>
      <td>files</td>
    </tr>
  </tbody>
</table>