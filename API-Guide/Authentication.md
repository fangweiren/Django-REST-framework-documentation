# 认证 (Authentication)
身份验证需要可插拔。—— Jacob Kaplan-Moss, ["REST worst practices"](https://jacobian.org/writing/rest-worst-practices/)

身份验证是将传入请求与一组标识凭据 (例如请求来自的用户或其签名的令牌) 相关联的机制。然后，[权限](http://www.django-rest-framework.org/api-guide/permissions/)和[限制](http://www.django-rest-framework.org/api-guide/throttling/)策略可以使用这些凭据来确定是否应允许该请求。

REST framework 提供了一些开箱即用的身份验证方案，并且还允许您实现自定义方案。

身份验证始终在视图的最开始运行，在权限和限制检查发生之前以及允许任何其他代码继续执行之前。

`request.user` 属性通常被设置为 `contrib.auth` 包的 `User` 类的实例。

`request.auth` 属性用于任何其他身份验证信息，例如，它可用于表示请求已签名的身份验证令牌。

***

**注意**：不要忘记，**身份验证本身不会允许或不允许传入的请求**，它只识别发出请求的凭证。

有关如何为 API 设置权限策略的信息，请参阅[权限文档](http://www.django-rest-framework.org/api-guide/permissions/)。

***

## 如何确定身份验证 (How authentication is determined)
身份验证方案始终定义为类列表。REST framework 将尝试对列表中的每个类进行身份验证，并使用成功进行身份验证的第一个类的返回值设置 `request.user` 和 `request.auth`。

如果没有类进行身份验证，`request.user` 将被设置为 `django.contrib.auth.models.AnonymousUser` 的实例，`request.auth` 将被设置为 `None`。

可以使用 `UNAUTHENTICATED_USER` 和 `UNAUTHENTICATED_TOKEN` 的设置修改未经身份认证的请求的 `request.user` 和 `request.auth` 的值。

## 设置身份验证方案 (Setting the authentication scheme)
可以使用 `DEFAULT_AUTHENTICATION_CLASSES` 设置全局的默认身份验证方案。比如：
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    )
}
```
您还可以使用基于 `APIView` 类的视图在每个视图或每个视图集的基础上设置身份验证方案。
```python
from rest_framework.authentication import SessionAuthentication, BasicAuthentication
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    authentication_classes = (SessionAuthentication, BasicAuthentication)
    permission_classes = (IsAuthenticated,)

    def get(self, request, format=None):
        content = {
            'user': unicode(request.user),  # `django.contrib.auth.User` instance.
            'auth': unicode(request.auth),  # None
        }
        return Response(content)
```
或者，如果您使用基于函数的视图的 `@api_view` 装饰器。
```python
@api_view(['GET'])
@authentication_classes((SessionAuthentication, BasicAuthentication))
@permission_classes((IsAuthenticated,))
def example_view(request, format=None):
    content = {
        'user': unicode(request.user),  # `django.contrib.auth.User` instance.
        'auth': unicode(request.auth),  # None
    }
    return Response(content)
```

## 未经授权和禁止响应 (Unauthorized and Forbidden responses)
当未经验证的请求被拒绝许可时，有两种不同的错误代码可能是适当的。

- [HTTP 401 未经授权](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.2)
- [HTTP 403 权限被拒绝](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.4)

HTTP 401 响应必须始终包含 `WWW-Authenticate` 标头，指示客户端如何进行身份验证。HTTP 403 响应不包括 `WWW-Authenticate` 标头。

使用的响应类型取决于身份验证方案。尽管可以使用多种认证方案，但是仅可以使用一种方案来确定响应的类型。**在确定响应类型时使用视图上设置的第一个身份验证类。**

注意，当请求可以成功进行身份验证时，但仍然被拒绝执行该请求的许可，不管认证方案如何，总是使用 `403 Permission Denied` 响应。

## Apache mod_wsgi 具体配置 (Apache mod_wsgi specific configuration)
请注意，如果[使用 mod_wsgi 部署到 Apache](https://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIPassAuthorization.html)，授权标头默认不会传递给 WSGI 应用程序，因为它假定由 Apache 处理身份验证，而不是在应用程序级别处理。

如果您正在部署到 Apache，并使用任何基于非会话的身份验证，则需要明确配置 mod_wsgi 以将所需的标头传递给应用程序。这可以通过在适当的上下文中指定 `WSGIPassAuthorization` 指令并将其设置为 `'On'` 来完成。
```python
# 这可以在服务器配置、虚拟主机、目录或 .htaccess 中执行
WSGIPassAuthorization On
```

***

# API 参考 (API Reference)
## BasicAuthentication
此身份验证方案使用 [HTTP 基本身份验证](https://tools.ietf.org/html/rfc2617)，对用户的用户名和密码进行签名。基本身份验证通常仅适用于测试。

如果成功通过身份验证，`BasicAuthentication` 提供以下凭据。

- `request.user` 将是 Django `User` 实例。
- `request.auth` 将是 `None`。

拒绝许可的未经身份验证的响应将导致带有相应的 WWW-Authenticate 标头的 `HTTP 401 Unauthorized` 响应。例如：
```python
WWW-Authenticate: Basic realm="api"
```
**注意**：如果您在产品中使用 `BasicAuthentication`，您必须确保您的 API 仅在 `https` 上可用。您还应确保您的 API 客户端始终在登录时重新请求用户名和密码，并且不会存储这些详细信息到持久存储器中。

## TokenAuthentication
此身份验证方案使用简单的基于令牌的 HTTP 身份验证方案。令牌认证适用于客户端-服务器设置，例如本地桌面和移动客户端。

要使用 `TokenAuthentication` 方案，您需要[配置身份验证类](http://www.django-rest-framework.org/api-guide/authentication/#setting-the-authentication-scheme)以包含 `TokenAuthentication`，并在 `INSTALLED_APPS` 设置中另外包含 `rest_framework.authtoken`：
```python
INSTALLED_APPS = (
    ...
    'rest_framework.authtoken'
)
```

***

**注意**：确保在更改设置后运行 `manage.py migrate`。`rest_framework.authtoken` 应用程序提供 Django 数据库迁移。

***

您还需要为用户创建令牌。
```python
from rest_framework.authtoken.models import Token

token = Token.objects.create(user=...)
print token.key
```
要使客户端进行身份验证，令牌密钥应包含在 `Authorization` HTTP 标头中。密钥应以字符串文字“Token”为前缀，空格分隔两个字符串。例如：
```python
Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b
```
**注意**：如果想要在标头中使用不同的关键字，例如 `Bearer`，只需将 `TokenAuthentication` 子类化并设置 `keyword` 类变量。

如果成功通过身份验证，`TokenAuthentication` 提供以下凭据。

- `request.user` 将是 Django `User` 实例。
- `request.auth` 将是 `rest_framework.authtoken.models.Token` 实例。

拒绝许可的未经身份验证的响应将导致带有相应的 WWW-Authenticate 标头的 `HTTP 401 Unauthorized` 响应。例如：
```python
WWW-Authenticate: Token
```
`curl` 命令行工具可用于测试令牌认证 API。例如：
```python
curl -X GET http://127.0.0.1:8000/api/example/ -H 'Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b'
```

***

**注意**：如果您在产品中使用 `TokenAuthentication`，您必须确保您的 API 仅在 `https` 上可用。

***

#### 生成令牌 (Generating Tokens)
##### 通过使用信号 (By using signals)
如果您想要每个用户都拥有一个自动生成的令牌，您只需捕获用户的 `post_save` 信号即可。
```python
from django.conf import settings
from django.db.models.signals import post_save
from django.dispatch import receiver
from rest_framework.authtoken.models import Token

@receiver(post_save, sender=settings.AUTH_USER_MODEL)
def create_auth_token(sender, instance=None, created=False, **kwargs):
    if created:
        Token.objects.create(user=instance)
```
请注意，您需要确保将此代码段放在已安装的 `models.py` 模块中，或者在启动时由 Django 导入的其他位置。

如果您已经创建了一些用户，则可以为所有现有用户生成令牌，如下所示：
```python
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

for user in User.objects.all():
    Token.objects.get_or_create(user=user)
```

##### 通过暴露 api 端点 (By exposing an api endpoint)
当使用 `TokenAuthentication` 时，你可能希望为客户端提供一种获取给定用户名和密码的令牌的机制。REST framework 提供了一个内置的视图来提供此行为。要使用它，请将 `obtain_auth_token` 视图添加到你的 URLconf：
```python
from rest_framework.authtoken import views
urlpatterns += [
    url(r'^api-token-auth/', views.obtain_auth_token)
]
```
请注意，模式的 URL 部分可以是您要使用的任何内容。

当使用表单数据或 JSON 将有效的 `username` 和 `password` 字段 POST 到视图时，`obtain_auth_token` 视图将返回 JSON 响应：
```python
{ 'token' : '9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b' }
```
请注意，默认的 `obtain_auth_token` 视图显式使用 JSON 请求和响应，而不是使用 settings 中默认的渲染器和解析器类。

默认情况下，没有应用于 `obtain_auth_token` 视图的权限或限制。如果您希望应用限制，则需要重写视图类，并使用 `throttle_classes` 属性将它们包括在内。

如果需要自定义版本的 `obtain_auth_token` 视图，可以通过子类化 `ObtainAuthToken` 类，并在 url conf 中使用它来实现。

例如，您可以返回令牌值之外的其他用户信息：
```python
from rest_framework.authtoken.views import ObtainAuthToken
from rest_framework.authtoken.models import Token
from rest_framework.response import Response

class CustomAuthToken(ObtainAuthToken):

    def post(self, request, *args, **kwargs):
        serializer = self.serializer_class(data=request.data,
                                           context={'request': request})
        serializer.is_valid(raise_exception=True)
        user = serializer.validated_data['user']
        token, created = Token.objects.get_or_create(user=user)
        return Response({
            'token': token.key,
            'user_id': user.pk,
            'email': user.email
        })
```
在你的 `urls.py` 中：
```python
urlpatterns += [
    url(r'^api-token-auth/', CustomAuthToken.as_view())
]
```

##### With Django admin
还可以通过管理界面手动创建令牌。如果您使用的是大型用户基础，我们建议您修补 `TokenAdmin` 类以根据需要对其进行自定义，更具体地说，将 `user` 字段声明为`raw_field`。

`your_app/admin.py`：
```python
from rest_framework.authtoken.admin import TokenAdmin

TokenAdmin.raw_id_fields = ('user',)
```

#### 使用 Django manage.py 命令 (Using Django manage.py command)
从版本3.6.4开始，可以使用以下命令生成用户令牌：
```python
./manage.py drf_create_token <username>
```
此命令将返回给定用户的 API 令牌，如果它不存在则创建它：
```python
Generated token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b for user user1
```
如果您想要重新生成令牌 (例如如果它已被破坏或泄露)，您可以传递一个额外的参数：
```python
./manage.py drf_create_token -r <username>
```

## SessionAuthentication
此身份验证方案使用 Django 的默认会话后端进行身份验证。会话身份验证适用于与您的网站在同一会话上下文中运行的 AJAX 客户端。

如果成功通过身份验证，则 `SessionAuthentication` 提供以下凭据。

- `request.user` 是 Django `User` 实例。
- `request.auth` 是 `None`。

未经许可的未经身份验证的响应将导致 `HTTP 403 Forbidden` 响应。

如果您正在使用带有 SessionAuthentication 的 AJAX 样式的 API，则需要确保为任何“不安全” HTTP 方法调用 (例如 `PUT`，`PATCH`，`POST` 或 `DELETE` 请求) 包含有效的 CSRF 令牌。有关更多详细信息，请参阅 [Django CSRF 文档](https://docs.djangoproject.com/en/stable/ref/csrf/#ajax)。

**警告**：创建登录页面时，始终使用 Django 的标准登录视图。这将确保您的登录视图得到适当保护。

REST framework 中的 CSRF 验证与标准 Django 的工作方式略有不同，因为需要同时支持基于会话和非会话的身份验证。这意味着只有经过身份验证的请求才需要 CSRF 令牌，并且可以在没有 CSRF 令牌的情况下发送匿名请求。此行为不适用于登录视图，登录视图应始终应用 CSRF 验证。

## RemoteUserAuthentication
此身份验证方案允许您将身份验证委托给 Web 服务器，该服务器设置 `REMOTE_USER` 环境变量。

要使用它，你必须在你的 `AUTHENTICATION_BACKENDS` 设置中有 `django.contrib.auth.backends.RemoteUserBackend` （或者子类）。默认情况下，`RemoteUserBackend` 为尚不存在的用户名创建 `User` 对象。要更改此行为和其他行为，请参阅 [Django 文档](https://docs.djangoproject.com/en/stable/howto/auth-remote-user/)。

如果成功通过身份验证，`RemoteUserAuthentication` 提供以下凭据：

- `request.user` 是 Django `User` 实例。
- `request.auth` 是 `None`。

有关配置身份验证方法的信息，请参阅 Web 服务器的文档，例如：

- [Apache Authentication How-To](https://httpd.apache.org/docs/2.4/howto/auth.html)
- [NGINX (Restricting Access)](https://www.nginx.com/resources/admin-guide/#restricting_access)

# 自定义身份验证 (Custom authentication)
要实现自定义的身份验证方案，要继承 `BaseAuthentication` 类并且重写 `.authenticate(self, request)` 方法。如果认证成功，该方法应返回 `(user, auth)` 的二元元组，否则返回 `None`。

在某些情况下，您可能想要从 `.authenticate()` 方法引发 `AuthenticationFailed` 异常，而不是返回 `None`。

通常，您应采取的方法是：

- 如果未尝试验证，返回 `None`。还将检查任何其他正在使用的身份验证方案。
- 如果尝试验证失败，则引发 `AuthenticationFailed` 异常。无论是否进行任何权限检查，都将立即返回错误响应，并且不会检查任何其他身份验证方案。

您还可以重写 `.authenticate_header(self, request)` 方法。如果实现，则应返回将用作 `HTTP 401 Unauthorized` 响应中的 `WWW-Authenticate` 标头的值的字符串。

如果 `.authenticate_header()` 方法未被重写，则身份验证方案将在未经验证的请求被拒绝访问时返回 `HTTP 403 Forbidden` 响应。

***

**注意**：当您的自定义身份验证器被请求对象的 `.user` 或 `.auth` 属性调用时，您可能会看到 `AttributeError` 作为 `WrappedAttributeError` 被重新引发。这对于防止原始异常被外部属性访问所抑制是必要的。Python 不会识别来自您的自定义身份验证器的 `AttributeError`，而是会假设请求对象没有 `.user` 或 `.auth` 属性。这些错误应由您的验证器修复或以其他方式处理。

***

## 举个栗子
以下示例将对在自定义请求标头中名为 'X_USERNAME' 的用户名指定的用户的任何传入请求进行身份验证。
```python
from django.contrib.auth.models import User
from rest_framework import authentication
from rest_framework import exceptions

class ExampleAuthentication(authentication.BaseAuthentication):
    def authenticate(self, request):
        username = request.META.get('X_USERNAME')
        if not username:
            return None

        try:
            user = User.objects.get(username=username)
        except User.DoesNotExist:
            raise exceptions.AuthenticationFailed('No such user')

        return (user, None)
```

***

# 第三方包 (Third party packages)
以下是可用的第三方包。

## Django OAuth Toolkit
[Django OAuth Toolkit](https://github.com/evonove/django-oauth-toolkit) 包提供了 OAuth 2.0 支持，并且兼容 Python 2.7 和 Python 3.3+。这个包使用优秀的 [OAuthLib](https://github.com/idan/oauthlib)，由 [Evonove](https://github.com/evonove/) 维护。该软件包有很完善的文档，并得到很好的支持，目前是我们推荐使用的 OAuth 2.0 支持软件包。

#### 安装和配置 (Installation & configuration)
使用 `pip` 安装。
```python
pip install django-oauth-toolkit
```
把这个包添加到你的 `INSTALLED_APPS` 中，并修改您的 REST framework 设置。
```python
INSTALLED_APPS = (
    ...
    'oauth2_provider',
)

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'oauth2_provider.contrib.rest_framework.OAuth2Authentication',
    )
}
```
有关更多详细信息，请参阅 [Django REST框架 - 入门](https://django-oauth-toolkit.readthedocs.io/en/latest/rest-framework/getting_started.html)文档。

## Django REST framework OAuth
[Django REST framework OAuth](https://jpadilla.github.io/django-rest-framework-oauth/) 包为 REST framework 提供 OAuth1 和 OAuth2 支持。

此软件包以前直接包含在 REST framework 包中，但现在作为第三方包支持和维护。

#### 安装和配置 (Installation & configuration)
使用 `pip` 安装软件包。
```python
pip install djangorestframework-oauth
```
有关配置和使用的详细信息，请参阅 Django REST framework OAuth 文档的[身份验证](https://jpadilla.github.io/django-rest-framework-oauth/authentication/)和[权限](https://jpadilla.github.io/django-rest-framework-oauth/permissions/)。

## Digest Authentication
HTTP 摘要认证是一种广泛实施的方案，旨在取代 HTTP 基本认证，并提供简单的加密认证机制。[Juan Riaza](https://github.com/juanriaza) 维护着 [djangorestframework-digestauth](https://github.com/juanriaza/django-rest-framework-digestauth) 包，它为 REST framework 提供 HTTP 摘要认证支持。

## Django OAuth2 Consumer
[Rediker Software](https://github.com/Rediker-Software) 的 [Django OAuth2 Consumer](https://github.com/Rediker-Software/doac) 是另一个[为 REST framework 提供 OAuth 2.0 支持](https://github.com/Rediker-Software/doac/blob/master/docs/integrations.md#)的软件包。该软件包包含令牌的令牌范围权限，允许对你的 API 进行更细粒度的访问。

## JSON Web Token Authentication
JSON Web Token 是一个相当新的标准，可用于基于令牌的身份验证。与内置 TokenAuthentication 方案不同，JWT Authentication 不需要使用数据库来验证令牌。[Blimp](https://github.com/GetBlimp) 维护着 [djangorestframework-jwt](https://github.com/GetBlimp/django-rest-framework-jwt) 软件包，它提供了 JWT Authentication 类，以及客户端获取给定用户名和密码的 JWT 的机制。JWT 身份验证的另一个包是 [djangorestframe -simplejwt](https://github.com/davesque/django-rest-framework-simplejwt)，它提供了不同的特性以及可插入的令牌黑名单应用程序。

## Hawk HTTP Authentication
[HawkREST](https://hawkrest.readthedocs.io/en/latest/) 库以 [Mohawk](https://mohawk.readthedocs.io/en/latest/) 库为基础，允许您在 API 中处理 [Hawk](https://github.com/hueniverse/hawk) 签名的请求和响应。Hawk 让双方使用共享密钥签名的消息彼此安全地进行通信。它基于 [HTTP MAC 访问认证](https://tools.ietf.org/html/draft-hammer-oauth-v2-mac-token-05)(基于 [OAuth 1.0](https://oauth.net/core/1.0a/) 的部分)。

## HTTP Signature Authentication
HTTP 签名(目前为 [IETF 草案](https://datatracker.ietf.org/doc/draft-cavage-http-signatures/))提供了一种实现 HTTP 消息的源认证和消息完整性的方法。与 [Amazon 的 HTTP 签名方案](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)类似，许多服务使用它，它允许无状态的每个请求的身份验证。[Elvio Toccalino](https://github.com/etoccalino/) 维护 [djangorestframework-httpsignature](https://github.com/etoccalino/django-rest-framework-httpsignature) 包，提供了一个易于使用的 HTTP 签名身份验证机制。

## Djoser
[Djoser](https://github.com/sunscrapers/djoser) 库提供了一组视图来处理诸如注册，登录，注销，密码重置和帐户激活等基本操作。该包使用自定义用户模型，并使用基于令牌的身份验证。这是一个可以使用 REST 实现的 Django 身份验证系统。

## django-rest-auth
[Django-rest-auth](https://github.com/Tivix/django-rest-auth) 库提供了一组 REST API 端点，用于注册，身份验证(包括社交媒体身份验证)，密码重置，检索和更新用户详细信息等。通过拥有这些 API 端点，您的客户端应用程序(如 AngularJS、iOS、Android 等)可以通过 REST API 独立地与 Django 后端站点通信，用于用户管理。

## django-rest-framework-social-oauth2
[Django-rest-framework-social-oauth2](https://github.com/PhilipGarnero/django-rest-framework-social-oauth2) 库提供了一种将社交插件(facebook、twitter、google 等)集成到您的身份验证系统和简单的 oauth2 设置的简单方法。使用这个库，您将能够基于外部令牌(例如 facebook 访问令牌)对用户进行身份验证，将这些令牌转换为“内部” oauth2 令牌，并使用和生成 oauth2 令牌来验证您的用户。

## django-rest-knox
[Django-rest-knox](https://github.com/James1345/django-rest-knox) 库提供模型和视图，以比内置 TokenAuthentication 方案更安全和可扩展的方式处理基于令牌的身份验证 - 考虑到单页应用程序和移动客户端。它为每个客户端提供令牌，并在提供一些其他身份验证(通常是基本身份验证)时生成它们，以删除令牌(提供服务器强制注销)，并删除所有令牌(注销用户登录的所有客户端)的视图。

## drfpasswordless
[drfpasswordless](https://github.com/aaronn/django-rest-framework-passwordless) 为 Django REST Framework 自己的 TokenAuthentication 方案添加了(Medium，Square Cash灵感)无密码支持。用户登录并使用发送到联系人点(如电子邮件地址或手机号码)的令牌进行注册。
