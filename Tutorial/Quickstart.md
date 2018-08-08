# 环境
Python 3.5.2  
Django 2.0.6  
djangorestframework 3.8.2  
httpie 0.9.9

# 快速开始
我们将创建一个简单的 API 来允许管理员用户查看和编辑系统中的用户和组。

## 项目设置
创建一个名为 `tutorial` 的新 Django 项目，然后创建一个名为 `quickstart` 的新应用。
```python
# 创建项目目录
mkdir tutorial
cd tutorial

# 创建 virtualenv 环境来隔离我们本地的包依赖关系
virtualenv env
source env/bin/activate  # 在 Windows 上使用 `env\Scripts\activate`

# 在创建的虚拟环境中安装 Django 和 Django REST framework
pip install django
pip install djangorestframework

# 创建一个新项目，并只有一个应用
django-admin.py startproject tutorial .  # Note the trailing '.' character
cd tutorial
django-admin.py startapp quickstart
cd ..
```
项目布局应该如下所示：
```python
(venv) fang@ubuntu:~/tutorial$ pwd
/home/fang/tutorial
(venv) fang@ubuntu:~/tutorial$ find .
...
./manage.py
./tutorial
./tutorial/urls.py
./tutorial/wsgi.py
./tutorial/settings.py
./tutorial/quickstart
./tutorial/quickstart/apps.py
./tutorial/quickstart/tests.py
./tutorial/quickstart/migrations
./tutorial/quickstart/migrations/__init__.py
./tutorial/quickstart/models.py
./tutorial/quickstart/admin.py
./tutorial/quickstart/views.py
./tutorial/quickstart/__init__.py
./tutorial/__init__.py
```
在项目目录中创建应用程序可能看起来很不寻常。使用项目的名称空间避免了与外部模块的名称冲突(话题超出了快速入门的范围)。

现在第一次同步您的数据库：
```python
(venv) fang@ubuntu:~/tutorial$ python manage.py migrate
```
我们还将创建一个名为 `admin` 的初始用户，其密码为 `password123`。稍后在示例中，我们将验证该用户。
```python
(venv) fang@ubuntu:~/tutorial$ python manage.py createsuperuser --email admin@example.com --username admin
```
一旦你建立了一个数据库并创建了初始用户并且准备就绪，打开应用程序的目录，我们开始编码......

## 序列化
首先我们要定义一些序列化器。我们来创建一个名为 `tutorial/quickstart/serializers.py` 的新模块，我们将用它来表示数据。
```python
from django.contrib.auth.models import User, Group
from rest_framework import serializers


class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ('url', 'username', 'email', 'groups')


class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Group
        fields = ('url', 'name')
```
请注意，在本例中我们用到了超链接关系，使用 `HyperlinkedModelSerializer`。您还可以使用主键和各种其他关系，但超链接是很棒的 RESTful 设计。

## 视图
好了，我们最好写一些视图。打开 `tutorial/quickstart/views.py` 并输入。
```python
from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from tutorial.quickstart.serializers import UserSerializer, GroupSerializer


class UserViewSet(viewsets.ModelViewSet):
    """
    允许用户查看或编辑的 API 端点。
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer


class GroupViewSet(viewsets.ModelViewSet):
    """
    允许组查看或编辑的 API 端点。
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
```
我们将所有通用行为分组到 `ViewSets` 类中，而不是编写多个视图。

如果需要，我们可以轻松地将这些视图分解为单个视图，但使用视图集可以使视图逻辑组织得非常好，并且非常简洁。

## URLs
好的，现在让我们配置 API 的 URL。在 `tutorial / urls.py` 上...
```
from django.conf.urls import url, include
from rest_framework import routers

from tutorial.quickstart import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'group', views.GroupViewSet)

# 使用自动 URL 路由连接 API。
# 另外，我们还包括可浏览 API 的登录 URL。
urlpatterns = [
    url(r'^', include(router.urls)),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```
因为我们使用视图集而不是视图，所以我们可以通过简单地向路由器类注册视图集来自动为我们的 API 生成 URL conf。

同样，如果我们需要更多地控制 API URL，我们可以简单地使用常规的基于类的视图，并明确地编写 URL conf。

最后，我们将包括默认的登录和注销视图，以用于可浏览的 API。这是可选的，但如果您的 API 需要身份验证并且您想使用可浏览的 API，则会很有用。

## 设置
将 `rest_framework` 添加到 `INSTALLED_APPS`。设置模块将位于 `tutorial/settings.py` 中
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
]
```
好的，我们完成了。

## 测试我们的 API
我们现在准备测试我们构建的 API。让我们从命令行启动服务器。
```python
$ python manage.py runserver
```
我们现在可以从命令行使用 curl 等工具访问我们的 API ...
```python
(venv) fang@ubuntu:~/tutorial$ curl -H 'Accept: application/json; indent=4' -u admin:password123 http://127.0.0.1:8000/users/
[
    {
        "url": "http://127.0.0.1:8000/users/2/",
        "username": "tom",
        "email": "atom@example.com",
        "groups": []
    },
    {
        "url": "http://127.0.0.1:8000/users/1/",
        "username": "admin",
        "email": "admin@example.com",
        "groups": []
    }
```
或者命令行工具 `httpie` ...
```python
(venv) fang@ubuntu:~/tutorial$ http -a admin:password123 http://127.0.0.1:8000/users/
HTTP/1.1 200 OK
Allow: GET, POST, HEAD, OPTIONS
Content-Length: 198
Content-Type: application/json
Date: Mon, 25 Jun 2018 14:22:19 GMT
Server: WSGIServer/0.2 CPython/3.5.2
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

[
    {
        "email": "atom@example.com",
        "groups": [],
        "url": "http://127.0.0.1:8000/users/2/",
        "username": "tom"
    },
    {
        "email": "admin@example.com",
        "groups": [],
        "url": "http://127.0.0.1:8000/users/1/",
        "username": "admin"
    }
]
```
或者直接通过浏览器访问 URL `http://127.0.0.1:8000/users/` ...
![f_38619336.png](http://pan.16mb.com/data/f_38619336.png)

如果你正在使用浏览器访问，请确保从右上角的进行登录。

太棒了，那很容易！

如果您想更深入地了解 REST framework 是如何通过[本教程](http://www.iamnancy.top/post/167/)组合在一起的，或者开始浏览 [API 指南](http://www.django-rest-framework.org/#api-guide)。
