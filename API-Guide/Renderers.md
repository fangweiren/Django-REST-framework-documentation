# 渲染器 (Renderers)
在将 TemplateResponse 实例返回给客户端之前，必须渲染它。渲染过程采用模板和上下文的中间表示，并将其转换为可以提供给客户端的最终字节流。—— Django 文档

REST framework 包含许多内置的渲染器 (Renderer) 类，允许您返回各种媒体类型的响应。还支持自定义渲染器，这使您可以灵活地设计自己的媒体类型。

## 如何确定渲染器 (How the renderer is determined)
视图的有效渲染器集合始终被定义为类列表。进入视图时，REST框架将对传入请求执行内容协商，并确定最合适的渲染器以满足请求。

内容协商的基本过程涉及检查请求的 `Accept` 标头，以确定它在响应中期望的媒体类型。可选地，URL 上的格式后缀可以用于显式地请求特定的表示。例如，URL `http://example.com/api/users_count.json` 可能是始终返回 JSON 数据的端点。

有关更多信息，请参阅有关[内容协商](http://www.django-rest-framework.org/api-guide/content-negotiation/)的文档。

## 设置渲染器 (Setting the renderers)
可以使用 `DEFAULT_RENDERER_CLASSES` 设置全局的默认渲染器集。例如，以下设置将使用 `JSON` 作为主要媒体类型，并且还包含自描述 API。
```python
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    )
}
```
您还可以使用基于 `APIView` 类的视图设置用于单个视图或视图集的渲染器。
```python
from django.contrib.auth.models import User
from rest_framework.renderers import JSONRenderer
from rest_framework.response import Response
from rest_framework.views import APIView

class UserCountView(APIView):
    """
    返回 JSON 中活跃用户数量的视图。
    """
    renderer_classes = (JSONRenderer, )

    def get(self, request, format=None):
        user_count = User.objects.filter(active=True).count()
        content = {'user_count': user_count}
        return Response(content)
```
或者，如果您使用基于函数的视图的 `@api_view` 装饰器。
```python
@api_view(['GET'])
@renderer_classes((JSONRenderer,))
def user_count_view(request, format=None):
    """
    A view that returns the count of active users in JSON.
    """
    user_count = User.objects.filter(active=True).count()
    content = {'user_count': user_count}
    return Response(content)
```

## 渲染器类的排序 (Ordering of renderer classes)
在为 API 指定渲染器类时，考虑要分配给每个媒体类型的优先级非常重要。如果客户端未指定可接受的表现形式，例如发送 `Accept：*/*` 标头，或者根本不包含 `Accept` 标头，则 REST framework 将选择列表中的第一个渲染器用于响应。

例如，如果您的 API 提供 JSON 响应和 HTML 可浏览 API，您可能希望使 `JSONReNDER` 成为默认的渲染器，以便向不指定 `Accept` 标头的客户端发送 `JSON` 响应。

如果您的 API 包含可以根据请求同时提供常规网页和 API 响应的视图，那么你可以考虑将  `TemplateHTMLRenderer` 设置为默认渲染器，以便与发送[损坏的 accept 标头](http://www.gethifi.com/blog/browser-rest-http-accept-headers)的旧浏览器很好地配合使用。

***

# API 参考 (API Reference)
## JSONRenderer
使用 utf-8 编码将请求数据渲染为 `JSON`。

请注意，默认风格包含 unicode 字符，并使用紧凑风格渲染 (没有多余的空白) 响应：
```python
{"unicode black star":"★","value":999}
```
客户端可能另外包括 `'indent'` 媒体类型参数，在这种情况下，返回的 `JSON` 将缩进。例如  `Accept: application/json; indent=4`。
```python
{
    "unicode black star": "★",
    "value": 999
}
```
使用 `UNICODE_JSON` 和 `COMPACT_JSON` 设置键可以更改默认的 JSON 编码样式。

**.media_type**: `application/json`

**.format**: `'.json'`

**.charset**: `None`

## TemplateHTMLRenderer
使用 Django 的标准模板渲染将数据渲染为 HTML。与其他渲染器不同，传递给 `Response` 的数据不需要序列化。此外，与其他渲染器不同，您可能想要在创建 `Response` 时包含 `template_name` 参数。

TemplateHTMLRenderer 将创建 `RequestContext`，使用 `response.data` 作为上下文字典，并确定用于渲染上下文的模板名称。

模板名称由 (按优先顺序) 确定：

1. 传递给响应的显式 `template_name` 参数。  
2. 在此类上设置显式的 `.template_name` 属性。  
3. 调用 `view.get_template_names()` 的返回结果。

使用 `TemplateHTMLRenderer` 的视图示例：
```python
class UserDetail(generics.RetrieveAPIView):
    """
    回给定用户的模板化 HTML 表示的视图。
    """
    queryset = User.objects.all()
    renderer_classes = (TemplateHTMLRenderer,)

    def get(self, request, *args, **kwargs):
        self.object = self.get_object()
        return Response({'user': self.object}, template_name='user_detail.html')
```
可以使用 `TestPielHTMLReDeReor` 来使用 REST framework 返回常规 HTML 页面，或者从单个端点返回 HTML 和 API 响应。

如果你正在构建使用 `TemplateHTMLRenderer` 以及其他渲染器类的网站，您应该考虑将`TemplateHTMLRenderer` 列为 `renderer_classes` 列表中的第一个类，因此，对于发送格式不正确的 `ACCEPT:` 标头，它甚至会优先考虑浏览器。

有关 `TemplateHTMLRenderer` 用法的更多示例，请参阅 [HTML&Forms 主题页](http://www.django-rest-framework.org/topics/html-and-forms/)。

**.media_type**: `text/html`

**.format**: `'.html'`

**.charset**: `utf-8`

也可以看看： `StaticHTMLRenderer`

## StaticHTMLRenderer
一个简单的渲染器，只返回预渲染的 HTML。与其他渲染器不同，传递给响应对象的数据应该是表示要返回的内容的字符串。

使用 `StaticHTMLRenderer` 的视图示例：
```python
@api_view(('GET',))
@renderer_classes((StaticHTMLRenderer,))
def simple_html_view(request):
    data = '<html><body><h1>Hello, world</h1></body></html>'
    return Response(data)
```
可以使用 `StaticHTMLRenderer` 来使用 REST framework 返回常规 HTML 页面，或者从单个端点返回 HTML 和 API 响应。

**.media_type**: `text/html`

**.format**: `'.html'`

**.charset**: `utf-8`

也可以看看： `TemplateHTMLRenderer`

## BrowsableAPIRenderer
为可浏览的 API 将数据渲染为 HTML：

![](http://www.django-rest-framework.org/img/quickstart.png)

此渲染器将确定哪个其他渲染器将被赋予最高优先级，并使用它来显示 HTML 页面中 API 样式的响应。

**.media_type**: `text/html`

**.format**: `'.api'`

**.charset**: `utf-8`

**.template**: `'rest_framework/api.html'`

#### 自定义 BrowsableAPIRenderer
默认情况下，响应内容将使用除 `BrowsableAPIRenderer` 之外的最高优先级渲染器进行渲染。如果您需要自定义此行为，例如使用 HTML 作为默认返回格式，但在可浏览的 API 中使用 JSON，则可以通过重写 `get_default_renderer()` 方法来实现。例如：
```python
class CustomBrowsableAPIRenderer(BrowsableAPIRenderer):
    def get_default_renderer(self, view):
        return JSONRenderer()
```

## AdminRenderer
将数据渲染为 HTML 以进行类似管理的显示：

![](https://q1mi.github.io/Django-REST-framework-documentation/img/admin.png)

此渲染器适用于 CRUD 样式的 Web API，它还应提供用户友好的界面来管理数据。

请注意，对于其输入具有嵌套或列表序列化程序的视图将无法与 `AdminRenderer` 一起使用，因为 HTML 表单无法正确支持它们。

**注意**：当在数据中存在正确配置的 `URL_FIELD_NAME` (默认为 `url` ) 属性时，`AdminRenderer` 才能包含指向详细信息页面的链接。对于 `HyperlinkedModelSerializer`，情况就是如此，但是对于 `ModelSerializer` 或者简单的 `Serializer` 类，你需要确保显式地包含该字段。例如，我们在这里使用模型  `get_absolute_url` 方法：
```python
class AccountSerializer(serializers.ModelSerializer):
    url = serializers.CharField(source='get_absolute_url', read_only=True)

    class Meta:
        model = Account
```
**.media_type**: `text/html`

**.format**: `'.admin'`

**.charset**: `utf-8`

**.template**: `'rest_framework/admin.html'`

## HTMLFormRenderer
将序列化程序返回的数据渲染为 HTML 表单。此渲染器的输出不包括封闭的 `<form>` 标记，隐藏的 CSRF 输入或任何提交按钮。

此渲染器不是直接使用，而是可以通过将序列化器实例传递给 `render_form` 模板标记来替代模板。
```python
{% load rest_framework %}

<form action="/submit-report/" method="post">
    {% csrf_token %}
    {% render_form serializer %}
    <input type="submit" value="Save" />
</form>
```
有关更多信息，请参阅 [HTML 和表单](http://www.django-rest-framework.org/topics/html-and-forms/)文档。

**.media_type**: `text/html`

**.format**: `'.form'`

**.charset**: `utf-8`

**.template**: `'rest_framework/horizontal/form.html'`

## MultiPartRenderer
此渲染器用于渲染 HTML 多部分表单数据。**它不适合作为响应渲染器**，而是用于使用 REST framework 的[测试客户端和测试请求工厂](http://www.django-rest-framework.org/api-guide/testing/)来创建测试请求。

**.media_type**: `multipart/form-data; boundary=BoUnDaRyStRiNg`

**.format**: `'.multipart'`

**.charset**: `utf-8`

***

# 自定义渲染器 (Custom renderers)
要实现自定义渲染器，你应该重写 `BaseRenderer`，设置 `.media_type` 和 `.format` 属性，并且实现 `.render(self, data, media_type=None, renderer_context=None)` 方法。

该方法应当返回一个 bytestring，它将被用于 HTTP 响应的主体。

传递给 `.render()` 方法的参数是：

**data**

请求数据，由 `Response()` 例化设置。

**media_type=None**

可选的。如果提供，这是接受的媒体类型，由内容协商阶段确定。

根据客户端的 `Accept:`标头，这可能比渲染器的 `media_type` 属性更具体，并且可能包括媒体类型参数。例如 `"application/json; nested=true"`。

**renderer_context=None**

可选的。如果提供，这是由视图提供的上下文信息的字典。

默认情况下，这将包括以下键：`view`, `request`, `response`, `args`, `kwargs`。

## 举个栗子
下面是一个纯文本渲染器示例，它将返回一个以 `data` 参数作为响应内容的响应。
```python
from django.utils.encoding import smart_unicode
from rest_framework import renderers


class PlainTextRenderer(renderers.BaseRenderer):
    media_type = 'text/plain'
    format = 'txt'

    def render(self, data, media_type=None, renderer_context=None):
        return data.encode(self.charset)
```

## 设置字符集 (Setting the character set)
默认情况下，渲染器类被假定为使用 `UTF-8` 编码。要使用其他编码，请在渲染器上设置 `charset` 属性。
```python
class PlainTextRenderer(renderers.BaseRenderer):
    media_type = 'text/plain'
    format = 'txt'
    charset = 'iso-8859-1'

    def render(self, data, media_type=None, renderer_context=None):
        return data.encode(self.charset)
```
注意，如果渲染器类返回 unicode 字符串，则响应内容将被 `Response` 类强制为 bytestring，而设置在渲染器上的字符集属性用于确定编码。

如果渲染器返回表示原始二进制内容的字节串，则应将字符集值设置为 `None`，这将确保响应的 `Content-Type` 标头不会设置字符集值。

在某些情况下，您可能还希望将 `render_style` 属性设置为 `'binary'`。这样做也将确保可浏览 API 不会试图将二进制内容显示为字符串。
```python
class JPEGRenderer(renderers.BaseRenderer):
    media_type = 'image/jpeg'
    format = 'jpg'
    charset = None
    render_style = 'binary'

    def render(self, data, media_type=None, renderer_context=None):
        return data
```

***

# 高级渲染器用法 (Advanced renderer usage)
您可以使用 REST framework 的渲染器做一些非常灵活的事情。一些例子...

- 根据请求的媒体类型，从同一端点提供平面或嵌套表示。
- 提供常规 HTML 网页和来自同一端点的基于 JSON 的 API 响应。
- 为要使用的 API 客户端指定多种类型的 HTML 表示。
- 未指定渲染器的媒体类型，例如使用 `media_type = 'image/*'`，并使用 `Accept` 标头来更改响应的编码。

## 媒体类型的不同行为 (Varying behaviour by media type)
在某些情况下，您可能希望视图根据所接受的媒体类型使用不同的序列化样式。如果需要这样做，可以访问 `request.accepted_renderer` 以确定将用于响应的协商渲染器。

举个栗子：
```python
@api_view(('GET',))
@renderer_classes((TemplateHTMLRenderer, JSONRenderer))
def list_users(request):
    """
    可以返回系统中用户的 JSON 或 HTML 表示的视图。
    """
    queryset = Users.objects.filter(active=True)

    if request.accepted_renderer.format == 'html':
	    # TemplateHTMLRenderer 采用上下文字典，并且还需要 template_name 名称。
	    # 它不需要序列化。
        data = {'users': queryset}
        return Response(data, template_name='list_users.html')

    # JSONRenderer 需要正常的序列化数据。
    serializer = UserSerializer(instance=queryset)
    data = serializer.data
    return Response(data)
```

## 未指定媒体类型 (Underspecifying the media type)
在某些情况下，您可能希望渲染器提供一系列媒体类型。在这种情况下，你可以不指定应该响应的媒体类型，通过使用  `media_type` 值 (诸如 `image/*` 或 `*/*` ) 。

如果您未指定渲染器的媒体类型，则应确保在使用 `content_type` 属性返回响应时显式指定媒体类型。
```python
return Response(data, content_type='image/png')
```

## 设计媒体类型 (Designing your media types)
对于许多 Web API 的目的，具有超链接关系的简单 `JSON` 响应可能是足够的。如果您想完全拥抱 RESTful 设计和 [HATEOAS](http://timelessrepo.com/haters-gonna-hateoas)，则需要更详细地考虑媒体类型的设计和使用。

用 [Roy Fielding 的话](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)说，“REST API 应该花费几乎所有的描述性工作来定义用于表示资源和驱动应用程序状态的媒体类型，或者为现有的标准媒体类型定义扩展关系名称和/或超文本启用标记。"。

有关自定义媒体类型的良好示例，请参阅 GitHub 的使用自定义 [application/vnd.github+json](https://developer.github.com/v3/media/) 媒体类型，以及 Mike Amundsen 的 IANA 认可的  [application/vnd.collection+json](http://www.amundsen.com/media-types/collection/) 基于 JSON 超媒体。

## HTML 错误视图 (HTML error views)
通常，渲染器都会表现相同，无论是处理常规响应，还是由异常引起的响应，例如 `Http404` 或 `PermissionDenied` 异常，或 `APIException` 的子类。

如果您正在使用 `TemplateHTMLRenderer` 或 `StaticHTMLRenderer` 并引发异常，行为略有不同，并反映 [Django 对错误视图的默认处理](https://docs.djangoproject.com/en/stable/topics/http/views/#customizing-error-views)。

由 HTML 渲染器引发和处理的异常将尝试按优先顺序使用以下方法之一进行渲染。

- 加载并渲染名为 `{status_code}.html` 的模板。
- 加载并渲染名为 `api_exception.html` 的模板。
- 渲染 HTTP 状态码和文本，例如 “404 Not Found”。

模板将用一个包含 `status_code` 和 `details` 键的 `RequestContext` 渲染。

**注意**：如果 `DEBUG=True`，Django 的标准回溯错误页面将被展示，而不是渲染 HTTP 状态码和文本。

***

# 第三方包 (Third party packages)
以下第三方包都是可用的。

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

## JSONP
[REST framework JSONP](https://jpadilla.github.io/django-rest-framework-jsonp/) 提供 JSONP 渲染支持。它之前直接包含在 REST framework 包中，现在被替代为第三方包支持。

***

**警告**：如果你需要跨域的 AJAX 请求，你通常应该使用更现代化的 [CORS](https://www.w3.org/TR/cors/) 方法代替 `JSONP`。有关更多详细信息，请参阅 [CORS 文档](http://www.django-rest-framework.org/topics/ajax-csrf-cors/)。

`JSONP` 方法本质上是一个浏览器破解，并且[仅适用于全局可读的 API 端点](https://stackoverflow.com/questions/613962/is-jsonp-safe-to-use)，其中 `GET` 请求是未经认证的并且不需要任何用户权限。

***

#### 安装和配置 (Installation & configuration)
使用 pip 安装。
```python
$ pip install djangorestframework-jsonp
```
修改您的 REST framework 设置。
```python
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework_jsonp.renderers.JSONPRenderer',
    ),
}
```

## MessagePack
[MessagePack](https://msgpack.org/) 是一种快速，高效的二进制序列化格式。[Juan Riaza](https://github.com/juanriaza) 维护着 [djangorestframework-msgpack](https://github.com/juanriaza/django-rest-framework-msgpack) 包，它为 REST framework 提供 MessagePack 渲染器和解析器支持。

## CSV
逗号分隔值是纯文本表格数据格式，可以轻松导入到电子表格应用中。[Mjumbe Poe](https://github.com/mjumbewu) 维护着 [djangorestframework-csv](https://github.com/mjumbewu/django-rest-framework-csv) 包，它为 REST framework 提供了 CSV 渲染器支持。

## UltraJSON
[UltraJSON](https://github.com/esnme/ultrajson) 是一个优化的 C JSON 编码器，可以显著提高 JSON 渲染速度。[Jacob Haslehurst](https://github.com/hzy) 维护着使用 UJSON 包实现 JSON 渲染的 [drf-ujson-renderer](https://github.com/gizmag/drf-ujson-renderer) 包。

## CamelCase JSON
[djangorestframework-camel-case](https://github.com/vbabiy/djangorestframework-camel-case) 为 REST framework 提供了驼峰式 JSON 渲染器和解析器。这使序列化程序可以使用 Python 风格的下划线字段名，但是在 API 中显示成 Javascript 样式的驼峰字段名。它由 [Vitaly Babiy](https://github.com/vbabiy) 维护着。

## Pandas (CSV, Excel, PNG)
[Django REST Pandas](https://github.com/wq/django-rest-pandas) 提供了一个序列化器和渲染器，通过 [Pandas](https://pandas.pydata.org/) DataFrame API 支持额外的数据处理和输出。Django REST Pandas 包括 Pandas 风格的 CSV 文件，Excel 表格(包括 `.xls` 和 `.xlsx`)以及许多[其他格式](https://github.com/wq/django-rest-pandas#supported-formats)的渲染器。由 [S. Andrew Sheppard](https://github.com/sheppard) 维护，作为 [wq 项目](https://github.com/wq)的一部分。

## LaTeX
[Rest Framework Latex](https://github.com/mypebble/rest-framework-latex) 提供了一个使用 Laulatex 输出 PDF 的渲染器。它由 [Pebble (S/F Software)](https://github.com/mypebble) 维护着。
