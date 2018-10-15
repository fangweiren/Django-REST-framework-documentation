# 第三方包 (Third Party Packages)
软件生态系统 […] 建立一个社区，进一步加速知识、内容、问题、专业知识和技能的共享。—— [Jan Bosch](http://www.software-ecosystems.com/Software_Ecosystems/Ecosystems.html)

## 关于第三方包 (About Third Party Packages)
第三方软件包允许开发人员共享扩展 Django REST framework 功能的代码，以支持其他用例。

我们**支持**、**鼓励**并**强烈支持**创建第三方包来封装新的行为，而不是直接向 Django REST framework 添加额外的功能。

我们的目标是尽可能简单地创建第三方软件包，同时保持简单且维护良好的核心 API。通过推广第三方软件包，我们确保软件包的责任仍由其作者承担。如果一个包被证明是受欢迎的，那么它总是可以被考虑包含到核心 REST framework 中。

如果您对新功能有所了解，请考虑如何将其打包为第三方包。我们总是很乐意在[邮件列表](https://groups.google.com/forum/#!forum/django-rest-framework)上讨论想法。

## 如何创建第三方包 (How to create a Third Party Package)
### 创建您的包 (Creating your package)
您可以使用[此 cookiecutter 模板](https://github.com/jpadilla/cookiecutter-django-rest-framework)快速创建可重用的 Django REST Framework 包。Cookiecutter 从项目模板创建项目。虽然是可选的，但是这个 cookiecutter 模板包括 Django REST framework 和其他包中的最佳实践，以及 Travis CI 配置、Tox 配置和合理的 setup.py，以便于 PyPI 注册/分发。

注意：如果您有备用 cookiecuter 包，请告诉我们，以便我们也可以链接到它。

#### 运行初始 cookiecutter 命令 (Running the initial cookiecutter command)
要运行初始 cookiecutter 命令，首先需要安装 Python `cookiecutter` 包。
```python
$ pip install cookiecutter
```

安装 `cookiecutter` 后，只需运行以下命令即可创建新项目。
```python
$ cookiecutter gh:jpadilla/cookiecutter-django-rest-framework
```

系统会提示您一些问题，然后回答它们，然后它会根据这些值在当前工作目录中创建您的 Python 包。
```python
full_name (default is "Your full name here")? Johnny Appleseed
email (default is "you@example.com")? jappleseed@example.com
github_username (default is "yourname")? jappleseed
pypi_project_name (default is "dj-package")? djangorestframework-custom-auth
repo_name (default is "dj-package")? django-rest-framework-custom-auth
app_name (default is "djpackage")? custom_auth
project_short_description (default is "Your project description goes here")?
year (default is "2014")?
version (default is "0.1.0")?
```

#### 把它带到 Github 上 (Getting it onto GitHub)
要将项目放在 GitHub 上，您需要一个存储库才能使用它。您可以[在此处](https://github.com/new)创建新存储库。如果您需要帮助，请查看 GitHub 上的 [Create A Repo](https://help.github.com/articles/create-a-repo/) 文章。

#### 添加到 Travis CI (Adding to Travis CI)
我们建议使用 Travis CI，这是一种托管的持续集成服务，可以与 GitHub 完美集成，并且可以免费用于公共存储库。

要开始使用 Travis CI，请使用您的 GitHub 帐户[登录](https://travis-ci.org/)。登录后，转到您的[配置文件页面](https://travis-ci.org/profile)并为所需的存储库启用服务挂钩。

如果您使用 cookiecutter 模板，您的项目将包含一个 `.travis.yml` 文件，Travis CI 将使用该文件来构建项目并运行测试。默认情况下，每次推送到存储库或创建 Pull Request 时都会触发构建。

#### 上传到 PyPI (Uploading to PyPI)
一旦您至少有了一个原型工作和测试运行，您就应该在 PyPI 上发布它，以允许其他人通过 `pip` 安装它。

在发布到 PyPI 之前，您必须[注册](https://pypi.python.org/pypi?%3Aaction=register_form)一个帐户。

要在 PyPI 上注册您的包，请运行以下命令。
```python
$ python setup.py register
```

如果这是第一次发布到 PyPI，系统将提示您登录。

注意：在发布之前，您需要确保您有最新的支持 `wheel` 的 pip，以及安装 `wheel` 包。
```python
$ pip install --upgrade pip
$ pip install wheel
```

在此之后，每次要在 PyPI 上发布新版本时，只需运行以下命令即可。
```python
$ python setup.py publish
You probably want to also tag the version now:
    git tag -a {0} -m 'version 0.1.0'
    git push --tags
```

在向 PyPI 发布新版本后，标记版本并将其作为 GitHub 发行版提供始终是一个好主意。

我们建议对您的包版本遵循[语义版本控制](https://semver.org/)。

### 发展 (Development)
#### 版本要求 (Version requirements)
cookiecutter 模板假定将为 Python 和 Django 提供一组受支持的版本。确保您正确更新您的要求，文档，`tox.ini`，`.travis.yml` 和 `setup.py` 以匹配您希望支持的版本集。

#### 测试 (Tests)
cookiecutter 模板包含 `runtests.py`，它使用 `pytest` 包作为测试运行器。

在运行之前，您需要安装几个测试要求 (依赖)。
```python
$ pip install -r requirements.txt
```

安装依赖后，您可以运行 `runtests.py`。
```python
$ ./runtests.py
```

使用更简洁的输出样式运行。
```python
$ ./runtests.py --fast
```

不要运行 flake8 代码检查。
```python
$ ./runtests.py --nolint
```

只运行 flake8 代码检查，不要运行测试。
```python
$ ./runtests.py --lintonly
```

为给定的测试用例运行测试。
```python
$ ./runtests.py MyTestCase
```

为给定的测试方法运行测试。
```python
$ ./runtests.py MyTestCase.test_this_method
```

为给定的测试方法以较短形式运行测试。
```python
$ ./runtests.py test_this_method
```

要针对不同版本的需求 (例如 Django) 对多个版本的 Python 运行测试，我们建议使用 `tox`。[Tox](https://tox.readthedocs.io/en/latest/) 是一种通用的 virtualenv 管理和测试命令行工具。

首先，全局安装 `tox`。
```python
$ pip install tox
```

要运行 `tox`，只需运行：
```python
$ tox
```

要运行特定的 `tox` 环境：
```python
$ tox -e envlist
```

`envlist` 是一个以逗号分隔的值，用于指定要对其运行测试的环境。要查看所有可能的测试环境的列表，请运行：
```python
$ tox -l
```

#### 版本兼容性 (Version compatibility)
有时，为了确保您的代码适用于各种不同版本的 Django、Python 或第三方库，您需要根据环境运行稍微不同的代码。以这种方式分支的任何代码都应该被隔离到 `compat.py` 模块中，并且应该提供代码库其余部分可以使用的单个公共接口。

查看 Django REST framework 的 [compat.py](https://github.com/encode/django-rest-framework/blob/master/rest_framework/compat.py) 作为示例。

### 一旦您的包可用 (Once your package is available)
一旦您的包在 PyPI 上被正确记录并可用，您可能希望与发现它有用的其他人共享它。

#### 添加到 Django REST framework 网格 (Adding to the Django REST framework grid)
我们建议将您的包添加到 Django Packages 上的 [REST Framework](https://www.djangopackages.com/grids/g/django-rest-framework/) 网格中。

#### 添加到 Django REST framework 文档 (Adding to the Django REST framework docs)
在 GitHub 上创建一个 Pull Request 或 Issue，我们将从主 REST framework 文档中添加一个链接。您可以在最适用的 API 指南部分的**第三方软件包**下添加您的软件包，例如[身份验证](https://www.django-rest-framework.org/api-guide/authentication/)或[权限](https://www.django-rest-framework.org/api-guide/permissions/)。您还可以在[第三方包](https://www.django-rest-framework.org/topics/third-party-packages/#existing-third-party-packages)小节下链接您的包。

#### 在讨论组上宣布 (Announce on the discussion group.)
您还可以通过[讨论组](https://groups.google.com/forum/#!forum/django-rest-framework)让其他人了解您的包。

## 现有第三方包 (Existing Third Party Packages)
Django REST Framework 拥有越来越多的开发人员，软件包和资源社区。

查看详细介绍 [Django Packages](https://www.djangopackages.com/grids/g/django-rest-framework/) 中 Django REST Framework 周围所有包和生态系统的网格。

要提交新内容，请[打开问题](https://github.com/encode/django-rest-framework/issues/new)或[创建拉取请求](https://github.com/encode/django-rest-framework/compare)。

### 认证 (Authentication)
- [djangorestframework-digestauth](https://github.com/juanriaza/django-rest-framework-digestauth) - 提供摘要访问身份验证支持。
- [django-oauth-toolkit](https://github.com/evonove/django-oauth-toolkit) - 提供 OAuth 2.0 支持。
- [doac](https://github.com/Rediker-Software/doac) - 提供 OAuth 2.0 支持。
- [djangorestframework-jwt](https://github.com/GetBlimp/django-rest-framework-jwt) - 提供 JSON Web 令牌认证支持。
- [djangorestframework-simplejwt](https://github.com/davesque/django-rest-framework-simplejwt) - 提供 JSON Web 令牌认证支持的替代包。
- [hawkrest](https://github.com/kumar303/hawkrest) - 提供 Hawk HTTP 授权。
- [djangorestframework-httpsignature](https://github.com/etoccalino/django-rest-framework-httpsignature) - 提供易于使用的 HTTP 签名身份验证机制。
- [djoser](https://github.com/sunscrapers/djoser) - 提供一组视图来处理注册、登录、注销、密码重置和帐户激活等基本操作。
- [django-rest-auth](https://github.com/Tivix/django-rest-auth/) - 提供一组 REST API 端点，用于注册、身份验证 (包括社交媒体身份验证)、密码重置、检索和更新用户详细信息等。
- [drf-oidc-auth](https://github.com/ByteInternet/drf-oidc-auth) - 为 DRF 实现 OpenID Connect 令牌认证。
- [drfpasswordless](https://github.com/aaronn/django-rest-framework-passwordless) - 通过电子邮件和手机号码添加 (Medium，Square Cash inspired) 无密码登录和注册。

### 权限 (Permissions)
- [drf-any-permissions](https://github.com/kevin-brown/drf-any-permissions) - 提供备用权限处理。
- [djangorestframework-composed-permissions](https://github.com/niwibe/djangorestframework-composed-permissions) - 提供定义复杂权限的简单方法。
- [rest_condition](https://github.com/caxap/rest_condition) - 以简单方便的方式构建复杂权限的另一个扩展。
- [dry-rest-permissions](https://github.com/Helioscene/dry-rest-permissions) - 提供一种为单个 api 操作定义权限的简单方法。

### 序列化器 (Serializers)
- [django-rest-framework-mongoengine](https://github.com/umutbozkurt/django-rest-framework-mongoengine) - 支持使用 MongoDB 作为 Django REST framework 存储层的序列化器类。
- [djangorestframework-gis](https://github.com/djangonauts/django-rest-framework-gis) - 地理附加组件
- [djangorestframework-hstore](https://github.com/djangonauts/django-rest-framework-hstore) - 支持 django-hstore DictionaryField 模型字段及其 schema 模式特性的序列化器类。
- [djangorestframework-jsonapi](https://github.com/django-json-api/django-rest-framework-json-api) - 提供解析器、渲染器、序列化器和其他工具，以帮助构建符合 jsonapi.org 规范的 API。
- [html-json-forms](https://github.com/wq/html-json-forms) - 提供一个算法和序列化器来处理每个 (非活动) 规范的 HTML JSON 表单提交。
- [django-rest-framework-serializer-extensions](https://github.com/evenicoulddoit/django-rest-framework-serializer-extensions) - 启用黑名单/白名单字段，并基于每个视图/请求有条件地扩展子序列化器。
- [djangorestframework-queryfields](https://github.com/wimglenn/djangorestframework-queryfields) - Serializer mixin 允许客户端控制将在 API 响应中发送的字段。

### 序列化器字段 (Serializer fields)
- [drf-compound-fields](https://github.com/estebistec/drf-compound-fields) - 提供 “复合” 序列化器字段，例如简单值列表。
- [django-extra-fields](https://github.com/Hipo/drf-extra-fields) - 提供额外的序列化器字段。
- [django-versatileimagefield](https://github.com/WGBH/django-versatileimagefield) - 为 Django 的库存 `ImageField` 提供简单替换，可以轻松地从单个字段提供多种尺寸/再现的图像。有关 DRF 特定的实施文档，[请单击此处](https://django-versatileimagefield.readthedocs.io/en/latest/drf_integration.html)。

### 视图 (Views)
- [djangorestframework-bulk](https://github.com/miki725/django-rest-framework-bulk) - 实现通用视图混合以及一些常见的具体通用视图，以允许通过 API 请求应用批量操作。
- [django-rest-multiple-models](https://github.com/MattBroach/DjangoRestMultipleModels) - 提供通用视图 (和 mixin)，用于通过单个 API 请求发送多个序列化模型和/或查询集。

### 路由器 (Routers)
- [drf-nested-routers](https://github.com/alanjds/drf-nested-routers) - 提供用于处理嵌套资源的路由器和关系字段。
- [wq.db.rest](https://wq.io/docs/about-rest) - 提供具有合理默认 URL 和视图集的管理员样式模型注册 API。

### 解析器 (Parsers)
- [djangorestframework-msgpack](https://github.com/juanriaza/django-rest-framework-msgpack) - 提供 MessagePack 渲染器和解析器支持。
- [djangorestframework-jsonapi](https://github.com/django-json-api/django-rest-framework-json-api) - 提供解析器、渲染器、序列化器和其他工具，以帮助构建符合 jsonapi.org 规范的 API。
- [djangorestframework-camel-case](https://github.com/vbabiy/djangorestframework-camel-case) - 提供驼峰式 JSON 渲染器和解析器。

### 渲染器 (Renderers)
- [djangorestframework-csv](https://github.com/mjumbewu/django-rest-framework-csv) - 提供 CSV 渲染器支持。
- [djangorestframework-jsonapi](https://github.com/django-json-api/django-rest-framework-json-api) - 提供解析器、渲染器、序列化器和其他工具，以帮助构建符合 jsonapi.org 规范的 API。
- [drf_ujson](https://github.com/gizmag/drf-ujson-renderer) - 使用 UJSON 包实现 JSON 渲染。
- [rest-pandas](https://github.com/wq/django-rest-pandas) - 包括 Excel、CSV 和 SVG 格式的 Pandas DataFrame-powered 渲染器。
- [djangorestframework-rapidjson](https://github.com/allisson/django-rest-framework-rapidjson) - 使用解析器和渲染器提供 rapidjson 支持。

### 过滤 (Filtering)
- [djangorestframework-chain](https://github.com/philipn/django-rest-framework-chain) - 允许任意链接的关系和查找过滤器。
- [django-url-filter](https://github.com/miki725/django-url-filter) - 允许以安全的方式通过人性化的 URL 过滤数据。它是一个不依赖于 DRF 的通用库，但它提供了与 DRF 的简单集成。
- [drf-url-filter](https://github.com/manjitkumar/drf-url-filters) 是一个简单的 Django 应用程序，它以简洁、可配置的方式在 drf `ModelViewSet` 的 `Queryset` 上应用过滤器。它还支持对传入查询参数及其值的验证。

### 杂项 (Misc)
- [cookiecutter-django-rest](https://github.com/agconti/cookiecutter-django-rest) - 负责设置和配置的 cookiecutter 模板，因此您可以专注于使您的 REST apis 非常棒。
- [djangorestrelationalhyperlink](https://github.com/fredkingham/django_rest_model_hyperlink_serializers_project) - 一个超链接序列化器，可以通过超链接来改变关系，但在其他方面类似于超链接模型序列化器。
- [django-rest-swagger](https://github.com/marcgibbons/django-rest-swagger) - Swagger UI 的 API 文档生成器。
- [django-rest-framework-proxy](https://github.com/eofs/django-rest-framework-proxy) - 将传入请求重定向到另一个 API 服务器的代理。
- [gaiarestframework](https://github.com/AppsFuel/gaiarestframework) - 用于 django-rest-framework 的实用工具
- [drf-extensions](https://github.com/chibisov/drf-extensions) - 自定义扩展的集合
- [ember-django-adapter](https://github.com/dustinfarris/ember-django-adapter) - 用于处理 Ember.js 的适配器
- [django-versatileimagefield](https://github.com/WGBH/django-versatileimagefield) - 为 Django 的库存 `ImageField` 提供简单替换，可以轻松地从单个字段提供多种尺寸/再现的图像。有关 DRF 特定的实施文档，[请单击此处](https://django-versatileimagefield.readthedocs.io/en/latest/drf_integration.html)。
- [drf-tracking](https://github.com/aschn/drf-tracking) - 用于跟踪 DRF API 视图请求的实用工具。
- [drf_tweaks](https://github.com/ArabellaTech/drf_tweaks) - 具有一步验证 (以及更多) 的序列化程序，没有计数的分页和其他调整。
- [django-rest-framework braces](https://github.com/dealertrack/django-rest-framework-braces) - 用于处理 Django Rest Framework 的实用工具的集合。最值得注意的是 [FormSerializer](https://django-rest-framework-braces.readthedocs.io/en/latest/overview.html#formserializer) 和 [SerializerForm](https://django-rest-framework-braces.readthedocs.io/en/latest/overview.html#serializerform)，它们是 DRF 序列化器和 Django 表单之间的适配器。
- [drf-haystack](https://drf-haystack.readthedocs.io/en/latest/) - Django Rest Framework 的 Haystack 搜索
- [django-rest-framework-version-transforms](https://github.com/mrhwick/django-rest-framework-version-transforms) - 允许 delta 转换的使用对 DRF 资源表示的版本控制。
- [django-rest-messaging](https://github.com/raphaelgyory/django-rest-messaging)，[django-rest-messaging-centrifugo](https://github.com/raphaelgyory/django-rest-messaging-centrifugo) 和 [django-rest-messaging-js](https://github.com/raphaelgyory/django-rest-messaging-js) - 使用 DRM 的实时可插拔消息传递服务。
- [djangorest-alchemy](https://github.com/dealertrack/djangorest-alchemy) - SQLAlchemy 对 REST framework 的支持。
- [djangorestframework-datatables](https://github.com/izimobil/django-rest-framework-datatables) - Django REST framework 和[数据表](https://datatables.net/)之间的无缝集成。
