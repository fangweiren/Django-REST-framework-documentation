# 模式 (Schemas)
机器可读的[模式]描述了通过 API 可以使用哪些资源，它们的 url 是什么，它们是如何表示的，它们支持哪些操作。—— Heroku, [Heroku 平台 API 的 JSON 模式](https://blog.heroku.com/archives/2014/1/8/json_schema_for_heroku_platform_api)

API 模式是一个有用的工具，允许一系列用例，包括生成参考文档，或驱动可与您的 API 交互的动态客户端库。

## 安装 Core API (Install Core API)
您需要安装 `coreapi` 软件包才能为 REST framework 添加模式支持。
```python
pip install coreapi
```

## 内部模式表示 (Internal schema representation)
REST framework 使用 [Core API](http://www.coreapi.org/) 以独立于格式的表示模拟模式信息。然后，这些信息可以被渲染成各种不同的模式格式，或者用于生成 API 文档。

使用 Core API 时，模式被表示为 `Document`，它是有关 API 信息的顶级容器对象。可用的 API 交互使用 `Link` 对象表示。每个链接包括 URL，HTTP 方法，并且可能包括 `Field` 实例的列表，其描述 API 端点可以接受的任何参数。`Link` 和 `Field` 实例还可能包含描述，允许将 API 模式渲染到用户文档中。

以下是包含单个 `search` 端点的 API 描述示例：
```python
coreapi.Document(
    title='Flight Search API',
    url='https://api.example.org/',
    content={
        'search': coreapi.Link(
            url='/search/',
            action='get',
            fields=[
                coreapi.Field(
                    name='from',
                    required=True,
                    location='query',
                    description='City name or airport code.'
                ),
                coreapi.Field(
                    name='to',
                    required=True,
                    location='query',
                    description='City name or airport code.'
                ),
                coreapi.Field(
                    name='date',
                    required=True,
                    location='query',
                    description='Flight date in "YYYY-MM-DD" format.'
                )
            ],
            description='Return flight availability and prices.'
        )
    }
)
```

## 模式输出格式 (Schema output formats)
为了在 HTTP 响应中呈现，必须将内部表示渲染为响应中使用的实际字节。

[Core JSON](http://www.coreapi.org/specification/encoding/#core-json-encoding) 被设计为与 Core API 一起使用的规范格式。REST framework 包含用于处理此媒体类型的渲染器类，该类可用作 `renderers.CoreJSONRenderer`。

### 替代模式格式 (Alternate schema formats)
其他模式格式 (如 [Open API](https://openapis.org/) (“Swagger”)，[JSON HyperSchema](http://json-schema.org/latest/json-schema-hypermedia.html) 或 [API Blueprint](https://apiblueprint.org/)) 也可以通过实现自定义渲染器类来支持处理将 `Document` 实例转换为字节串表示。

如果有一个 Core API 编解码器包支持编码为您想要使用的格式，则可以使用编解码器来实现该渲染器类。

#### 举个栗子
例如，`openapi_codec` 包提供对 Open API (“Swagger”) 式的编码或解码支持：
```python
from rest_framework import renderers
from openapi_codec import OpenAPICodec

class SwaggerRenderer(renderers.BaseRenderer):
    media_type = 'application/openapi+json'
    format = 'swagger'

    def render(self, data, media_type=None, renderer_context=None):
        codec = OpenAPICodec()
        return codec.dump(data)
```

## 模式 vs 超媒体 (Schemas vs Hypermedia)
值得指出的是，Core API 也可用于模拟超媒体响应，这为 API 模式提供了另一种交互方式。

通过 API 模式，整个可用接口作为单个端点呈现在前端。然后，对各个 API 端点的响应通常会以普通数据的形式呈现，而不会在每个响应中包含任何进一步的交互。

使用超媒体，客户端会显示包含数据和可用交互的文档。每次交互都会生成一个新文档，详细说明当前状态和可用交互。

关于使用 REST framework 构建超媒体 API 的更多信息和支持计划在将来的版本中发布。
***

# 创建模式 (Creating a schema)
REST framework 包括用于自动生成模式的功能，或允许您显式指定。

## 手动模式规范 (Manual Schema Specification)
要手动指定模式，请创建 Core API `Document`，类似于上面的示例。
```python
schema = coreapi.Document(
    title='Flight Search API',
    content={
        ...
    }
)
```

## 自动模式生成 (Automatic Schema Generation)
`SchemaGenerator` 类提供自动模式生成。

`SchemaGenerator` 处理路由 URL patterns 列表并编译适当结构化的 Core API 文档。

基本用法只是为模式提供标题并调用 `get_schema()`：
```python
generator = schemas.SchemaGenerator(title='Flight Search API')
schema = generator.get_schema()
```

## 每个视图模式自定义 (Per-View Schema Customisation)
默认情况下，视图内省由可通过 `APIView` 上的 `schema` 属性访问的 `AutoSchema` 实例执行的。这为视图、请求方法和路径提供了适当的 Core API `Link` 对象：
```python
auto_schema = view.schema
coreapi_link = auto_schema.get_link(...)
```
(在编译模式时，`SchemaGenerator` 为每个视图、允许的方法和路径调用 `view.schema.get_link()`。)

***
**注意**：对于基本的 `APIView` 子类，默认自省本质上仅限于 URL kwarg 路径参数。对于包含所有提供的基于类的视图的 `GenericAPIView` 子类，`AutoSchema` 将尝试内省序列化器、分页和过滤器字段，以及提供更丰富的路径字段描述。(这里的关键钩子是相关的 `GenericAPIView` 属性和方法：`get_serializer`，`pagination_class`，`filter_backends` 等。)
***

要自定义 `Link` 生成，您可以：

- 使用 `manual_fields` kwarg 在您的视图上实例化 `AutoSchema`：
```python
from rest_framework.views import APIView
from rest_framework.schemas import AutoSchema

class CustomView(APIView):
    ...
    schema = AutoSchema(
        manual_fields=[
            coreapi.Field("extra_field", ...),
        ]
    )
```

&ensp;&ensp;这允许在没有子类化的情况下扩展最常见的情况。

- 提供具有更复杂自定义的 `AutoSchema` 子类：
```python
from rest_framework.views import APIView
from rest_framework.schemas import AutoSchema

class CustomSchema(AutoSchema):
    def get_link(...):
        # Implement custom introspection here (or in other sub-methods)

class CustomView(APIView):
    ...
    schema = CustomSchema()
```

&ensp;&ensp;这提供了对视图内省的完全控制。

- 在视图上实例化 `ManualSchema`，为视图显式地提供 Core API `Fields`：
```python
from rest_framework.views import APIView
from rest_framework.schemas import ManualSchema

class CustomView(APIView):
    ...
    schema = ManualSchema(fields=[
        coreapi.Field(
            "first_field",
            required=True,
            location="path",
            schema=coreschema.String()
        ),
        coreapi.Field(
            "second_field",
            required=True,
            location="path",
            schema=coreschema.String()
        ),
    ])
```

&ensp;&ensp;这允许手动指定某些视图的模式，同时保持其他地方的自动生成。

您可以通过将 `schema` 设置为 `None` 来禁用视图的模式生成：
```python
 class CustomView(APIView):
        ...
        schema = None  # Will not appear in schema
```

***
**注意**：有关 `SchemaGenerator` 以及 `AutoSchema` 和 `ManualSchema` 描述符的完整详细信息，请参阅[下面的 API 参考](http://www.django-rest-framework.org/api-guide/schemas/#api-reference)。
***

# 添加模式视图 (Adding a schema view)
根据您的需要，有几种不同的方法可以向您的 API 添加模式视图。

## get_schema_view 快捷方式 (The get_schema_view shortcut)
在项目中包含模式的最简单方法是使用 `get_schema_view()` 函数。
```python
from rest_framework.schemas import get_schema_view

schema_view = get_schema_view(title="Server Monitoring API")

urlpatterns = [
    url('^$', schema_view),
    ...
]
```

一旦添加了视图，您就可以发出 API 请求来检索自动生成的模式定义。
```python
$ http http://127.0.0.1:8000/ Accept:application/coreapi+json
HTTP/1.0 200 OK
Allow: GET, HEAD, OPTIONS
Content-Type: application/vnd.coreapi+json

{
    "_meta": {
        "title": "Server Monitoring API"
    },
    "_type": "document",
    ...
}
```

`get_schema_view()` 的参数是：

`title`

可用于为模式定义提供描述性标题。

`url`

可用于为模式定义传递规范 URL。
```python
schema_view = get_schema_view(
    title='Server Monitoring API',
    url='https://www.example.org/api/'
)
```

`urlconf`

表示您想要生成 API 模式的 URL conf 的导入路径的字符串。这默认为 Django 的 ROOT_URLCONF 设置的值。
```python
schema_view = get_schema_view(
    title='Server Monitoring API',
    url='https://www.example.org/api/',
    urlconf='myproject.urls'
)
```

`renderer_classes`

可用于传递用于渲染 API 根端点的渲染器类集合。
```python
from rest_framework.schemas import get_schema_view
from rest_framework.renderers import CoreJSONRenderer
from my_custom_package import APIBlueprintRenderer

schema_view = get_schema_view(
    title='Server Monitoring API',
    url='https://www.example.org/api/',
    renderer_classes=[CoreJSONRenderer, APIBlueprintRenderer]
)
```

`patterns`

限制模式内省的url模式列表。如果您只想在模式中公开 `myproject.api` url：
```python
schema_url_patterns = [
    url(r'^api/', include('myproject.api.urls')),
]

schema_view = get_schema_view(
    title='Server Monitoring API',
    url='https://www.example.org/api/',
    patterns=schema_url_patterns,
)
```

`generator_class`

可用于指定要传递给 `SchemaView` 的 `SchemaGenerator` 子类。

`authentication_classes`

可用于指定将应用于模式端点的身份验证类列表。默认为 `settings.DEFAULT_AUTHENTICATION_CLASSES`

`permission_classes`

可用于指定将应用于模式端点的权限类列表。默认为 `settings.DEFAULT_PERMISSION_CLASSES`

## 使用显式的模式视图 (Using an explicit schema view)
如果您需要比 `get_schema_view()` 快捷方式稍微多些的控制权，那么您可以直接使用 `SchemaGenerator` 类来自动生成 `Document` 实例，并从视图中返回该实例。

这个选项使您灵活地使用任何您想要的行为设置模式端点。例如，您可以将不同的权限、限制或身份验证策略应用于模式端点。

下面是使用 `SchemaGenerator` 和视图来返回模式的示例。

**views.py:**
```python
from rest_framework.decorators import api_view, renderer_classes
from rest_framework import renderers, response, schemas

generator = schemas.SchemaGenerator(title='Bookings API')

@api_view()
@renderer_classes([renderers.CoreJSONRenderer])
def schema_view(request):
    schema = generator.get_schema(request)
    return response.Response(schema)
```

**urls.py:**
```python
urlpatterns = [
    url('/', schema_view),
    ...
]
```

您还可以为不同的用户提供不同的模式，具体取决于他们可用的权限。此方法可用于确保未经身份验证的请求呈现与已验证请求不同的模式，或者确保根据角色使不同用户可以看到 API 的不同部分。

为了呈现一个通过用户权限过滤的端点的模式，您需要将 `request` 参数传递给 `get_schema()` 方法，如下所示：
```python
@api_view()
@renderer_classes([renderers.CoreJSONRenderer])
def schema_view(request):
    generator = schemas.SchemaGenerator(title='Bookings API')
    return response.Response(generator.get_schema(request=request))
```

## 显式模式定义 (Explicit schema definition)
自动生成方法的替代方法是通过在代码库中声明 `Document` 对象来显式指定 API 模式。这样做会多一点工作，但确保您完全控制模式表示。
```python
import coreapi
from rest_framework.decorators import api_view, renderer_classes
from rest_framework import renderers, response

schema = coreapi.Document(
    title='Bookings API',
    content={
        ...
    }
)

@api_view()
@renderer_classes([renderers.CoreJSONRenderer])
def schema_view(request):
    return response.Response(schema)
```

## 静态模式文件 (Static schema file)
最后一个选项是使用可用格式之一 (如 Core JSON 或 Open API) 将 API 模式编写为静态文件。

然后您可以：

- 将模式定义写为静态文件，并[直接提供静态文件](https://docs.djangoproject.com/en/stable/howto/static-files/)。
- 编写使用 `Core API` 加载的模式定义，然后根据客户端请求渲染为多种可用格式之一。

***

# 模式作为文档 (Schemas as documentation)
API 模式的一个常见用法是使用它们来构建文档页面。

REST framework 中的模式生成使用文档字符串自动填充模式文档中的描述。

这些描述将基于：

- 对应方法的文档字符串 (如果存在)。
- 类文档字符串中的命名部分，可以是单行或多行。
- 类文档字符串。

## 举个栗子
`APIView`，带有显式的方法文档字符串。
```python
class ListUsernames(APIView):
    def get(self, request):
        """
        Return a list of all user names in the system.
        """
        usernames = [user.username for user in User.objects.all()]
        return Response(usernames)
```

`ViewSet`，带有显式地动作文档字符串。
```python
class ListUsernames(ViewSet):
    def list(self, request):
        """
        Return a list of all user names in the system.
        """
        usernames = [user.username for user in User.objects.all()]
        return Response(usernames)
```

类文档字符串中包含部分的通用视图，使用单行样式。
```python
class UserList(generics.ListCreateAPIView):
    """
    get: List all the users.
    post: Create a new user.
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = (IsAdminUser,)
```

类文档字符串中包含部分的通用视图，使用多行样式。
```python
class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited.

    retrieve:
    Return a user instance.

    list:
    Return all users, ordered by most recently joined.
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer
```

***

# API 参考 (API Reference)
## SchemaGenerator
遍历路由 URL patterns 列表的类，为每个视图请求模式并整理生成的 CoreAPI 文档。

通常，您将使用单个参数实例化 `SchemaGenerator`，如下所示：
```python
generator = SchemaGenerator(title='Stock Prices API')
```

参数：

- `title` **必需** - API 的名称。
- `url` - API 模式的根 URL。除非模式被包含在路径前缀下，否则此选项不是必需的。
- `patterns` - 生成模式时检查的 URL 列表。默认为项目的 URL conf。
- `urlconf` - 生成模式时使用的 URL conf 模块名称。 默认为 `settings.ROOT_URLCONF`。

### get_schema(self, request)
返回表示 API 模式的 `coreapi.Document` 实例。
```python
@api_view
@renderer_classes([renderers.CoreJSONRenderer])
def schema_view(request):
    generator = schemas.SchemaGenerator(title='Bookings API')
    return Response(generator.get_schema())
```
`request` 参数是可选的，如果要将每个用户权限应用于结果模式生成，则可以使用该参数。

### get_links(self, request)
返回一个嵌套字典，其中包含应包含在 API 模式中的所有链接。

如果您想修改生成的模式的结果结构，这是一个很好的重写点，因为您可以用不同的布局构建新的字典。

## AutoSchema
处理用于模式生成的各个视图的自省的类。

`AutoSchema` 通过 `schema` 属性附加到 `APIView`。

`AutoSchema` 构造函数采用单个关键字参数 `manual_fields`。

**`manual_fields`**：将添加到生成的字段的 `coreapi.Field` 实例 `list`。具有匹配 `name` 的生成字段将被覆盖。
```python
class CustomView(APIView):
    schema = AutoSchema(manual_fields=[
        coreapi.Field(
            "my_extra_field",
            required=True,
            location="path",
            schema=coreschema.String()
        ),
    ])
```

对于更高级的自定义子类 `AutoSchema` 来自定义模式生成。
```python
class CustomViewSchema(AutoSchema):
    """
    Overrides `get_link()` to provide Custom Behavior X
    """

    def get_link(self, path, method, base_url):
        link = super().get_link(path, method, base_url)
        # Do something to customize link here...
        return link

class MyView(APIView):
  schema = CustomViewSchema()
```

以下方法可用于覆盖。

### get_link(self, path, method, base_url)
返回与给定视图相对应的 `coreapi.Link` 实例。

这是主要的入口点。如果需要为特定视图提供自定义行为，则可以重写此方法。

### get_description(self, path, method)
返回用作链接描述的字符串。默认情况下，这是基于上面 “模式作为文档” 一节中描述的视图文档字符串。

### get_encoding(self, path, method)
当与给定视图交互时，返回指示任何请求体的编码的字符串。例如 `'application/json'`。对于不期望请求主体的视图，可能返回空白字符串。

### get_path_fields(self, path, method):
返回 `coreapi.Link()` 实例列表。用于 URL 中的每个路径参数。

### get_serializer_fields(self, path, method)
返回 `coreapi.Link()` 实例列表。用于视图使用的序列化器类中的每个字段。

### get_pagination_fields(self, path, method)
返回 `coreapi.Link()` 实例列表，由视图使用的任何分页类上的 `get_schema_fields()` 方法返回。

### get_filter_fields(self, path, method)
返回 `coreapi.Link()` 实例列表，由视图使用的任何过滤器类上的 `get_schema_fields()` 方法返回。

### get_manual_fields(self, path, method)
返回 `coreapi.Field()` 实例列表以添加或替换生成的字段。默认为 (可选) `anual_fields` 传递给 `AutoSchema` 构造函数。

可以通过 `path` 或 `method` 重写自定义手动字段。例如，每个方法的调整可能如下所示：
```python
def get_manual_fields(self, path, method):
    """Example adding per-method fields."""

    extra_fields = []
    if method=='GET':
        extra_fields = # ... list of extra fields for GET ...
    if method=='POST':
        extra_fields = # ... list of extra fields for POST ...

    manual_fields = super().get_manual_fields(path, method)
    return manual_fields + extra_fields
```

### update_fields(fields, update_with)
实用的 `staticmethod`。封装逻辑按 `Field.name` 从列表中添加或替换字段。可重写以调整替换标准。

## ManualSchema
允许手动为模式提供 `coreapi.Field` 实例列表，以及一个可选的描述。
```python
class MyView(APIView):
  schema = ManualSchema(fields=[
        coreapi.Field(
            "first_field",
            required=True,
            location="path",
            schema=coreschema.String()
        ),
        coreapi.Field(
            "second_field",
            required=True,
            location="path",
            schema=coreschema.String()
        ),
    ]
  )
```

`ManualSchema` 构造函数有两个参数：

**`fields`**：`coreapi.Field` 实例列表。必需。

**`description`**：字符串描述。可选的。

**`encoding`**：默认 `None`。字符串编码，例如 `application/json`。可选的。

***

## Core API
本文档给出了用于表示 API 模式的 `coreapi` 包中的组件的简要概述。

请注意，这些类是从 `coreapi` 包导入的，而不是从 `rest_framework` 包导入。

### Document
表示 API 模式的容器。

`title`

API 的名称。

`url`

API 的规范 URL。

`content`

一个字典，包含模式包含的 `Link` 对象。

为了向模式提供更多结构，`content` 字典可以嵌套，通常嵌套二层。例如：
```python
content={
    "bookings": {
        "list": Link(...),
        "create": Link(...),
        ...
    },
    "venues": {
        "list": Link(...),
        ...
    },
    ...
}
```

### Link
表示单个 API 端点。

`url`

端点的 URL。可能是 URI 模板，例如 `/users/{username}/`。

`action`

与端点关联的 HTTP 方法。请注意，支持多个 HTTP 方法的 url 应该对应于每个 HTTP 方法的单个链接。

`fields`

`Field` 实例列表，描述输入上的可用参数。

`description`

端点的含义和预期用途的简短描述。

### Field
表示给定 API 端点上的单个输入参数。

`name`

输入的描述性名称。

`required`

布尔值，表示客户端是否需要包含值，或者参数是否可以省略。

`location`

确定如何将信息编码到请求中。应该是以下字符串之一：

**"path"**

包含在模板化的 URI 中。例如，`/products/{product_code}/` 的 `url` 值可以与 `"path"` 字段一起使用，以处理 URL 路径中的 API 输入，例如 `/products/slim-fit-jeans/`。

这些字段通常与[项目 URL conf 中的命名参数](https://docs.djangoproject.com/en/stable/topics/http/urls/#named-groups)对应。

**"query"**

包含为 URL 查询参数。例如 `?search=sale`。通常用于 `GET` 请求。

这些字段通常与视图上的分页和过滤控件相对应。

**"form"**

包含在请求正文中，作为 JSON 对象或 HTML 表单的单个项。例如 `{"colour": "blue", ...}`。通常用于 `POST`，`PUT` 和 `PATCH` 请求。多个 `"form"` 字段可能包含在单个链接上。

这些字段通常与视图上的序列化器字段相对应。

**"body"**

包含为完整的请求主体。通常用于 `POST`, `PUT` 和 `PATCH` 请求。链接上可能存在不超过一个 `"body"` 字段。不能与 `"form"` 字段一起使用。

这些字段通常与使用 `ListSerializer` 验证请求输入的视图或文件上传视图相对应。

`encoding`

**"application/json"**

JSON 编码的请求内容。对应于使用 `JSONParser` 的视图。仅在 `Link` 中包含一个或多个 `location="form"` 字段或单个 `location="body"` 字段时有效。

**"multipart/form-data"**

多部分编码的请求内容。对应于使用 `MultiPartParser` 的视图。仅在 `Link` 中包含一个或多个 `location="form"` 字段时有效。

**"application/x-www-form-urlencoded"**

URL 编码的请求内容。对应于使用 `FormParser` 的视图。仅在 `Link` 中包含一个或多个 `location="form"` 字段时有效。

**"application/octet-stream"**

二进制上传请求内容。对应于使用 `FileUploadParser` 的视图。仅在 `Link` 中包含 `location="body"` 字段时有效。

`description`
输入字段的含义和预期用途的简短描述。

# 第三方包 (Third party packages)
## drf-yasg - Yet Another Swagger Generator
[drf-yasg](https://github.com/axnsan12/drf-yasg/) 生成适合代码生成的 [OpenAPI](https://openapis.org/) 文档 - 嵌套模式，命名模型，响应主体，枚举/模式/最小/最大验证器，表单参数等。

## DRF OpenAPI
[DRF OpenAPI](https://github.com/limdauto/drf_openapi) 以 [OpenAPI](https://openapis.org/) 格式渲染由 Django Rest Framework 生成的模式。
