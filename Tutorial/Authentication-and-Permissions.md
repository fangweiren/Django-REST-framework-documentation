# 教程 4：认证和权限
目前我们的 API 对谁可以编辑或删除 snippet 代码没有任何限制。我们希望有一些更高级的行为来确保：

- snippets 代码始终与创建者相关联。
- 只有经过身份验证的用户可以创建 snippets。
- 只有 snippets 的创建者可以更新或删除它。
- 未经身份验证的请求应具有完全只读访问权限。

## 将信息添加到我们的模型
我们将对 `Snippet` 模型类进行一些更改。首先，我们添加几个字段。其中一个字段将用于表示创建 snippet 代码的用户。另一个字段将用于存储高亮显示的 HTML 内容表示的代码。

将以下两个字段添加到 `models.py` 文件中的 `Snippet` 模型中。
```python
class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)  # 是否显示行号
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)
    owner = models.ForeignKey('auth.User', related_name='snippets', on_delete=models.CASCADE)
    highlighted = models.TextField()
```
我们还需要确保在保存模型时，使用 `pygments` 代码高亮库填充 highlighted 字段。

我们需要导入额外的模块：
```python
from pygments.lexers import get_lexer_by_name
from pygments.formatters.html import HtmlFormatter
from pygments import highlight
```
现在我们可以在我们的模型类中添加一个 `.save()` 方法：
```python
def save(self, *args, **kwargs):
    """
    使用 pygments 库创建一个高亮显示的 HTML 表示的 snippet 代码。
    """
    lexer = get_lexer_by_name(self.language)
    linenos = 'table' if self.linenos else False
    options = {'title': self.title} if self.title else {}
    formatter = HtmlFormatter(style=self.style, linenos=linenos, full=True, **options)
    self.highlighted = highlight(self.code, lexer, formatter)
    super(Snippet, self).save(*args, **kwargs)
```
完成这些工作后，我们需要更新我们的数据库表。通常这种情况我们会创建一个数据库迁移来实现这一点，但作为教程的目的，让我们删除数据库并重新开始。
```python
rm -f db.sqlite3
rm -r snippets/migrations
python manage.py makemigrations snippets
python manage.py migrate
```
您可能还想创建几个不同的用户，以用于测试 API。最快捷的方法是使用 `createsuperuser` 命令。
```python
python manage.py createsuperuser
```

## 为我们的用户模型添加端点
现在我们有一些用户可以使用，我们最好将这些用户的表示添加到我们的 API 中。创建一个新的序列化器很简单，在 `serializers.py` 文件中添加：
```python
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())

    class Meta:
        model = User
        fields = ('id', 'username', 'snippets')
```
由于 `'snippets'` 在用户模型上是反向关系，当在使用 `ModelSerializer` 类时，它将不会被默认包含，所以我们需要为它添加一个显式字段。

我们还会在 `views.py` 中添加几个视图。我们想要使用用户表示的只读视图，所以我们将使用 `ListAPIView` 和 `RetrieveAPIView` 通用的基于类的视图。
```python
from django.contrib.auth.models import User


class UserList(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer


class UserDetail(generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```
确保也导入了 `UserSerializer` 类
```python
from snippets.serializers import UserSerializer
```
最后，我们需要通过在 URL conf 中引用这些视图来将这些视图添加到 API 中。将以下内容添加到 `urls.py` 中的模式中。
```python
url(r'^users/$', views.UserList.as_view()),
url(r'^users/(?P<pk>[0-9]+)/$', views.UserDetail.as_view()),
```

## 将 Snippets 和用户关联
现在，如果我们创建了一个 snippet 代码，那么无法将创建 snippet 的用户与 snippet 实例关联起来。用户不是作为序列化表示的一部分发送的，而是作为传入请求的属性。

我们处理方式是在我们的 snippet 视图上覆盖 `.perform_create()` 方法，这允许我们修改实例保存的管理方式，并处理传入请求或请求的 URL 中隐含的任何信息。

在 `SnippetList` 视图类中(`snippets/views.py`)，添加以下方法：
```python
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```
现在我们的序列化器的 `create()` 方法将被传递一个附加的 `'owner'` 字段以及来自请求的验证数据。

## 更新我们的序列化器
现在 snippets 与创建它们的用户相关联了，让我们更新我们的 `SnippetSerializer` 以体现这一点。将以下字段添加到 `serializers.py` 中的序列化器的定义中：
```python
owner = serializers.ReadOnlyField(source='owner.username')
```
注意：确保您还将 `'owner'` 添加到内部 `Meta` 类的字段列表中。

这个字段正在做一件很有趣的事情。`source` 参数控制哪个属性用于填充字段，并且可以指向序列化实例上的任何属性。它也可以采用如上所示的点符号(.)，在这种情况下，它将以与 Django  模板语言相似的方式遍历给定的属性。

我们添加的字段是无类型的 `ReadOnlyField` 类，与其他类型的字段相反，如 `CharField`，`BooleanField` 等...无类型的 `ReadOnlyField` 始终是只读的，只能用于序列化表示，但不能用于在反序列化时更新模型实例。我们在这里也可以使用 `CharField(read_only=True)`。

## 添加视图所需的权限
现在，snippets 和用户相关联，我们希望确保只有经过身份验证的用户才能创建，更新和删除 snippets。

REST framework 包含许多权限类，我们可以用来限制谁可以访问给定的视图。在这种情况下，我们需要的是 `IsAuthenticatedOrReadOnly` 类，它将确保经过身份验证的请求获得读写访问权限，未经身份验证的请求将获得只读访问权限。

首先要在视图模块中导入以下内容
```python
from rest_framework import permissions
```
然后，将以下属性添加到 `SnippetList` 和 `SnippetDetail` 视图类中。
```python
permission_classes = (permissions.IsAuthenticatedOrReadOnly,)
```

## 添加登录到可浏览 API
如果您现在打开浏览器并导航到可浏览的 API，那么您将发现无法再创建新的 snippets。为了做到这一点，我们需要能够以用户身份登录。

我们可以通过编辑项目级 `urls.py` 文件(`tutorial/urls.py`)中的 URLconf 来添加可浏览 API 的登录视图。

在文件顶部添加以下导入：
```python
from django.conf.urls import include
```
并且，在文件末尾添加一个模式以包括可浏览的 API 的登录和注销视图。
```python
urlpatterns = [
    url(r'^api-auth/', include('rest_framework.urls')),
]
```
模式的 `r'^api-auth/'` 部分实际上可以是你要使用的任何 URL。

现在，如果您再次打开浏览器并刷新页面，则会在页面右上方看到一个“登录”链接。如果您用前面创建的用户登录，就可以再次创建 snippets。

一旦创建了一些 snippets 后，导航到 `'/users/'` 端点，并注意到在每个用户的 'snippets' 字段中包含与每个用户相关联的 snippet id 的列表。

## 对象级权限
实际上，我们希望所有人都可以看到所有 snippets，但也要确保只有创建 snippet 的用户才能更新或删除它。

为此，我们需要创建一个自定义权限。

在 snippets 应用中，创建一个新文件 `permissions.py`
```python
from rest_framework import permissions


class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    自定义权限只允许对象的所有者编辑它。
    """
    def has_object_permission(self, request, view, obj):
        # 读取权限被允许用于任何请求，
        # 所以我们始终允许 GET，HEAD 或 OPTIONS 请求。
        if request.method in permissions.SAFE_METHODS:
            return True

        # 写入权限只允许给 snippet 的所有者。
        return obj.owner == request.user
```
现在我们可以通过编辑 `SnippetDetail` 视图类中的 `permission_classes` 属性来将自定义权限添加到我们的snippet 实例端点：
```python
permission_classes = (permissions.IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly,)
```
确保要先导入 `IsOwnerOrReadOnly` 类。
```python
from snippets.permissions import IsOwnerOrReadOnly
```
现在，如果再次打开浏览器，你会发现 “DELETE” 和 “PUT” 操作只出现在以 snippet 创建者的身份登录的 snippet 实例端点上。

## 使用 API 进行身份验证
因为现在我们在 API 上有一组权限，如果我们想要编辑任何 snippet，我们需要验证我们的请求。我们还没有设置任何认证类，因此当前应用的是默认的 `SessionAuthentication` 和 `BasicAuthentication`。

当我们通过 Web 浏览器与 API 进行交互时，我们可以登录，然后浏览器会话将为请求提供所需的身份验证。

如果我们以编程方式与 API 进行交互，那么我们需要在每个请求上明确提供身份验证凭据。

如果我们尝试创建一个没有进行身份验证的 snippet，我们会得到一个错误：
```python
(venv) fangweiren@ubuntu:~/Django_REST_framework$ http POST http://127.0.0.1:8000/snippets/ code="print 123"
HTTP/1.1 403 Forbidden
Allow: GET, HEAD, OPTIONS
Content-Length: 58
Content-Type: application/json
Date: Sun, 17 Jun 2018 14:56:36 GMT
Server: WSGIServer/0.2 CPython/3.5.2
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

{
    "detail": "Authentication credentials were not provided."
}
```
我们可以通过添加我们之前创建的用户的用户名和密码来发送成功的请求。
```python
(venv) fangweiren@ubuntu:~/Django_REST_framework$ http -a djrest:aa112211 POST http://127.0.0.1:8000/snippets/ code="print 789"
HTTP/1.1 201 Created
Allow: GET, POST, HEAD, OPTIONS
Content-Length: 110
Content-Type: application/json
Date: Mon, 18 Jun 2018 14:55:44 GMT
Server: WSGIServer/0.2 CPython/3.5.2
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

{
    "code": "print 789",
    "id": 4,
    "language": "python",
    "linenos": false,
    "owner": "djrest",
    "style": "friendly",
    "title": ""
}
```

## 总结
我们现在已经在我们的 Web API 上获得了相当精细的一组权限集合，并为系统用户和他们创建的 snippet 提供了终点。

在[本教程的第 5 部分](http://www.iamnancy.top/post/171/)中，我们将介绍如何通过为高亮显示 snippet 创建 HTML 端点来将所有内容联结在一起，并通过对系统内的关系使用超链接来提高 API 的凝聚力。
