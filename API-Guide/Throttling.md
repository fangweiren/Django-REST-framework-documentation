# 限流 (Throttling)
限流类似于[权限](http://www.django-rest-framework.org/api-guide/permissions/)，因为它确定是否应该授权请求。限流阀表示临时状态，并用来控制客户端对 API 的请求速率。

与权限一样，可能会使用多种限流方式。您的 API 可能对未经身份验证的请求进行限流，对经过身份验证的请求限流较少。

您可能想要使用多种限流的另一种方案是，如果您需要对 API 的不同部分施加不同的约束，由于某些服务特别是资源密集型。

如果您想同时施加突发节流速率和持续节流速率，还可以使用多个限流阀。例如，您可能希望将用户限制为每分钟最多 60 个请求，每天 1000 个请求。

限流阀不一定只指限速请求。例如，存储服务可能还需要限制带宽，而付费数据服务可能希望限制访问的记录的特定数量。

## 如何确定限流 (How throttling is determined)
与权限和身份验证一样，REST framework 中的限流始终定义为类列表。

在运行视图的主体之前，检查列表中的每个限流阀。如果任何限流检查失败，将引发 `exceptions.Throttled` 异常，并且该视图的主体将不会再执行。

## 设置限流策略 (Setting the throttling policy)
使用 `DEFAULT_THROTTLE_CLASSES` 和 `DEFAULT_THROTTLE_RATES` settings 可以设置全局的默认限流策略。例如：
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ),
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day'
    }
}
```
`DEFAULT_THROTTLE_RATES` 中使用的频率描述可能包括 `second`，`minute`，`hour` 或 `day` 作为限流周期。

您还可以使用基于 `APIView` 类的视图，在每个视图或每个视图集的基础上设置限流策略。
```python
from rest_framework.response import Response
from rest_framework.throttling import UserRateThrottle
from rest_framework.views import APIView

class ExampleView(APIView):
    throttle_classes = (UserRateThrottle,)

    def get(self, request, format=None):
        content = {
            'status': 'request was permitted'
        }
        return Response(content)
```
或者使用基于函数的视图的 `@api_view` 装饰器。
```python
@api_view(['GET'])
@throttle_classes([UserRateThrottle])
def example_view(request, format=None):
    content = {
        'status': 'request was permitted'
    }
    return Response(content)
```

## 如何识别客户端 (How clients are identified)
`X-Forwarded-For` HTTP 标头和 `REMOTE_ADDR` WSGI 变量用于为限流的唯一标识的客户端 IP 地址。如果存在 `X-Forwarded-For` 标头，则将使用它，否则将使用 WSGI 环境中的 `REMOTE_ADDR` 变量的值。

如果您需要严格标识唯一的客户端 IP 地址，则需要首先通过设置 `NUM_PROXIES` 设置来配置 API 运行的应用程序代理的数量。此设置应为零或更大的整数。如果设置为非零，则一旦任何应用程序代理 IP 地址首先被排除，客户端 IP 将被标识为 `X-Forwarded-For` 标头中的最后一个 IP 地址。如果设置为零，则 `REMOTE_ADDR` 值将始终用作识别 IP 地址。

重要的是要理解，如果配置 `NUM_PROXIES` 设置，那么唯一的 [NAT](https://en.wikipedia.org/wiki/Network_address_translation) 网关后面的所有客户端将被视为单个客户端。

关于 `X-Forwarded-For` 标头如何工作以及识别远程客户端 IP 的更多上下文可以[在这里找到](http://oxpedia.org/wiki/index.php?title=AppSuite:Grizzly#Multiple_Proxies_in_front_of_the_cluster)。

## 设置缓存 (Setting up the cache)
EST framework 提供的限流类使用 Django 的缓存后端。你应该确保你已经设置了适当的[缓存设置](https://docs.djangoproject.com/en/stable/ref/settings/#caches)。对于简单的设置，`LocMemCache` 后端的默认值应该没问题。有关更多详细信息，请参阅 Django 的[缓存文档](https://docs.djangoproject.com/en/stable/topics/cache/#setting-up-the-cache)。

如果您需要使用 `'default'` 以外的缓存，则可以通过创建自定义限流类并设置 `cache` 属性来实现。例如：
```python
class CustomAnonRateThrottle(AnonRateThrottle):
    cache = get_cache('alternate')
```
您还需要记住在 `'DEFAULT_THROTTLE_CLASSES'` settings 键中设置自定义的限流类，或者使用 `throttle_classes` 视图属性。

***

# API 参考
## AnonRateThrottle
`AnonRateThrottle` 只会限制未经认证的用户。传入请求的 IP 地址用于生成唯一的键来限流。

允许的请求率由以下之一决定 (按优先顺序)。

- 类上的 `rate` 属性，可以通过重写 `AnonRateThrottle` 并设置其属性来提供。
- `DEFAULT_THROTTLE_RATES['anon']` 设置。

如果您想限制未知来源的请求率，`AnonRateThrottle` 是合适的。

## UserRateThrottle
`UserRateThrottle` 将通过 API 限制用户的给定请求率。用户 id 用于生成唯一的密钥来加以限制。未经身份验证的请求将回退到使用传入请求的 IP 地址生成唯一的密钥来进行限制。

允许的请求率由以下之一决定 (按优先顺序)。

- 类上的 `rate` 属性，可以通过重写 `UserRateThrottle` 并设置其属性来提供。
- `DEFAULT_THROTTLE_RATES['user']` 设置。

API 可能同时具有多个 `UserRateThrottles`。为此，请重写 `UserRateThrottle` 并为每个类设置唯一的 “范围”。

例如，可以通过使用以下类来实现多个用户限流率...
```python
class BurstRateThrottle(UserRateThrottle):
    scope = 'burst'

class SustainedRateThrottle(UserRateThrottle):
    scope = 'sustained'
```
...和以下设置。
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'example.throttles.BurstRateThrottle',
        'example.throttles.SustainedRateThrottle'
    ),
    'DEFAULT_THROTTLE_RATES': {
        'burst': '60/min',
        'sustained': '1000/day'
    }
}
```
如果希望对每个用户进行简单的全局速率限制，那么 `UserRateThrottle` 是合适的。

## ScopedRateThrottle
`ScopedRateThrottle` 类可用于限制对 API 特定部分的访问。只有当正在访问的视图包含 `.throttle_scope` 属性时才会应用此限制。然后通过将请求的 “范围” 与唯一的用户 id 或 IP 地址连接起来形成唯一的限流密钥。

允许的请求率由 `DEFAULT_THROTTLE_RATES` setting 使用请求 “范围” 中的键确定。

例如，给出以下视图...
```python
class ContactListView(APIView):
    throttle_scope = 'contacts'
    ...

class ContactDetailView(APIView):
    throttle_scope = 'contacts'
    ...

class UploadView(APIView):
    throttle_scope = 'uploads'
    ...
```
...和以下设置。
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.ScopedRateThrottle',
    ),
    'DEFAULT_THROTTLE_RATES': {
        'contacts': '1000/day',
        'uploads': '20/day'
    }
}
```
用户对 `ContactListView` 或 `ContactDetailView` 的请求将被限制为每天 1000 次。用户对 `UploadView` 的请求将被限制为每天 20 次。

***

# 自定义限流 (Custom throttles)
要创建自定义限流，请重写 `BaseThrottle` 类并实现 `.allow_request(self, request, view)`。如果请求被允许，该方法应返回 `True`，否则返回 `False`。

还可以选择重写 `.wait()` 方法。如果实现，`.wait()` 应返回在尝试下一次请求之前等待的推荐秒数，或者返回 `None`。如果 `.allow_request()` 先前已经返回 `False`，则只会调用 `.wait()` 方法。

如果 `.wait()` 方法被实现并且请求受到限制，那么 `Retry-After` 标头将包含在响应中。

## 举个栗子
以下是速率限制的示例，它将随机地限制 10 次请求中的 1 次。
```python
import random

class RandomRateThrottle(throttling.BaseThrottle):
    def allow_request(self, request, view):
        return random.randint(1, 10) != 1
```
