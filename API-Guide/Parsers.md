# 解析器 (Parsers)
与表单编码相比，机器交互 web 服务更倾向于使用更结构化的格式来发送数据，因为它们发送的数据比简单表单更复杂。—— Malcom Tredinnick, Django developers group

REST framework 包含许多内置的 Parser 类，允许您接受各种媒体类型的请求。还支持自定义解析器，这使您可以灵活地设计API接受的媒体类型。

## 如何确定解析器 (How the parser is determined)
视图的有效解析器集始终定义为类列表。访问 `request.data` 时，REST framework 将检查传入请求的 `Content-Type` 标头，并确定用于解析请求内容的解析器。

***

**注意**：在开发客户端应用程序时，请始终记住确保在 HTTP 请求中发送数据时设置 `Content-Type` 标头。

如果您没有设置内容类型，大多数客户端将默认使用`'application/x-www-form-urlencoded'`，这可能不是您想要的。

例如，如果使用带有[.ajax() 方法](https://api.jquery.com/jQuery.ajax/)的 jQuery 发送 `json` 编码数据，您应确保包含 `contentType: 'application/json'` 设置。
***

## 设置解析器 (Setting the parsers)
可以使用 `DEFAULT_PARSER_CLASSES` 设置默认的全局解析器。例如，以下设置将只允许带有 `JSON` 内容的请求，而不是默认的 JSON 或表单数据。
```python
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',
    )
}
```
您还可以使用基于 `APIView` 类的视图设置用于单个视图或视图集的解析器。
```python
from rest_framework.parsers import JSONParser
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    """
    接受带有 JSON 内容的 POST 请求的视图。
    """
    parser_classes = (JSONParser,)

    def post(self, request, format=None):
        return Response({'received data': request.data})
```
或者，如果您使用带有基于函数的视图的 `@api_view` 装饰器。
```python
from rest_framework.decorators import api_view
from rest_framework.decorators import parser_classes
from rest_framework.parsers import JSONParser

@api_view(['POST'])
@parser_classes((JSONParser,))
def example_view(request, format=None):
    """
    接受带有 JSON 内容的 POST 请求的视图。
    """
    return Response({'received data': request.data})
```

***

# API 参考 (API Reference)
## JSONParser
解析 `JSON` 请求内容。

**.media_type**: `application/json`

## FormParser
解析 HTML 表单内容。`request.data` 将用 `QueryDict` 数据填充。

您通常希望同时使用 `FormParser` 和 `MultiPartParser`，以便完全支持 HTML 表单数据。

**.media_type**: `application/x-www-form-urlencoded`

## MultiPartParser
解析多部分 HTML 表单内容，支持文件上传。`request.data` 将被 `QueryDict` 填充。

您通常希望同时使用 `FormParser` 和 `MultiPartParser`，以便完全支持 HTML 表单数据。

**.media_type**: `multipart/form-data`

## FileUploadParser
解析原始文件上传内容。`request.data` 属性将是一个包含上传文件的单个键 `'file'` 的字典。

如果使用 `FileUploadParser` 调用了 `filename` URL 关键字参数的视图，则该参数将用作文件名。

如果没有 `filename`  URL 关键字参数被调用，那么客户端必须在 HTTP 报头 `Content-Disposition` 中设置文件名。例如 `Content-Disposition: attachment; filename=upload.jpg`。

**.media_type**: `*/*`

##### 注意：
- `FileUploadParser` 适用于可以将文件作为原始数据请求上传的本地客户端。对于基于 Web 的上传，或者对于具有分段上传支持的本地客户端，您应该使用 `MultiPartParser` 解析器。
- 由于该解析器的 `media_type` 匹配任何内容类型，因此 `FileUploadParser` 通常应该是 API 视图上唯一的解析器集。
- `FileUploadParser` 遵循 Django 的标准 `FILE_UPLOAD_HANDLERS` 设置，和 `request.upload_handlers` 属性。有关更多详细信息，请参阅 [Django 文档](https://docs.djangoproject.com/en/stable/topics/http/file-uploads/#upload-handlers)。

##### 基本用法示例：
```python
# views.py
class FileUploadView(views.APIView):
    parser_classes = (FileUploadParser,)

    def put(self, request, filename, format=None):
        file_obj = request.data['file']
        # ...
        # 用上传的文件做一些事情
        # ...
        return Response(status=204)

# urls.py
urlpatterns = [
    # ...
    url(r'^upload/(?P<filename>[^/]+)$', FileUploadView.as_view())
]
```

***

# 自定义解析器 (Custom parsers)
要实现一个自定义解析器，你应该重写 `BaseParser`，设置 `.media_type` 属性，并实现 `.parse(self，stream，media_type，parser_context)` 方法。

该方法应返回将用于填充 `request.data` 属性的数据。

传递给 `.parse()` 的参数是：

### stream
表示请求主体的流状对象。

### media_type
可选的。如果提供，这是传入请求内容的媒体类型。

根据请求的 `Content-Type:` 标头，这可能比渲染器的 `media_type` 属性更具体，可能包括媒体类型参数。例如 `"text/plain; charset=utf-8"`。

### parser_context
可选的。如果提供，该参数将是包含解析请求内容可能需要的任何附加上下文的字典。

默认情况下，将包含以下键：`view`, `request`, `args`, `kwargs`。

## 举个栗子
以下是一个纯文本解析器示例，它将使用表示请求主体的字符串填充 `request.data` 属性。
```python
class PlainTextParser(BaseParser):
    """
    纯文本解析器。
    """
    media_type = 'text/plain'

    def parse(self, stream, media_type=None, parser_context=None):
        """
        只需返回表示请求主体的字符串。
        """
        return stream.read()
```

***

# 第三方包 (Third party packages)
以下是可用的第三方包。

## YAML
[REST framework YAML](https://jpadilla.github.io/django-rest-framework-yaml/) 提供 [YAML](http://www.yaml.org/) 解析和渲染支持。它以前直接包含在REST framework 包中，现在被替代为第三方包支持。

#### 安装和配置 (Installation & configuration)
使用 pip 安装。
```python
$ pip install djangorestframework-yaml
```
修改您的 REST framework 设置。
```python
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework_yaml.parsers.YAMLParser',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework_yaml.renderers.YAMLRenderer',
    ),
}
```

## XML
[REST Framework XML](https://jpadilla.github.io/django-rest-framework-xml/) 提供了一个简单的非正式 XML 格式。它以前直接包含在 REST framework 包中，现在被替代为第三方包支持。

#### 安装和配置 (Installation & configuration)
使用 pip 安装。
```python
$ pip install djangorestframework-xml
```
修改您的 REST framework 设置。
```python
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework_xml.parsers.XMLParser',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework_xml.renderers.XMLRenderer',
    ),
}
```

## MessagePack
[MessagePack](https://msgpack.org/) 是一种快速，高效的二进制序列化格式。[Juan Riaza](https://github.com/juanriaza) 维护着 [djangorestframework-msgpack](https://github.com/juanriaza/django-rest-framework-msgpack) 包，它为 REST framework 提供 MessagePack 渲染器和解析器支持。

## CamelCase JSON
[djangorestframework-camel-case](https://github.com/vbabiy/djangorestframework-camel-case) 为 REST framework 提供了驼峰式 JSON 渲染器和解析器。这使序列化程序可以使用 Python 风格的下划线字段名，但是在 API 中显示成 Javascript 样式的驼峰字段名。它由 [Vitaly Babiy](https://github.com/vbabiy) 维护着。
