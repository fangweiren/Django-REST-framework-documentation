# 过滤 (Filtering)
Manager 提供的根 QuerySet 描述了数据库表中的所有对象。但是，通常您只需要选择完整对象集的一个子集。—— [Django 文档](https://docs.djangoproject.com/en/stable/topics/db/queries/#retrieving-specific-objects-with-filters)

REST framework 的通用列表视图的默认行为是返回模型管理器的整个查询集。通常您会希望 API 限制查询集返回的条目。

过滤子类 `GenericAPIView` 的任何视图的查询集的最简单方法是重写 `.get_queryset()` 方法。

重写此方法允许您以多种不同方式自定义视图返回的查询集。

## 过滤当前用户 (Filtering against the current user)
您可能希望过滤查询集以确保只返回与当前经过身份验证的用户发出的请求相关的结果。

您可以通过基于 `request.user` 的值进行过滤来完成此操作。

举个栗子：
```python
from myapp.models import Purchase
from myapp.serializers import PurchaseSerializer
from rest_framework import generics

class PurchaseList(generics.ListAPIView):
    serializer_class = PurchaseSerializer

    def get_queryset(self):
        """
        此视图应返回当前已验证用户的所有 purchases 列表。
        """
        user = self.request.user
        return Purchase.objects.filter(purchaser=user)
```

## 过滤 URL (Filtering against the URL)
另一种过滤方式可能涉及限制基于 URL 某些部分的查询集。

例如，如果 URL 配置包含这样的条目：
```python
url('^purchases/(?P<username>.+)/$', PurchaseList.as_view()),
```
然后，您可以编写一个视图，该视图返回由 URL 的用户名部分过滤的 purchase 查询集：
```python
class PurchaseList(generics.ListAPIView):
    serializer_class = PurchaseSerializer

    def get_queryset(self):
        """
        这个视图应该返回由 URL 的用户名部分确定的用户的所有 purchases 列表。
        """
        username = self.kwargs['username']
        return Purchase.objects.filter(purchaser__username=username)
```

## 过滤查询参数 (Filtering against query parameters)
过滤初始查询集的最后一个示例是根据 url 中的查询参数确定初始查询集。

我们可以重写 `.get_queryset()` 来处理诸如 `http://example.com/api/purchases?username=denvercoder9` 的 URL，并且只有在 URL 中包含 `username` 参数时才过滤查询集：
```python
class PurchaseList(generics.ListAPIView):
    serializer_class = PurchaseSerializer

    def get_queryset(self):
        """
        通过过滤 URL 中的 `username` 查询参数，可以选择将返回的 purchases 限制为给定的用户。
        """
        queryset = Purchase.objects.all()
        username = self.request.query_params.get('username', None)
        if username is not None:
            queryset = queryset.filter(purchaser__username=username)
        return queryset
```
***

# 通用过滤 (Generic Filtering)
除了能够重写默认的查询集外，REST framework 还包括对通用过滤后端的支​​持，允许您轻松地构建复杂的搜索和过滤器。

通用过滤器还可以在可浏览的 API 和管理 API 中将自身显示为 HTML 控件。

![filter-controls.png](http://pan.16mb.com/data/f_85463608.png)

## 设置过滤器后端 (Setting filter backends)
可以使用 `DEFAULT_FILTER_BACKENDS` setting 全局设置默认的过滤器后端。例如：
```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ('django_filters.rest_framework.DjangoFilterBackend',)
}
```
您还可以使用基于类的视图 (`GenericAPIView`)，在每个视图或每个视图集的基础上设置过滤器后端。
```python
import django_filters.rest_framework
from django.contrib.auth.models import User
from myapp.serializers import UserSerializer
from rest_framework import generics

class UserListView(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    filter_backends = (django_filters.rest_framework.DjangoFilterBackend,)
```

## 过滤和对象查找 (Filtering and object lookups)
请注意，如果为视图配置了过滤器后端，那么除了用于过滤列表视图之外，它还将用于过滤用于返回单个对象的查询集。

例如，给定前面的示例以及 ID 为 4675 的产品，以下 URL 将返回相应的对象，或返回 404 响应，具体取决于给定的产品实例是否满足过滤条件：
```python
http://example.com/api/products/4675/?category=clothing&max_price=10.00
```

## 重写初始查询集 (Overriding the initial queryset)
请注意，您可以同时使用重写的 `.get_queryset()` 和通用过滤，并且一切都将按预期工作。例如，如果 `Product` 与 `User` (名为 `purchase`) 有多对多的关系，则您可能需要编写像这样的视图：
```python
class PurchasedProductsList(generics.ListAPIView):
    """
    Return a list of all the products that the authenticated
    user has ever purchased, with optional filtering.
    """
    model = Product
    serializer_class = ProductSerializer
    filter_class = ProductFilter

    def get_queryset(self):
        user = self.request.user
        return user.purchase_set.all()
```
***

# API 指南 (API Guide)
## DjangoFilterBackend
`django-filter` 库包含 `DjangoFilterBackend` 类，它支持 REST framework 高度自定义字段过滤。

要使用 `DjangoFilterBackend`，首先安装 `django-filter`。然后将 `django_filters` 添加到 Django 的 `INSTALLED_APPS` 中
```python
pip install django-filter
```
您现在应该将过滤器后端添加到您的设置：
```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ('django_filters.rest_framework.DjangoFilterBackend',)
}
```
或者将过滤器后端添加到单个视图或视图集。
```python
from django_filters.rest_framework import DjangoFilterBackend

class UserListView(generics.ListAPIView):
    ...
    filter_backends = (DjangoFilterBackend,)
```
如果您需要的是简单的基于等式的过滤，则可以在视图或视图集上设置 `filter_fields` 属性，列出您想要过滤的一组字段。
```python
class ProductList(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = (DjangoFilterBackend,)
    filter_fields = ('category', 'in_stock')
```
这将自动为给定字段创建一个 `FilterSet` 类，并允许您发出如下请求：
```python
http://example.com/api/products?category=clothing&in_stock=True
```
对于更高级的过滤要求，您可以指定视图应使用的 `FilterSet` 类。您可以在 [django-filter 文档](https://django-filter.readthedocs.io/en/latest/index.html)中阅读有关 `FilterSet` 的更多信息。还建议您阅读有关 [DRF integration](https://django-filter.readthedocs.io/en/latest/guide/rest_framework.html) 的部分。

## SearchFilter
`SearchFilter` 类支持简单的基于单个查询参数的搜索，并且基于 [Django 管理员的搜索功能](https://docs.djangoproject.com/en/stable/ref/contrib/admin/#django.contrib.admin.ModelAdmin.search_fields)。

在使用时，可浏览的 API 将包含 `SearchFilter` 控件：

![search-filter.png](http://pan.16mb.com/data/f_19520707.png)

仅当视图具有 `search_fields` 属性集时，才会应用 `SearchFilter` 类。`search_fields` 属性应该是模型上文本类型字段的名称列表，例如 `CharField` 或 `TextField`。
```python
class UserListView(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    filter_backends = (filters.SearchFilter,)
    search_fields = ('username', 'email')
```
这将允许客户端通过查询来过滤列表中的项，例如：
```python
http://example.com/api/users?search=russell
```
您还可以使用查找 API 双下划线表示法对 ForeignKey 或 ManyToManyField 执行相关查找：
```python
search_fields = ('username', 'email', 'profile__profession')
```
默认情况下，搜索将使用不区分大小写的部分匹配。搜索参数可能包含多个搜索项，它们应该是空格和/或逗号分隔的。如果使用多个搜索项，则只有在所有提供的项匹配的情况下，对象才会返回到列表中。

可以通过在 `search_fields` 之前添加各种字符来限制搜索行为。

- '^' 开始搜索。
- '=' 完全匹配。
- '@' 全文搜索。(目前只支持 Django 的 MySQL 后端。)
- '$' 正则表达式搜索。

举个栗子：
```python
search_fields = ('=username', '=email')
```
默认情况下，搜索参数被命名为 `'search'`，但这可能会被 `SEARCH_PARAM` 设置覆盖。

有关更多详细信息，请参阅 [Django 文档](https://docs.djangoproject.com/en/stable/ref/contrib/admin/#django.contrib.admin.ModelAdmin.search_fields)。
***

## OrderingFilter
`OrderingFilter` 类支持简单查询参数控制结果的排序。

![f_52271811.png](http://pan.16mb.com/data/f_52271811.png)

默认情况下，查询参数被命名为 `'ordering'`，但这可能会被 `ORDERING_PARAM` 设置覆盖。

例如，要按 username 对用户排序：
```python
http://example.com/api/users?ordering=username
```
客户端还可以通过在字段名称前加上 '-' 来指定反向排序，如下所示：
```python
http://example.com/api/users?ordering=-username
```
还可以指定多个排序：
```python
http://example.com/api/users?ordering=account,username
```

### 指定可以对哪些字段进行排序 (Specifying which fields may be ordered against)
建议您显式指定 API 应该允许在排序过滤器中使用哪些字段。您可以通过在视图上设置 `ordering_fields` 属性来完成此操作，如下所示：
```python
class UserListView(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    filter_backends = (filters.OrderingFilter,)
    ordering_fields = ('username', 'email')
```
这有助于防止意外的数据泄漏，例如允许用户对密码哈希字段或其他敏感数据进行排序。

如果您在视图上未指定 `ordering_fields` 属性，则过滤器类将默认允许用户过滤由 `serializer_class` 属性指定的序列化器中的任何可读字段。

如果您确信视图使用的查询集不包含任何敏感数据，则还可以通过使用特殊值 `'__all__'` 显式指定允许在任何模型字段或查询集聚合上进行排序的视图。
```python
class BookingsListView(generics.ListAPIView):
    queryset = Booking.objects.all()
    serializer_class = BookingSerializer
    filter_backends = (filters.OrderingFilter,)
    ordering_fields = '__all__'
```

### 指定默认排序 (Specifying a default ordering)
如果在视图上设置了 `ordering` 属性，则将用作默认排序。

通常，您通过在初始查询集上设置 `order_by` 来控制，但是使用视图上的 `ordering` 参数允许您以某种方式指定顺序，然后可以将其作为上下文自动传递到渲染的模板。如果它们被用于排序结果，这使得自动渲染不同的列标题成为可能。
```python
class UserListView(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    filter_backends = (filters.OrderingFilter,)
    ordering_fields = ('username', 'email')
    ordering = ('username',)
```
`ordering` 属性可以是字符串或者字符串列表/元组。
***

## DjangoObjectPermissionsFilter
`DjangoObjectPermissionsFilter` 旨在与 `django-guardian` 软件包一起使用，添加了自定义 `'view'` 的权限。过滤器将确保查询集仅返回用户具有适当查看权限的对象。

如果您正在使用的是 `DjangoObjectPermissionsFilter`，那么您可能还需要添加适当的对象权限类，以确保用户只有在具有适当对象权限的情况下才能对实例进行操作。做到这一点的最简单方法是子类化 `DjangoObjectPermissions` 并为  `perms_map` 属性添加 `'view'` 权限。

使用 `DjangoObjectPermissionsFilter` 和 `DjangoObjectPermissions` 的完整示例可能如下所示。

**permissions.py**：
```python
class CustomObjectPermissions(permissions.DjangoObjectPermissions):
    """
    Similar to `DjangoObjectPermissions`, but adding 'view' permissions.
    """
    perms_map = {
        'GET': ['%(app_label)s.view_%(model_name)s'],
        'OPTIONS': ['%(app_label)s.view_%(model_name)s'],
        'HEAD': ['%(app_label)s.view_%(model_name)s'],
        'POST': ['%(app_label)s.add_%(model_name)s'],
        'PUT': ['%(app_label)s.change_%(model_name)s'],
        'PATCH': ['%(app_label)s.change_%(model_name)s'],
        'DELETE': ['%(app_label)s.delete_%(model_name)s'],
    }
```
**views.py**：
```python
class EventViewSet(viewsets.ModelViewSet):
    """
    如果用户有 'view' 权限，视图集只列出事件，如果用户有适当的 'view', 'add', 'change' or 'delete' 权限，则只允许对单个事件进行操作。
    """
    queryset = Event.objects.all()
    serializer_class = EventSerializer
    filter_backends = (filters.DjangoObjectPermissionsFilter,)
    permission_classes = (myapp.permissions.CustomObjectPermissions,)
```
有关为模型添加 `'view'` 权限的更多信息，请参阅 `django-guardian` 文档的[相关部分](https://django-guardian.readthedocs.io/en/latest/userguide/assign.html)和[此博客文章](https://blog.nyaruka.com/adding-a-view-permission-to-django-models)。
***

# 自定义通用过滤 (Custom generic filtering)
您还可以提供自己的通用过滤后端，或编写可安装的应用程序供其他开发人员使用。

为此，请重写 `BaseFilterBackend`，并重写 `.filter_queryset(self, request, queryset, view)` 方法。该方法应返回一个新的过滤查询集。

除了允许客户端执行搜索和过滤外，通用过滤器后端对于限制哪些对象对任何给定的请求或用户可见是有用的。

## 举个栗子
例如，您可能需要将用户限制为只能看到他们创建的对象。
```python
class IsOwnerFilterBackend(filters.BaseFilterBackend):
    """
    只允许用户查看自己的对象的筛选器。
    """
    def filter_queryset(self, request, queryset, view):
        return queryset.filter(owner=request.user)
```
我们可以通过重写视图上的 `get_queryset()` 来实现相同的行为，但使用过滤器后端允许您更轻松地将此限制添加到多个视图，或者将其应用于整个 API。

## 自定义界面 (Customizing the interface)
通用过滤器还可以在可浏览的 API 中呈现接口。为此，您应该实现 `to_html()` 方法来返回过滤器的渲染 HTML 表示。此方法应具有以下签名：

`to_html(self, request, queryset, view)`

该方法应返回渲染的 HTML 字符串。

## 分页和模式 (Pagination & schemas)
您还可以通过实现 `get_schema_fields()` 方法，使过滤器控件可用于 REST framework 提供的模式自动生成。此方法应具有以下签名：

`get_schema_fields(self, view)`

该方法应该返回一个 `coreapi.Field` 实例列表。

# 第三方包 (Third party packages)
以下第三方包提供了额外的过滤器实现。

## Django REST framework filters package
[django-rest-framework-filters 包](https://github.com/philipn/django-rest-framework-filters)与 `DjangoFilterBackend` 类一起工作，并允许您轻松地创建跨关系的过滤器，或者为给定字段创建多个过滤器查找类型。

## Django REST framework full word search filter
[djangorestframework-word-filter](https://github.com/trollknurr/django-rest-framework-word-search-filter) 是作为 `filters.SearchFilter` 的替代品开发的，将在文本中搜索全文，或者完全匹配。

## Django URL Filter
[django-url-filter](https://github.com/miki725/django-url-filter) 提供了一种通过人性化 URL 过滤数据的安全方法。它的工作原理与 DRF 序列化器和字段非常类似，在某种意义上，它们可以嵌套，除非它们被称为过滤器集和过滤器。这提供了过滤相关数据的简便方法。此库也是通用的，因此它可用于过滤其他数据源，而不仅仅是Django `QuerySets`。

## drf-url-filters
[drf-url-filter](https://github.com/manjitkumar/drf-url-filters) 是一个简单的 Django 应用程序，它以简洁，可配置的方式在 drf `ModelViewSet` 的 `Queryset` 上应用过滤器。它还支持对传入查询参数及其值的验证。一个漂亮的 python 包 `Voluptuous` 被用于对传入的查询参数进行验证。voluptuous 的最好部分是可以根据查询参数要求定义自己的验证。
