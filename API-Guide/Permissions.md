# 权限 (Permissions)
认证或识别本身通常不足以获得对信息或代码的访问。为此，请求访问的实体必须具有授权。—— [Apple 开发人员文档](https://developer.apple.com/library/mac/#documentation/security/Conceptual/AuthenticationAndAuthorizationGuide/Authorization/Authorization.html)

与[身份验证](http://www.django-rest-framework.org/api-guide/authentication/)和[限流](http://www.django-rest-framework.org/api-guide/throttling/)一起，权限确定是否应该授予或拒绝访问请求。

在允许任何其他代码继续之前，权限检查始终在视图的最开始运行。权限检查通常会使用 `request.user` 和 `request.auth` 属性中的认证信息来确定是否允许传入请求。

权限用于授予或拒绝不同类别的用户访问 API 的不同部分。

最简单的权限类型是允许访问任何经过身份验证的用户，并拒绝访问任何未经身份验证的用户。这对应于 REST framework 中的 `IsAuthenticated` 类。

稍微宽松的权限会允许通过身份验证的用户完全访问，而未经身份验证的用户只能进行只读访问。这对应于 REST framework 中的 `IsAuthenticatedOrReadOnly` 类。

## 如何确定权限 (How permissions are determined)
REST framework 中的权限始终被定义为权限类的列表。

在运行视图的主体之前，列表中的每个权限都会被检查。如果任何权限检查失败，则会引发 `exceptions.PermissionDenied` 或 `exceptions.NotAuthenticated` 异常，并且视图的主体不会再运行。

当权限检查失败时，根据以下规则，将返回 “403 Forbidden” 或 “401 Unauthorized” 响应：

- 请求成功通过身份验证，但权限被拒绝。 —— 将返回 403 Forbidden 响应。
- 请求未成功通过身份验证，并且最高优先级身份验证类未使用 `WWW-Authenticate` 标头。—— 将返回 403 Forbidden 响应。
- 请求未成功通过身份验证，并且最高优先级身份验证类使用 `WWW-Authenticate` 标头。—— 将返回 HTTP 401 Unauthorized 响应，并附带适当的 `WWW-Authenticate` 标头。

## 对象级权限 (Object level permissions)
REST framework 权限还支持对象级权限。对象级权限用于确定是否允许用户对特定对象进行操作，该特定对象通常是指模型实例。

当 `.get_object()` 被调用时，对象级权限由 REST framework 的通用视图运行。与视图级权限一样，如果用户不被允许对给定对象进行操作，则会引发 `exceptions.PermissionDenied` 异常。

如果您正在编写自己的视图并希望强制执行对象级权限，或者在通用视图上重写 `get_object` 方法，那么您需要在检索对象时显式地调用视图上的 `.check_object_permissions(request, obj)` 方法。

这将引发 `PermissionDenied` 或 `NotAuthenticated` 异常，或者只是在视图具有适当的权限时才返回。

举个栗子：
```python
def get_object(self):
    obj = get_object_or_404(self.get_queryset(), pk=self.kwargs["pk"])
    self.check_object_permissions(self.request, obj)
    return obj
```

#### 对象级权限的限制 (Limitations of object level permissions)
出于性能原因，在返回对象列表时，通用视图不会自动将对象级权限应用于查询集中的每个实例。

通常，当您使用对象级权限时，您还需要适当地[过滤查询集](http://www.django-rest-framework.org/api-guide/filtering/)，以确保用户只能看到他们被允许查看的实例。

## 设置权限策略 (Setting the permission policy)
可以使用 `DEFAULT_PERMISSION_CLASSES` setting 全局设置默认权限策略。例如：
```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    )
}
```
如果未指定，则此设置默认为允许无限制访问：
```python
'DEFAULT_PERMISSION_CLASSES': (
   'rest_framework.permissions.AllowAny',
)
```
您还可以使用基于类的视图 (APIView) 在每个视图或每个视图集的基础上设置身份验证策略。
```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

class ExampleView(APIView):
    permission_classes = (IsAuthenticated,)

    def get(self, request, format=None):
        content = {
            'status': 'request was permitted'
        }
        return Response(content)
```
或者，使用基于函数的视图的 `@api_view` 装饰器。
```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

@api_view(['GET'])
@permission_classes((IsAuthenticated, ))
def example_view(request, format=None):
    content = {
        'status': 'request was permitted'
    }
    return Response(content)
```
**注意**：当通过类属性或装饰器设置新的权限类时，您会告诉视图忽略 **settings.py** 文件上设置的默认列表。

***

# API 参考
## AllowAny
`AllowAny` 权限类将允许不受限制的访问，**不管请求是经过身份验证还是未经身份验证**。

此权限不是严格要求的，因为您可以通过使用空列表或元组进行权限设置来获得相同的结果，但您可能会发现指定此类很有用，因为它使意图明确。

## IsAuthenticated
`IsAuthenticated` 权限类将拒绝任何未经身份验证的用户的权限，否则允许权限。

如果您希望 API 仅供注册用户访问，则此权限适用。

## IsAdminUser
`IsAdminUser` 权限类将拒绝任何用户的权限，除非在 `user.is_staff` 为 `True` 的情况下权限将被允许。

如果您希望 API 仅对受信任的管理员子集进行访问，则此权限是合适的。

## IsAuthenticatedOrReadOnly
`IsAuthenticatedOrReadOnly` 将允许经过身份验证的用户执行任何请求。如果请求方法是 “安全” 的方法之一 (`GET`、`HEAD`、`OPTIONS`)，则只允许未授权用户的请求。

如果您希望 API 允许匿名用户拥有读取权限，并且只允许对经过身份验证的用户拥有写入权限，则此权限是合适的。

## DjangoModelPermissions
此权限类与 Django 的标准 `django.contrib.auth` [模型权限](https://docs.djangoproject.com/en/stable/topics/auth/customizing/#custom-permissions)绑定。此权限只能应用于具有 `.queryset` 属性集的视图。只有在用户经过身份验证并分配了相关模型权限时，才会获得授权。

- `POST` 请求要求用户在模型上具有 `add` 权限。
- `PUT` 和 `PATCH` 请求要求用户在模型上具有 `change` 权限。
- `DELETE` 请求要求用户在模型上具有 `delete` 权限。

还可以重写默认行为以支持自定义模型权限。例如，你可能想要包含 `GET` 请求的 `view` 模型权限。

要使用自定义模型权限，请重写 `DjangoModelPermissions` 并设置 `.perms_map` 属性。有关详细信息，请参阅源代码。

#### 使用不包含 `queryset` 属性的视图 (Using with views that do not include a `queryset` attribute.)
如果您将此权限与重写的 `get_queryset()` 方法的视图一起使用，则视图上可能没有 `queryset` 属性。在这种情况下，我们建议还用前哨查询集标记视图，以便该类可以确定所需的权限。例如：
```python
queryset = User.objects.none()  # Required for DjangoModelPermissions
```

## DjangoModelPermissionsOrAnonReadOnly
与 `DjangoModelPermissions` 类似，但也允许未经身份验证的用户对 API 的进行只读访问。

## DjangoObjectPermissions
该权限类与 Django 的标准[对象权限框架](https://docs.djangoproject.com/en/stable/topics/auth/customizing/#handling-object-permissions)绑定，该框架允许模型上的每个对象权限。要使用此权限类，您还需要添加支持对象级权限的权限后端，例如 [django-guardian](https://github.com/lukaszb/django-guardian)。

与 `DjangoModelPermissions` 一样，此权限只能应用于具有 `.queryset` 属性或 `.get_queryset()` 方法的视图。只有在用户经过身份验证且分配了相关的每个对象权限和相关的模型权限时，才会获得授权。

- `POST` 请求要求用户在模型实例上具有 `add` 权限。
- `PUT` 和 `PATCH` 请求要求用户在模型实例上具有 `change` 权限。
- `DELETE` 请求要求用户在模型实例上具有 `delete` 权限。

请注意，`DjangoObjectPermissions` **不需要** `django-guardian` 软件包，并且应同样支持其他对象级后端。

与 `DjangoModelPermissions` 一样，您可以通过重写 `DjangoObjectPermissions` 并设置 `.perms_map` 属性来使用自定义模型权限。有关详细信息，请参阅源代码。

***

**注意**：如果您需要 `GET`，`HEAD` 和 `OPTIONS` 请求的对象级 `view` 权限，则还需要考虑添加 `DjangoObjectPermissionsFilter` 类，以确保列表端点只返回包含用户具有适当查看权限的对象的结果。

***
***

# 自定义权限 (Custom permissions)
要实现自定义权限，请重写 `BasePermission` 并实现以下方法中的一个或两个：

- `.has_permission(self, request, view)`
- `.has_object_permission(self, request, view, obj)`

如果请求被授予访问权限，则方法应返回 `True`，否则返回 `False`。

如果您需要测试请求是读操作还是写操作，则应该根据常量 `SAFE_METHODS` 检查请求方法，该常量是包含 `'GET'`，`'OPTIONS'` 和 `'HEAD'` 的元组。例如：
```python
if request.method in permissions.SAFE_METHODS:
    # Check permissions for read-only request
else:
    # Check permissions for write request
```
***
**注意**：只有在视图级别 `has_permission` 检查通过时才会调用实例级别的 `has_object_permission` 方法。还要注意，为了运行实例级检查，视图代码应显式调用 `.check_object_permissions(request, obj)`。如果您使用的是通用视图，则默认情况下将为您处理。(基于函数的视图需要显式检查对象权限，在失败时引发 `PermissionDenied`。)
***

如果测试失败，自定义权限将引发 `PermissionDenied` 异常。要更改与异常相关的错误消息，请直接在自定义权限上实现 `message` 属性。否则，将使用 `PermissionDenied` 中的 `default_detail` 属性。
```python
from rest_framework import permissions

class CustomerAccessPermission(permissions.BasePermission):
    message = 'Adding customers not allowed.'

    def has_permission(self, request, view):
         ...
```

## 举个栗子
以下是根据黑名单检查传入请求的 IP 地址的权限类的示例，并且如果 IP 已被列入黑名单，则拒绝该请求。
```python
from rest_framework import permissions

class BlacklistPermission(permissions.BasePermission):
    """
    对列入黑名单的IP进行全局权限检查。
    """

    def has_permission(self, request, view):
        ip_addr = request.META['REMOTE_ADDR']
        blacklisted = Blacklist.objects.filter(ip_addr=ip_addr).exists()
        return not blacklisted
```
除了针对所有传入请求运行的全局权限之外，您还可以创建对象级权限，仅运行针对影响特定对象实例的操作。例如：
```python
class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    对象级权限，仅允许对象的所有者编辑它。
    假设模型实例具有 `owner` 属性。
    """

    def has_object_permission(self, request, view, obj):
        # 任何请求都允许读取权限，
        # 所以我们总是允许 GET，HEAD 或 OPTIONS 请求。
        if request.method in permissions.SAFE_METHODS:
            return True

        # 实例必须具有名为 `owner` 的属性。
        return obj.owner == request.user
```
请注意，通用视图将检查适当的对象级权限，但如果您正在编写自己的自定义视图，则需要确保检查自己的对象级权限。您可以通过在拥有对象实例后从视图中调用 `self.check_object_permissions(request, obj)` 来完成此操作。如果任何对象级权限检查失败，此调用将引发适当的 `APIException`，否则将简单地返回。

还要注意，通用视图仅检查检索单个模型实例的视图的对象级权限。如果需要对列表视图进行对象级别过滤，则需要单独过滤查询集。有关详细信息，请参阅[过滤文档](http://www.django-rest-framework.org/api-guide/filtering/)。
***

# 第三方包 (Third party packages)
以下是可用的第三方包。

## Composed Permissions
[Composed Permissions](https://github.com/niwibe/djangorestframework-composed-permissions) 包使用小的和可重用组件来提供一种定义复杂和多深度 (带逻辑运算符) 权限对象的简单方法。

## REST Condition
[REST Condition](https://github.com/caxap/rest_condition) 包是以简单方便的方式构建复杂权限的另一种扩展。该扩展允许您将权限与逻辑运算符组合在一起。

## DRY Rest Permissions
[DRY Rest Permissions](https://github.com/Helioscene/dry-rest-permissions) 包提供了为单个默认和自定义操作定义不同权限的能力。此包适用于具有从应用程序数据模型中定义的关系派生的权限的应用程序。它还支持通过 API 的序列化器将权限检查返回给客户端应用程序。此外，它还支持向默认和自定义列表操作添加权限，以限制每个用户检索的数据。

## Django Rest Framework Roles
[Django Rest Framework Roles](https://github.com/computer-lab/django-rest-framework-roles) 包使您在多种类型的用户上参数化 API 更轻松。

## Django Rest Framework API Key
[Django Rest Framework API Key](https://github.com/manosim/django-rest-framework-api-key) 包允许您确保对服务器发出的每个请求都需要 API 密钥头。您可以从 django 管理界面生成一个。

## Django Rest Framework Role Filters
[Django Rest Framework Role Filters](https://github.com/allisson/django-rest-framework-role-filters) 包提供对多种类型角色的简单过滤。
