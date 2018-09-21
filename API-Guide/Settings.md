# 设置 (Settings)
命名空间是一个很棒的主意 - 让我们做更多的事情吧！—— [The Zen of Python](https://www.python.org/dev/peps/pep-0020/)

REST framework 的配置全部在单个 Django 设置中命名的，名为 `REST_FRAMEWORK`。

例如，您的项目的 `settings.py` 文件可能包含以下内容：
```python
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework.renderers.JSONRenderer',
    ),
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',
    )
}
```

## 访问设置 (Accessing settings)
如果您需要访问项目中 REST framework 的 API 设置的值，则应使用 `api_settings` 对象。例如。
```python
from rest_framework.settings import api_settings

print(api_settings.DEFAULT_AUTHENTICATION_CLASSES)
```

`api_settings` 对象将检查任何用户定义的设置，否则将回退到默认值。 任何使用字符串导入路径引用类的设置都会自动导入并返回被引用的类，而不是字符串文字。

***

# API 参考 (API Reference)
## API 策略设置 (API policy settings)
*以下设置控制基本 API 策略，并应用于每个基于 `APIView` 类的视图或基于 `@api_view` 函数的视图。*

#### DEFAULT_RENDERER_CLASSES
渲染器类的列表或元组，用于确定返回 `Response` 对象时可能使用的默认渲染器集。

默认：
```python
(
    'rest_framework.renderers.JSONRenderer',
    'rest_framework.renderers.BrowsableAPIRenderer',
)
```

#### DEFAULT_PARSER_CLASSES
解析器类的列表或元组，用于确定访问 `request.data` 属性时使用的默认解析器集。

默认：
```python
(
    'rest_framework.parsers.JSONParser',
    'rest_framework.parsers.FormParser',
    'rest_framework.parsers.MultiPartParser'
)
```

#### DEFAULT_AUTHENTICATION_CLASSES
身份验证类的列表或元组，用于确定在访问 `request.user` 或 `request.auth` 属性时使用的默认身份验证器集。

默认：
```python
(
    'rest_framework.authentication.SessionAuthentication',
    'rest_framework.authentication.BasicAuthentication'
)
```

#### DEFAULT_PERMISSION_CLASSES
权限类的列表或元组，用于确定在视图开始时检查的默认权限集。权限必须由列表中的每个类授予。

默认：
```python
(
    'rest_framework.permissions.AllowAny',
)
```

#### DEFAULT_THROTTLE_CLASSES
限流类的列表或元组，用于确定在视图开始时检查的默认限流类集。

默认：`()`

#### DEFAULT_CONTENT_NEGOTIATION_CLASS
内容协商类，在给定传入请求的情况下，确定如何为响应选择渲染器。

默认：`'rest_framework.negotiation.DefaultContentNegotiation'`

#### DEFAULT_SCHEMA_CLASS
将用于模式生成的视图检查器类。

默认：`'rest_framework.schemas.AutoSchema'`

***

## 通用视图设置 (Generic view settings)
*以下设置控制通用基于类的视图的行为。*

#### DEFAULT_PAGINATION_SERIALIZER_CLASS
***
**此设置已被删除。**

分页 API 不使用序列化器来确定输出格式，您需要在分页类上重写 `get_paginated_response` 方法，以便指定如何控制输出格式。
***

#### DEFAULT_FILTER_BACKENDS
用于通用过滤的过滤器后端类列表。如果设置为 `None`，则禁用通用过滤。

#### PAGINATE_BY
***
**此设置已被删除。**

有关[设置分页样式](http://www.django-rest-framework.org/api-guide/pagination/#modifying-the-pagination-style)的进一步指导，请参阅分页文档。
***

#### PAGE_SIZE
用于分页的默认页面大小。如果设置为 `None`，则默认情况下禁用分页。

默认：`None`

#### PAGINATE_BY_PARAM
***
**此设置已被删除。**

有关[设置分页样式](http://www.django-rest-framework.org/api-guide/pagination/#modifying-the-pagination-style)的进一步指导，请参阅分页文档。
***

#### MAX_PAGINATE_BY
***
**此设置已被删除。**

有关[设置分页样式](http://www.django-rest-framework.org/api-guide/pagination/#modifying-the-pagination-style)的进一步指导，请参阅分页文档。
***

### SEARCH_PARAM
查询参数的名称，可用于指定 `SearchFilter` 使用的搜索词。

默认：`search`

#### ORDERING_PARAM
查询参数的名称，可用于指定 `OrderingFilter` 返回的结果的排序。

默认：`ordering`

***

## 版本控制设置 (Versioning settings)
#### DEFAULT_VERSION
当没有版本控制信息存在时，应该用于 `request.version` 的值。

默认：`None`

#### ALLOWED_VERSIONS
如果设置，则此值将限制版本控制方案可能返回的版本集合，并且如果提供的版本不在此集合中，则会引发错误。

默认：`None`

#### VERSION_PARAM
应该用于任何版本控制参数的字符串，例如媒体类型或 URL 查询参数。

默认：`'version'`

***

## 认证设置 (Authentication settings)
*以下设置控制未经身份验证的请求的行为。*

#### UNAUTHENTICATED_USER
应该用于为未经身份验证的请求初始化 `request.user` 的类。(如果要完全删除身份验证，例如从 `INSTALLED_APPS` 中删除 `django.contrib.auth`，将 `UNAUTHENTICATED_USER` 设置为 `None`。)

默认：`django.contrib.auth.models.AnonymousUser`

#### UNAUTHENTICATED_TOKEN
应该用于为未经身份验证的请求初始化 `request.auth` 的类。

默认：`None`

***

## 测试设置 (Test settings)
*以下设置控制 APIRequestFactory 和 APIClient 的行为*

#### TEST_REQUEST_DEFAULT_FORMAT
发出测试请求时应使用的默认格式。

这应该与 `TEST_REQUEST_RENDERER_CLASSES` 设置中某个渲染器类的格式相匹配。

默认：`'multipart'`

#### TEST_REQUEST_RENDERER_CLASSES
构建测试请求时支持的渲染器类。

构建测试请求时可以使用任何这些渲染器类的格式，例如：`client.post('/users', {'username': 'jamie'}, format='json')`

默认：
```python
(
    'rest_framework.renderers.MultiPartRenderer',
    'rest_framework.renderers.JSONRenderer'
)
```

***

## 模式生成控件 (Schema generation controls)
#### SCHEMA_COERCE_PATH_PK
如果设置，则在生成模式路径参数时，将 URL conf 中的 `'pk'` 标识符映射到实际字段名称上。通常这将是 `'id'`。这给出了更合适的表示，因为 “主键” 是实现细节，而 “标识符” 是更一般的概念。

默认：`True`

#### SCHEMA_COERCE_METHOD_NAMES
如果设置，则用于将内部视图集方法名称映射到模式生成中使用的外部操作名称。这允许我们生成更适合外部表示的名称，而不是代码库内部使用的名称。

默认：`{'retrieve': 'read', 'destroy': 'delete'}`

***

## 内容类型控件 (Content type controls)
#### URL_FORMAT_OVERRIDE
通过在请求 URL 中使用 `format=…` 查询参数，可用于覆盖默认内容协商 `Accept` 标头行为的 URL 参数的名称。

例如：`http://example.com/organizations/?format=csv`

如果此设置的值为 `None`，则 URL 格式覆盖将被禁用。

默认：`'format'`

#### FORMAT_SUFFIX_KWARG
URL conf 中用于提供格式后缀的参数名称。使用 `format_suffix_patterns` 包含后缀 URL patterns 时应用此设置。

例如：`http://example.com/organizations.csv/`

默认：`'format'`

***

## 日期和时间格式 (Date and time formatting)
*以下设置用于控制如何解析和渲染日期和时间表示。*

#### DATETIME_FORMAT
默认情况下用于渲染 `DateTimeField` 序列化器字段的输出的格式字符串。如果为 `None`，那么 `DateTimeField` 序列化器字段将返回 Python `datetime` 对象，并且日期时间编码将由渲染器决定。

可以是 `None`，`'iso-8601'` 或 Python [strftime 格式](https://docs.python.org/3/library/time.html#time.strftime)字符串中的任何一个。

默认：`'iso-8601'`

#### DATETIME_INPUT_FORMATS
默认情况下应用于解析 `DateTimeField` 序列化器字段输入的格式字符串列表。

可以是包含字符串 `'iso-8601'` 或 Python [strftime 格式](https://docs.python.org/3/library/time.html#time.strftime)字符串的列表。

默认：`['iso-8601']`

#### DATE_FORMAT
默认情况下用于渲染 `DateField` 序列化器字段的输出的格式字符串。如果为 `None`，那么 `DateField` 序列化器字段将返回 Python `date` 对象，并且日期编码将由渲染器决定。

可以是 `None`，`'iso-8601'` 或 Python [strftime 格式](https://docs.python.org/3/library/time.html#time.strftime)字符串中的任何一个。

默认：`'iso-8601'`

#### DATE_INPUT_FORMATS
默认情况下应用于解析 `DateField` 序列化器字段的输入的格式字符串列表。

可以是包含字符串 `'iso-8601'` 或 Python [strftime 格式](https://docs.python.org/3/library/time.html#time.strftime)字符串的列表。

默认：`['iso-8601']`

#### TIME_FORMAT
默认情况下用于渲染 `TimeField` 序列化器字段的输出的格式字符串。如果为 `None`，那么 `TimeField` 序列化器字段将返回 Python `time` 对象，并且时间编码将由渲染器决定。

可以是 `None`，`'iso-8601'` 或 Python [strftime 格式](https://docs.python.org/3/library/time.html#time.strftime)字符串中的任何一个。

默认：`'iso-8601'`

#### TIME_INPUT_FORMATS
默认情况下应用于解析 `TimeField` 序列化器字段的输入的格式字符串列表。

可以是包含字符串 `'iso-8601'` 或 Python [strftime 格式](https://docs.python.org/3/library/time.html#time.strftime)字符串的列表。

默认：`['iso-8601']`

***

## 编码 (Encodings)
#### UNICODE_JSON
设置为 `True` 时，JSON 响应将允许响应中的 unicode 字符。例如：
```python
{"unicode black star":"★"}
```

当设置为 `False` 时，JSON 响应将转义非 ascii 字符，如下所示：
```python
{"unicode black star":"\u2605"}
```

两种样式都符合 [RFC 4627](https://www.ietf.org/rfc/rfc4627.txt)，并且在语法上是有效的 JSON。在检查 API 响应时，首选 unicode 样式是因为它更加用户友好。

默认：`True`

#### COMPACT_JSON
当设置为 `True` 时，JSON 响应将返回紧凑表示，在 `':'` 和 `','` 字符之后没有间隔。例如：
```python
{"is_admin":false,"email":"jane@example"}
```

设置为 `False` 时，JSON 响应将返回稍微更详细的表示，如下所示：
```python
{"is_admin": false, "email": "jane@example"}
```

默认样式是返回缩小的响应，符合 [Heroku 的 API 设计指南](https://github.com/interagent/http-api-design#keep-json-minified-in-all-responses)。

默认：`True`

#### STRICT_JSON
当设置为 `True` 时，JSON 渲染和解析只会遵循语法上有效的 JSON，而 Python 的 `json` 模块接受的扩展浮点值 (`nan`，`inf`，`-inf`) 引发异常。这是推荐的设置，因为通常不支持这些值。例如，Javascript 的 `JSON.Parse` 和 PostgreSQL 的 JSON 数据类型都不接受这些值。

设置为 `False` 时，JSON 渲染和解析将是允许的。但是，这些值仍然无效，需要在代码中进行特殊处理。

默认：`True`

#### COERCE_DECIMAL_TO_STRING
当在不支持原生十进制类型的 API 表示中返回十进制对象时，通常最好以字符串的形式返回值。这避免了二进制浮点实现所产生的精度损失。

设置为 `True` 时，序列化器 `DecimalField` 类将返回字符串而不是 `Decimal` 对象。设置为 `False` 时，序列化器将返回 `Decimal` 对象，默认 JSON 编码器将作为浮点数返回。

默认：`True`

***

## 视图名称和描述 (View names and descriptions)
**以下设置用于生成视图名称和描述，如用于响应 `OPTIONS` 请求，以及用于可浏览 API 。**

#### VIEW_NAME_FUNCTION
表示在生成视图名称时应该使用的函数的字符串。

这应该是具有以下签名的函数：
```python
view_name(cls, suffix=None)
```

- `cls`：视图类。通常，名称函数会在生成描述性名称时会通过访问 `cls.__name__` 来检查类的名称。
- `suffix`：区分视图集中各个视图时使用的可选后缀。

默认：`'rest_framework.views.get_view_name'`

#### VIEW_DESCRIPTION_FUNCTION
表示生成视图描述时应使用的函数的字符串。

可以更改此设置以支持默认 markdown 以外的标记样式。例如，您可以使用它来支持在可浏览的 API 中输出的视图文档字符串中的 `rst` 标记。

这应该是具有以下签名的函数：
```python
view_description(cls, html=False)
```

- `cls`：视图类。通常，描述函数在生成描述时会通过访问 `cls.__doc__` 来检查类的文档字符串。
- `html`：指示是否需要 HTML 输出的布尔值。在可浏览的 API 中使用时为 `True`，在生成 `OPTIONS` 响应时使用 `False`。

默认：`'rest_framework.views.get_view_description'`

## HTML 选择字段截取 (HTML Select Field cutoffs)
用于在可浏览 API 中[渲染关系字段的选择字段截取](http://www.django-rest-framework.org/api-guide/relations/#select-field-cutoffs)的全局设置。

#### HTML_SELECT_CUTOFF
`html_cutoff` 值的全局设置。必须是整数。

默认：1000

#### HTML_SELECT_CUTOFF_TEXT
表示 `html_cutoff_text` 的全局设置的字符串。

默认：`"More than {count} items..."`

***

## 杂项设置 (Miscellaneous settings)
#### EXCEPTION_HANDLER
一个字符串，表示在为任何给定异常返回响应时应使用的函数。如果函数返回 `None`，则会引发 500 错误。

可以更改此设置以支持除默认 `{"detail": "Failure..."}` 响应之外的错误响应。例如，您可以使用它来提供API响应，例如 `{"errors": [{"message": "Failure...", "code": ""} ...]}`。

这应该是具有以下签名的函数：
```python
exception_handler(exc, context)
```

- `exc`：异常。

默认：`'rest_framework.views.exception_handler'`

#### NON_FIELD_ERRORS_KEY
一个字符串，表示应该用于不引用特定字段的序列化器错误的键，而是一般错误。

默认：`'non_field_errors'`

#### URL_FIELD_NAME
一个字符串，表示应该用于 `HyperlinkedModelSerializer` 生成的 URL 字段的键。

默认：`'url'`

#### NUM_PROXIES
0 或更大的整数，可用于指定 API 在后台运行的应用程序代理的数量。这允许限制更准确地识别客户端 IP 地址。如果设置为 `None`，那么限流类将使用不太严格的 IP 匹配。

默认：`None`
