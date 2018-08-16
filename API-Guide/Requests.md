# Requests
如果你正在做基于 REST 的 Web 服务的东西...你应该忽略 request.POST。—— Malcom Tredinnick，Django 开发人员小组

REST framework 的 `Request` 类扩展了标准的 `HttpRequest`，增加了对 REST framework 灵活的请求解析和请求认证的支持。
***

# 请求解析
REST framework 的 Request 对象提供了灵活的请求解析，允许您使用 JSON 数据或其他媒体类型像通常处理表单数据相同的方式处理请求。

## .data
`request.data` 返回请求主体的解析内容。这与标准的 `request.POST` 和 `request.FILES` 属性类似，除了：

- 它包括所有解析的内容，包括文件和非文件输入。
- 它支持解析 `POST` 以外的 HTTP 方法的内容，这意味着您可以访问 `PUT` 和 `PATCH` 请求的内容。
- 它支持 REST framework 的灵活请求解析，而不仅仅是支持表单数据。例如，您可以像处理传入表单数据一样处理传入的 JSON 数据。

有关更多详细信息，请参阅[解析器文档](http://www.django-rest-framework.org/api-guide/parsers/)。

## .query_params
`request.query_params` 是 `request.GET` 更准确的命名同义词。

为了使代码内部更清晰，我们推荐使用 `request.query_params` 而不是 Django 的标准`request.GET`。这样做有助于保持您的代码库更加正确和明显 —— 任何 HTTP 方法类型都可能包含查询参数，而不仅仅是 `GET` 请求。

## .parsers
`APIView` 类或 `@api_view` 装饰器将根据视图上设置的 `parser_classes` 或根据 `DEFAULT_PARSER_CLASSES` 设置确保将此属性自动设置为 `Parser` 实例列表。

您通常不需要访问此属性。

***
注意：如果客户端发送格式错误的内容，则访问 `request.data` 可能会引发 `ParseError`。默认情况下，REST framework 的 `APIView` 类或 `@api_view` 装饰器将捕获错误并返回 `400 Bad Request` 响应。

如果客户端发送的请求的内容类型无法解析，则会引发 `UnsupportedMediaType` 异常，默认情况下会被捕获并返回 `415 Unsupported Media Type` 响应。
***

# 内容协商 (Content negotiation)
请求公开了一些属性，这些属性允许您确定内容协商阶段的结果。这允许您实现一些行为，例如为不同的媒体类型选择不同的序列化方案。

## .accepted_renderer
渲染器实例是由内容协商阶段选择的。

## .accepted_media_type
内容协商阶段被接受的字符串表示的媒体类型。
***

# 认证 (Authentication)
REST framework 提供灵活的每个请求认证，使您能够：

- 为您的 API 的不同部分使用不同的身份验证策略。
- 支持使用多个身份验证策略。
- 提供与传入请求关联的用户和令牌信息。

## .user
`request.user` 通常会返回 `django.contrib.auth.models.User` 的一个实例，但其行为取决于正在使用的身份验证策略。

如果请求未经身份验证，则 `request.user` 的默认值是 `django.contrib.auth.models.AnonymousUser` 的实例。

有关详细信息，请参阅[身份验证文档](http://www.django-rest-framework.org/api-guide/authentication/)。

## .auth
`request.auth` 返回任何附加的认证上下文。`request.auth` 的确切行为取决于正在使用的身份验证策略，但它通常可能是请求通过身份验证的令牌实例。

如果请求未经身份验证，或者没有附加上下文，则 `request.auth` 的默认值为 `None`。

有关详细信息，请参阅[身份验证文档](http://www.django-rest-framework.org/api-guide/authentication/)。

## .authenticators
`APIView` 类或 `@api_view` 装饰器将根据视图上设置的 `authentication_classes` 或根据 `DEFAULT_AUTHENTICATORS` 设置确保将此属性自动设置为 `Authentication` 实例列表。

您通常不需要访问此属性。

***
注意：调用 `.user` 或 `.auth` 属性时可能会引发 `WrappedAttributeError` 异常。这些错误源自认证器 (authenticator) 作为标准 `AttributeError`，但是为了防止它们被外部属性访问修改，有必要重新设置为不同的异常类型。Python 将无法识别来自认证器 (authenticator) 的 `AttributeError`，并会立即假定请求对象没有 `.user` 或 `.auth` 属性。认证器 (authenticator) 需要修复。
***

# 浏览器增强
REST framework 支持一些浏览器增强功能，例如基于浏览器的 `PUT`，`PATCH` 和 `DELETE` 表单。

## .method
`request.method` 返回大写字符串表示的请求 HTTP 方法。

透明地支持基于浏览器的 `PUT`，`PATCH` 和 `DELETE` 表单。

有关更多信息，请参阅[浏览器增强功能文档](http://www.django-rest-framework.org/topics/browser-enhancements/)。

## .content_type
`request.content_type` 返回表示 HTTP 请求正文的媒体类型的字符串对象，如果没有提供媒体类型，则返回空字符串。

您通常不需要直接访问请求的内容类型，因为您通常会依赖 REST framework 的默认请求解析行为。

如果确实需要访问请求的内容类型，则应该优先使用 `.content_type` 属性，而不是使用 `request.META.get('HTTP_CONTENT_TYPE')`，因为它为基于浏览器的非表单内容提供了透明支持。

有关更多信息，请参阅[浏览器增强功能文档](http://www.django-rest-framework.org/topics/browser-enhancements/)。

## .stream
`request.stream` 返回一个代表请求主体内容的流。

您通常不需要直接访问请求的内容，因为您通常会依赖 REST framework 的默认请求解析行为。
***

# 标准的 HttpRequest 属性
由于 REST framework 的 `Request` 扩展了 Django 的 `HttpRequest`，所有其他标准属性和方法也可用。例如 `request.META` 和 `request.session` 字典可以正常使用。

请注意，由于实现原因，`Request` 类不会从 `HttpRequest` 类继承，而是使用组合扩展类。
