### 示例模板

##### 这是一个模板示例，帮助您了解如何使用它。

##### 如果一个 JSON 文件中包含 {"useTemplate":true, "path":"template/\*.md", var:{...}}，那么该 JSON 文件将被转换为一个使用其中的模板和变量的 Markdown 文件；

##### 请看下面的例子：

##### 标题是：{{var.title}}
##### 内容是：{{var.content}}

##### 我可以在上级菜单中使用变量：
#### 授权人姓名是：{{var.auth.name}}