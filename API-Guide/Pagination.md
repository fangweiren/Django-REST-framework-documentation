# 分页 (Pagination)
Django 提供了一些类来帮助您管理分页数据，也就是说，这些数据通过 “前/下” 链接被分割到多个页面。—— [Django 文档](https://docs.djangoproject.com/en/stable/topics/pagination/)

REST framework 包含对自定义分页样式的支持。这允许您修改如何将大的结果集分成单个数据页面。

分页 API 支持：

- 分页链接是作为响应内容的一部分提供的。
- 分页链接被包含在响应标头中，例如 `Content-Range` 或 `Link`。

内置样式目前都使用被包含作为响应内容的一部分的链接。使用可浏览 API 时，此样式更易于访问。

只有在使用通用视图或视图集时，才会自动执行分页。如果您使用常规 `APIView`，则需要自己调用分页 API 以确保返回分页响应。有关示例，请参阅 `mixins.ListModelMixin` 和 `generics.GenericAPIView` 类的源代码。

可以通过将分页类设置为 `None` 来关闭分页。

## 设置分页样式 (Setting the pagination style)
可以使用 `DEFAULT_PAGINATION_CLASS` 和 `PAGE_SIZE` 设置键全局设置分页样式。例如，要使用内置的限制/偏移分页，您可以这样做：
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 100
}
```

请注意，您需要同时设置分页类和应使用的页面大小。默认情况下，`DEFAULT_PAGINATION_CLASS` 和 `PAGE_SIZE` 都是 `None`。

您还可以通过使用 `pagination_class` 属性在单个视图上设置分页类。通常，您希望在整个 API 中使用相同的分页样式，尽管您可能希望在每个视图的基础上更改分页的各个方面，例如默认或最大页大小。

## 修改分页样式 (Modifying the pagination style)
如果您想修改分页样式的特定方面，您需要重写其中一个分页类，并设置要更改的属性。
```python
class LargeResultsSetPagination(PageNumberPagination):
    page_size = 1000
    page_size_query_param = 'page_size'
    max_page_size = 10000

class StandardResultsSetPagination(PageNumberPagination):
    page_size = 100
    page_size_query_param = 'page_size'
    max_page_size = 1000
```

然后，您可以使用 `.pagination_class` 属性将新样式应用到视图：
```python
class BillingRecordsView(generics.ListAPIView):
    queryset = Billing.objects.all()
    serializer_class = BillingRecordsSerializer
    pagination_class = LargeResultsSetPagination
```

或者使用 `DEFAULT_PAGINATION_CLASS` 设置键全局应用样式。例如：
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'apps.core.pagination.StandardResultsSetPagination'
}
```

***

# API 参考 (API Reference)
## PageNumberPagination
此分页样式在请求查询参数中接受单个数字页码。

**Request**：
```python
GET https://api.example.org/accounts/?page=4
```

**Response**：
```python
HTTP 200 OK
{
    "count": 1023
    "next": "https://api.example.org/accounts/?page=5",
    "previous": "https://api.example.org/accounts/?page=3",
    "results": [
       …
    ]
}
```

#### 设置 (Setup)
要全局启用 `PageNumberPagination` 样式，请使用以下配置，并按要求设置 `PAGE_SIZE`：
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 100
}
```

在 `GenericAPIView` 子类上，您还可以设置 `pagination_class` 属性以便在每个视图的基础上选择 `PageNumberPagination`。

#### 配置 (Configuration)
`PageNumberPagination` 类包含许多可以重写以修改分页样式的属性。

要设置这些属性，您应该重写 `PageNumberPagination` 类，然后启用上面的自定义分页类。

- `django_paginator_class` - 要使用的 Django Paginator 类。默认是 `django.core.paginator.Paginator`，对于大多数用例应该没问题。
- `page_size` - 表示页面大小的数值。如果设置，则会覆盖 `PAGE_SIZE` 设置。默认值与 `PAGE_SIZE` 设置键相同。
- `page_query_param` - 指示要用于分页控件的查询参数的名称的字符串值。
- `page_size_query_param` - 如果设置，则这是指示允许客户端基于每个请求设置页面大小的查询参数的名称的字符串值。默认为 `None`，表示客户端可能无法控制所请求的页面大小。
- `max_page_size` - 如果设置，则这是指示允许的最大请求页面大小的数值。该属性仅在 `page_size_query_param` 也被设置时有效。
- `last_page_strings` - 字符串值得列表或元组，指示可与 `page_query_param` 一起使用的值，用以请求集合中的最终页面。默认为 `('last',)`
- `template` - 在可浏览 API 中渲染分页控件时使用的模板的名称。可能会被覆盖以修改渲染样式，或设置为 `None` 以完全禁用 HTML 分页控件。默认为 `"rest_framework/pagination/numbers.html"`。

***

## LimitOffsetPagination
这种分页样式反映了查找多个数据库记录时使用的语法。客户端包括 “limit” 和 “offset” 查询参数。limit 表示要返回的最大项数，相当于其他样式中的 `page_size`。offset 表示查询相对于整套未标记项的起始位置。

**Request**：
```python
GET https://api.example.org/accounts/?limit=100&offset=400
```

**Response**：
```python
HTTP 200 OK
{
    "count": 1023
    "next": "https://api.example.org/accounts/?limit=100&offset=500",
    "previous": "https://api.example.org/accounts/?limit=100&offset=300",
    "results": [
       …
    ]
}
```

#### 设置 (Setup)
要全局启用 `LimitOffsetPagination` 样式，请使用以下配置：
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination'
}
```

或者，您也可以设置 `PAGE_SIZE` 键。如果还使用 `PAGE_SIZE` 参数，则 `limit` 查询参数将是可选的，并且可被客户端省略。

在 `GenericAPIView` 子类上，您还可以设置 `pagination_class` 属性以便在每个视图的基础上选择 `LimitOffsetPagination`。

#### 配置 (Configuration)
`LimitOffsetPagination` 类包含许多可以重写以修改分页样式的属性。

要设置这些属性，您应该重写 `LimitOffsetPagination` 类，然后启用上面的自定义分页类。

- `default_limit` - 如果客户端在查询参数中未提供一个值，则指示使用限制的数值。默认与 `PAGE_SIZE` 设置键相同的值。
- `limit_query_param` - 指示 “limit” 查询参数的名称的字符串值。默认为 `'limit'`。
- `offset_query_param` - 指示 “offset” 查询参数的名称的字符串值。默认为 `'offset'`。
- `max_limit` - 如果设置，则这是指示客户端可能请求的最大允许限度的数值。默认为 `None`。
- `template` - 在可浏览 API 中渲染分页控件时使用的模板的名称。可能会被覆盖以修改渲染样式，或设置为 `None` 以完全禁用 HTML 分页控件。默认为 `"rest_framework/pagination/numbers.html"`。

***

## CursorPagination
基于游标的分页提供了一个不透明的 “游标” 指示器，客户端可以使用该指示器来翻阅结果集。这种分页样式只提供正向和反向控件，并且不允许客户端导航到任意位置。

基于游标的分页要求结果集中的项具有唯一且不变的排序。这种排序通常是记录上的创建时间戳，因为这提供了排序的一致性。

基于游标的分页比其他方案更复杂。它还要求结果集提供固定顺序，并且不允许客户端任意索引到结果集。然而，它确实提供了以下好处：

- 提供一致的分页视图。正确使用 `CursorPagination` 可确保客户端在分页记录时永远不会看到相同的项两次，即使在分页过程中其他客户端插入新项。
- 支持使用非常大的数据集。对于极大的数据集，使用基于偏移的分页样式的分页可能变得低效或无法使用。基于游标的分页方案具有固定时间属性，并且不会随着数据集大小的增加而减慢。

#### 细节和限制 (Details and limitations)
正确使用基于游标的分页需要稍微注意细节。您需要考虑希望该方案应用什么排序。默认是按 `"-created"` 排序。这假设在模型实例上**必须有一个 'created' 时间戳字段**，并且将显示 “时间轴” 样式的分页视图，其中最近添加的项是第一个。

您可以通过覆盖分页类的 `'ordering'` 属性，或者将 `OrderingFilter` 过滤器类与 `CursorPagination` 一起使用来修改排序。当与 `OrderingFilter` 一起使用时，您应该强烈考虑限制用户可能排序的字段。

正确使用游标分页应该有一个满足以下条件的排序字段：

- 应该是一个不变的值，例如时间戳、slug 或其他仅在创建时设置一次的字段。
- 应该是唯一的，或者几乎是唯一的。毫秒精度时间戳就是一个很好的例子。游标分页的这种实现使用了智能的 “位置 + 偏移” 样式，允许它正确地支持非严格唯一的值作为排序。
- 应该是可以强制为字符串的非空值。
- 不应该是浮点数。精度误差容易导致不正确的结果。提示：使用小数代替。(如果你已经有了一个浮点字段，并且必须对它进行分页，那么[这里能找到使用小数来限制精度的 `CurrPosiPrimes` 子类示例](https://gist.github.com/keturn/8bc88525a183fd41c73ffb729b8865be#file-fpcursorpagination-py)。)
- 该字段应具有数据库索引。

使用不满足这些约束的排序字段通常仍然有效，但是您将失去游标分页的一些好处。

关于我们用于游标分页的实现的更多技术细节，“[为 Disqus API 构建游标](http://cramer.io/2011/03/08/building-cursors-for-the-disqus-api)” 博客文章对基本方法进行了很好的概述。

#### 设置 (Setup)
要全局启用 `CursorPagination` 样式，请使用以下配置，按要求修改 `PAGE_SIZE`：
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.CursorPagination',
    'PAGE_SIZE': 100
}
```

在 `GenericAPIView` 子类上，您还可以设置 `pagination_class` 属性以便在每个视图的基础上选择 `CursorPagination`。

#### 配置 (Configuration)
`CursorPagination` 类包含许多可以重写以修改分页样式的属性。

要设置这些属性，您应该重写 `CursorPagination` 类，然后启用上面的自定义分页类。

- `page_size` = 指示页面大小的数值。如果设置，则会覆盖 `PAGE_SIZE` 设置。默认与 `PAGE_SIZE` 设置键相同的值。
- `cursor_query_param` = 指示 “cursor” 查询参数的名称的字符串值。默认为 `'cursor'`。
- `ordering` = 这应该是一个字符串或字符串列表，指示将应用基于游标的分页的字段。例如： `ordering = 'slug'`。默认为 `-created`。该值还可以通过在视图上使用 `OrderingFilter` 来覆盖。
- `template` = 在可浏览 API 中渲染分页控件时使用的模板的名称。可能会被覆盖以修改渲染样式，或设置为 `None` 以完全禁用 HTML 分页控件。默认为 `"rest_framework/pagination/previous_and_next.html"`。

***

# 自定义分页样式 (Custom pagination styles)
要创建自定义分页序列化类，您应该子类化 `pagination.BasePagination` 并重写 `paginate_queryset(self, queryset, request, view=None)` 和 `get_paginated_response(self, data)` 方法：

- `paginate_queryset` 方法传递初始查询集，并且应返回一个只包含请求页面中的数据的可迭代对象。
- `get_paginated_response` 方法传递序列化的页面数据，并返回 `Response` 实例。

请注意，`paginate_queryset` 方法可以在分页实例上设置状态，而后 `get_paginated_response` 方法可以使用它。

## 举个栗子
假设我们想用一个修改后的格式替换默认分页输出样式，该格式包含嵌套 “links” 键下的下一个和上一个链接。我们可以像这样指定自定义分页类：
```python
class CustomPagination(pagination.PageNumberPagination):
    def get_paginated_response(self, data):
        return Response({
            'links': {
                'next': self.get_next_link(),
                'previous': self.get_previous_link()
            },
            'count': self.page.paginator.count,
            'results': data
        })
```

然后我们需要在配置中设置自定义类：
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'my_project.apps.core.pagination.CustomPagination',
    'PAGE_SIZE': 100
}
```

请注意，如果您关心如何在可浏览 API 的响应中显示键的顺序，则可以在构造分页响应的主体时选择使用 `OrderedDict`，但这是可选的。

## 使用您的自定义分页类 (Using your custom pagination class)
要默认使用您的自定义分页类，请使用 `DEFAULT_PAGINATION_CLASS` 设置：
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'my_project.apps.core.pagination.LinkHeaderPagination',
    'PAGE_SIZE': 100
}
```

列表端点的 API 响应现在将包含一个 `Link` 标头，而不是将分页链接作为响应主体的一部分包括在内，例如：

## 分页和模式 (Pagination & schemas)
您还可以通过实现 `get_schema_fields()` 方法使分页控件可用于 REST framework 提供的模式自动生成。此方法应具有以下签名：

`get_schema_fields(self, view)`

该方法应返回 `coreapi.Field` 实例的列表。

***

![f_14127127.png](http://pan.16mb.com/data/f_14127127.png)

*使用 'Link header' 的自定义分页样式*
***

# HTML 分页控件 (HTML pagination controls)
默认情况下，使用分页类将导致 HTML 分页控件显示在可浏览的 API 中。有两种内置显示样式。`PageNumberPagination` 和 `LimitOffsetPagination` 类显示带有上一个和下一个控件的页码列表。`CursorPagination` 类显示更简单的样式，仅显示上一个和下一个控件。

## 自定义控件 (Customizing the controls)
您可以覆盖渲染 HTML 分页控件的模板。这两种内置式样是：

- `rest_framework/pagination/numbers.html`
- `rest_framework/pagination/previous_and_next.html`

在全局模板目录中提供这些路径中的任一个模板将重写相关分页类的默认渲染。

或者，您可以通过继承现有类来完全禁用 HTML 分页控件，将 `template=None` 设置为类的属性。然后，您需要配置 `DEFAULT_PAGINATION_CLASS` 设置键，以将您的自定义类用作默认分页样式。

#### 低级 API (Low-level API)
用于确定分页类是否应显示控件的低级 API 在分页实例上公开为 `display_page_controls` 属性。如果需要显示 HTML 分页控件，则应在 `paginate_queryset` 方法中将自定义分页类设置为 `True`。

`.to_html()` 和 `.get_html_context()` 方法也可以在自定义分页类中重写，以便进一步自定义控件的渲染方式。
***

# 第三方包 (Third party packages)
以下是可用的第三方包。

## DRF-extensions
[`DRF-extensions` 包](https://chibisov.github.io/drf-extensions/docs/)包括 [`PaginateByMax` 混合类](https://chibisov.github.io/drf-extensions/docs/#paginatebymaxmixin)，它允许您的 API 客户端指定 `?page_size=max` 以获得允许的最大页面大小。

## drf-proxy-pagination
[`drf-proxy-pagination` 包](https://github.com/tuffnatty/drf-proxy-pagination)中包含 `ProxyPagination` 类，允许选择带有查询参数的分页类。

## link-header-pagination
[`django-rest-framework-link-header-pagination` 包](https://github.com/tbeadle/django-rest-framework-link-header-pagination)中包含 `LinkHeaderPagination` 类，它通过 [Github 开发者文档](http://www.django-rest-framework.org/api-guide/github-link-pagination)中描述的 HTTP `Link` 标头提供分页。
