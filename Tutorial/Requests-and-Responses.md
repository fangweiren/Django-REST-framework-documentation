# 教程 2：请求和响应
从现在开始，我们将真正开始接触 REST framework 的核心。 我们来介绍几个基本的构建模块。

## 请求对象 (request objects)
REST framework 引入了一个扩展了常规 `HttpRequest` 的 `Request` 对象，并提供了更灵活的请求解析。`Request` 对象的核心功能是 `request.data` 属性，它与 `request.POST` 类似，但对于使用 Web API 更加有用。
```python
request.POST # 只处理表单数据。 只适用于 'POST' 方法。
request.data # 处理任意数据。 适用于 'POST'，'PUT' 和 'PATCH' 方法。
```

## 响应对象 (Response objects)
REST framework 还引入了一个 `Response` 对象，该对象是一种获取未渲染内容的 `TemplateResponse` 类型，并使用内容协商来确定正确内容类型返回给客户端。
```python
return Response(data) # 渲染成客户端请求的内容类型。
```

## 状态码 (Status codes)
在您的视图中使用数字 HTTP 状态码并不总是利于代码的阅读，如果写错代码，很容易被忽略。REST framework 为每个状态码提供更明确的标识符，譬如 `status` 模块中的  `HTTP_400_BAD_REQUEST`。使用它们是一个好主意，而不是使用数字标识符。

## 包装 API 视图 (Wrapping API views)
REST framework 提供了两种编写 API 视图的封装。

1. 用于基于函数视图的 `@api_view` 装饰器。
2. 用于基于类视图的 `APIView` 类。

这些视图封装提供了一些功能，例如确保你的视图能够接收 `Request` 实例，并将上下文添加到 `Response` 对象，使得内容协商可以正常的运作。

视图封装还内置了一些行为，例如在适当的时候返回 `405 Method Not Allowed` 响应，并处理访问错误的输入 `request.data` 时触发的任何 `ParseError` 异常。

## 组合在一起
好的，让我们开始使用这些新的组件来写几个视图。

我们不再需要 `views.py` 中 `JSONResponse` 类，所以把它删除掉。然后，我们可以重构我们的视图。
```python
@api_view(['GET', 'POST'])
def snippet_list(request):
    """
    列出所有的代码 snippet，或创建一个新的 snippet。
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```
我们的实例视图比前面的示例有所改进。它稍微简洁一点，现在的代码与我们使用 Forms API 时非常相似。我们还使用了指定的状态码，这使得响应更加明显。

以下是 `views.py` 模块中单个 snippet 的视图。
```python
@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    """
    获取，更新或删除一个代码 snippet
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
这对我们来说应该都是非常熟悉的 - 它和正常 Django 视图并没有什么不同。

注意，我们不再显式地将请求或响应绑定到给定的内容类型。`request.data` 可以处理传入的 `json` 请求，但它也可以处理其他格式。同样，我们返回带有数据的响应对象，但允许 REST framework 将响应渲染成正确的内容类型。

## 为我们的网址添加可选的格式后缀
为了利用我们的响应不再被硬连接到单个内容类型的事实，让我们将格式后缀的支持添加到我们的API端点。使用格式后缀，为我们提供了明确指向给定格式的 URL，这意味着我们的 API 可以处理一些 URLs，类似这样的格式 http://example.com/api/items/4.json。

像下面这样在这两个视图中添加一个 `format` 关键字参数。
```python
def snippet_list(request, format=None):
```
和
```python
def snippet_detail(request, pk, format=None):
```
现在更新 `snippet/urls.py` 文件，在现有的 urls 基础上追加一组 `format_suffix_patterns` 。
```python
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
    url(r'^snippets/(?P<pk>[0-9]+)$', views.snippet_detail),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```
我们不一定需要添加这些额外的 url 模式，但它给了我们一个简单，清晰的方式来引用特定的格式。

## 它看起来如何？
继续从命令行测试 API，就像我们在教程第一部分所做的那样。一切都非常类似，尽管我们对一些无效的请求有更好的错误处理。

我们可以像以前一样获取所有 snippet 的列表。
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ http http://127.0.0.1:8000/snippets/
HTTP/1.1 200 OK
Allow: POST, OPTIONS, GET
Content-Length: 317
Content-Type: application/json
Date: Thu, 14 Jun 2018 15:49:40 GMT
Server: WSGIServer/0.2 CPython/3.5.2
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

[
    {
        "code": "foo = \"bar\"\n",
        "id": 1,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    },
    {
        "code": "print \"hello, world\"\n",
        "id": 2,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    },
    {
        "code": "print \"hello, world\"",
        "id": 3,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    }
]
```
我们可以通过使用 `Accept` 请求头，来控制我们返回的响应的格式。
```python
http http://127.0.0.1:8000/snippets/ Accept:application/json  # 请求 JSON
http http://127.0.0.1:8000/snippets/ Accept:text/html         # 请求 HTML
```
或者通过追加格式后缀：
```python
http http://127.0.0.1:8000/snippets.json  # JSON 后缀
http http://127.0.0.1:8000/snippets.api   # 可浏览 API 后缀
```
类似的，我们可以通过 `Content-Type` 请求头来控制我们发送请求的格式。
```python
# POST 表单数据
(env) fang@ubuntu:~/django_rest_framework/tutorial$ http --form POST http://127.0.0.1:8000/snippets/ code="print 123"
HTTP/1.1 201 Created
Allow: POST, OPTIONS, GET
Content-Length: 93
Content-Type: application/json
Date: Thu, 14 Jun 2018 16:00:37 GMT
Server: WSGIServer/0.2 CPython/3.5.2
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

{
    "code": "print 123",
    "id": 4,
    "language": "python",
    "linenos": false,
    "style": "friendly",
    "title": ""
}

# POST JSON 数据
(env) fang@ubuntu:~/django_rest_framework/tutorial$ http --json POST http://127.0.0.1:8000/snippets/ code="print 456"
HTTP/1.1 201 Created
Allow: POST, OPTIONS, GET
Content-Length: 93
Content-Type: application/json
Date: Thu, 14 Jun 2018 16:02:24 GMT
Server: WSGIServer/0.2 CPython/3.5.2
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

{
    "code": "print 456",
    "id": 5,
    "language": "python",
    "linenos": false,
    "style": "friendly",
    "title": ""
}
```
如果你向上述 `http` 请求中添加 `--debug` ，则可以在请求头中查看请求类型。

现在可以在浏览器中访问 `http://127.0.0.1:8000/snippets/` 查看 API 。

### 可视化
由于 API 是基于客户端发起的请求来选择响应内容的格式，因此，当接收到来自浏览器的请求时，会默认以 `HTML` 格式来描述数据。这允许 API 返回网页完全可浏览的 HTML。

拥有可浏览的网页 API 是一个巨大的胜利，并且使开发和使用 API 更加容易。它也大大降低了其他开发人员检查和使用 API 的门槛。

有关可浏览的 API 功能以及如何对其进行定制的更多信息，请参阅[可浏览的 API](http://www.django-rest-framework.org/topics/browsable-api/)主题。

## 下一步是什么？
在[教程的第三部分](http://www.iamnancy.top/post/169/)，我们将开始使用基于类的视图，并查看通用视图如何减少我们需要编写的代码量。
