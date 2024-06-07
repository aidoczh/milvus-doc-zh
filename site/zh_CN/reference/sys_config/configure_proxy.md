---
id: configure_proxy.md
related_key: configure
group: system_configuration.md
summary: 学习如何配置 Milvus 的代理。
title: 代理相关配置
---

# 代理相关配置

本主题介绍了 Milvus 的代理相关配置。

代理是系统的访问层，也是用户的端点。它验证客户端请求并减少返回的结果。

在本节中，您可以配置代理端口、系统限制等。

## `proxy.port`

<table id="proxy.port">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>代理的 TCP 端口。</td>
      <td>19530</td>
    </tr>
  </tbody>
</table>

## `proxy.grpc.serverMaxRecvSize`

<table id="proxy.grpc.serverMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>代理可以接收的每个 RPC 请求的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>536870912</td>
    </tr>
  </tbody>
</table>

## `proxy.grpc.serverMaxSendSize`

<table id="proxy.grpc.serverMaxSendSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在接收 RPC 请求时，代理可以发送的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>536870912</td>
    </tr>
  </tbody>
</table>

## `proxy.grpc.clientMaxRecvSize`

<table id="proxy.grpc.clientMaxRecvSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在发送 RPC 请求时，代理可以接收的每个响应的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>104857600</td>
    </tr>
  </tbody>
</table>

## `proxy.grpc.clientMaxSendSize`

<table id="proxy.grpc.clientMaxSendSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>代理可以发送的每个 RPC 请求的最大大小。</li>
        <li>单位：字节</li>
      </td>
      <td>104857600</td>
    </tr>
  </tbody>
</table>

## `proxy.timeTickInterval`

<table id="proxy.timeTickInterval">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>代理同步时间标记的间隔。</li>
        <li>单位：毫秒</li>
      </td>
      <td>200</td>
    </tr>
  </tbody>
</table>

## `proxy.msgStream.timeTick.bufSize`

<table id="proxy.msgStream.timeTick.bufSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        在生成消息时，代理的 timeTick 消息流中可以缓冲的最大消息数量。
      </td>
      <td>512</td>
    </tr>
  </tbody>
</table>

## `proxy.maxNameLength`

<table id="proxy.maxNameLength">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>在 Milvus 中可以创建的名称或别名的最大长度，包括集合名称、集合别名、分区名称和字段名称。</td>
      <td>255</td>
    </tr>
  </tbody>
</table>

## `proxy.maxFieldNum`

<table id="proxy.maxFieldNum">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>在创建集合时可以创建的字段的最大数量。</td>
      <td>64</td>
    </tr>
  </tbody>
</table>

## `proxy.maxDimension`

<table id="proxy.maxDimension">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>在创建集合时向量可以具有的最大维数。</td>
      <td>32768</td>
    </tr>
  </tbody>
</table>

## `proxy.maxShardNum`

<table id="proxy.maxShardNum">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>在创建集合时可以创建的分片的最大数量。</td>
      <td>64</td>
    </tr>
  </tbody>
</table>

## `proxy.maxTaskNum`

<table id="proxy.maxShardNum">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>代理的任务队列中可以存在的最大任务数量。</td>
      <td>1024</td>
    </tr>
  </tbody>
</table>

## `proxy.maxVectorFieldNum`

<table id="proxy.maxVectorFieldNum">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>在集合中可以指定的最大向量字段数量。取值范围：[1, 10]。</td>
      <td>4</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.enable`

<table id="proxy.accessLog.enable">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>是否启用访问日志功能。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.filename`

<table id="proxy.accessLog.filename">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
## `proxy.accessLog.localPath`

<table id="proxy.accessLog.localPath">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>存储访问日志文件的本地文件夹路径。当<code>proxy.accessLog.filename</code>不为空时，可以指定此参数。</td>
      <td>/tmp/milvus_access</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.maxSize`

<table id="proxy.accessLog.maxSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>单个访问日志文件允许的最大大小。如果日志文件大小达到此限制，将触发轮换过程。此过程封存当前访问日志文件，创建新的日志文件，并清空原始日志文件的内容。</li>
        <li>单位：MB</li>
      </td>
      <td>64</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.rotatedTime`

<table id="proxy.accessLog.rotatedTime">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>旋转单个访问日志文件允许的最大时间间隔。达到指定时间间隔后，将触发轮换过程，导致创建新的访问日志文件并封存先前的文件。</li>
        <li>单位：秒</li>
      </td>
      <td>0</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.maxBackups`

<table id="proxy.accessLog.maxBackups">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>可保留的封存访问日志文件的最大数量。如果封存的访问日志文件数量超过此限制，将删除最旧的文件。</td>
      <td>8</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.minioEnable`

<table id="proxy.accessLog.minioEnable">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>是否将本地访问日志文件上传到MinIO。当<code>proxy.accessLog.filename</code>不为空时，可以指定此参数。</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.remotePath`

<table id="proxy.accessLog.remotePath">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>用于上传访问日志文件的对象存储路径。</td>
      <td>access_log/</td>
    </tr>
  </tbody>
</table>
</tbody>
</table>

## `proxy.accessLog.remoteMaxTime`

<table id="proxy.accessLog.remoteMaxTime">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>允许上传访问日志文件的时间间隔。如果日志文件的上传时间超过此间隔，文件将被删除。将值设置为<code>0</code>将禁用此功能。</td>
      <td>0</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.base.format`

<table id="proxy.accessLog.base.format">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>具有动态指标的日志格式。此格式默认适用于所有方法。有关指标的更多信息，请参阅<a href="configure_access_logs.md">配置访问日志</a>。</td>
      <td>空字符串</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.<custom_formatter_name>.format`

<table id="proxy.accessLog.<custom_formatter_name>.format">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>您为特定方法配置的自定义日志格式。此参数与<code>proxy.accessLog.&lt;custom_formatter_name&gt;.methods</code>一起使用。</td>
      <td>空字符串</td>
    </tr>
  </tbody>
</table>

## `proxy.accessLog.<custom_formatter_name>.methods`

<table id="proxy.accessLog.<custom_formatter_name>.methods">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>自定义格式适用的方法。此参数与<code>proxy.accessLog.&lt;custom_formatter_name&gt;.format</code>一起使用。</td>
      <td>空字符串</td>
    </tr>
  </tbody>
</table>