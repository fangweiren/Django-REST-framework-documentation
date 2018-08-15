# 教程 7：Schemas 和客户端库
schema 是一个机器可读文档，描述可用的 API 端点，它们的 URL 以及它们支持的操作。

Schemas 可以是自动生成文档的有用工具，也可以用来驱动可以与 API 交互的动态客户端库。

## Core API
为了提供 schema 支持，REST framework 使用 [Core API](www.coreapi.org)。

Core API 是用于描述 API 的文档规范。它用于提供可用端点的内部表示格式和 API 公开的可能交互。它既可以用于服务器端，也可以用于客户端。

当使用服务器端时，Core API 允许 API 支持对各种 schema 或超媒体格式的渲染。

当使用客户端时，Core API 允许动态驱动的客户端库，可以与任何公开支持的 schema 或超媒体格式的 API 进行交互。

## 添加 schema
REST framework 支持显式定义的 schema 视图或自动生成的 schemas。由于我们使用视图集和路由器，因此我们可以简单地使用自动 schema 生成。

您需要安装 `coreapi` Python 包以便包含 API 模式。
```python
$ pip install coreapi
```
我们现在可以通过在我们的 URL 配置中包含一个自动生成的 schema 视图来为我们的 API 包含schema。
```python
from rest_framework.schemas import get_schema_view

schema_view = get_schema_view(title='Pastebin API')

urlpatterns = [
    url(r'^schema/$', schema_view),
    ...
]
```
如果你在浏览器中访问 /schema/ 端点，那么你现在应该看到 `corejson` 表示成为可选项。
![f_34764973.png](http://pan.16mb.com/data/f_34764973.png)

我们还可以通过在 `Accept` 头中指定所需的内容类型来从命令行请求 schema。
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ http http://127.0.0.1:8000/schema/ Accept:application/coreapi+json
HTTP/1.1 200 OK
Allow: GET, HEAD, OPTIONS
Content-Length: 1893
Content-Type: application/coreapi+json
Date: Wed, 20 Jun 2018 10:44:13 GMT
Server: WSGIServer/0.2 CPython/3.5.2
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

{
    "_meta": {
        "title": "Pastebin API",
        "url": "http://127.0.0.1:8000/schema/"
    },
    "_type": "document",
    ...
```
默认的输出风格是使用 `Core JSON` 编码。

还支持其他架构格式，如 `Open API` (以前称为Swagger)。

## 使用命令行客户端
现在我们的 API 公开了一个 schema 端点，我们可以使用一个动态客户端库来与 API 进行交互。为了演示这一点，我们使用 Core API 命令行客户端。

命令行客户端可用作 `coreapi-cli` 软件包：
```python
$ pip install coreapi-cli
```
现在检查它在命令行上是否可用...
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ coreapi
Usage: coreapi [OPTIONS] COMMAND [ARGS]...

  Command line client for interacting with CoreAPI services.

  Visit http://www.coreapi.org for more information.

Options:
  --version  Display the package version number.
  --help     Show this message and exit.

Commands:
  action       Interact with the active document.
  bookmarks    Add, remove and show bookmarks.
  clear        Clear the active document and other state.
  codecs       Manage the installed codecs.
  credentials  Configure request credentials.
  describe     Display description for link at given PATH.
  dump         Dump a document to console.
  get          Fetch a document from the given URL.
  headers      Configure custom request headers.
  history      Navigate the browser history.
  load         Load a document from disk.
  reload       Reload the current document.
  show         Display the current document.
```
首先，我们将使用命令行客户端加载 API schema。
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ coreapi get http://127.0.0.1:8000/schema/
<Pastebin API "http://127.0.0.1:8000/schema/">
    snippets: {
        list([page])
        read(id)
        highlight(id)
    }
    users: {
        list([page])
        read(id)
    }
```
我们还没有进行身份验证，所以现在我们只能看到只读端点，与我们如何设置 API 的权限一致。

让我们尝试使用命令行客户端列出现有的 snippets：
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ coreapi action snippets list
{
    "count": 4,
    "next": null,
    "previous": null,
    "results": [
        {
            "url": "http://127.0.0.1:8000/snippets/1/",
            "id": 1,
            "highlight": "http://127.0.0.1:8000/snippets/1/highlight/",
            "owner": "djrest",
            "title": "test one",
            "code": "hello, world",
            "linenos": false,
            "language": "abap",
            "style": "abap"
        },
        ...
    ]
}
```
一些 API 端点需要命名参数。例如，为了要获取特定的高亮 HTML 表示的 snippet，我们需要提供一个 id。
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ coreapi action snippets highlight --param id=1
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
   "http://www.w3.org/TR/html4/strict.dtd">

<html>
<head>
  <title>test one</title>
  <meta http-equiv="content-type" content="text/html; charset=None">
  <style type="text/css">
td.linenos { background-color: #f0f0f0; padding-right: 10px; }
span.lineno { background-color: #f0f0f0; padding: 0 5px 0 5px; }
pre { line-height: 125%; }
  </style>
</head>
<body>
<h2>test one</h2>

<div class="highlight"><pre><span></span><span class="nv">hello</span><span class="p">,</span> <span class="nv">world</span>
</pre></div>
</body>
</html>
```

## 认证我们的客户端
如果我们希望能够创建，编辑和删除 snippets，我们需要进行有效性用户身份验证。在这种情况下，我们只使用基本的 auth。

请确保使用您的实际的用户名和密码替换下面的 `<username>` 和 `<password>`。
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ coreapi credentials add 127.0.0.1 djrest:aa112211 --auth basic
Added credentials
127.0.0.1 "Basic ZGpyZXN0OmFhMTEyMjEx"
```
现在，如果我们再次获取 schema，我们应该能够看到一组完整的可用交互。
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ coreapi reload
<Pastebin API "http://127.0.0.1:8000/schema/">
    snippets: {
        list([page])
        create(code, [title], [linenos], [language], [style])
        read(id)
        update(id, code, [title], [linenos], [language], [style])
        partial_update(id, [title], [code], [linenos], [language], [style])
        delete(id)
        highlight(id)
    }
    users: {
        list([page])
        read(id)
    }
```
我们现在可以与这些端点进行交互。例如，要创建一个新的 snippet：
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ coreapi action snippets create --param title="Example" --param code="print('hello, world')"
{
    "url": "http://127.0.0.1:8000/snippets/6/",
    "id": 6,
    "highlight": "http://127.0.0.1:8000/snippets/6/highlight/",
    "owner": "djrest",
    "title": "Example",
    "code": "print('hello, world')",
    "linenos": false,
    "language": "python",
    "style": "friendly"
}
```
并删除 snippet：
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ coreapi action snippets delete --param id=6
```
除了命令行客户端之外，开发人员还可以使用客户端库与 API 进行交互。Python 客户端库是第一个可用的客户端库，并且计划很快发布 Javascript 客户端库。

有关自定义 schema 生成和使用 Core API 客户端库的更多详细信息，您需要参阅完整的文档。

## 回顾我们的工作
拥有非常小的代码量，现在我们有了一个完整的 pastebin Web API，它完全是 Web 可浏览的，包括一个模式驱动的客户端库，并完成了身份验证、每个对象权限和多个渲染器格式。

我们已经走过了设计过程的每个步骤，并且看到如果我们需要自定义任何东西，我们都可以按部就班的简单地使用常规的 Django 视图实现。

您可以在 GitHub 上查看[最终的教程代码](https://github.com/fangweiren/Django_REST_framework)，或者在[沙箱](https://restframework.herokuapp.com/)中试用一个实例。

## 勇往直前
我们已经到达了教程的最后。如果您想更多地参与 REST framework 项目，可以从以下几个地方开始：

- 通过审查和提交问题，并提出拉取请求，为 [GitHub](https://github.com/encode/django-rest-framework) 做出贡献。
- 加入 [REST framework 讨论组](https://groups.google.com/forum/?fromgroups#!forum/django-rest-framework)，并帮助构建社区。
- 在 Twitter 上关注[作者](https://twitter.com/_tomchristie)并打个招呼。

**现在就去构建非常棒的东西吧。**
