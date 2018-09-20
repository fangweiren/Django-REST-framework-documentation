# 测试 (Testing)
天有不测风云，码有祸福旦夕 (指明了测试在编程开发中的重要性)。—— [Jacob Kaplan-Moss](https://jacobian.org/writing/django-apps-with-buildout/#s-create-a-test-wrapper)

REST framework 包含一些扩展 Django 现有测试框架的辅助类，并改进对 API 请求的支持。

# APIRequestFactory
扩展 [Django 的现有的 `RequestFactory` 类](https://docs.djangoproject.com/en/stable/topics/testing/advanced/#django.test.client.RequestFactory)。

## 创建测试请求 (Creating test requests)
`APIRequestFactory` 类支持与 Django 的标准 `RequestFactory` 类几乎相同的 API。这意味着标准的 `.get()`，`.post()`，`.put()`，`.patch()`，`.delete()`，`.head()` 和 `.options()` 方法都可用。
```python
from rest_framework.test import APIRequestFactory

# 使用标准 RequestFactory API 创建表单 POST 请求
factory = APIRequestFactory()
request = factory.post('/notes/', {'title': 'new idea'})
```

#### 使用 `format` 参数 (Using the `format` argument)
创建请求主体 (如 `post`，`put` 和 `patch`) 方法包括 `format` 参数，这使得使用除 multipart 表单数据以外的内容类型生成请求变得容易。例如：
```python
# 创建 JSON POST 请求
factory = APIRequestFactory()
request = factory.post('/notes/', {'title': 'new idea'}, format='json')
```
默认情况下，可用的格式是 `'multipart'` 和 `'json'`。为了与 Django 的现有的 `RequestFactory` 兼容，默认格式是 `'multipart'`。

要支持更广泛的请求格式集，或更改默认格式，[请参阅配置部分](http://www.django-rest-framework.org/api-guide/testing/#configuration)。

#### 显式编码请求主体 (Explicitly encoding the request body)
如果您需要显式编码请求主体，则可以通过设置 `content_type` 标志来完成。例如：
```python
request = factory.post('/notes/', json.dumps({'title': 'new idea'}), content_type='application/json')
```

#### PUT 和 PATCH 与表单数据 (PUT and PATCH with form data)
Django 的 `RequestFactory` 和 REST framework 的 `APIRequestFactory` 之间值得注意的一个区别是 multipart 表单数据将被编码为除 `.post()` 以外的方法。

例如，使用 `APIRequestFactory`，您可以像这样做一个表单 PUT 请求：
```python
factory = APIRequestFactory()
request = factory.put('/notes/547/', {'title': 'remember to email dave'})
```
使用 Django 的 `RequestFactory`，您需要自己显式编码数据：
```python
from django.test.client import encode_multipart, RequestFactory

factory = RequestFactory()
data = {'title': 'remember to email dave'}
content = encode_multipart('BoUnDaRyStRiNg', data)
content_type = 'multipart/form-data; boundary=BoUnDaRyStRiNg'
request = factory.put('/notes/547/', content, content_type=content_type)
```

## 强制验证 (Forcing authentication)
当使用请求工厂直接测试视图时，能够直接验证请求通常很方便，而不必构造正确的验证凭证。

要强制验证请求，请使用 `force_authenticate()` 方法。
```python
from rest_framework.test import force_authenticate

factory = APIRequestFactory()
user = User.objects.get(username='olivia')
view = AccountDetail.as_view()

# 对视图进行身份验证请求...
request = factory.get('/accounts/django-superstars/')
force_authenticate(request, user=user)
response = view(request)
```
该方法的签名是 `force_authenticate(request, user=None, token=None)`。在进行呼叫时，可以设置用户和令牌中的一个或两个。

例如，当使用令牌强行进行身份验证时，您可能会执行以下操作：
```python
user = User.objects.get(username='olivia')
request = factory.get('/accounts/django-superstars/')
force_authenticate(request, user=user, token=user.auth_token)
```

***
**注意**：`force_authenticate` 直接将 `request.user` 设置为内存中的 `user` 实例。如果要更新已保存的 `user` 状态的多个测试中重新使用相同的 `user` 实例，则可能需要在测试之间调用 `refresh_from_db()`。
***
**注意**：使用 `APIRequestFactory` 时，返回的对象是 Django 的标准 `HttpRequest`，而不是 REST framework 的 `Request` 对象，只有在调用视图后才会生成。

这意味着直接在请求对象上设置属性可能并不总是具有您期望的效果。例如，直接设置 `.token` 将不起作用，并且直接设置 `.user` 仅在使用会话身份验证时才有效。
```python
# 如果正在使用 `SessionAuthentication`，请求将只进行身份验证。
request = factory.get('/accounts/django-superstars/')
request.user = user
response = view(request)
```

***

## 强制 CSRF 验证 (Forcing CSRF validation)
默认情况下，使用 `APIRequestFactory` 创建的请求在传递给 REST framework 视图时不会应用 CSRF 验证。如果您需要显式启用 CSRF 验证，则可以通过在实例化工厂时设置 `enforce_csrf_checks` 标志来实现。
```python
factory = APIRequestFactory(enforce_csrf_checks=True)
```

***
**注意**：值得注意的是，Django 的标准 `RequestFactory` 不需要包含此选项，因为当使用常规的 Django 时，CSRF 验证发生在中间件中，当直接测试视图时不会运行。当使用 REST framework 时，CSRF 验证发生在视图内，因此请求工厂需要禁用视图级 CSRF 检查。
***

# APIClient
扩展 [Django 现有的 `Client` 类](https://docs.djangoproject.com/en/stable/topics/testing/tools/#the-test-client)。

## 发出请求 (Making requests)
`APIClient` 类支持与 Django 的标准 `Client` 类相同的请求接口。这意味着标准的 `.get()`，`.post()`，`.put()`，`.patch()`，`.delete()`，`.head()` 和 `.options()` 方法都可用。例如：
```python
from rest_framework.test import APIClient

client = APIClient()
client.post('/notes/', {'title': 'new idea'}, format='json')
```
要支持更广泛的请求格式集，或更改默认格式，[请参阅配置部分](http://www.django-rest-framework.org/api-guide/testing/#configuration)。

## 认证 (Authenticating)
#### .login(**kwargs)
`login` 方法的功能与 Django 的常规 `Client` 类完全相同。这允许您对任何包含 `SessionAuthentication` 的视图进行身份验证。
```python
# 在登录会话的上下文中发出所有请求。
client = APIClient()
client.login(username='lauren', password='secret')
```
要退出，请照常调用注销方法。
```python
# 登出
client.logout()
```
`login` 方法适用于测试使用会话身份验证的 API，例如包含 AJAX 与 API 交互的网站。

#### .credentials(**kwargs)
`credentials` 方法可用于设置标头，然后测试客户端将包含在所有后续请求中。
```python
from rest_framework.authtoken.models import Token
from rest_framework.test import APIClient

# 在所有请求中包含适当的 `Authorization:` 标头。
token = Token.objects.get(user__username='lauren')
client = APIClient()
client.credentials(HTTP_AUTHORIZATION='Token ' + token.key)
```
请注意，第二次调用 `credentials` 会覆盖任何现有凭证。您可以通过调用不带参数的方法来取消设置任何现有凭据。
```python
# 停止包含任何凭据
client.credentials()
```
`credentials` 方法适用于测试需要验证标头的 API，例如基本身份验证，OAuth1 和 OAuth2 身份验证以及简单令牌身份验证方案。

#### .force_authenticate(user=None, token=None)
有时您可能希望完全绕过身份验证，并强制测试客户端的所有请求被自动视为已通过身份验证。

如果您正在测试 API 但不希望构建有效的身份验证凭据以发出测试请求，则这可能是一个有用的快捷方式。
```python
user = User.objects.get(username='lauren')
client = APIClient()
client.force_authenticate(user=user)
```
要取消对后续请求的身份验证，请调用 `force_authenticate` 将用户和/或令牌设置为 `None`。
```python
client.force_authenticate(user=None)
```

## CSRF 验证 (CSRF validation)
默认情况下，使用 `APIClient` 时不应用 CSRF 验证。如果您需要显式启用 CSRF 验证，则可以通过在实例化客户端时设置 `enforce_csrf_checks` 标志来实现。
```python
client = APIClient(enforce_csrf_checks=True)
```
通常，CSRF 验证将仅适用于任何会话身份验证视图。这意味着 CSRF 验证只有在客户端通过调用 `login()` 登录后才会发生。

***

# RequestsClient
REST framework 还包含一个客户端，用于使用流行的 Python 库 `requests` 与应用程序进行交互。 这可能是有用的，如果：

- 您期望主要从另一个 Python 服务与 API 进行交互，并且希望在与客户端看到的相同级别上测试该服务。
- 您希望以这样的方式编写测试，即它们也可以针对临时或实时环境运行。(请参阅下面的 “实时测试”。)

这公开了与直接使用请求会话完全相同的接口。
```python
client = RequestsClient()
response = client.get('http://testserver/users/')
assert response.status_code == 200
```
请注意，请求客户端要求您传递完全限定的 URL。

## `RequestsClient` 和使用数据库 (`RequestsClient` and working with the database)
如果您想编写仅与服务接口交互的测试，则 `RequestsClient` 类很有用。这比使用标准的 Django 测试客户端要严格一些，因为这意味着所有的交互必须通过 API。

如果您正在使用 `RequestsClient`，则需要确保测试设置和结果断言作为常规 API 调用执行，而不是直接与数据库模型交互。例如，与其检查 `Customer.objects.count() == 3`，不如列出客户端点，并确保它包含三个记录。

## 标头和身份验证 (Headers & Authentication)
[当使用标准的 `requests.Session` 实例](http://docs.python-requests.org/en/master/user/advanced/#session-objects)以相同的方式提供自定义标头和身份验证凭据。
```python
from requests.auth import HTTPBasicAuth

client.auth = HTTPBasicAuth('user', 'pass')
client.headers.update({'x-test': 'true'})
```

## CSRF
如果您正在使用 `SessionAuthentication`，则需要为 `POST`，`PUT`，`PATCH` 或 `DELETE` 请求包含 CSRF 令牌。

您可以通过遵循基于 JavaScript 的客户端使用的相同流程来执行此操作。首先发出 `GET` 请求以获取 CRSF 令牌，然后在以下请求中显示该令牌。

例如：
```python
client = RequestsClient()

# 获取 CSRF 令牌。
response = client.get('/homepage/')
assert response.status_code == 200
csrftoken = response.cookies['csrftoken']

# 与 API 交互。
response = client.post('/organisations/', json={
    'name': 'MegaCorp',
    'status': 'active'
}, headers={'X-CSRFToken': csrftoken})
assert response.status_code == 200
```

## 实时测试 (Live tests)
通过谨慎使用，`RequestsClient` 和 `CoreAPIClient` 都提供了编写测试用例的能力，这些测试用例可以在开发中运行，也可以直接针对登台服务器或生产环境运行。

使用此样式创建一些核心功能的基本测试是验证实时服务的有效方法。这样做可能需要仔细注意安装和卸载，以确保测试以不直接影响客户数据的方式运行。

***

# CoreAPIClient
CoreAPIClient 允许您使用 Python `coreapi` 客户端库与您的 API 进行交互。
```python
# Fetch the API schema
client = CoreAPIClient()
schema = client.get('http://testserver/schema/')

# Create a new organisation
params = {'name': 'MegaCorp', 'status': 'active'}
client.action(schema, ['organisations', 'create'], params)

# Ensure that the organisation exists in the listing
data = client.action(schema, ['organisations', 'list'])
assert(len(data) == 1)
assert(data == [{'name': 'MegaCorp', 'status': 'active'}])
```

## 标头和身份验证 (Headers & Authentication)
自定义标头和身份验证可以与 `CoreAPIClient` 以类似于 `RequestsClient` 的方式一起使用。
```python
from requests.auth import HTTPBasicAuth

client = CoreAPIClient()
client.session.auth = HTTPBasicAuth('user', 'pass')
client.session.headers.update({'x-test': 'true'})
```

***

# API 测试用例 (API Test cases)
REST framework 包含以下测试用例类，它们镜像现有的 Django 测试用例类，但使用 `API​​Client` 而不是 Django 的默认 `Client`。

- `APISimpleTestCase`
- `APITransactionTestCase`
- `APITestCase`
- `APILiveServerTestCase`

## 举个栗子
您可以像使用常规 Django 测试用例类一样使用任何 REST framework 的测试用例类。`self.client` 属性将是 `APIClient` 实例。
```python
from django.urls import reverse
from rest_framework import status
from rest_framework.test import APITestCase
from myproject.apps.core.models import Account

class AccountTests(APITestCase):
    def test_create_account(self):
        """
        确保我们可以创建新的帐户对象。
        """
        url = reverse('account-list')
        data = {'name': 'DabApps'}
        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Account.objects.count(), 1)
        self.assertEqual(Account.objects.get().name, 'DabApps')
```

***

# URLPatternsTestCase
REST framework 还提供了一个测试用例类，用于在每个类的基础上隔离 `urlpatterns`。请注意，这继承自 Django 的 `SimpleTestCase`，并且很可能需要与另一个测试用例类混合使用。

## 举个栗子
```python
from django.urls import include, path, reverse
from rest_framework.test import APITestCase, URLPatternsTestCase


class AccountTests(APITestCase, URLPatternsTestCase):
    urlpatterns = [
        path('api/', include('api.urls')),
    ]

    def test_create_account(self):
        """
        确保我们可以创建新的帐户对象。
        """
        url = reverse('account-list')
        response = self.client.get(url, format='json')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data), 1)
```

***

# 测试响应 (Testing responses)
## 检查响应数据 (Checking the response data)
当检查测试响应的有效性时，检查创建响应的数据通常更方便，而不是检查完全渲染的响应。

例如，检查 `response.data` 更容易：
```python
response = self.client.get('/users/4/')
self.assertEqual(response.data, {'id': 4, 'username': 'lauren'})
```

而不是检查解析 `response.content` 的结果：
```python
response = self.client.get('/users/4/')
self.assertEqual(json.loads(response.content), {'id': 4, 'username': 'lauren'})
```

## 渲染响应 (Rendering responses)
如果您使用 `API​​RequestFactory` 直接测试视图，则返回的响应将不会渲染，因为模板响应的渲染由 Django 的内部请求 - 响应循环执行。为了访问 `response.content`，您首先需要渲染响应。
```python
view = UserDetail.as_view()
request = factory.get('/users/4')
response = view(request, pk='4')
response.render()  # 没有这个，无法访问`response.content`。
self.assertEqual(response.content, '{"username": "lauren", "id": 4}')
```

***

# 配置 (Configuration)
## 设置默认格式 (Setting the default format)
可以使用 `TEST_REQUEST_DEFAULT_FORMAT` 设置键设置用于发出测试请求的默认格式。例如，要始终对测试请求默认使用 JSON 而不是标准的 multipart 表单请求，请在 `settings.py` 文件中设置以下内容：
```python
REST_FRAMEWORK = {
    ...
    'TEST_REQUEST_DEFAULT_FORMAT': 'json'
}
```

## 设置可用的格式 (Setting the available formats)
如果需要使用除 multipart 或 json 请求之外的其他东西来测试请求，可以通过设置`TEST_REQUEST_RENDERER_CLASSES` 设置来完成。

例如，要在测试请求中添加对 `format ='html'` 的支持，您可能在 `settings.py` 文件中有类似的内容。
```python
REST_FRAMEWORK = {
    ...
    'TEST_REQUEST_RENDERER_CLASSES': (
        'rest_framework.renderers.MultiPartRenderer',
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.TemplateHTMLRenderer'
    )
}
```
