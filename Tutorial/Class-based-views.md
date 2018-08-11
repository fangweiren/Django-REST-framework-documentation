# 教程 3：基于类的视图
我们也可以使用基于类的视图来编写 API 视图，而不是基于函数的视图。正如我们将看到的，这是一个强大的模式，可以让我们重用通用功能，并帮助我们保持代码 [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself)。

## 使用基于类的视图重写我们的 API
我们将首先将基于类的视图重写根视图。所有这些都涉及一些对 `views.py` 的重构。
```python
class SnippetList(APIView):
    """
    列出所有的代码 snippet，或创建一个新的 snippet。
    """
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```
到现在为止还挺好。它看起来与以前的情况非常相似，但是我们在不同的 HTTP 方法之间有更好的分离。我们还需要更新 `views.py` 中的实例视图。
```python
class SnippetDetail(APIView):
    """
    获取，更新或删除一个代码 snippet
    """
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
看起来不错，它现在仍然非常类似于基于函数的视图。

使用基于类的视图，我们现在还需要稍微重构 `snippets/urls.py`。
```python
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views


urlpatterns = [
    url(r'^snippets/$', views.SnippetList.as_view()),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```
好的，我们完成了。 如果你运行开发服务器，一切都应该像以前一样工作。

## 使用混合 (mixins)
使用基于类的视图的优势之一是我们可以很容易撰写可重复使用的行为。

到目前为止，我们使用的创建/获取/更新/删除操作和我们创建的任何基于模型的 API 视图非常相似。这些常见的行为是在 REST framework 的 mixin 类中实现的。

让我们来看看我们是如何通过使用 mixin 类编写视图的。这是我们的 `snippets/views.py` 模块。
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics


class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```
我们将花一点时间仔细检查这里发生了什么。我们使用 `GenericAPIView` 构建了视图，并且用上了 `ListModelMixin` 和 `CreateModelMixin`。

基类提供核心功能，而 mixin 类提供 `.list()` 和 `.create()` 操作。然后我们明确地将 `get` 和 `post` 方法绑定到适当的操作。到目前为止显而易见。
```python
class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```
非常相似。我们再次使用 `GenericAPIView` 类来提供核心功能，并添加 mixins 来提供 `.retrieve()`，`.update()` 和 `.destroy()` 操作。

## 使用通用的基于类的视图
我们使用 mixin 类，使用比以前稍少的代码重写了视图，但我们可以更进一步。REST framework 提供了一套已经混合的通用视图，我们可以简化 `views.py `模块。
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import generics


class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```
哇，非常简洁。我们可以使用很多现成的代码，让我们的代码看起来非常清晰、简洁，惯用的 Django。

接下来，我们将转到[本教程的第 4 部分](http://www.iamnancy.top/post/170/)，我们将介绍如何处理 API 的身份验证和权限。
