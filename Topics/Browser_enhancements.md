# 浏览器增强 (Browser enhancements)
“重载 POST 有两个无争议的用途。第一种是为不支持 PUT 或 DELETE 的 Web 浏览器等客户端模拟 HTTP 的统一接口” —— [RESTful Web Services](https://www.amazon.com/RESTful-Web-Services-Leonard-Richardson/dp/0596529260), Leonard Richardson & Sam Ruby.

为了让可浏览的 API 发挥作用，REST framework 需要提供一些浏览器增强功能。

从 3.3.0 版本开始，这些都是通过 javascript 启用的，使用 [ajax-form](https://github.com/encode/ajax-form) 库。

## 基于浏览器的 PUT、DELETE 等... (Browser based PUT, DELETE, etc...)
[AJAX 表单库](https://github.com/encode/ajax-form)支持基于浏览器的 HTML 表单上的 `PUT`、`DELETE` 和其他方法。

包含库后，在表单上使用 `data-method` 属性，如下所示：
```python
<form action="/" data-method="PUT">
    <input name='foo'/>
    ...
</form>
```

请注意，在 3.3.0 之前，这种支持是基于服务器端而不是基于 JavaScript 的。由于在请求解析中引入的微妙问题，不再支持方法重载样式 (在 Ruby on Rails 中使用)。

## 基于浏览器的非表单内容提交 (Browser based submission of non-form content)
[AJAX 表单库](https://github.com/encode/ajax-form)使用具有 `data-override='content-type'` 和 `data-override='content'` 属性的表单字段支持基于浏览器的内容类型 (例如 JSON) 提交。

举个栗子：
```python
    <form action="/">
        <input data-override='content-type' value='application/json' type='hidden'/>
        <textarea data-override='content'>{}</textarea>
        <input type="submit"/>
    </form>
```

请注意，在 3.3.0 之前，这种支持是基于服务器端而不是基于 JavaScript 的。

## 基于 URL 的格式后缀 (URL based format suffixes)
REST framework 可以采用 `?format=json` 样式的 URL 参数，这对于确定应从视图返回哪种内容类型是一个有用的快捷方式。

使用 `URL_FORMAT_OVERRIDE` 设置控制此行为。

## 基于 HTTP 标头的方法覆盖 (HTTP header based method overriding)
在版本 3.3.0 之前，支持半扩展头 `X-HTTP-Method-Override` 来覆盖请求方法。此行为不再是核心行为，但可以根据需要使用中间件添加。

举个栗子：
```python
METHOD_OVERRIDE_HEADER = 'HTTP_X_HTTP_METHOD_OVERRIDE'

class MethodOverrideMiddleware(object):
    def process_view(self, request, callback, callback_args, callback_kwargs):
        if request.method != 'POST':
            return
        if METHOD_OVERRIDE_HEADER not in request.META:
            return
        request.method = request.META[METHOD_OVERRIDE_HEADER]
```

## 基于 URL 的接受标头 (URL based accept headers)
直到 3.3.0 版本，REST framework 包含了对 `?accept=application/json` 样式 URL 参数的内置支持，这将允许重写 `Accept` 标头。

自引入内容协商 API 以来，此行为不再包含在核心中，但如果需要，可以使用自定义内容协商类添加。

例如：
```python
class AcceptQueryParamOverride()
    def get_accept_list(self, request):
       header = request.META.get('HTTP_ACCEPT', '*/*')
       header = request.query_params.get('_accept', header)
       return [token.strip() for token in header.split(',')]
```

## HTML5 不支持 PUT 和 DELETE 表单吗? (Doesn't HTML5 support PUT and DELETE forms?)
不。它曾一度用于支持 `PUT` 和 `DELETE` 表单，但后来从[规范中删除](https://www.w3.org/TR/html5-diff/#changes-2010-06-24)了。目前[仍在讨论](http://amundsen.com/examples/put-delete-forms/)添加对 `PUT` 和 `DELETE` 的支持，以及如何支持除表单编码数据之外的内容类型。
