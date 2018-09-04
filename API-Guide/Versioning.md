# 版本控制 (Versioning)
对接口进行版本控制只是一种杀死已部署客户端的 “礼貌” 方式。—— [Roy Fielding](https://www.slideshare.net/evolve_conference/201308-fielding-evolve/31)

API 版本控制允许您更改不同客户端之间的行为。REST framework 提供了许多不同的版本控制方案。

版本控制由传入的客户端请求决定，并且可以基于请求 URL，也可以基于请求头。

有许多有效的方法来处理版本控制。[非版本化的系统也可能是合适的](https://www.infoq.com/articles/roy-fielding-on-versioning)，特别是如果您正在为超出您控制之外的多个客户端的非常长期的系统进行工程设计。

## REST framework 的版本控制 (Versioning with REST framework)
当启用 API 版本控制时，`request.version` 属性将包含对应于传入客户端请求中请求的版本的字符串。

默认情况下，版本控制未启用，`request.version` 将始终返回 `None`。

#### 基于版本改变行为 (Varying behavior based on the version)
如何改变 API 行为取决于您，但您可能通常需要的一个示例是在新版本中切换到不同的序列化样式。例如：
```python
def get_serializer_class(self):
    if self.request.version == 'v1':
        return AccountSerializerVersion1
    return AccountSerializer
```

#### 版本化 API 的反向 URL (Reversing URLs for versioned APIs)
REST framework 包含的 `reverse` 函数与版本控制方案相关联。您需要确保将当前 `request` 包含为关键字参数，如下所示。
```python
from rest_framework.reverse import reverse

reverse('bookings-list', request=request)
```

上述函数将应用适用于请求版本的任何 URL 转换。例如：

- 如果正在使用 `NamespacedVersioning`，并且 API 版本为 'v1'，那么使用的 URL 查找将是 `'v1:bookings-list'`，它可能解析为像 `http://example.org/v1/bookings/` 这样的 URL。
- 如果正在使用 `QueryParameterVersioning`，并且 API 版本为 `1.0`，那么返回的 URL 可能类似于 `http://example.org/bookings/?version=1.0`

#### 版本化 API 和超链接序列化器 (Versioned APIs and hyperlinked serializers)
将超链接序列化样式与基于 URL 的版本控制方案一起使用时，请确保将请求作为上下文包含在序列化器中。
```python
def get(self, request):
    queryset = Booking.objects.all()
    serializer = BookingsSerializer(queryset, many=True, context={'request': request})
    return Response({'all_bookings': serializer.data})
```

这样做将允许任何返回的 URL 包含适当的版本控制。

## 配置版本控制方案 (Configuring the versioning scheme)
版本控制方案由 `DEFAULT_VERSIONING_CLASS` 设置键定义。
```python
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.NamespaceVersioning'
}
```

除非显式设置，否则 `DEFAULT_VERSIONING_CLASS` 的值将为 `None`。在这种情况下，`request.version` 属性将始终返回 `None`。

您还可以在单个视图上设置版本控制方案。通常，您不需要这样做，因为全局使用单个版本控制方案更有意义。如果您确实需要这样做，请使用 `versioning_class` 属性。
```python
class ProfileList(APIView):
    versioning_class = versioning.QueryParameterVersioning
```

#### 其他版本设置 (Other versioning settings)
以下设置键也用于控制版本控制：

- `DEFAULT_VERSION`。当没有版本控制信息存在时，用于 `request.version` 的值。默认为 `None`。
- `ALLOWED_VERSIONS`。如果设置，则该值将限制版本控制方案可能返回的版本集，并且如果提供的版本不在此集合中，则会引发错误。请注意，用于 `DEFAULT_VERSION` 设置的值始终被认为是 `ALLOWED_VERSIONS` 集的一部分 (除非它是 `None`)。默认为 `None`。
- `VERSION_PARAM`。应用于任何版本控制参数的字符串，例如媒体类型或 URL 查询参数。默认为 `'version'`。

您还可以通过定义自己的版本控制方案并使用 `default_version`，`allowed_versions` 和 `version_param` 类变量，在每个视图或每个视图集的基础上设置版本控制类以及这三个值。例如，如果您想使用 `URLPathVersioning`：
```python
from rest_framework.versioning import URLPathVersioning
from rest_framework.views import APIView

class ExampleVersioning(URLPathVersioning):
    default_version = ...
    allowed_versions = ...
    version_param = ...

class ExampleView(APIVIew):
    versioning_class = ExampleVersioning
```

***

# API 参考 (API Reference)
## AcceptHeaderVersioning
此方案要求客户端在 `Accept` 标头中将版本指定为媒体类型的一部分。该版本作为媒体类型参数包含在内，它补充了主要媒体类型。

以下是使用 accept 标头版本控制样式的 HTTP 请求示例。
```python
GET /bookings/ HTTP/1.1
Host: example.com
Accept: application/json; version=1.0
```

在上面的示例请求中 `request.version` 属性将返回字符串 `'1.0'`。

基于 Accept header 的版本控制[通常被认为](http://blog.steveklabnik.com/posts/2011-07-03-nobody-understands-rest-or-http#i_want_my_api_to_be_versioned)是[最佳实践](https://github.com/interagent/http-api-design/blob/master/en/foundations/require-versioning-in-the-accepts-header.md)，尽管其他样式可能更适合您的客户端需求。

#### 使用带有供应商媒体类型的 accept 标头 (Using accept headers with vendor media types)
严格地说，`json` 媒体类型未指定为[包含额外参数](https://tools.ietf.org/html/rfc4627#section-6)。如果您正在构建一个明确指定的公共 API，您可以考虑使用[供应商媒体类型](https://en.wikipedia.org/wiki/Internet_media_type#Vendor_tree)。为此，使用具有自定义媒体类型的基于 JSON 的渲染器配置您的渲染器：
```python
class BookingsAPIRenderer(JSONRenderer):
    media_type = 'application/vnd.megacorp.bookings+json'
```

您的客户端请求现在看起来像这样：
```python
GET /bookings/ HTTP/1.1
Host: example.com
Accept: application/vnd.megacorp.bookings+json; version=1.0
```

## URLPathVersioning
此方案要求客户端将版本指定为 URL 路径的一部分。
```python
GET /v1/bookings/ HTTP/1.1
Host: example.com
Accept: application/json
```

您的 URL conf 必须包含一个与 `'version'` 关键字参数相匹配的模式，以便版本控制方案可以使用此信息。
```python
urlpatterns = [
    url(
        r'^(?P<version>(v1|v2))/bookings/$',
        bookings_list,
        name='bookings-list'
    ),
    url(
        r'^(?P<version>(v1|v2))/bookings/(?P<pk>[0-9]+)/$',
        bookings_detail,
        name='bookings-detail'
    )
]
```

## NamespaceVersioning
对于客户端，此方案与 `URLPathVersioning` 相同。唯一的区别是，它是如何在 Django 应用程序中配置的，因为它使用 URL 命名空间而不是 URL 关键字参数。
```python
GET /v1/something/ HTTP/1.1
Host: example.com
Accept: application/json
```

使用此方案，`request.version` 属性是根据与传入请求路径匹配的 `namespace` 确定的。

在下面的示例中，我们给出了一组视图，其中有两个不同的可能的 URL 前缀，每个前缀都位于不同的名称空间中：
```python
# bookings/urls.py
urlpatterns = [
    url(r'^$', bookings_list, name='bookings-list'),
    url(r'^(?P<pk>[0-9]+)/$', bookings_detail, name='bookings-detail')
]

# urls.py
urlpatterns = [
    url(r'^v1/bookings/', include('bookings.urls', namespace='v1')),
    url(r'^v2/bookings/', include('bookings.urls', namespace='v2'))
]
```

如果您只需要简单的版本控制方案，则 `URLPathVersioning` 和 `NamespaceVersioning` 都是合理的。`URLPathVersioning` 方法可能更适合小型临时项目，而 `NamespaceVersioning` 可能更容易管理大型项目。

## HostNameVersioning
主机名版本控制方案要求客户端将请求的版本指定为URL中主机名的一部分。

例如，以下是对 `http://v1.example.com/bookings/` URL 的 HTTP 请求：
```python
GET /bookings/ HTTP/1.1
Host: v1.example.com
Accept: application/json
```

默认情况下，此实现要求主机名与此简单正则表达式匹配：
```python
^([a-zA-Z0-9]+)\.[a-zA-Z0-9]+\.[a-zA-Z0-9]+$
```

请注意，第一个组用括号括起来，表示这是主机名的匹配部分。

在调试模式中，使用 `HostNameVersioning` 方案可能会很尴尬，因为您通常会访问原始的 IP 地址 (如 127.0.0.1)。有关如何[使用自定义子域访问本地主机](https://reinteractive.net/posts/199-developing-and-testing-rails-applications-with-subdomains)的各种在线教程，在这种情况下您可能会发现有用的子域。

如果您有基于版本将传入请求路由到不同服务器的要求，则基于主机名的版本控制特别有用，因为您可以为不同的 API 版本配置不同的 DNS 记录。

## QueryParameterVersioning
此方案是一种简单样式，其中包含版本作为 URL 中的查询参数。例如：
```python
GET /something/?version=0.1 HTTP/1.1
Host: example.com
Accept: application/json
```

***

# 自定义版本控制方案 (Custom versioning schemes)
要实现自定义版本控制方案，请继承 `BaseVersioning` 并覆盖 `.determine_version` 方法。

## 举个栗子
以下示例使用自定义 `X-API-Version` 标头来确定所请求的版本。
```python
class XAPIVersionScheme(versioning.BaseVersioning):
    def determine_version(self, request, *args, **kwargs):
        return request.META.get('HTTP_X_API_VERSION', None)
```

如果您的版本控制方案基于请求 URL，您还需要更改版本化 URL 的确定方式。为了做到这一点，您应重写类的 `.reverse()` 方法。有关示例，请参阅源代码。
