# 格式后缀 (Format suffixes)
第 6.2.1 节并未说明应始终使用内容协商。—— Roy Fielding, [REST 讨论邮件列表](http://tech.groups.yahoo.com/group/rest-discuss/message/5857)

Web API 的常见模式是在 URL 上使用文件扩展名来为给定的媒体类型提供端点。例如，'http://example.com/api/users.json' 用于提供 JSON 表示。

在 URLconf 中为您的 API 添加格式后缀模式到每个条目是容易出错和非 DRY 的，因此 REST framework 提供了将这些模式添加到 URLconf 的快捷方式。

## format_suffix_patterns
**签名**：format_suffix_patterns(urlpatterns, suffix_required=False, allowed=None)

返回一个 URL 模式列表，其中包括附加到提供的每个 URL 模式的格式后缀模式。

参数：

- **urlpatterns**：必需。URL 模式列表。
- **suffix_required**：可选。布尔值，指示 URL 中的后缀是可选的还是强制的。默认为 `False`，意味着默认情况下后缀是可选的。
- **allowed**：可选。有效格式后缀的列表或元组。如果没有提供，将使用通配符格式后缀模式。

举个栗子：
```python
from rest_framework.urlpatterns import format_suffix_patterns
from blog import views

urlpatterns = [
    url(r'^/$', views.apt_root),
    url(r'^comments/$', views.comment_list),
    url(r'^comments/(?P<pk>[0-9]+)/$', views.comment_detail)
]

urlpatterns = format_suffix_patterns(urlpatterns, allowed=['json', 'html'])
```

在使用 `format_suffix_patterns` 时，您必须确保将 `'format'` 关键字参数添加到相应的视图。例如：
```python
@api_view(('GET', 'POST'))
def comment_list(request, format=None):
    # do stuff...
```

或者使用基于类的视图：
```python
class CommentList(APIView):
    def get(self, request, format=None):
        # do stuff...

    def post(self, request, format=None):
        # do stuff...
```

可以使用 `FORMAT_SUFFIX_KWARG` 设置来修改使用的 kwarg 的名称。

另请注意，`format_suffix_patterns` 不支持陷入到 `include` URL 模式。

### 使用 `i18n_patterns` (Using with `i18n_patterns`)
如果使用 Django 提供的 `i18n_patterns` 函数以及 `format_suffix_patterns`，则应确保将 `i18n_patterns` 函数应用到最终或最外层函数。例如：
```python
url patterns = [
    …
]

urlpatterns = i18n_patterns(
    format_suffix_patterns(urlpatterns, allowed=['json', 'html'])
)
```

***

## 查询参数格式 (Query parameter formats)
格式后缀的替代方法是在查询参数中包含所请求的格式。REST framework 默认提供此选项，并且它在可浏览的 API 中用于在不同的可用表示之间切换。

要使用其短格式的选择表示，请使用 `format` 查询参数。例如： `http://example.com/organizations/?format=csv`。

此查询参数的名称可以使用 `URL_FORMAT_OVERRIDE` 设置进行修改。设置该值为 `None` 以禁用此行为。

***

## 接受标头和格式后缀 (Accept headers vs. format suffixes)
在一些 Web 社区中似乎有这样一种观点，即文件名扩展不是 RESTful 模式，应始终使用 `HTTP Accept` 标头。

这实际上是一种误解。例如，引用 Roy Fielding 的以下引文，讨论查询参数媒体类型指示符的相对优点。文件扩展媒体类型指示符：

“这就是为什么我总是喜欢扩展。这两种选择都与 REST 无关。” - Roy Fielding，[REST 讨论邮件列表](http://tech.groups.yahoo.com/group/rest-discuss/message/14844)

引用中没有提到 Accept 标头，但是它清楚地表明格式后缀应该被认为是可接受的模式。
