# 记录您的 API (Documenting your API)
REST API 应该花费几乎所有的描述性工作来定义用于表示资源和驱动应用程序状态的媒体类型。—— Roy Fielding, [REST APIs must be hypertext driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)

REST framework 提供对 API 文档的内置支持。还有一些很棒的第三方文档工具可用。

## 内置 API 文档 (Built-in API documentation)
内置的 API 文档包括：

- API 端点的文档。
- 为每个可用的 API 客户端库自动生成代码示例。
- 支持 API 交互。

### 安装 (Installation)
`coreapi` 库是 API 文档的必需品。确保安装最新版本。`pygments` 和 `markdown` 库是可选的，但推荐使用。

要安装 API 文档，您需要将其包含在项目 URLconf 中：
```python
from rest_framework.documentation import include_docs_urls

urlpatterns = [
    ...
    url(r'^docs/', include_docs_urls(title='My API title'))
]
```

这将包括两个不同的视图：

- `/docs/` - 文档页面本身。
- `/docs/schema.js` - 公开 API 模式的 JavaScript 资源。

***
**注意**：默认情况下，`include_docs_urls` 配置底层 `SchemaView` 以生成公共模式。这意味着视图不会使用 `request` 实例进行实例化。即在视图内部 `self.request` 将为 `None`。

要与检查 `self.request` 的行为方法 (例如 `get_serializer` 或 `get_serializer_class` 等) 兼容，或者特别是 `self.request.user` 可能需要调整以处理这种情况。

您可以通过使用 `public = False` 调用 `include_docs_urls` 来确保为 `request` 实例提供视图：
```python
from rest_framework.documentation import include_docs_urls

urlpatterns = [
    ...
    # 使用有效的`request`实例生成模式：
    url(r'^docs/', include_docs_urls(title='My API title', public=False))
]
```

***

### 记录您的视图 (Documenting your views)
您可以通过包含描述每个可用操作的文档字符串来记录您的视图。例如：
```python
class UserList(generics.ListAPIView):
    """
    Return a list of all the existing users.
    """
```

如果视图支持多种方法，则应使用 `method:` 样式分隔符拆分文档。
```python
class UserList(generics.ListCreateAPIView):
    """
    get:
    Return a list of all the existing users.

    post:
    Create a new user instance.
    """
```

使用视图集时，应使用相关的操作名称作为分隔符。
```python
class UserViewSet(viewsets.ModelViewSet):
    """
    retrieve:
    Return the given user.

    list:
    Return a list of all the existing users.

    create:
    Create a new user instance.
    """
```

### `documentation` API 参考 (`documentation` API Reference)
`rest_framework.documentation` 模块提供了三个帮助函数来帮助配置交互式 API 文档，`include_docs_urls` (用法如上所示)，`get_docs_view` 和 `get_schemajs_view`。

`include_docs_urls` 使用 `get_docs_view` 和 `get_schemajs_view` 来生成分别公开 API 模式的文档页面和 JavaScript 资源的 url patterns。他们公开了以下自定义选项。(`get_docs_view` 和 `get_schemajs_view` 最终调用 `rest_frameworks.schemas.get_schema_view()`，请参阅 Schemas 文档以获取更多选项。)

#### `include_docs_urls`

- `title`：默认为 `None`。可用于为模式定义提供描述性标题。
- `description`：默认为 `None`。可用于为模式定义提供描述。
- `schema_url`：默认为 `None`。可用于传递模式的规范基本 URL。
- `public`：默认为 `True`。模式应被视为公共的吗？如果在没有被传递到视图的 `request` 实例的情况下生成 `True` 模式。
- `patterns`：默认为 `None`。生成模式时要检查的 URL 列表。如果为 `None`，将使用项目的 URL conf。
- `generator_class`：默认为 `rest_framework.schemas.SchemaGenerator`。可用于指定要传递给 `SchemaView` 的 `SchemaGenerator` 子类。
- `authentication_classes`：默认为 `api_settings.DEFAULT_AUTHENTICATION_CLASSES`。可用于将自定义身份验证类传递给 `SchemaView`。
- `permission_classes`：默认为 `api_settings.DEFAULT_PERMISSION_CLASSES`。可用于将自定义权限类传递给 `SchemaView`。
- `renderer_classes`：默认为 `None`。可用于将自定义渲染器类传递给 `SchemaView`。

#### `get_docs_view`

- `title`：默认为 `None`。可用于为模式定义提供描述性标题。
- `description`：默认为 `None`。可用于为模式定义提供描述。
- `schema_url`：默认为 `None`。可用于传递模式的规范基本 URL。
- `public`：默认为 `True`。如果在没有被传递到视图的 `request` 实例的情况下生成 `True` 模式。
- `patterns`：默认为 `None`。生成模式时要检查的 URL 列表。如果为 `None`，将使用项目的 URL conf。
- `generator_class`：默认为 `rest_framework.schemas.SchemaGenerator`。可用于指定要传递给 `SchemaView` 的 `SchemaGenerator` 子类。
- `authentication_classes`：默认为 `api_settings.DEFAULT_AUTHENTICATION_CLASSES`。可用于将自定义身份验证类传递给 `SchemaView`。
- `permission_classes`：默认为 `api_settings.DEFAULT_PERMISSION_CLASSES`。可用于将自定义权限类传递给 `SchemaView`。
- `renderer_classes`：默认为 `None`。可用于将自定义渲染器类传递给 `SchemaView`。如果为 `None`，`SchemaView` 将配置 `DocumentationRenderer` 和 `CoreJSONRenderer` 渲染器，对应于(默认) `html` 和 `corejson` 格式。

#### `get_schemajs_view`

- `title`：默认为 `None`。可用于为模式定义提供描述性标题。
- `description`：默认为 `None`。可用于为模式定义提供描述。
- `schema_url`：默认为 `None`。可用于传递模式的规范基本 URL。
- `public`：默认为 `True`。如果在没有被传递到视图的 `request` 实例的情况下生成 `True` 模式。
- `patterns`：默认为 `None`。生成模式时要检查的 URL 列表。如果为 `None`，将使用项目的 URL conf。
- `generator_class`：默认为 `rest_framework.schemas.SchemaGenerator`。可用于指定要传递给 `SchemaView` 的 `SchemaGenerator` 子类。
- `authentication_classes`：默认为 `api_settings.DEFAULT_AUTHENTICATION_CLASSES`。可用于将自定义身份验证类传递给 `SchemaView`。
- `permission_classes`：默认为 `api_settings.DEFAULT_PERMISSION_CLASSES`。可用于将自定义权限类传递给 `SchemaView`。

### 自定义代码示例 (Customising code samples)
内置 API 文档包括为每个可用API客户端库自动生成的代码示例。

您可以通过继承 `DocumentationRenderer` 来自定义这些示例，将 `languages` 设置为您希望支持的语言列表：
```python
from rest_framework.renderers import DocumentationRenderer


class CustomRenderer(DocumentationRenderer):
    languages = ['ruby', 'go']
```

对于每种语言，您需要提供一个 `intro` 模板，详细安装说明等，以及用于发出 API 请求的通用模板，可以填充个人请求详情。有关示例，请参阅[捆绑语言的模板](https://github.com/encode/django-rest-framework/tree/master/rest_framework/templates/rest_framework/docs/langs)。

***

## 第三方包 (Third party packages)
有许多成熟的第三方软件包可用于提供 API 文档。

#### drf-yasg - Yet Another Swagger Generator
[drf-yasg](https://github.com/axnsan12/drf-yasg/) 是不使用 Django Rest Framework 提供的模式生成实现的 [Swagger](https://swagger.io/) 生成工具。

它旨在尽可能多地实现 [OpenAPI](https://openapis.org/) 规范 - 嵌套模式，命名模型，响应主体，枚举/模式/最小/最大验证器，表单参数等。- 并生成可与代码生成工具如 (`swagger-codegen`) 一起使用的文档。

这也转换为非常有用的 `swagger-ui` 形式的交互式文档查看器：

![](http://www.django-rest-framework.org/img/drf-yasg.png)

#### DRF OpenAPI
[DRF OpenAPI](https://github.com/limdauto/drf_openapi/) 通过 Django Rest Framework 公开开箱即用的模式弥补了 OpenAPI 规范和工具链之间的差距。它的目标是：

- 无需任何代码更改即可放入任何现有 DRF 项目中。
- 在请求模式和响应模式之间提供明确的区分。
- 为每个模式提供版本控制机制。支持按版本范围语法定义模式，例如 >1.0，<=2.0
- 支持多种响应代码，而不仅仅是 200
- 所有这些信息都应绑定到视图方法，而不是视图类。

它还试图与 DRF 提供的成熟模式生成机制保持同步。

![](http://www.django-rest-framework.org/img/drf-openapi.png)

***

#### DRF Docs
[DRF Docs](https://github.com/ekonstantinidis/django-rest-framework-docs) 允许您记录使用 Django REST Framework 制作的 Web API，它由 Emmanouil Konstantinidis 编写。它开箱即用，其设置不应超过几分钟。完整的文档可以在[网站](https://www.drfdocs.com/)上找到，同时还有一个[演示](http://demo.drfdocs.com/)可供人们看看它的样子。**Live API 端点**允许您以简洁的方式利用文档中的端点。

特点包括使用您的品牌自定义模板，设置根据环境和更多隐藏文档。

这个包和 Django REST Swagger 都有完整的文档记录，得到很好的支持，强烈推荐。

![](http://www.django-rest-framework.org/img/drfdocs.png)

***

#### Django REST Swagger
Marc Gibbons 的 [Django REST Swagger](https://github.com/marcgibbons/django-rest-swagger) 将 REST framework 与 [Swagger](https://swagger.io/) API 文档工具整合在一起。该软件包产生了良好表示的 API 文档，并包含用于测试 API 端点的交互式工具。

Django REST Swagger 支持 REST framework 版本 2.3 及更高版本。

Mark 也是 [REST Framework Docs](https://github.com/marcgibbons/django-rest-framework-docs) 包的作者，它为您的 API 提供简洁，简单的自动生成文档，但已弃用并已移至 Django REST Swagger。

此软件包和 DRF docs 均已完整记录，受到良好支持，强烈建议使用。

![](http://www.django-rest-framework.org/img/django-rest-swagger.png)

***

### DRF AutoDocs
Oleksander Mashianovs 的 [DRF Auto Docs](https://github.com/iMakedonsky/drf-autodocs) 自动化 api 渲染器。

毫不费力地收集几乎所有您编写的代码到文档中。

支持：

- 功能视图文档
- 树状结构
- 文档字符串：
- markdown
- 保留空格和换行符
- 用好的语法格式化
- 字段：
- 选择渲染
- help_text (指定 SerializerMethodField 输出等)
- 智能只读/必须渲染
- 端点属性：
- filter_backends
- authentication_classes
- permission_classes
- 额外 url 参数(GET params)

![](http://joxi.ru/52aBGNI4k3oyA0.jpg)

***

#### Apiary
有各种其他在线工具和服务可用于提供 API 文档。一个值得注意的服务是 [Apiary](https://apiary.io/)。使用 Apiary，您可以使用简单的 markdown 式语法来描述您的 API。生成的文档包括 API 交互，用于测试和原型设计的模拟服务器以及各种其他工具。

![](http://www.django-rest-framework.org/img/apiary.png)

***

## 自描述 API (Self describing APIs)
REST framework 提供的可浏览 API 使您的 API 可以完全自我描述。只需访问浏览器中的 URL 即可提供每个 API 端点的文档。

![](http://www.django-rest-framework.org/img/self-describing.png)

***

#### 设置标题 (Setting the title)
可浏览 API 中使用的标题是从视图类名称或函数名称生成的。任何尾随 `View` 或 `ViewSet` 的后缀都会被剥离，字符串是大小写边界或下划线分隔的空格。

例如，视图 `UserListView` 在可浏览 API 中显示时将被命名为 `User List`。

使用视图集时，会为每个生成的视图附加适当的后缀。例如，视图设置 `UserViewSet` 将生成名为 `User List` 和 `User Instance` 的视图。

#### 设置描述 (Setting the description)
可浏览 API 中的描述是从视图或视图集的文档字符串生成的。

如果安装了python `markdown` 库，则可以在文档字符串中使用 [markdown 语法](https://daringfireball.net/projects/markdown/)，并在可浏览的 API 中将其转换为 HTML。例如：
```python
class AccountListView(views.APIView):
    """
    返回系统中所有**活动**帐户的列表。

    有关如何激活帐户的更多详细信息，请[参阅此处][ref]。

    [ref]: http://example.com/activating-accounts
    """
```

请注意，使用视图集时，基本文档字符串用于所有生成的视图。例如用于列表和检索视图 (这句有问题，暂时这样。such as for the the list and retrieve views)，使用模式中描述的文档字符串部分[作为文档：示例](http://www.django-rest-framework.org/api-guide/schemas/#example)。

#### `OPTIONS` 方法 (The `OPTIONS` method)
REST framework API 还支持使用 `OPTIONS` HTTP 方法以编程方式访问的描述。视图将使用元数据 (包括名称，描述以及它接受和响应的各种媒体类型) 响应 `OPTIONS` 请求。

当使用通用视图时，任何 `OPTIONS` 请求都将额外地响应有关任何可用 `POST` 或 `PUT` 操作的元数据，描述序列化器上有哪些字段。

您可以通过覆盖 `options` 视图方法和/或提供自定义元数据类来修改 `OPTIONS` 请求的响应行为。例如：
```python
def options(self, request, *args, **kwargs):
    """
    不要在 OPTIONS 响应中包含视图描述。
    """
    meta = self.metadata_class()
    data = meta.determine_metadata(request, self)
    data.pop('description')
    return data
```

有关更多详细信息，请参阅[元数据文档](http://www.django-rest-framework.org/api-guide/metadata/)。

***

## 超媒体方法 (The hypermedia approach)
要完全 RESTful API，应在其发送的响应中将其可用操作显示为超媒体控件。

在这种方法中，与其预先记录可用的 API 端点，不如集中于所使用的媒体类型上的描述。对任何给定 URL 可能采取的可用操作不是严格固定的，而是通过返回文档中存在的链接和表单控件来使其可用。

要实现超媒体 API，您需要为 API 确定适当的媒体类型，并为该媒体类型实现自定义渲染器和解析器。文档的 [REST, Hypermedia & HATEOAS](http://www.django-rest-framework.org/topics/rest-hypermedia-hateoas/) 部分包括指向背景阅读的指针，以及各种超媒体格式的链接。
