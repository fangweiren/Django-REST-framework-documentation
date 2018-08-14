# 教程 6：ViewSets ＆ Routers
REST framework 包括一个用于处理 `ViewSets` 的抽象，它允许开发人员集中精力对 API 的状态和交互进行建模，并保留 URL 结构，根据通用约定自动处理。

`ViewSet` 类与 `View` 类几乎相同，只是它们提供诸如 `read` 或 `update` 等操作，而不提供诸如 `get` 或 `put` 等方法处理程序。

一个 `ViewSet` 类最后时刻只绑定一组方法处理程序，当它被实例化为一组视图时，通常通过使用一个 `Router` 类来处理定义复杂的 url。

## 使用 ViewSets 重构
我们来看看当前的一组视图，并将它们重构为视图集。

首先让我们将 `UserList` 和 `UserDetail` 视图重构为单个 `UserViewSet`。我们可以删除这两个视图，并用一个类替换它们：
```python
class UserViewSet(viewsets.ReadOnlyModelViewSet):
    """
    这个视图集自动提供 `list` 和 `detail` 操作。
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
```
这里我们使用了 `ReadOnlyModelViewSet` 类来自动提供默认的 “只读” 操作。我们仍然像我们使用常规视图时一样设置 `queryset` 和 `serializer_class` 属性，但我们不再需要为两个单独的类提供相同的信息。

接下来我们将替换 `SnippetList`，`SnippetDetail` 和 `SnippetHighlight` 视图类。我们可以删除这三个视图，并再次用一个类替换它们。
```python
class SnippetViewSet(viewsets.ModelViewSet):
    """
    这个视图集自动提供 `list`，`create`，`retrieve`，`update`和`destroy`操作。

    另外我们还提供了一个额外的 `highlight` 操作。
    """
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly,)

    @action(detail=True, renderer_classes=[renderers.StaticHTMLRenderer])
    def highlight(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```
这次我们使用 `ModelViewSet` 类来获得完整的默认读写操作集。

请注意，我们还使用了 `@action` 装饰器来创建一个名为 `highlight` 的自定义操作。这个装饰器可以用来添加任何不符合标准 `create`/`update`/`delete` 样式的自定义端点。

使用 `@action` 装饰器的自定义操作默认会响应 `GET` 请求。如果我们需要响应 `POST` 请求的操作，我们可以使用 `methods` 参数。

自定义操作的 URL 默认取决于方法名称本身。如果要更改 URL 的构造方式，可以包含 `url_path` 作为装饰器关键字参数。

## 明确地将 ViewSets 绑定到 URL
当我们定义 URLConf 时，处理程序方法只能绑定到操作上。为了看看到底发生了什么，让我们首先从我们的 ViewSets 中明确地创建一组视图。

在 `snippets/urls.py` 文件中，我们将 `ViewSet` 类绑定到一组具体视图中。
```python
from snippets.views import SnippetViewSet, UserViewSet, api_root
from rest_framework import renderers

snippet_list = SnippetViewSet.as_view({
    'get': 'list',
    'post': 'create'
})
snippet_detail = SnippetViewSet.as_view({
    'get': 'retrieve',
    'put': 'update',
    'patch': 'partial_update',
    'delete': 'destroy'
})
snippet_highlight = SnippetViewSet.as_view({
    'get': 'highlight'
}, renderer_classes=[renderers.StaticHTMLRenderer])
user_list = UserViewSet.as_view({
    'get': 'list'
})
user_detail = UserViewSet.as_view({
    'get': 'retrieve'
})
```
请注意，我们是如何从每个 `ViewSet` 类创建多个视图，通过将 http 方法绑定到每个视图所需的操作中。

现在我们已经将资源绑定到具体的视图中，我们可以像往常一样在 URL conf 中注册视图。
```python
urlpatterns = format_suffix_patterns([
    url(r'^$', api_root),
    url(r'^snippets/$', snippet_list, name='snippet-list'),
    url(r'^snippets/(?P<pk>[0-9]+)/$', snippet_detail, name='snippet-detail'),
    url(r'^snippets/(?P<pk>[0-9]+)/highlight/$', snippet_highlight, name='snippet-highlight'),
    url(r'^users/$', user_list, name='user-list'),
    url(r'^users/(?P<pk>[0-9]+)/$', user_detail, name='user-detail')
])
```

## 使用 Routers
因为我们使用 `ViewSet` 类而不是 `View` 类，所以实际上我们不需要自己设计 URL。将资源连接到视图和 URL 的约定可以使用 `Router` 类自动处理。我们需要做的就是使用路由器注册相应的视图集，然后让它执行其余操作。

这是我们重新连线的 `snippets/urls.py` 文件。
```python
from django.conf.urls import url, include
from rest_framework.routers import DefaultRouter

from snippets import views

# 创建路由器并注册我们的视图。
router = DefaultRouter()
router.register(r'snippets', views.SnippetViewSet)
router.register(r'users', views.UserViewSet)

# API URL 现在由路由器自动确定。
urlpatterns = [
    url(r'^', include(router.urls))
]
```
用路由器注册视图集类似于提供 urlpattern。我们包括两个参数——视图的 URL 前缀和视图集本身。

我们使用的 `DefaultRouter` 类也为我们自动创建了 API 根视图，所以我们现在可以从 `views` 模块中删除 `api_root` 方法。

## 视图与视图集之间的权衡
使用视图集可以是一个非常有用的抽象。它有助于确保 URL 约定在您的 API 中保持一致，最大限度地减少编写所需的代码量，并允许您专注于 API 提供的交互和表示，而不是 URL conf 的细节。

这并不意味着它总是正确的做法。当使用基于类的视图而不是基于函数的视图时，也有类似的权衡考虑。使用视图集不像单独构建视图那样明确。

在[本教程的第 7 部分](http://www.iamnancy.top/post/173/)中，我们将介绍如何添加 API 模式，以及如何使用客户端库或命令行工具与我们的 API 进行交互。
