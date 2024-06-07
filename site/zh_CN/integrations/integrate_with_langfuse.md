---
id: integrate_with_langfuse.md
summary: 这是一个简单的食谱，演示了如何使用 LlamaIndex Langfuse 集成。它使用 Milvus Lite 来存储文档和查询。
title: 食谱 LlamaIndex & Milvus 集成
---

# 食谱 - LlamaIndex & Milvus 集成

<a target="_blank" href="https://colab.research.google.com/github/langfuse/langfuse-docs/blob/main/cookbook/integration_llama-index_milvus-lite.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/>
</a>

这是一个简单的食谱，演示了如何使用[LlamaIndex Langfuse 集成](https://langfuse.com/docs/integrations/llama-index/get-started)。它使用 Milvus Lite 来存储文档和查询。

[Milvus Lite](https://github.com/milvus-io/milvus-lite/) 是 Milvus 的轻量级版本，Milvus 是一个开源的向量数据库，为具有向量嵌入和相似度搜索的 AI 应用提供支持。

## 设置

确保已安装 `llama-index` 和 `langfuse`。


```python
$ pip install llama-index langfuse llama-index-vector-stores-milvus --upgrade
```

初始化集成。从[Langfuse 项目设置](https://cloud.langfuse.com)获取 API 密钥，并用你的密钥值替换 public_key 和 secret_key。此示例使用 OpenAI 进行嵌入和聊天完成，因此您还需要在环境变量中指定您的 OpenAI 密钥。


```python
import os

# 从项目设置页面获取项目的密钥
# https://cloud.langfuse.com
os.environ["LANGFUSE_PUBLIC_KEY"] = ""
os.environ["LANGFUSE_SECRET_KEY"] = ""
os.environ["LANGFUSE_HOST"] = "https://cloud.langfuse.com" # 🇪🇺 欧洲地区
# os.environ["LANGFUSE_HOST"] = "https://us.cloud.langfuse.com" # 🇺🇸 美国地区

# 您的 OpenAI 密钥
os.environ["OPENAI_API_KEY"] = ""
```


```python
from llama_index.core import Settings
from llama_index.core.callbacks import CallbackManager
from langfuse.llama_index import LlamaIndexCallbackHandler
 
langfuse_callback_handler = LlamaIndexCallbackHandler()
Settings.callback_manager = CallbackManager([langfuse_callback_handler])
```

## 使用 Milvus Lite 进行索引


```python
from llama_index.core import Document

doc1 = Document(text="""
麦克斯韦尔·“麦克斯”·席尔弗斯坦（Maxwell "Max" Silverstein）是一位备受赞誉的电影导演、编剧和制片人，生于1978年10月25日，美国马萨诸塞州波士顿。从小就是电影爱好者的他，早年开始拍摄家庭影片，使用超级8摄影机。他的热情使他进入南加州大学（USC）主修电影制作。最终，他在派拉蒙影业开始了他的职业生涯，担任助理导演。席尔弗斯坦的导演处女作《未见之门》是一部心理惊悚片，在圣丹斯电影节上获得认可，标志着他成功导演生涯的开端。
""")
doc2 = Document(text="""
在他的职业生涯中，席尔弗斯坦以其多样化的作品和独特的叙事技巧而备受赞誉。他巧妙地融合悬疑、人类情感和微妙的幽默在他的故事情节中。他的著名作品包括《飘忽的回声》、《宁静的黄昏》以及获得奥斯卡奖的科幻史诗作品《事件地平线的边缘》。他对电影的贡献主要围绕着探讨人性、关系的复杂性以及探索现实和感知。在幕后，他是一位致力于慈善事业的慈善家，与妻子和两个孩子居住在洛杉矶。
""")
```
```python
# 示例索引构建 + LLM 查询

from llama_index.core import VectorStoreIndex
from llama_index.core import StorageContext
from llama_index.vector_stores.milvus import MilvusVectorStore


vector_store = MilvusVectorStore(
    uri="tmp/milvus_demo.db", dim=1536, overwrite=False
)
storage_context = StorageContext.from_defaults(vector_store=vector_store)

index = VectorStoreIndex.from_documents(
    [doc1, doc2], storage_context=storage_context
)
```

## 查询


```python
# 查询
response = index.as_query_engine().query("他小时候做了什么？")
print(response)
```


```python
# 聊天
response = index.as_chat_engine().chat("他小时候做了什么？")
print(response)
```

## 在 Langfuse 中探索追踪


```python
# 由于我们希望立即在 Langfuse 中看到结果，我们需要刷新回调处理程序
langfuse_callback_handler.flush()
```

完成！✨ 您可以在 Langfuse 项目中看到索引和查询的追踪。

示例追踪（公共链接）：
1. [查询](https://cloud.langfuse.com/project/cloramnkj0002jz088vzn1ja4/traces/2b26fc72-044f-4b0b-a3c3-485328975161)
2. [查询（聊天）](https://cloud.langfuse.com/project/cloramnkj0002jz088vzn1ja4/traces/72503163-2b25-4693-9cc9-56190b8e32b9)

在 Langfuse 中的追踪：

![Langfuse 追踪](https://static.langfuse.com/llamaindex-langfuse-docs.gif)


## 对更高级功能感兴趣吗？

查看完整的[集成文档](https://langfuse.com/docs/integrations/llama-index/get-started)以了解更多关于高级功能以及如何使用它们的信息：

- 与 Langfuse Python SDK 和其他集成的互操作性
- 向追踪添加自定义元数据和属性
```