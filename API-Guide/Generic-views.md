# 通用视图 (Generic views)
Django 的通用视图…被开发为常用使用模式的捷径…他们在视图开发中发现了一些常见的习语和模式，并将它们抽象出来，这样就可以快速编写数据的共同视图，而不必重复自己。——[Django Documentation](https://docs.djangoproject.com/en/stable/ref/class-based-views/#base-vs-generic-views)

基于类的视图的一个关键好处是，它允许您组合一些可重用行为。REST framework 利用这一点，通过提供许多预构建的视图来提供常用模式。

REST framework 提供的通用视图允许您快速构建紧密映射到数据库模型的 API 视图。

如果通用视图不适合您的 API 需求，则可以下拉使用常规 `APIView` 类，或者重用通用视图使用的 mixins 和基类来组成自己的可重用通用视图集。

## 举个栗子
通常，在使用通用视图时，您将重写视图，并设置几个类属性。
```python
from django.contrib.auth.models import User
from myapp.serializers import UserSerializer
from rest_framework import generics
from rest_framework.permissions import IsAdminUser

class UserList(generics.ListCreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = (IsAdminUser,)
```
对于更复杂的情况，您可能还想重写视图类中的各种方法。例如。
```python
class UserList(generics.ListCreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = (IsAdminUser,)

    def list(self, request):
        # Note the use of `get_queryset()` instead of `self.queryset`
        queryset = self.get_queryset()
        serializer = UserSerializer(queryset, many=True)
        return Response(serializer.data)
```
对于非常简单的情况，您可能想要使用 `.as_view()` 方法传递任何类属性。例如，您的 URLconf 可能包含类似于以下条目的内容：
```python
url(r'^/users/', ListCreateAPIView.as_view(queryset=User.objects.all(), serializer_class=UserSerializer), name='user-list')
```

# API 参考 (API Reference)
## GenericAPIView
该类扩展了REST framework 的 `APIView` 类，为标准列表和详细视图添加了通常所需的行为。

每个具体的通用视图都是通过将 `GenericAPIView` 类和一个或多个 minxin 类相互结合来构建的。

### 属性 (Attributes)
**基本设置**：
以下属性控制基本视图行为。

- `queryset` —— 用于从该视图返回对象的查询集。通常，您必须设置此属性，或重写 `get_queryset()` 方法。如果您正在重写视图方法，那么重要的是调用 `get_queryset()`，而不是直接访问此属性，因为 `queryset` 将被评估一次，并且这些结果将被缓存用于所有后续请求。
- `serializer_class` —— 用于验证和反序列化输入以及序列化输出的序列化类。通常，您必须设置此属性，或重写 `get_serializer_class()` 方法。
- `lookup_field` —— 用于执行各个模型实例的对象查找的模型字段。默认为 `'pk'`。注意，在使用超链接 API 时，如果需要使用自定义值，则需要确保 API 视图和序列化器类都设置查找字段。
- `lookup_url_kwarg` —— 用于对象查找的 URL 关键字参数。URL conf 应该包含与此值相对应的关键字参数。如果取消设置默认值，使用与 `lookup_field` 相同的值。

**分页**：
与列表视图一起使用时，以下属性用于控制分页。

- `pagination_class` —— 在对列表结果进行分页时应该使用的分页类。默认为与 `DEFAULT_PAGINATION_CLASS` 设置相同的值，即 `'rest_framework.pagination.PageNumberPagination'`。设置 `pagination_class = None` 将禁用此视图的分页。

**Filtering**：

- `filter_backends` —— 用于过滤查询集的过滤后端类列表。默认值与 `DEFAULT_FILTER_BACKENDS` 设置的值相同。

### 方法
**基本方法**：
`get_queryset(self)`
返回用于列表视图的查询集，并用于详细视图中查找的基础。默认返回 `queryset` 属性指定的查询集。

应该始终使用此方法，而不是直接访问 `self.queryset`，因为 `self.queryset` 只被评估一次，并且这些结果将被缓存用于所有后续请求。

可以重写来提供动态行为，例如返回查询集，这是针对请求的用户所特有的。

举个栗子：
```python
def get_queryset(self):
    user = self.request.user
    return user.accounts.all()
```
`get_object(self)`
返回用于详细视图的对象实例。默认使用 `lookup_field` 参数来过滤基本查询集。

可以重写以提供更复杂的行为，例如基于多个 URL kwarg 的对象查找。

举个栗子：
```python
def get_object(self):
    queryset = self.get_queryset()
    filter = {}
    for field in self.multiple_lookup_fields:
        filter[field] = self.kwargs[field]

    obj = get_object_or_404(queryset, **filter)
    self.check_object_permissions(self.request, obj)
    return obj
```
请注意，如果您的 API 不包含任何对象级权限，您可以选择性地排除 `self.check_object_permissions`，并简单地从 `get_object_or_404` 查找中返回对象。

`filter_queryset(self, queryset)`

给定一个查询集，使用任何后端过滤器进行过滤，返回一个新的查询集。

举个栗子：
```python
def filter_queryset(self, queryset):
    filter_backends = (CategoryFilter,)

    if 'geo_route' in self.request.query_params:
        filter_backends = (GeoRouteFilter, CategoryFilter)
    elif 'geo_point' in self.request.query_params:
        filter_backends = (GeoPointFilter, CategoryFilter)

    for backend in list(filter_backends):
        queryset = backend().filter_queryset(self.request, queryset, view=self)

    return queryset
```
`get_serializer_class(self)`

返回用于序列化的类。默认返回 `serializer_class` 属性。

可以重写以提供动态行为，例如使用不同的序列化器进行读写操作，或者为不同类型的用户提供不同的序列化器。

举个栗子：
```python
def get_serializer_class(self):
    if self.request.user.is_staff:
        return FullAccountSerializer
    return BasicAccountSerializer
```
**保存和删除钩子**：

以下方法由 `mixin` 类提供，可以很轻松的重写对象的保存和删除行为。

- `perform_create(self，serializer)` —— 在保存新对象实例时由 `CreateModelMixin`  调用。
- `perform_update(self，serializer)` —— 在保存现有对象实例时由 `UpdateModelMixin` 调用。
- `perform_destroy(self，instance)` —— 在删除对象实例时由 `DestroyModelMixin` 调用。

这些钩子特别适用于设置请求中隐含的属性，但不是请求数据的一部分。例如，您可以基于请求用户或基于 URL 关键字参数在对象上设置属性。
```python
def perform_create(self, serializer):
    serializer.save(user=self.request.user)
```
这些重写要点对于添加保存对象之前或之后发生的行为（如发送确认电子邮件或记录更新）也特别有用。
```python
def perform_update(self, serializer):
    instance = serializer.save()
    send_email_confirmation(user=self.request.user, modified=instance)
```
您还可以使用这些钩子通过引发 `ValidationError()` 来提供额外的验证。如果您需要在数据库保存时应用一些验证逻辑，这将非常有用。例如：
```python
def perform_create(self, serializer):
    queryset = SignupRequest.objects.filter(user=self.request.user)
    if queryset.exists():
        raise ValidationError('You have already signed up')
    serializer.save(user=self.request.user)
```
**注意**：这些方法替换了旧版本的 2.x `pre_save`，`post_save`，`pre_delete` 和 `post_delete` 方法，这些方法不再可用。

**其他方法**：

您通常不需要重写以下方法，但如果您使用 `GenericAPIView` 编写自定义视图，则可能需要调用它们。

- `get_serializer_context(self)` —— 返回包含应该被提供给序列化器的任何附加上下文的字典。默认包括 “request”，“view” 和 “format” 键。
- `get_serializer(self，instance = None，data = None，many = False，partial = False)` —— 返回一个序列化器实例。
- `get_paginated_response(self，data)` —— 返回分页样式的 `Response` 对象。
- `paginate_queryset(self, queryset)` —— 根据需要为查询集分页，或者返回一个页面对象；如果没有为该视图配置分页，则为 `None`。
- `filter_queryset(self, queryset)` —— 给定一个查询集，使用任何后端过滤器进行过滤，返回一个新的查询集。

***

# Mixins
mixin 类提供用于提供基本视图行为的操作。请注意，mixin 类提供了操作方法，而不是直接定义处理方法，如 `.get()` 和 `.post()`。这允许更灵活的行为组合。

mixin 类可以从 `rest_framework.mixins` 导入。

## ListModelMixin
提供 `.list(request，* args，** kwargs)` 方法，它实现列出查询集。

如果填充了查询集，则返回 `200 OK` 响应，并将查询集的序列化表示作为响应的主体。响应数据可以选择分页。

## CreateModelMixin
提供 `.create(request，* args，** kwargs)` 方法，用于实现创建和保存新模型实例。

如果对象被创建，则返回一个 `201 Created` 响应，并将该对象的序列化表示形式作为响应的主体。如果该表示包含名为 `url` 的键，则响应的 `Location` header 将填充该值。

如果为创建对象而提供的请求数据无效，则将返回 `400 Bad Request` 响应，并将错误详细信息作为响应的主体。

## RetrieveModelMixin
提供 `.retrieve(request，* args，** kwargs)` 方法，用于实现在响应中返回现有模型实例。

如果可以检索对象，则返回一个 `200 OK` 响应，并将对象的序列化表示作为响应的主体。否则它将返回 `404 Not Found`。

## UpdateModelMixin
提供 `.update(request，* args，** kwargs)` 方法，用于实现更新和保存现有模型实例。

还提供了 `.partial_update(request，* args，** kwargs)` 方法，该方法与 `update` 方法类似，不同之处在于更新的所有字段都是可选的。这允许支持 HTTP `PATCH` 请求。

如果对象被更新，则返回 `200 OK` 响应，并将对象的序列化表示作为响应的主体。

如果为更新对象而提供的请求数据无效，则将返回 `400 Bad Request` 响应，并将错误详细信息作为响应的主体。

## DestroyModelMixin
提供 `.destroy(request，* args，** kwargs)` 方法，用于实现对现有模型实例的删除。

如果对象被删除，则返回 `204 No Content`，否则它将返回 `404 Not Found`。
***

# 具体视图类 (Concrete View Classes)
以下类是具体的通用视图。如果你使用通用视图，这通常是你要工作的级别，除非你需要大量定制的行为。

可以从 `rest_framework.generics` 导入视图类。

## CreateAPIView
用于**仅创建**端点。

提供 `post` 方法处理程序。

扩展：[GenericAPIView](http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview), [CreateModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#createmodelmixin)

## ListAPIView
用**只读**端点表示**模型实例的集合**。

提供 `get` 方法处理程序。

扩展：[GenericAPIView](http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview), [ListModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#listmodelmixin)

## RetrieveAPIView
用**只读**端点表示**单个模型实例**。

提供 `get` 方法处理程序。

扩展：[GenericAPIView](http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview), [RetrieveModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#retrievemodelmixin)

## DestroyAPIView
用于**仅删除单个模型实例**的端点。

提供 `delete` 方法处理程序。

扩展：[GenericAPIView](http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview), [DestroyModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#destroymodelmixin)

## UpdateAPIView
用于**只更新单个模型实例**的端点。

提供 `put` 和 `patch` 方法处理程序。

扩展：[GenericAPIView](http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview), [UpdateModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#updatemodelmixin)

## ListCreateAPIView
用**读写**端点来表示**模型实例的集合**。

提供 `get` 和 `post` 方法处理程序。

扩展：[GenericAPIView](http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview), [ListModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#listmodelmixin), [CreateModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#createmodelmixin)

## RetrieveUpdateAPIView
用于**读取或更新**端点以表示**单个模型实例**。

提供 `get`，`put` 和 `patch` 方法处理程序。

扩展：[GenericAPIView](http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview), [RetrieveModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#retrievemodelmixin), [UpdateModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#updatemodelmixin)

## RetrieveDestroyAPIView
用于**读取或删除**端点以表示**单个模型实例**。

提供 `get` 和 `delete` 方法处理程序。

扩展：[GenericAPIView](http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview), [RetrieveModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#retrievemodelmixin), [DestroyModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#destroymodelmixin)

## RetrieveUpdateDestroyAPIView
用于**读写删除**端点以表示**单个模型实例**。

提供 `get` ，`put`，`patch` 和 `delete` 方法处理程序。

扩展：[GenericAPIView](http://www.django-rest-framework.org/api-guide/generic-views/#genericapiview), [RetrieveModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#retrievemodelmixin), [UpdateModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#updatemodelmixin), [DestroyModelMixin](http://www.django-rest-framework.org/api-guide/generic-views/#destroymodelmixin)

***

# 自定义通用视图 (Customizing the generic views)
通常您希望使用现有的通用视图，但使用一些稍微定制的行为。如果您发现自己在多个位置重复使用某些自定义行为，则可能需要将该行为重构为公共类，然后可以根据需要将其应用于任何视图或视图集。

## 创建自定义 mixins (Creating custom mixins)
例如，如果您需要根据 URL conf 中的多个字段查找对象，则可以创建如下所示的 mixin 类：
```python
class MultipleFieldLookupMixin(object):
    """
    将此 mixin 应用于任何视图或视图集，以基于`lookup_fields`属性获得多个字段过滤，而不是默认的单字段过滤。
    """
    def get_object(self):
        queryset = self.get_queryset()             # 获取基本查询集
        queryset = self.filter_queryset(queryset)  # 应用过滤器
        filter = {}
        for field in self.lookup_fields:
            if self.kwargs[field]: # 忽略空字段
                filter[field] = self.kwargs[field]
        obj = get_object_or_404(queryset, **filter)  # 查找对象
        self.check_object_permissions(self.request, obj)
        return obj
```
然后可以在需要应用自定义行为的任​​何时候，将该 mixin 应用于视图或视图集。
```python
class RetrieveUserView(MultipleFieldLookupMixin, generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    lookup_fields = ('account', 'username')
```
如果您需要使用自定义行为，则使用自定义 mixins 是一个不错的选择。

## 创建自定义基类 (Creating custom base classes)
如果您在多个视图中使用 mixin，则可以更进一步，创建自己的一组基本视图，然后可以在整个项目中使用。例如：
```python
class BaseRetrieveView(MultipleFieldLookupMixin,
                       generics.RetrieveAPIView):
    pass

class BaseRetrieveUpdateDestroyView(MultipleFieldLookupMixin,
                                    generics.RetrieveUpdateDestroyAPIView):
    pass
```
如果您的自定义行为始终需要在整个项目中的大量视图中重复，那么使用自定义基类是一个不错的选择。

***

# PUT as create
在版本 3.0 之前，REST framework mixins 将 `PUT` 视为更新或创建操作，这取决于对象是否已经存在。

允许 `PUT` 作为创建操作是有问题的，因为它必然暴露有关对象存在或不存在的信息。同样不明显的是，透明地允许重新创建以前删除的实例，这必然是比简单地返回 `404` 响应更好的默认行为。

两种样式 “`PUT` as 404” 和 “`PUT` as create” 在不同情况下都可以有效，但从版本 3.0 开始，我们现在使用 `404` 行为作为默认值，因为它更简单，更明显。

如果您需要通用的 `PUT-as-create` 行为，您可能希望将类似此类的 [`AllowPUTAsCreateMixin`](https://gist.github.com/tomchristie/a2ace4577eff2c603b1b) 类作为 mixin 包含在您的视图中。

***

# 第三方包
以下第三方包提供了其他通用视图实现。

## Django REST Framework bulk
[django-rest- frameworkbulk 包](https://github.com/miki725/django-rest-framework-bulk)实现了通用的视图混合，以及一些通用的具体视图，允许通过 API 请求应用批量操作。

## Django Rest Multiple Models
[Django Rest Multiple Models](https://github.com/MattBroach/DjangoRestMultipleModels) 提供了一个通用视图 (和 mixin)，用于通过单个 API 请求发送多个序列化模型和/或查询集。
