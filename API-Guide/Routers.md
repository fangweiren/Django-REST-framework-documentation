# Routers
资源路由允许您快速声明给定资源控制器的所有公共路由。而不是为索引声明单独的路由…灵活多样的路由在一行代码中声明它们。—— [Ruby on Rails 文档](http://guides.rubyonrails.org/routing.html)

一些 Web 框架 (如 Rails) 提供了自动确定应用程序的 url 应该如何映射到处理传入请求的逻辑的功能。

REST framework 增加了对 Django 自动 URL 路由的支持，并为您提供一种简单、快速和一致的方式，将视图逻辑连接到一组 URL。

## 用法 (Usage)
这是一个使用 `SimpleRouter` 的简单 URL conf 的示例。
```python
from rest_framework import routers

router = routers.SimpleRouter()
router.register(r'users', UserViewSet)
router.register(r'accounts', AccountViewSet)
urlpatterns = router.urls
```
`register()` 方法有两个必需参数：

- `prefix` —— 用于此路由集的 URL 前缀。
- `viewset` —— 视图集类。

可选地，您还可以指定一个附加的参数：

- `base_name` —— 用于创建的 URL 名称的基础。如果未设置，basename 将根据视图集的 `queryset` 属性 (如果它有的话) 自动生成。注意，如果视图集不包含 `queryset` 属性，那么在注册视图集时必须设置 `base_name`。

上面的示例将生成以下 URL 模式：

- URL 模式: `^users/$` 名称: `'user-list'`
- URL 模式: `^users/{pk}/$` 名称: `'user-detail'`
- URL 模式: `^accounts/$` 名称: `'account-list'`
- URL 模式: `^accounts/{pk}/$` 名称: `'account-detail'`

***

**注意**：`base_name` 参数用于指定视图名称模式的初始部分。在上面的示例中，是 `user` 或 `account` 部分。

通常，您不需要指定 `base_name` 参数，但是如果您有一个定义了自定义 `get_queryset` 方法的视图集，那么该视图集可能没有 `.queryset` 属性集。如果您尝试注册该视图集，您将看到如下错误：
```python
'base_name' argument not specified, and could not automatically determine the name from the viewset, as it does not have a '.queryset' attribute.
```
这意味着您在注册视图集时需要显式设置 `base_name` 参数，因为它无法从模型名称自动确定。

***

### 使用路由器的 `include` (Using `include` with routers)
路由器实例上的 `.urls` 属性仅仅是 URL 模式的标准列表。关于如何包含这些 url，有许多不同的样式。

例如，您可以将 `router.urls` 附加到现有视图列表中......
```python
router = routers.SimpleRouter()
router.register(r'users', UserViewSet)
router.register(r'accounts', AccountViewSet)

urlpatterns = [
    url(r'^forgot-password/$', ForgotPasswordFormView.as_view()),
]

urlpatterns += router.urls
```
另外，你可以使用 Django 的 `include` 函数，像这样...
```python
urlpatterns = [
    url(r'^forgot-password/$', ForgotPasswordFormView.as_view()),
    url(r'^', include(router.urls)),
]
```
您可以在应用程序命名空间中使用 `include`：
```python
urlpatterns = [
    url(r'^forgot-password/$', ForgotPasswordFormView.as_view()),
    url(r'^api/', include((router.urls, 'app_name'))),
]
```
或应用程序和实例命名空间：
```python
urlpatterns = [
    url(r'^forgot-password/$', ForgotPasswordFormView.as_view()),
    url(r'^api/', include((router.urls, 'app_name'), namespace='instance_name')),
]
```
有关详细信息，请参阅 Django 的 [URL 命名空间文档](https://docs.djangoproject.com/en/1.11/topics/http/urls/#url-namespaces)和 [`include` API 参考](https://docs.djangoproject.com/en/2.0/ref/urls/#include)。

***
**注意**：如果对超链接序列化器使用命名空间，则还需要确保序列化器上的任何 `view_name` 参数都能正确反映命名空间。在上面的示例中，您需要为超链接到用户详细视图的序列化字段包含一个诸如 `view_name='api:user-detail'` 之类的参数。

使用类似 `%(model_name)-detail` 模式自动生成 `view_name`。除非您的模型名称实际上发生冲突，否则在使用超链接序列化器时，最好不要对 Django REST framework 视图进行命名。

***

### 额外操作的路由 (Routing for extra actions)
视图集可以通过使用 `@action` 装饰器装饰方法来[标记路由的额外操作](http://www.django-rest-framework.org/api-guide/viewsets/#marking-extra-actions-for-routing)。这些额外的操作将包含在生成的路由中。例如，在 `UserViewSet` 类中给出 `set_password` 方法：
```python
from myapp.permissions import IsAdminOrIsSelf
from rest_framework.decorators import action

class UserViewSet(ModelViewSet):
    ...

    @action(methods=['post'], detail=True, permission_classes=[IsAdminOrIsSelf])
    def set_password(self, request, pk=None):
        ...
```
将生成以下路由：

- URL 模式: `^users/{pk}/set_password/$`
- URL 名称: `'user-set-password'`

默认情况下，URL 模式基于方法名，URL 名称是 `ViewSet.basename` 和带连字符的方法名称的组合。如果您不想使用这些值中的任何一个默认值，您可以向 `@action` 装饰器提供 `url_path` 和 `url_name` 参数。

例如，如果要将自定义操作的 URL 更改为 `^users/{pk}/change-password/$`，则可以编写：
```python
from myapp.permissions import IsAdminOrIsSelf
from rest_framework.decorators import action

class UserViewSet(ModelViewSet):
    ...

    @action(methods=['post'], detail=True, permission_classes=[IsAdminOrIsSelf],
            url_path='change-password', url_name='change_password')
    def set_password(self, request, pk=None):
        ...
```
上面的示例现在将生成以下 URL 模式：

- URL path: `^users/{pk}/change-password/$`
- URL name: `'user-change_password'`

# API 指南 (API Guide)
## SimpleRouter
此路由器包含标准 `list`，`create`，`retrieve`，`update`，`partial_update` 和 `destroy` 操作的路由。视图集还可以使用 `@action` 装饰器标记被路由的其他方法。
<table>
   <tr>
      <td>URL Style</td>
      <td>HTTP Method</td>
      <td>Action</td>
      <td>URL Name</td>
   </tr>
   <tr>
      <td rowspan="2">{prefix}/</td>
      <td>GET</td>
      <td>list</td>
      <td rowspan="2">{basename}-list</td>
   </tr>
   <tr>
      <td>POST</td>
      <td>create</td>
   </tr>
   <tr>
      <td>{prefix}/{url_path}/</td>
      <td>GET, or as specified by `methods` argument</td>
      <td>`@action(detail=False)` decorated method</td>
      <td>{basename}-{url_name}</td>
   </tr>
   <tr>
      <td rowspan="4">{prefix}/{lookup}/</td>
      <td>GET</td>
      <td>retrieve</td>
      <td rowspan="4">{basename}-detail</td>
   </tr>
   <tr>
      <td>PUT</td>
      <td>update</td>
   </tr>
   <tr>
      <td>PATCH</td>
      <td>partial_update</td>
   </tr>
   <tr>
      <td>DELETE</td>
      <td>destroy</td>
   </tr>
   <tr>
      <td>{prefix}/{lookup}/{url_path}/</td>
      <td>GET, or as specified by `methods` argument</td>
      <td>`@action(detail=True)` decorated method</td>
      <td>{basename}-{url_name}</td>
   </tr>
</table>

默认情况下，`SimpleRouter` 创建的 URL 附加了一个尾部斜杠。在实例化路由器时，可以通过将 `trailing_slash` 参数设置为 `False` 来修改此行为。例如：

```python
router = SimpleRouter(trailing_slash=False)
```

在 Django 中，尾部斜杠是常规的，但在某些其他框架 (如 Rails) 中默认不使用。您选择使用哪种样式主要是偏好问题，尽管某些 javascript 框架可能需要特定的路由样式。

路由器将匹配包含除斜杠和句点字符以外的任何字符的查找值。对于更严格 (或宽松) 的查找模式，请在视图集上设置 `lookup_value_regex` 属性。例如，您可以将查找限制为有效的 UUID：

```python
class MyModelViewSet(mixins.RetrieveModelMixin, viewsets.GenericViewSet):
    lookup_field = 'my_model_id'
    lookup_value_regex = '[0-9a-f]{32}'
```

## DefaultRouter
此路由器与上面的 `SimpleRouter` 类似，但另外包括一个默认的 API 根视图，它返回包含所有列表视图的超链接的响应。它还为可选的 `.json` 样式格式后缀生成路由。
<table>
   <tr>
      <td>URL Style</td>
      <td>HTTP Method</td>
      <td>Action</td>
      <td>URL Name</td>
   </tr>
   <tr>
      <td>[.format]</td>
      <td>GET</td>
      <td>automatically generated root view</td>
      <td>api-root</td>
   </tr>
   <tr>
      <td rowspan="2">{prefix}/[.format]</td>
      <td>GET</td>
      <td>list</td>
      <td rowspan="2">{basename}-list</td>
   </tr>
   <tr>
      <td>POST</td>
      <td>create</td>
   </tr>
   <tr>
      <td>{prefix}/{url_path}/[.format]</td>
      <td>GET, or as specified by `methods` argument</td>
      <td>`@action(detail=False)` decorated method</td>
      <td>{basename}-{url_name}</td>
   </tr>
   <tr>
      <td rowspan="4">{prefix}/{lookup}/[.format]</td>
      <td>GET</td>
      <td>retrieve</td>
      <td rowspan="4">{basename}-detail</td>
   </tr>
   <tr>
      <td>PUT</td>
      <td>update</td>
   </tr>
   <tr>
      <td>PATCH</td>
      <td>partial_update</td>
   </tr>
   <tr>
      <td>DELETE</td>
      <td>destroy</td>
   </tr>
   <tr>
      <td>{prefix}/{lookup}/{url_path}/[.format]</td>
      <td>GET, or as specified by `methods` argument</td>
      <td>`@action(detail=True)` decorated method</td>
      <td>{basename}-{url_name}</td>
   </tr>
</table>

与 `SimpleRouter` 一样，在实例化路由器时通过将 `trailing_slash` 参数设置为 `False` 来删除 URL 路由上的尾部斜杠。

```python
router = DefaultRouter(trailing_slash=False)
```

# 自定义路由器 (Custom Routers)
实现自定义路由器不是您经常需要做的事情，但是如果您对 API 的 URL 如何构造有特定的要求，它会很有用。这样做允许您以可重用的方式封装 URL 结构，以确保您不必为每个新视图显式编写 URL 模式。

实现自定义路由的最简单方法是对现有路由类之一进行子类化。`.routes` 属性用于对将映射到每个视图集的 URL 模式进行模板化。`.routes` 属性是 `Route` 元组列表。

`Route` 命名元组的参数：

**url**：表示被路由的 URL 的字符串。可能包含以下格式字符串：

- `{prefix}` —— 用于这组路由的 URL 前缀。
- `{lookup}` —— 用于匹配单个实例的查找字段。
- `{trailing_slash}` —— “/” 或空字符串，取决于 `trailing_slash` 参数。

**mapping**：HTTP 方法名称到视图方法的映射

**name**：在 `reverse` 调用时使用的 URL 名称。可能包含以下格式字符串：

- `{basename}` —— 用于创建的 URL 名称的基础。

**initkwargs**：实例化视图时应传递的任何其他参数的字典。请注意，`detail`，`basename` 和 `suffix` 参数是视图集内省保留的，并也可由可浏览 API 使用来生成视图名称和痕迹链接。

## 自定义动态路由 (Customizing dynamic routes)
您还可以自定义 `@action` 装饰器的路由方式。在 `.routes` 列表中包含名为 tuple 的 `DynamicRoute`，将 `detail` 参数设置为适用于基于列表和基于详细信息的路由。除了 `detail` 之外，`DynamicRoute` 的参数是：

**url**：表示被路由的 URL 的字符串。可以包含与 `Route` 相同的格式字符串，并额外接受 `{url_path}` 格式字符串。

**name**：在 `reverse` 调用时使用的 URL 名称。可能包含以下格式字符串：

- `{basename}` —— 用于创建的 URL 名称的基础。
- `{url_name}` —— 提供给 `@action` 的 `url_name`。

**initkwargs**：实例化视图时应传递的任何其他参数的字典。

## 举个栗子
下面的示例将只路由 `list` 和 `retrieve` 操作，不使用斜杠约定。
```python
from rest_framework.routers import Route, DynamicRoute, SimpleRouter

class CustomReadOnlyRouter(SimpleRouter):
    """
    用于只读 API 的路由器，不使用尾部斜杠。
    """
    routes = [
        Route(
            url=r'^{prefix}$',
            mapping={'get': 'list'},
            name='{basename}-list',
            detail=False,
            initkwargs={'suffix': 'List'}
        ),
        Route(
            url=r'^{prefix}/{lookup}$',
            mapping={'get': 'retrieve'},
            name='{basename}-detail',
            detail=True,
            initkwargs={'suffix': 'Detail'}
        ),
        DynamicRoute(
            url=r'^{prefix}/{lookup}/{url_path}$',
            name='{basename}-{url_name}',
            detail=True,
            initkwargs={}
        )
    ]
```
让我们来看看 `CustomReadOnlyRouter` 为一个简单的视图集生成的路由。

`views.py`：

```python
class UserViewSet(viewsets.ReadOnlyModelViewSet):
    """
    提供标准操作的视图集
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
    lookup_field = 'username'

    @action(detail=True)
    def group_names(self, request, pk=None):
        """
        返回给定用户所属的所有组名称的列表。
        """
        user = self.get_object()
        groups = user.groups.all()
        return Response([group.name for group in groups])
```

`urls.py` :

```python
router = CustomReadOnlyRouter()
router.register('users', UserViewSet)
urlpatterns = router.urls
```

将生成以下映射...

| URL | HTTP Method | Action | URL Name |
| - | - | - | - |
| /users | GET | list | user-listAction |
| /users/{username} | GET | retrieve | 	user-detail |
| /users/{username}/group-names | GET | group_names | user-group-names |

有关设置 `.routes` 属性的另一个示例，请参阅 `SimpleRouter` 类的源代码。

## 高级定制路由器 (Advanced custom routers)
如果您想提供整个自定义行为，可以重写 `BaseRouter` 并重写 `get_urls(self)` 方法。该方法应检查已注册的视图集并返回 URL 模式列表。可以通过访问 `self.registry` 属性来检查已注册的 prefix (前缀)，viewsets (视图集) 和 basename tuoles (基本名称元组)。

您可能还想重写 `get_default_base_name（self，viewset）` 方法，或者在向路由器注册视图集时始终显式设置 `base_name` 参数。

# 第三方包 (Third Party Packages)
以下第三方包也可用。

## DRF Nested Routers
[drf-nested-routers 包](https://github.com/alanjds/drf-nested-routers)提供了用于处理嵌套资源的路由器和关系字段。

## ModelRouter (wq.db.rest)
[wq.db 包](https://wq.io/wq.db)提供了一个高级的 [ModelRouter](https://wq.io/docs/router) 类 (和单例实例)，它使用 `register_model()` API 扩展了 `DefaultRouter`。与 Django 的 `admin.site.register` 非常相似，`rest.router.register_model` 唯一需要的参数是模型类。url 前缀、序列化器和视图集的合理默认值将从模型和全局配置中推断出来。
```python
from wq.db import rest
from myapp.models import MyModel

rest.router.register_model(MyModel)
```

## DRF-extensions
[`DRF-extensions` 包](https://chibisov.github.io/drf-extensions/docs/)提供了用于创建[嵌套视图集](https://chibisov.github.io/drf-extensions/docs/#nested-routes)、具有[可定制端点名称](https://chibisov.github.io/drf-extensions/docs/#controller-endpoint-name)的[集合级控制器](https://chibisov.github.io/drf-extensions/docs/#collection-level-controllers)的[路由器](https://chibisov.github.io/drf-extensions/docs/#routers)。
