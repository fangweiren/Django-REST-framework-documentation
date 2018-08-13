# 教程 5：关系和超链接 API
目前我们的 API 中的关系是通过使用主键来表示。在本教程的这一部分中，我们将改进 API 的内聚力和可发现性，而不是使用超链接来进行关系。

## 为我们的 API 的根地址创建端点
现在我们有 'snippets' 和 'users' 端点，但我们没有一个指向我们 API 的入口。要创建一个，我们将使用基于函数的常规视图和我们前面介绍的 `@api_view` 装饰器。在你的 `snippets / views.py` 中添加：
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework.reverse import reverse


@api_view(['GET'])
def api_root(request, format=None):
    return Response({
        'users': reverse('user-list', request=request, format=format),
        'snippets': reverse('snippet-list', request=request, format=format)
    })
```
这里应该注意两件事。首先，我们使用 REST framework 的 reverse 函数来返回完全限定的URL；其次，URL 模式通过我们稍后将在我们的 `snippets/urls.py` 中声明的便利名称进行标识。

## 为高亮显示 snippets 创建端点
另一个明显的事情是我们的 pastebin API 仍然缺少高亮显示代码的端点。

与所有其他 API 端点不同，我们不想使用 JSON，而只是呈现 HTML 表示。REST framework 提供了两种 HTML 渲染器，一种是使用模板来处理渲染的 HTML，另一种是处理预渲染的 HTML。第二个渲染器是我们要用于此端点的渲染器。

在创建代码高亮显示视图时，我们需要考虑的另一件事是不存在我们可以使用的具体的通用视图。我们不是返回一个对象实例，而是返回一个对象实例的属性。

我们将使用基类来表示实例，并创建我们自己的 `.get()` 方法，而不是使用具体的通用视图。在您的 `snippets/views.py` 中添加：
```python
from rest_framework import renderers
from rest_framework.response import Response

class SnippetHighlight(generics.GenericAPIView):
    queryset = Snippet.objects.all()
    renderer_classes = (renderers.StaticHTMLRenderer,)

    def get(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)
```
像往常一样，我们需要将我们创建的新视图添加到 URLconf 中。我们将在 `snippets/urls.py` 中为我们的新 API 根路径添加一个 url 模式：
```python
url(r'^$', views.api_root),
```
然后为高亮 snippet 添加一个 url 模式：
```python
url(r'^snippets/(?P<pk>[0-9]+)/highlight/$', views.SnippetHighlight.as_view()),
```

## 超链接我们的 API
处理实体之间的关系是 Web API 设计中更具挑战性的方面之一。这里有一些我们可能会选择代表关系的不同方法：

- 使用主键。
- 在实体之间使用超链接。
- 在相关实体上使用唯一的标识字段。
- 使用默认字符串代表相关实体。
- 将相关实体嵌套在父代表中。
- 一些其他自定义表示。

REST framework 支持所有这些样式，并且可以在正向或反向关系中应用它们，或者通过自定义管理器 (如通用外键) 应用它们。

在这种情况下，我们希望在实体之间使用超链接样式。为了做到这一点，我们将修改我们的序列化器来扩展 `HyperlinkedModelSerializer`，而不是现有的 `ModelSerializer`。

`HyperlinkedModelSerializer` 与 `ModelSerializer` 有以下区别：

- 它默认不包含 id 字段。
- 它包含一个 `url` 字段，使用 `HyperlinkedIdentityField`。
- 关系使用 `HyperlinkedRelatedField`，而不是 `PrimaryKeyRelatedField`。

我们可以轻松地重写现有的序列化器来使用超链接。在你的 `snippets/serializers.py` 中添加：
```python
class SnippetSerializer(serializers.HyperlinkedModelSerializer):
    owner = serializers.ReadOnlyField(source='owner.username')
    highlight = serializers.HyperlinkedIdentityField(view_name='snippet-highlight', format='html')

    class Meta:
        model = Snippet
        fields = ('url', 'id', 'highlight', 'owner',
                  'title', 'code', 'linenos', 'language', 'style')


class UserSerializer(serializers.HyperlinkedModelSerializer):
    snippets = serializers.HyperlinkedRelatedField(many=True, view_name='snippet-detail', read_only=True)

    class Meta:
        model = User
        fields = ('url', 'id', 'username', 'snippets')
```
请注意，我们还添加了一个新的 `'highlight'` 字段。该字段与 `url` 字段类型相同，不同之处在于它指向的是 `'snippet-highlight'` url 模式，而不是 `'snippet-detail'` url 模式。

因为我们已经包含了格式后缀的 URL，例如 `'.json'`，我们还需要在 `highlight` 字段上指出任何格式后缀的超链接返回应该使用 `.html'` 后缀。

## 确保我们的 URL 模式被命名
如果我们要有超链接的 API，我们需要确保命名 URL 模式。我们来看看我们需要命名的 URL 模式。

- 我们的 API 根地址是指 `'user-list'` 和 `'snippet-list'`。
- 我们的 snippet 序列化器包含一个指向 `'snippet-highlight'` 的字段。
- 我们的 user 序列化器包含一个指向 `'snippet-detail'` 的字段。
- 我们的 snippet 和 user 序列化器包括 `'url'` 字段，默认情况下将指向 `'{model_name}-detail'`，在这个例子中就是 `'snippet-detail'` 和 `'user-detail'`。

将所有这些名称添加到我们的 URLconf 后，我们最后的 `snippets/urls.py` 文件应如下所示：
```python
from django.conf.urls import url, include
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = format_suffix_patterns([
    url(r'^$', views.api_root),
    url(r'^snippets/$',
        views.SnippetList.as_view(),
        name='snippet-list'),
    url(r'^snippets/(?P<pk>[0-9]+)/$',
        views.SnippetDetail.as_view(),
        name='snippet-detail'),
    url(r'^snippets/(?P<pk>[0-9]+)/highlight/$',
        views.SnippetHighlight.as_view(),
        name='snippet-highlight'),
    url(r'^users/$',
        views.UserList.as_view(),
        name='user-list'),
    url(r'^users/(?P<pk>[0-9]+)/$',
        views.UserDetail.as_view(),
        name='user-detail')
])
```

## 添加分页
users 和 snippets 的列表视图最终会返回很多实例，所以我们真的要确保对结果进行分页，并允许 API 客户端遍历每个单独的页面。

我们可以通过稍微修改 `tutorial/settings.py` 文件来更改默认列表样式以使用分页。添加以下设置：
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
```
请注意，REST framework 中的所有设置都放在一个名为 `REST_FRAMEWORK` 的字典中，这有助于让他们与其他项目设置保持良好的分离。

如果需要，我们也可以自定义分页样式，但在这个例子中，我们将一直使用默认设置。

## 浏览 API
如果我们打开浏览器并导航到可浏览的 API，您会发现现在您可以通过简单的链接访问 API。

您还可以在 snippet 实例上看到 “highlight” 链接，这会带您跳转到代码高亮显示的 HTML 页面。

在[本教程的第 6 部分](http://www.iamnancy.top/post/172/)中，我们将介绍如何使用 ViewSets 和 Routers 来减少构建 API 所需的代码量。
