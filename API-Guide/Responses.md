# 响应 (Responses)
与基本的 HttpResponse 对象不同，TemplateResponse 对象保留了视图提供的用于计算响应的上下文的详细信息。在响应过程中，直到需要时才会计算最终的响应输出。—— Django 文档

REST framework 通过提供一个 `Response` 类来支持 HTTP 内容协商，该类允许您根据客户端请求返回可渲染为多种内容类型的内容。

`Response` 类的子类是 Django 的 `SimpleTemplateResponse`。响应对象使用数据进行初始化，数据应由本地 Python 基元组成。REST framework 然后使用标准的 HTTP 内容协商来确定它如何渲染最终响应内容。

您不需要使用 `Response` 类，如果需要，也可以从视图中返回常规的 `HttpResponse` 或 `StreamingHttpResponse` 对象。使用 `Response` 类只是为返回内容协商的 Web API 响应提供更好的接口，这些响应可以渲染为多种格式。

除非出于某种原因需要大量定制 REST framework，否则应始终对返回 `Response` 对象的视图使用 `API​​View` 类或 `@api_view` 函数。这样做可以确保视图执行内容协商，并在视图返回之前为响应选择适当的渲染器。

# 创建 responses
## Response()
语法：`Response(data, status=None, template_name=None, headers=None, content_type=None)`

与常规 `HttpResponse` 对象不同，您不会使用渲染的内容实例化 `Response` 对象。相反，您传递的是未渲染的数据，可能由任何 Python 基元组成。

由于 `Response` 类使用的渲染器不能处理复杂的数据类型，例如 Django 模型实例，所以需要在创建 `Response` 对象之前将数据序列化为基本数据类型。

您可以使用 REST framework 的 `Serializer` 类来执行数据序列化，或者使用您自己的自定义序列化。

参数：

- `data` ：响应的序列化数据。
- `status` ：响应的状态代码。默认为200。另请参阅[状态代码](http://www.django-rest-framework.org/api-guide/status-codes/)。
- `template_name` ：选择 `HTMLRenderer` 时使用的模板名称。
- `headers` ：响应中使用的 HTTP headers 的字典。
- `content_type` ：响应的内容类型。通常情况下，渲染器会根据内容协商的结果自动设置，但有些情况下需要明确指定内容类型。

***

# Attributes
## .data
未渲染的、序列化的响应数据。

## .status_code
HTTP 响应的数字状态码。

## .content
响应的渲染内容。在访问 `.content` 之前，必须先调用 `.render()` 方法。

## .template_name
`template_name` (如果提供)。只有当 `HTMLRenderer` 或其他自定义模板渲染器是响应的渲染器时才需要。

## .accepted_renderer
用于渲染响应的渲染器实例。

从视图返回响应之前由 `APIView` 或 `@api_view` 自动设置。

## .accepted_media_type
内容协商阶段选择的媒体类型。

从视图返回响应之前由 `APIView` 或 `@api_view` 自动设置。

## .renderer_context
将传递给渲染器的 `.render()` 方法的附加的上下文信息的字典。

从视图返回响应之前由 `APIView` 或 `@api_view` 自动设置。

***

# 标准 HttpResponse 属性
`Response` 类扩展了 `SimpleTemplateResponse`，并且响应中也提供了所有常用的属性和方法。例如，您可以用标准方式在响应中设置 headers：
```python
response = Response()
response['Cache-Control'] = 'no-cache'
```

## .render()
语法：`.render()`

与其他任何 `TemplateResponse` 一样，调用此方法将响应的序列化数据渲染为最终响应内容。调用 `.render()` 时，响应内容将设置为在 `accepted_renderer` 实例上调用 `.render(data，accepted_media_type，renderer_context)` 方法的结果。

您通常不需要自己调用 `.render()`，因为它是由 Django 的标准响应循环处理的。
