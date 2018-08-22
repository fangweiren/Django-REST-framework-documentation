# ViewSets
在路由确定了用于请求的控制器之后，控制器负责了解请求并生成适当的输出。—— [Ruby on Rails 文档](http://guides.rubyonrails.org/routing.html)

Django REST framework 允许您将一组相关视图的逻辑组合在一个单独的类中，称为 `ViewSet`。在其他框架中，您可能会发现概念上类似的实现，名为 “Resources” 或 “Controllers” 。

`ViewSet` 类只是一种基于类的视图类型，它不提供任何方法处理程序，如 `.get()` 或  `.post()`，而是提供诸如 `.list()` 和 `.create()` 之类的操作。

`ViewSet` 的方法处理程序使用 `.as_view()` 方法在最终确定视图时绑定相应操作。

通常，与其在 urlconf 中显式地注册视图集中的视图，倒不如用路由器类注册视图集，它自动为您确定 urlconf。

## 举个栗子
让我们定义一个简单的视图集，可用于列出或检索系统中的所有用户。
```python
from django.contrib.auth.models import User
from django.shortcuts import get_object_or_404
from myapps.serializers import UserSerializer
from rest_framework import viewsets
from rest_framework.response import Response

class UserViewSet(viewsets.ViewSet):
    """
    A simple ViewSet for listing or retrieving users.
    """
    def list(self, request):
        queryset = User.objects.all()
        serializer = UserSerializer(queryset, many=True)
        return Response(serializer.data)

    def retrieve(self, request, pk=None):
        queryset = User.objects.all()
        user = get_object_or_404(queryset, pk=pk)
        serializer = UserSerializer(user)
        return Response(serializer.data)
```
如果需要，我们可以将此视图集绑定到两个单独的视图中，如下所示：
```python
user_list = UserViewSet.as_view({'get': 'list'})
user_detail = UserViewSet.as_view({'get': 'retrieve'})
```
通常我们不会这样做，而是用一个路由器注册视图集，并允许自动生成 urlconf。
```python
from myapp.views import UserViewSet
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register(r'users', UserViewSet, base_name='user')
urlpatterns = router.urls
```
您通常希望使用提供默认行为集合的现有基类，而不是编写自己的视图集。举个栗子：
```python
class UserViewSet(viewsets.ModelViewSet):
    """
    用于查看和编辑用户实例的视图集。
    """
    serializer_class = UserSerializer
    queryset = User.objects.all()
```
使用 `ViewSet` 类比使用 `View` 类有两个主要优点。

- 重复的逻辑可以组合成一个类。在上面的示例中，我们只需要指定一次 `queryset`，它将在多个视图中使用。
- 通过使用 routers，我们不再需要处理自己的 URL 连接。

这两者各有优缺点。使用常规视图和 URL 配置文件更加明确，并为您提供更多控制。如果您想快速启动和运行，或者当您拥有大型 API 并希望始终执行一致的 URL 配置时， ViewSets 非常有用。

## 视图集操作
REST framework 中包含的默认路由器将为一套标准的创建/检索/更新/销毁样式操作提供路由，如下所示：
```python
class UserViewSet(viewsets.ViewSet):
    """
    示例空视图集演示由路由器类处理的标准操作。

    如果您使用的是格式后缀，请确保每个操作还包括 `format=None` 关键字参数。
    """

    def list(self, request):
        pass

    def create(self, request):
        pass

    def retrieve(self, request, pk=None):
        pass

    def update(self, request, pk=None):
        pass

    def partial_update(self, request, pk=None):
        pass

    def destroy(self, request, pk=None):
        pass
```

## 反思视图集操作
在调度期间，`ViewSet` 上提供了以下属性。

- `basename` —— 用于创建的 URL 名称的基础。
- `action` —— 当前操作的名称 (例如，list，create)。
- `detail` —— 布尔值指示当前动作是否配置为列表或详细视图。
- `suffix` —— 视图集类型的显示后缀 - 反射详细属性。

您可以检查这些属性以根据当前操作调整行为。例如，您可以限制除了与 `list` 类似的所有权限：
```python
def get_permissions(self):
    """
    实例化并返回该视图所需的权限列表。
    """
    if self.action == 'list':
        permission_classes = [IsAuthenticated]
    else:
        permission_classes = [IsAdmin]
    return [permission() for permission in permission_classes]
```

## 标记路由的额外操作
如果你有可路由的特殊方法，你可以用 `@action` 装饰器来标记它们。与常规操作一样，额外的操作可以用于对象列表或单个实例。要指出这一点，设置 `detail` 参数为 `True` 或 `False`。路由器将相应地配置其 URL 模式。例如，`DefaultRouter` 将在其 URL 模式中配置包含 `pk` 的详细操作。

更完整的额外操作例子：
```python
from django.contrib.auth.models import User
from rest_framework import status, viewsets
from rest_framework.decorators import action
from rest_framework.response import Response
from myapp.serializers import UserSerializer, PasswordSerializer

class UserViewSet(viewsets.ModelViewSet):
    """
    提供标准操作的视图集
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer

    @action(methods=['post'], detail=True)
    def set_password(self, request, pk=None):
        user = self.get_object()
        serializer = PasswordSerializer(data=request.data)
        if serializer.is_valid():
            user.set_password(serializer.data['password'])
            user.save()
            return Response({'status': 'password set'})
        else:
            return Response(serializer.errors,
                            status=status.HTTP_400_BAD_REQUEST)

    @action(detail=False)
    def recent_users(self, request):
        recent_users = User.objects.all().order('-last_login')

        page = self.paginate_queryset(recent_users)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(recent_users, many=True)
        return Response(serializer.data)
```
另外，装饰器可以为路由视图设置额外的参数。举个栗子：
```python
    @action(methods=['post'], detail=True, permission_classes=[IsAdminOrIsSelf])
    def set_password(self, request, pk=None):
       ...
```
这些装饰器将默认路由 `GET` 请求，但也可以通过设置 `methods` 参数来接受其他 HTTP 方法。例如：
```python
    @action(methods=['post', 'delete'], detail=True)
    def unset_password(self, request, pk=None):
       ...
```
两个新的操作将在 url `^users/{pk}/set_password/$` 和  `^users/{pk}/unset_password/$` 上可用。

要查看所有额外操作，请调用 `.get_extra_actions()` 方法。

## 反转操作 urls
如果你需要获取操作的 URL，请使用 `.reverse_action()` 方法。这是 `Reverse()` 的便利封装，它自动传递视图的 `request` 对象，并在 `url_name` 前加上 `.basename` 属性。

请注意，`basename` 是路由器在 `ViewSet` 注册期间提供的。如果不使用路由器，则必须向 `.as_view()` 方法提供 `basename` 参数。

使用上一节中的示例：
```python
>>> view.reverse_action('set-password', args=['1'])
'http://localhost:8000/api/users/1/set_password'
```
或者，您可以使用 `@action` 装饰器设置的 `url_name` 属性。
```python
>>> view.reverse_action(view.set_password.url_name, args=['1'])
'http://localhost:8000/api/users/1/set_password'
```
`.reverse_action()` 的 `url_name` 参数应该与 `@action` 装饰器匹配相同的参数。此外，此方法可用于反转默认操作，例如 `list` 和 `create`。

***

# API 参考
## ViewSet
`ViewSet` 类继承自 `APIView`。您可以使用任何标准属性 (例如 `permission_classes`，`authentication_classes`) 来控制视图集上的 API 策略。

`ViewSet` 类不提供任何操作的实现。为了使用 `ViewSet` 类，您将重写该类并显式地定义操作的实现。

## GenericViewSet
`GenericViewSet` 类继承自 `GenericAPIView` ，并提供默认的 `get_object`，`get_queryset` 方法和其他通用视图基本行为，但默认情况下不包含任何操作。

为了使用 `GenericViewSet` 类，您将重写该类，或混合所需的 mixin 类，或者显式定义操作实现。

## ModelViewSet
`ModelViewSet` 类继承自 `GenericAPIView`，并通过混合各种 mixin 类的行为来包含各种操作的实现。

`ModelViewSet` 类提供的操作有 `.list()`，`.retrieve()`，`.create()`，`.update()`，`.partial_update()` 和  `.destroy()`。

#### 举个栗子
因为 `ModelViewSet` 扩展了 `GenericAPIView` ，所以通常需要至少提供 `queryset` 和 `serializer_class` 属性。例如：
```python
class AccountViewSet(viewsets.ModelViewSet):
    """
    用于查看和编辑帐户的简单 ViewSet。
    """
    queryset = Account.objects.all()
    serializer_class = AccountSerializer
    permission_classes = [IsAccountAdminOrReadOnly]
```
注意，您可以使用 `GenericAPIView` 提供的任何标准属性或方法重写。例如，要动态确定它应该操作的查询集的 `ViewSet`，可以这样做：
```python
class AccountViewSet(viewsets.ModelViewSet):
    """
    一个简单的视图集，用于查看和编辑与用户相关的帐户。
    """
    serializer_class = AccountSerializer
    permission_classes = [IsAccountAdminOrReadOnly]

    def get_queryset(self):
        return self.request.user.accounts.all()
```
然而注意，当从您的 `ViewSet` 中移除 `queryset` 属性，任何关联的[路由器](http://www.django-rest-framework.org/api-guide/routers/)将无法自动导出模型的 base_name，因此您必须指定 `base_name` kwarg 作为[路由器注册](http://www.django-rest-framework.org/api-guide/routers/)的一部分。

还要注意，尽管这个类默认情况下提供了完整的创建/列表/检索/更新/销毁操作集，但是可以使用标准权限类来限制可用操作。

## ReadOnlyModelViewSet
`ReadOnlyModelViewSet` 类也继承自 `GenericAPIView`。与 `ModelViewSet` 一样，它还包括各种操作的实现，但与 `ModelViewSet` 不同，它只提供“只读”操作，`.list()` 和 `.retrieve()`。

#### 举个栗子
与 `ModelViewSet` 一样，您通常需要至少提供 `queryset` 和 `serializer_class` 属性。例如：
```python
class AccountViewSet(viewsets.ReadOnlyModelViewSet):
    """
    用于查看帐户的简单 ViewSet。
    """
    queryset = Account.objects.all()
    serializer_class = AccountSerializer
```
同样，与 `ModelViewSet` 一样，您可以使用 `GenericAPIView` 可用的任何标准属性和方法重写。

# 自定义视图集基类
您可能需要提供自定义 `ViewSet` 类，这些类没有完整的 `ModelViewSet` 操作集，或者以其他方式自定义行为。

## 举个栗子
要创建提供 `create`，`list` 和 `retrieve` 操作的基本视图集类，请继承 `GenericViewSet`，并混合所需的操作：
```python
from rest_framework import mixins

class CreateListRetrieveViewSet(mixins.CreateModelMixin,
                                mixins.ListModelMixin,
                                mixins.RetrieveModelMixin,
                                viewsets.GenericViewSet):
    """
    提供“检索”、“创建”和“列表”操作的视图集。

    若要使用它，请重写该类并设置 “.queryset” 和 “.serializer_class” 属性。
    """
    pass
```
通过创建自己的基本 `ViewSet` 类，可以提供在 API 中的多个视图集中重用的通用行为。
