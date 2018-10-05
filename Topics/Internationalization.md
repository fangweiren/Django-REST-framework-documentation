# 国际化 (Internationalization)
支持国际化不是可选的。它必须是核心功能。—— [Jannis Leidel，2015 年在 Django Under the Hood 演讲](https://youtu.be/Wa0VfS2q94Y)。

REST framework 附带可翻译的错误消息。您可以启用 [Django 的标准翻译机制](https://docs.djangoproject.com/en/1.7/topics/i18n/translation)以您的语言显示这些内容。

这样做可以让您：

- 使用标准 `LANGUAGE_CODE` Django 设置，选择英语以外的语言作为默认语言。
- 允许客户端使用 Django 中包含的 `LocaleMiddleware` 自己选择语言。API 客户端的典型用法是包含 `Accept-Language` 请求标头。

## 启用国际化 API (Enabling internationalized APIs)
您可以使用标准 Django `LANGUAGE_CODE` 设置更改默认语言：
```python
LANGUAGE_CODE = "es-es"
```

您可以通过将 `LocalMiddleware` 添加到 `MIDDLEWARE_CLASSES` 设置来打开每个请求的语言请求：
```python
MIDDLEWARE_CLASSES = [
    ...
    'django.middleware.locale.LocaleMiddleware'
]
```

当启用每个请求的国际化时，客户端请求将尽可能尊重 `Accept-Language` 标头。例如，让我们请求不受支持的媒体类型：

**Request**
```python
GET /api/users HTTP/1.1
Accept: application/xml
Accept-Language: es-es
Host: example.org
```

**Response**
```python
HTTP/1.0 406 NOT ACCEPTABLE

{"detail": "No se ha podido satisfacer la solicitud de cabecera de Accept."}
```

REST framework 包括这些内置的翻译，既适用于标准异常情况，也适用于序列化器验证错误。

请注意，翻译仅适用于错误字符串本身。错误消息的格式和字段名称的键将保持不变。示例 `400 Bad Request` 响应正文可能如下所示：
```python
{"detail": {"username": ["Esse campo deve ser unico."]}}
```

如果您想对响应的部分 (例如 `detail` 和 `non_field_errors`) 使用不同的字符串，那么可以使用自定义异常处理程序修改此行为。

#### 指定支持的语言集 (Specifying the set of supported languages.)
默认情况下，将支持所有可用语言。

如果您只想支持可用语言的子集，请使用 Django 的标准 `LANGUAGES` 设置：
```python
LANGUAGES = [
    ('de', _('German')),
    ('en', _('English')),
]
```

## 添加新的翻译 (Adding new translations)
REST framework 翻译使用 [Transifex](https://www.transifex.com/projects/p/django-rest-framework/) 在线管理。您可以使用 Transifex 服务添加新的翻译语言。然后，维护团队将确保这些翻译字符串包含在 REST framework 包中。

有时您可能需要在本地向项目添加翻译字符串。你可能需要这样做，如果：

- 您希望使用尚未在 Transifex 上翻译的语言使用 REST Framework。
- 您的项目包含自定义错误消息，这些消息不是 REST framework 的默认翻译字符串的一部分。

#### 在本地翻译新语言 (Translating a new language locally)
该指南假设您已经熟悉如何翻译 Django 应用程序。如果你不是，那么首先阅读 [Django 的翻译文档](https://docs.djangoproject.com/en/1.7/topics/i18n/translation)。

如果您要翻译新语言，则需要翻译现有的 REST framework 错误消息：

1. 创建一个要存储国际化资源的新文件夹。将此路径添加到 `LOCALE_PATHS` 设置。
2. 现在为要翻译的语言创建一个子文件夹。文件夹应该使用地区名称符号命名。例如：`de`，`pt_BR`，`es_AR`。
3. 现在将[基本翻译文件](https://raw.githubusercontent.com/encode/django-rest-framework/master/rest_framework/locale/en_US/LC_MESSAGES/django.po)从 REST framework 源代码复制到您的翻译文件夹中。
4. 编辑刚刚复制的 `django.po` 文件，翻译所有错误消息。
5. 运行 `manage.py compilemessages -l pt_BR` 以使 Django 可以使用的翻译。您应该看到像 `processing file django.po in <...>/locale/pt_BR/LC_MESSAGES` 的消息。
6. 重新启动开发服务器以查看更改是否生效。

如果您只翻译项目代码库中存在的自定义错误消息，则无需将 REST framework 源 `django.po` 文件复制到 `LOCALE_PATHS` 文件夹中，而只需运行 Django 的标准 `makemessages` 进程即可。

## 如何确定语言 (How the language is determined)
如果您要允许每个请求的语言首选项，则需要在 `MIDDLEWARE_CLASSES` 设置中包括 `django.middleware.locale.LocaleMiddleware`。

您可以在 [Django 文档](https://docs.djangoproject.com/en/1.7/topics/i18n/translation/#how-django-discovers-language-preference)中找到有关如何确定语言首选项的更多信息。作为参考，该方法是：

1. 首先，它在请求的 URL 中查找语言前缀。
2. 如果失败，它会在当前用户的会话中查找 `LANGUAGE_SESSION_KEY` 键。
3. 如果失败，它会查找 cookie。
4. 如果失败，它会查看 `Accept-Language` HTTP 标头。
5. 如果失败，它使用全局 `LANGUAGE_CODE` 设置。

对于 API 客户端，其中最合适的通常是使用 `Accept-Language` 标头；除非使用会话身份验证，否则会话和 cookie 将不可用，通常更好的做法是优先使用 API 客户端的 `Accept-Language` 标头，而不是使用语言 URL 前缀。
