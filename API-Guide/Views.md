# 基于类的视图
Django 的基于类的视图是一个受欢迎的背离旧风格的视图。 —— Reinout van Rees

REST framework 提供了一个 `APIView` 类，它是 Django 的 `View` 类的子类。

`APIView` 类与常规 `View` 类不同，有以下几种方法：

- 传递给处理程序方法的请求将是 REST framework 的 `Request` 实例，而不是 Django 的 `HttpRequest` 实例。
- 处理程序方法可能会返回 REST framework 的 `Response`，而不是 Django 的 `HttpResponse`。该视图将管理内容协商并在响应中设置正确的渲染器。
- 任何 `APIException` 异常都会被捕获并调解为适当的响应。
- 传入的请求将被认证，并且在将请求分派给处理程序方法之前将运行适当的的权限和/或限流检查。

使用 `APIView` 类与使用常规 `View` 类几乎是一样的，像往常一样，传入的请求被分派到适当的处理程序方法，如 `.get()` 或 `.post()` 。另外，可以在控制 API 策略的各个方面的类上设置多个属性。

例如：
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import authentication, permissions
from django.contrib.auth.models import User

class ListUsers(APIView):
    """
    查看以列出系统中的所有用户。

    * 需要令牌认证。
    * 只有管理员用户才能访问此视图。
    """
    authentication_classes = (authentication.TokenAuthentication,)
    permission_classes = (permissions.IsAdminUser,)

    def get(self, request, format=None):
        """
        返回所有用户的列表。
        """
        usernames = [user.username for user in User.objects.all()]
        return Response(usernames)
```

***
**注意**：Django REST framework 的 `APIView`，`GenericAPIView`，各种 `Mixins` 和 `Viewsets` 之间的完整方法，属性和关系初始(理解)很复杂。除了这里的文档之外，[Classy Django REST Framework](http://www.cdrf.co/) 资源还为每个 Django REST Framework 的基于类的视图提供了一个可浏览的参考，包含完整的方法和属性。
***

## API 策略属性
以下属性控制 API 视图的可插入方面。

### .renderer_classes
### .parser_classes
### .authentication_classes
### .throttle_classes
### .permission_classes
### .content_negotiation_class

## API 策略实例化方法
REST framework 使用以下方法来实例化各种可插入 API 策略。您通常不需要重写这些方法。

### .get_renderers(self)
### .get_parsers(self)
### .get_authenticators(self)
### .get_throttles(self)
### .get_permissions(self)
### .get_content_negotiator(self)
### .get_exception_handler(self)

## API 策略实现方法
在分派到处理程序方法之前调用以下方法。

### .check_permissions(self, request)
### .check_throttles(self, request)
### .perform_content_negotiation(self, request, force=False)

## 调度方法
以下方法由视图的 `.dispatch()` 方法直接调用。它们执行调用处理程序方法之前或之后的任何操作，例如 `.get()`，`.post()`，`put()`，`patch()` 和 `.delete()`。

### .initial(self, request, *args, **kwargs)
执行在调用处理程序方法之前需要执行的任何操作。该方法用于强制执行权限和限流，并执行内容协商。

您通常不需要重写此方法。

### .handle_exception(self, exc)
处理程序方法抛出的任何异常都将传递给此方法，该方法要么返回 `Response` 实例，要么重新引发异常。

默认实现处理 `rest_framework.exceptions.APIException` 的任何子类，以及 Django 的 `Http404` 和 `PermissionDenied` 异常，并返回相应的错误响应。

如果您需要自定义 API 返回的错误响应，您应该子类化此方法。

### .initialize_request(self, request, *args, **kwargs)
确保传递给处理程序方法的请求对象是 `Request` 的一个实例，而不是通常的 Django ` HttpRequest`。

您通常不需要重写此方法。

### .finalize_response(self, request, response, *args, **kwargs)
确保从处理程序方法返回的任何 `Response` 对象都被渲染成正确的内容类型，这是由内容协商决定的。

您通常不需要重写此方法。
***

# 基于函数的视图 (Function Based Views)
说 [基于类的视图] 永远是最好的解决方案是一个错误。——Nick Coghlan

REST framework 还允许您使用常规的基于函数的视图。它提供了一套简单的装饰器来包装你的基于函数的视图，以确保它们接收 `Request` (而不是通常的 Django `HttpRequest`)实例并允许它们返回 `Response` (而不是 Django `HttpResponse` )，并允许你配置该请求的处理方式。

## @api_view()
语法：`@api_view(http_method_names=['GET'])`

这个功能的核心是 `api_view` 装饰器，它接受你的视图应该响应的 HTTP 方法列表。例如，这就是如何编写一个非常简单的视图，只需手动返回一些数据：
```python
from rest_framework.decorators import api_view

@api_view()
def hello_world(request):
    return Response({"message": "Hello, world!"})
```
该视图将使用[设置](http://www.django-rest-framework.org/api-guide/settings/)中指定的默认渲染器，解析器，认证类等。

默认情况下，只有 `GET` 方法会被接受。其他方法将以“405 Method Not Allowed”进行响应。要改变这种行为，请指定视图允许的方法，如下所示：
```python
@api_view(['GET', 'POST'])
def hello_world(request):
    if request.method == 'POST':
        return Response({"message": "Got some data!", "data": request.data})
    return Response({"message": "Hello, world!"})
```

## API 策略装饰器 (API policy decorators)
为了覆盖默认设置，REST framework 提供了一系列可以添加到视图中的附加装饰器。这些必须在 `@api_view` 装饰器之后(下方)。例如，要创建一个使用[限流](http://www.django-rest-framework.org/api-guide/throttling/)来确保它每天只能由特定用户调用一次的视图，请使用 `@throttle_classes` 装饰器，传递一个限流类列表：
```python
from rest_framework.decorators import api_view, throttle_classes
from rest_framework.throttling import UserRateThrottle

class OncePerDayUserThrottle(UserRateThrottle):
        rate = '1/day'

@api_view(['GET'])
@throttle_classes([OncePerDayUserThrottle])
def view(request):
    return Response({"message": "Hello for today! See you tomorrow!"})
```
这些装饰器对应于上面描述的 `APIView` 子类上设置的属性。

可用的装饰者有：

- `@renderer_classes(...)`
- `@parser_classes(...)`
- `@authentication_classes(...)`
- `@throttle_classes(...)`
- `@permission_classes(...)`

每个装饰器都有一个参数，它必须是一个类列表或者类元组。

## 视图模式装饰器 (View schema decorator)
要覆盖基于函数的视图的默认模式生成，您可以使用 `@schema` 装饰器。这必须在 `@api_view` 装饰器之后(下方)。例如：
```python
from rest_framework.decorators import api_view, schema
from rest_framework.schemas import AutoSchema

class CustomAutoSchema(AutoSchema):
    def get_link(self, path, method, base_url):
        # override view introspection here...

@api_view(['GET'])
@schema(CustomAutoSchema())
def view(request):
    return Response({"message": "Hello for today! See you tomorrow!"})
```

该装饰器将接受一个 `AutoSchema` 实例，一个 `AutoSchema` 子类实例或 `ManualSchema` 实例，如[Schemas 文档](http://www.django-rest-framework.org/api-guide/schemas/)中所述。为了从模式生成中排除视图，您可以传递 `None`。
