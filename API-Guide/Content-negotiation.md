# 内容协商 (Content negotiation)
HTTP 具有用于 “内容协商” 的若干机制的规定—当有多个可用表示时，为给定响应选择最佳表示的过程。—— [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec12.html), Fielding et al.

内容协商是基于客户端或服务器首选项选择多个可能表示之一以返回到客户端的过程。

## 确定接受的渲染器 (Determining the accepted renderer)
REST framework 根据可用的渲染器，每个渲染器的优先级以及客户端的 `Accept:` 标头，使用简单的内容协商样式来确定应将哪些媒体类型返回给客户端。所使用的样式部分由客户端驱动，部分由服务器驱动。

1. 更具体的媒体类型优先于较不特定的媒体类型。
2. 如果多个媒体类型具有相同的特异性，则优先根据为给定视图配置的渲染器排序。

例如，给定以下 `Accept` 标头：
```python
application/json; indent=4, application/json, application/yaml, text/html, */*
```

每种给定媒体类型的优先级将是：

- `application/json; indent=4`
- `application/json`, `application/yaml` 和 `text/html`
- `*/*`

如果请求的视图仅配置了 `YAML` 和 `HTML` 的渲染器，则 REST framework 将选择`renderer_classes` 列表或 `DEFAULT_RENDERER_CLASSES` 设置中首先列出的渲染器。

有关 `HTTP Accept` 标头的更多信息，请参阅 [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)

***
**注意**：确定首选项时，REST framework 不会考虑 “q” 值。使用 “q” 值会对缓存产生负面影响，并且在作者看来，这是一种不必要且过于复杂的内容协商方法。

这是一种有效的方法，因为 HTTP 规范故意未指定服务器应如何根据基于客户端的首选项对基于服务器的首选项进行加权。
***

# 自定义内容协商 (Custom content negotiation)
您不太可能希望为 REST framework 提供自定义内容协商方案，但如果需要，您可以这样做。要实现自定义内容协商方案，请覆盖 `BaseContentNegotiation`。

REST framework 的内容协商类处理选择适当的请求解析器和适当的响应渲染器，因此您应实现 `.select_parser(request, parsers)` 和 `.select_renderer(request, renderers, format_suffix)` 方法。

`select_parser()` 方法应从可用解析器列表中返回一个解析器实例，如果没有解析器可以处理传入请求，则返回 `None`。

`select_renderer()` 方法应返回 (渲染器实例，媒体类型) 的二元组，或引发 `NotAcceptable` 异常。

## 举个栗子
以下是自定义内容协商类，它在选择适当的解析器或渲染器时会忽略客户端请求。
```python
from rest_framework.negotiation import BaseContentNegotiation

class IgnoreClientContentNegotiation(BaseContentNegotiation):
    def select_parser(self, request, parsers):
        """
        选择 `.parser_classes` 列表中的第一个解析器。
        """
        return parsers[0]

    def select_renderer(self, request, renderers, format_suffix):
        """
        选择 `.renderer_classes` 列表中的第一个渲染器。
        """
        return (renderers[0], renderers[0].media_type)
```

## 设置内容协商 (Setting the content negotiation)
使用 `DEFAULT_CONTENT_NEGOTIATION_CLASS` 设置可以全局设置默认内容协商类。例如，以下设置将使用我们的示例 `IgnoreClientContentNegotiation` 类。
```python
REST_FRAMEWORK = {
    'DEFAULT_CONTENT_NEGOTIATION_CLASS': 'myapp.negotiation.IgnoreClientContentNegotiation',
}
```

您还可以使用 `APIView` 基于类的视图设置用于单个视图或视图集的内容协商。
```python
from myapp.negotiation import IgnoreClientContentNegotiation
from rest_framework.response import Response
from rest_framework.views import APIView

class NoNegotiationView(APIView):
    """
    An example view that does not perform content negotiation.
    """
    content_negotiation_class = IgnoreClientContentNegotiation

    def get(self, request, format=None):
        return Response({
            'accepted media type': request.accepted_renderer.media_type
        })
```
