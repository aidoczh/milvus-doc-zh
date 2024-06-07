---
id: configure_log.md
related_key: configure
group: system_configuration.md
summary: 学习如何配置 Milvus 的日志记录。
title: 与日志相关的配置
---

# 与日志相关的配置

本主题介绍了 Milvus 的与日志相关的配置。

使用 Milvus 会生成一系列日志。默认情况下，Milvus 使用日志记录调试或更高级别的信息，输出到标准输出（stdout）和标准错误（stderr）。

在本节中，您可以配置系统日志输出。

## `log.level`

<table id="log.level">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Milvus 的日志级别。</li>
        <li>选项：<code>debug</code>、<code>info</code>、<code>warn</code>、<code>error</code>、<code>panic</code> 和 <code>fatal</code></li>
        <li>建议在测试和开发环境下使用 <code>debug</code> 级别，在生产环境中使用 <code>info</code> 级别。</li>
      </td>
      <td><code>debug</code></td>
    </tr>
  </tbody>
</table>

## `log.file.rootPath`

<table id="log.file.rootPath">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>日志文件的根路径。</li>
        <li>默认值为空，表示将日志文件输出到标准输出（stdout）和标准错误（stderr）。</li>
        <li>如果将此参数设置为有效的本地路径，Milvus 将在该路径写入和存储日志文件。</li>
        <li>请将此参数设置为您有写入权限的路径。</li>
      </td>
      <td>""</td>
    </tr>
  </tbody>
</table>

## `log.file.maxSize`

<table id="log.file.maxSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>日志文件的最大大小。</li>
        <li>单位：MB</li>
      </td>
      <td>300</td>
    </tr>
  </tbody>
</table>

## `log.file.maxAge`

<table id="log.file.maxAge">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>日志文件自动清除前的最大保留时间。</li>
        <li>单位：天</li>
        <li>最小值为 1。</li>
        <li>此参数仅在设置了有效的 <code>log.file.rootPath</code> 后生效。</li>
      </td>
      <td>10</td>
    </tr>
  </tbody>
</table>

## `log.file.maxBackups`

<table id="log.file.maxBackups">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>要备份的日志文件的最大数量。</li>
        <li>单位：天</li>
## `log.level`

<table id="log.level">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>日志级别的最小值为1。</li>
        <li>仅在设置了有效的<code>log.file.rootPath</code>后，此参数才会生效。</li>
      </td>
      <td>20</td>
    </tr>
  </tbody>
</table>

## `log.format`

<table id="log.format">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Milvus的日志格式。</li>
        <li>选项：<code>text</code> 和 <code>JSON</code></li>
      </td>
      <td><code>text</code></td>
    </tr>
  </tbody>
</table>