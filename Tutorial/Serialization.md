# 教程 1：序列化
## 介绍
本教程将涵盖创建一个简单的收录代码高亮展示的 Web API。整个过程, 将会介绍构成 REST framewoek 的各个组件，并让您全面了解各个组件是如何一起工作的。

该教程相当深入，所以你应该在开始之前准备一些曲奇饼干和一杯你最喜欢的啤酒。如果您只想快速浏览一下，则应该转而使用[快速入门文档](http://www.django-rest-framework.org/tutorial/quickstart/)。

***
**注意**：本教程的代码可以在 Github 的 [tomchristie/rest-framework-tutorial](https://github.com/encode/rest-framework-tutorial) 存储库中找到。完整的实现也可以在线作为一个沙盒版本进行测试。
***

## 搭建一个新的环境
在做其他事情之前，我们要用 virtualenv 创建一个新的虚拟环境。这将确保我们的软件包配置与我们正在处理的其他任何项目保持良好隔离。
```python
virtualenv env
source env/bin/activate
```
现在我们处于 virtualenv 环境中，我们可以安装我们需要的软件包。
```python
pip install django
pip install djangorestframework
pip install pygments  # 用于代码高亮
```
**注意**：要随时退出 virtualenv 环境，只需输入 `deactivate`。有关更多信息，请参阅 [virtualenv 文档](https://virtualenv.pypa.io/en/latest/index.html)。

## 入门开始
好了，我们现在要开始写代码了。首先，让我们来创建一个新的项目。
```python
cd ~
django-admin.py startproject tutorial
cd tutorial
```
完成上述步骤之后，我们可以创建一个用于创建简单的 Web API 的应用程序。
```python
python manage.py startapp snippets
```
我们需要将新建的 `snippets` 应用和 `rest_framework` 应用添加到 `INSTALLED_APPS`。让我们编辑 `tutorial/settings.py` 文件：
```python
INSTALLED_APPS = (
    ...
    'rest_framework',
    'snippets.apps.SnippetsConfig',
)
```
好了，我们的准备工作做完了。

## 创建一个模型
为了实现本教程的目的，我们将开始创建一个用于存储代码片段的简单的 `Snippet` 模型。然后继续编辑 `snippets/models.py` 文件。注意：良好的编程做法包括添加注释。尽管您可以在本教程代码的存储库版本中找到它们，但我们在此忽略它们以专注于代码本身。
```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

# 提取出了 pygments 支持的所有语言的词法分析程序
LEXERS = [item for item in get_all_lexers() if item[1]]
# 提取出了 pygments 支持的所有语言列表
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
# 提取出了 pygments 支持的所有格式化风格列表
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())


# Create your models here.
class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)  # 是否显示行号
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ('created',)
```
我们还需要为我们的代码段模型创建初始迁移 (initial migration)，并首次同步数据库 (migrate)。
```python
python manage.py makemigrations snippets
python manage.py migrate
```

## 创建一个序列化类
我们开始使用 Web API 的第一件事就是提供一种将代码片段实例序列化和反序列化为表示形式 (如 `json`) 的方法。我们可以通过声明与 Django forms 非常相似的序列化器 (serializers) 实现。在 `snippets` 的目录下创建一个名为 `serializers.py` 文件，并添加以下内容。
```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        给定验证过的数据创建并返回一个新的 Snippet 实例。
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        根据已验证的数据更新并返回已存在的 Snippet 实例。
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```
序列化器类的第一部分定义了获得序列化/反序列化的字段。`create()` 和 `update()` 方法定义了在调用 `serializer.save()` 时如何创建和修改完整的实例。  
序列化器类与 Django `Form` 类非常相似，并在各种字段中包含类似的验证标志，例如`required`，`max_length` 和 `default`。  
字段标志还可以控制 serializer 在某些情况下如何显示，比如渲染 HTML 的时候。上面的`{'base_template': 'textarea.html'}` 标志等同于在 Django `Form` 类中使用 `widget=widgets.Textarea`。这对于控制如何显示可浏览的 API 特别有用，我们将在本教程的后面看到。  
我们实际上也可以通过使用 `ModelSerializer` 类来节省一些时间，我们稍后会看到，但是现在我们将使用我们明确定义的 `serializer`。

## 使用 Serializers
在我们进一步了解之前，我们将熟悉使用我们的新的 `Serializer` 类。让我们进入 Django shell。
```python
python manage.py shell
```
好了，我们已经导入了几个模块，然后开始创建一些代码片段来处理。
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ python manage.py shell
Python 3.5.2 (default, Nov 23 2017, 16:37:01) 
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from snippets.models import Snippet
>>> from snippets.serializers import SnippetSerializer
>>> from rest_framework.renderers import JSONRenderer
>>> from rest_framework.parsers import JSONParser
>>> 
>>> snippet = Snippet(code='foo = "bar"\n')
>>> snippet.save()
>>> 
>>> snippet = Snippet(code='print "hello, world"\n')
>>> snippet.save()
>>> 
```
我们现在已经有了几个可用的片段实例，让我们看看序列化其中一个实例。
```python
>>> serializer = SnippetSerializer(snippet)
>>> serializer.data
{'language': 'python', 'code': 'print "hello, world"\n', 'style': 'friendly', 'id': 2, 'linenos': False, 'title': ''}
>>> 
```
此时，我们将模型实例转换为 Python 原生数据类型。要完成序列化过程，我们将数据渲染成 `json`。
```python
>>> content = JSONRenderer().render(serializer.data)
>>> content
b'{"id":2,"title":"","code":"print \\"hello, world\\"\\n","linenos":false,"language":"python","style":"friendly"}'
>>> 
```
反序列化是类似的。首先我们将一个流解析为 Python 原生数据类型...
```python
>>> from django.utils.six import BytesIO
>>> 
>>> stream = BytesIO(content)
>>> data = JSONParser().parse(stream)
>>> data
{'language': 'python', 'code': 'print "hello, world"\n', 'style': 'friendly', 'id': 2, 'linenos': False, 'title': ''}
>>> 
```
...然后我们将这些原生数据类型恢复成正常的对象实例。
```python
>>> serializer = SnippetSerializer(data=data)
>>> serializer.is_valid()
True
>>> serializer.validated_data
OrderedDict([('title', ''), ('code', 'print "hello, world"'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])
>>> serializer.save()
<Snippet: Snippet object (3)>
>>>
```
注意 API 工作形式和表单(forms)是多么相似。当我们开始使用我们的序列化类编写视图的时候，相似性会变得更加明显。  
我们也可以序列化查询集代替模型实例。为此，我们只需在序列化参数中添加一个 `many = True` 标志。
```python
>>> serializer = SnippetSerializer(Snippet.objects.all(), many=True)
>>> serializer.data
[OrderedDict([('id', 1), ('title', ''), ('code', 'foo = "bar"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 2), ('title', ''), ('code', 'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 3), ('title', ''), ('code', 'print "hello, world"'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])]
>>>
```

## 使用 ModelSerializers
我们的 `SnippetSerializer` 类中重复了很多包含在 `Snippet` 模型中的信息。如果我们能保持代码简洁，那就更好了。  
就像 Django 提供了 `Form` 类和 `ModelForm` 类一样，REST framework 包括 `Serializer` 类和 `ModelSerializer` 类。  
让我们看看使用 `ModelSerializer` 类重构我们的序列化类。再次打开 `snippets/serializers.py` 文件，并将 `SnippetSerializer` 类替换为以下内容。
```python
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```
序列化程序一个非常棒的属性就是可以通过打印其结构 (representation) 来检查序列化实例中的所有字段。
```python
>>> from snippets.serializers import SnippetSerializer
>>> serializer = SnippetSerializer()
>>> print(repr(serializer))
SnippetSerializer():
    id = IntegerField(read_only=True)
    title = CharField(allow_blank=True, max_length=100, required=False)
    code = CharField(style={'base_template': 'textarea.html'})
    linenos = BooleanField(required=False)
    language = ChoiceField(choices=[('abap', 'ABAP'), ('abnf', 'ABNF'), ('ada', 'Ada'), ..., ('zephir', 'Zephir')], default='python')
    style = ChoiceField(choices=[('abap', 'abap'), ('algol', 'algol'), ('algol_nu', 'algol_nu'), ..., ('xcode', 'xcode')], default='friendly')
>>> 
```
记住 `ModelSerializer` 类并不会做任何特别神奇的事情，它们只是创建序列化器类的快捷方式：  
- 自动确定一组字段。(不用重复去定义类属性)
- 默认简单实现的 `create()` 和 `update()` 方法。

## 使用我们的 Serializer 编写常规的 Django 视图
让我们看看如何使用我们新建的 Serializer 类编写一些 API 视图。目前我们不会使用任何 REST framework 的其他功能，我们只编写常规的 Django 视图。  
编辑 `snippets/views.py` 文件，并且添加以下内容。
```python
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
```
我们 API 的根视图支持列出所有现有的 snippet 或创建一个新的 snippet。
```python
@csrf_exempt
def snippet_list(request):
    """
    列出所有的代码 snippet，或创建一个新的 snippet。
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)
    
    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
    return JsonResponse(serializer.errors, status=400)
```
注意，因为我们希望能够从不具有 CSRF 令牌的客户端 POST 到此视图，所以我们需要将该视图标记为 `csrf_exempt`。这不是你通常想要做的事情，并且 REST framework 视图实际上比这更实用的行为，但它现在足够达到我们的目的。  
我们还需要一个与单个 snippet 相对应的视图，并可用于检索，更新或删除 snippet。
```python
@csrf_exempt
def snippet_detail(request, pk):
    """
    获取，更新或删除一个代码 snippet
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)
    
    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)
    
    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```
最后，我们需要把这些视图连起来。创建 `snippets/urls.py` 文件：
```python
from django.conf.urls import url
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.snippet_detail),
]
```
我们也需要在 `tutorial/urls.py` 文件中连接 root urlconf 来包含我们的 snippets 应用的URLs。
```python
from django.conf.urls import url, include

urlpatterns = [
    url(r'^', include('snippets.urls')),
]
```
值得注意的是，目前我们还没有处理好几个边缘案例。如果我们发送格式错误的 `json`，或者如果使用视图不处理的方法发出请求，那么我们最终将得到一个 500 “server error” 的响应。总之，现在我们暂时这么做。

## 测试我们在 Web API 上的第一次访问
现在为我们的 snippets 启动一个服务器。  
退出 shell...
```python
quit()
```
...并启动 Django 的开发服务器。
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ python manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).
June 13, 2018 - 10:17:11
Django version 2.0.6, using settings 'tutorial.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
另起一个终端窗口，我们可以测试服务器。  
我们可以使用 `curl` 或 `httpie` 来测试我们的 API。Httpie 是用 Python 编写的对用户友好的 http 客户端。让我们安装它。  
您可以使用 pip 安装 httpie：
```python
pip install httpie
```
最后，我们可以得到所有 snippet 的列表：
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ http http://127.0.0.1:8000/snippets/
HTTP/1.1 200 OK
Content-Length: 352
Content-Type: application/json
Date: Wed, 13 Jun 2018 10:23:58 GMT
Server: WSGIServer/0.2 CPython/3.5.2
X-Frame-Options: SAMEORIGIN

[
    {
        "code": "foo = \"bar\"\n",
        "id": 1,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    },
    {
        "code": "print \"hello, world\"\n",
        "id": 2,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    },
    {
        "code": "print \"hello, world\"",
        "id": 3,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    }
]
```
或者我们可以通过引用其 id 来获取特定的 snippet：
```python
(env) fang@ubuntu:~/django_rest_framework/tutorial$ http http://127.0.0.1:8000/snippets/2/
HTTP/1.1 200 OK
Content-Length: 119
Content-Type: application/json
Date: Wed, 13 Jun 2018 10:25:50 GMT
Server: WSGIServer/0.2 CPython/3.5.2
X-Frame-Options: SAMEORIGIN

{
    "code": "print \"hello, world\"\n",
    "id": 2,
    "language": "python",
    "linenos": false,
    "style": "friendly",
    "title": ""
}
```
同样，您可以通过在 Web 浏览器中访问这些 URL 来显示相同​​的 `json`。

## 我们现在在哪
到目前为止，我们做得不错，我们有一个与 Django 的 Forms API 非常相似序列化 API，和一些常规的 Django 视图。  
除了服务 `json` 响应之外，我们的 API 视图目前没有做特别的事情，并且还有我们仍然想要清理的一些错误处理边缘情况，但它是一个正常运行的 Web API。  
我们将在[本教程的第2部分](http://www.iamnancy.top/post/168/)看到我们如何开始改进这些事。
