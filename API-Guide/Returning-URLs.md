# 返回 URL (Returning URLs)
区别 REST 架构风格与其他基于网络风格的中心特征是它强调组件之间的统一接口。—— Roy Fielding，[架构风格与基于网络的软件体系结构设计](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_5)

通常，从 Web API 返回绝对 URI (例如 `http://example.com/foobar`) 可能是比返回相对 URI (`/foobar`) 更好。

这样做的好处是：

- 它更明确。
- 它为 API 客户端减少了工作量。
- 当在诸如 JSON 的表示中没有本地 URI 类型时，字符串的含义是没有歧义的。
- 它使得使用超链接标记 HTML 表示等事情变得很容易。

REST framework 提供了两个实用函数，使从 Web API 返回绝对 URI 更加简单。

您无需使用它们，但如果您这样做，那么自描述 API 将能够自动为您超链接其输出，这使得浏览 API 更加容易。

## reverse
**签名**：`reverse(viewname, *args, **kwargs)`

具有与 `django.urls.reverse` 相同的行为，除了它返回一个完全限定的 URL，使用请求来确定主机和端口。

你应该将**请求作为关键字参数**包含在该函数中，例如：
```python
from rest_framework.reverse import reverse
from rest_framework.views import APIView
from django.utils.timezone import now

class APIRootView(APIView):
    def get(self, request):
        year = now().year
        data = {
            ...
            'year-summary-url': reverse('year-summary', args=[year], request=request)
        }
        return Response(data)
```

## reverse_lazy
**签名**：`reverse_lazy(viewname, *args, **kwargs)`

具有与 `django.urls.reverse_lazy` 相同的行为，除了它返回一个完全限定的 URL，使用请求来确定主机和端口。

与 `reverse` 函数一样，你应该将**请求作为关键字参数**包含在函数中，例如：
```python
api_root = reverse_lazy('api-root', request=request)
```
