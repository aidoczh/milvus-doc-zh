---
id: roadmap.md
title: Milvus路线图
related_key: Milvus roadmap
summary: Milvus是一个用于支持人工智能应用的开源向量数据库。以下是我们的路线图，指导我们的发展方向。
---

# Milvus路线图

欢迎来到Milvus路线图！加入我们持续发展和演进Milvus的旅程。我们很高兴与您分享我们的成就、未来计划以及我们对未来的愿景。我们的路线图不仅仅是即将推出功能的列表，更体现了我们对创新的承诺和与社区合作的奉献精神。我们邀请您深入了解我们的路线图，提供反馈，并帮助塑造Milvus的未来！

## 路线图

| Category | Milvus 2.4.0（最近实现） | Milvus 2.5.0（预计在CY24年中发布） | 未来路线图（预计在CY24年内发布Milvus 3.0） |
| --- | --- | --- | --- |
| **AI开发者友好**<br><i>一个开发者友好的技术栈，融合了最新的人工智能创新</i> | **多向量与混合搜索**<br><i>用于多路召回和融合的框架</i><br><br>**GPU索引加速**<br><i>支持更高的QPS和更快的索引创建</i><br><br>**PyMilvus中的模型库**<br><i>为Milvus集成的嵌入模型</i> | **稀疏向量（GA）**<br><i>本地特征提取和关键词搜索</i><br><br>**Milvus Lite（GA）**<br><i>Milvus的轻量级内存版本</i><br><br>**嵌入模型库**<br><i>支持图像和多模态嵌入以及模型库中的重新排序模型</i> | **原始数据输入和输出**<br><i>支持Blob数据类型</i><br><br>**数据聚类**<br><i>数据共定位</i><br><br>**面向场景的向量搜索**<br><i>例如多目标搜索和NN过滤</i><br><br>**支持嵌入和重新排序端点** |
| **丰富功能**<br><i>增强的检索和数据管理功能</i> | **支持FP16、BF16数据类型**<br><i>这些机器学习数据类型有助于减少内存使用</i><br><br>**分组搜索**<br><i>聚合分割嵌入</i><br><br>**模糊匹配和倒排索引**<br><i>支持模糊匹配和倒排索引，适用于像varchar和int这样的标量类型</i> | 
<table>
    <tr>
        <td><strong>数组和 JSON 的倒排索引</strong><br><i>为数组和部分支持 JSON 进行索引</i><br><br><strong>位集索引</strong><br><i>提高执行速度和未来数据聚合</i><br><br><strong>截断集合</strong><br><i>允许在保留元数据的同时清除数据</i><br><br><strong>支持 NULL 和默认值</strong></td>
        <td><strong>支持更多数据类型</strong><br><i>例如日期时间、GIS</i><br><br><strong>高级文本过滤</strong><br><i>例如短语匹配</i><br><br><strong>主键去重</strong></td>
    </tr>
    <tr>
        <td><strong>成本效率与架构</strong><br><i>强调稳定性、成本效率、可扩展性和性能的先进系统</i></td>
        <td><strong>支持更多集合/分区</strong><br><i>在较小的集群中处理超过 10,000 个集合</i><br><br><strong>Mmap 优化</strong><br><i>在降低内存消耗和延迟之间取得平衡</i><br><br><strong>批量插入优化</strong><br><i>简化导入大型数据集</i></td>
        <td><strong>延迟加载</strong><br><i>数据通过读操作按需加载</i><br><br><strong>主要压缩</strong><br><i>根据配置重新分布数据以增强读取性能</i><br><br><strong>用于增长数据的 Mmap</strong><br><i>为扩展数据段而设计的 Mmap 文件</i></td>
        <td><strong>内存控制</strong><br><i>减少内存不足问题并提供全局内存管理</i><br><br><strong>LogNode 介绍</strong><br><i>确保全局一致性并解决根协调中的单点瓶颈</i><br><br><strong>存储格式 V2</strong><br><i>通用格式设计为基于磁盘的数据访问奠定基础</i></td>
    </tr>
    <tr>
        <td><strong>企业就绪</strong><br><i>旨在满足企业生产环境的需求</i></td>
        <td><strong>Milvus CDC</strong><br><i>数据复制功能</i><br><br><strong>访问日志增强</strong><br><i>详细记录以进行审计和追踪</i></td>
        <td><strong>新资源组</strong><br><i>增强的资源管理</i><br><br><strong>存储钩子</strong><br><i>支持自带密钥（BYOK）加密</i></td>
        <td><strong>动态副本数量调整</strong><br><i>便于动态更改副本数量</i><br><br><strong>动态模式修改</strong><br><i>例如，添加/删除字段，修改 varchar 长度</i><br><br><strong>Rust 和 C# SDK</strong></td>
    </tr>
</tbody>
</table>

- 我们的路线图通常分为三个部分：最近的发布、即将到来的下一个发布以及未来一年内的中长期愿景。
- 随着我们的进展，我们不断学习，偶尔调整我们的重点，根据需要添加或删除项目。
- 这些计划仅供参考，可能会有变化，并且可能会根据订阅服务而有所不同。
- 我们坚定地遵循我们的路线图，我们的[发布说明](release_notes.md)可作为参考。

## 如何贡献

作为一个开源项目，Milvus 依靠社区贡献不断发展。以下是您可以加入我们旅程的方式。

### 分享反馈

- 报告问题：遇到 bug 或有建议？在我们的[GitHub 页面](https://github.com/milvus-io/milvus/issues)上提出问题。

- 功能建议：有关于新功能或改进的想法？[我们很乐意听到！](https://github.com/milvus-io/milvus/discussions)

### 代码贡献

- Pull 请求：直接向我们的[代码库](https://github.com/milvus-io/milvus/pulls)贡献。无论是修复 bug、添加功能还是改进文档，我们都欢迎您的贡献。

- 开发指南：查看我们的[贡献者指南](https://github.com/milvus-io/milvus/blob/82915a9630ab0ff40d7891b97c367ede5726ff7c/CONTRIBUTING.md)，了解有关代码贡献的准则。

### 传播信息

- 社交分享：喜欢 Milvus 吗？在社交媒体和技术博客上分享您的使用案例和经验。

- 在 GitHub 上给我们点星：通过为我们的[GitHub 仓库](https://github.com/milvus-io/milvus)加星来表达您的支持。