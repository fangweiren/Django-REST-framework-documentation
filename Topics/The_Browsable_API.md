# 可浏览的 API (The Browsable API)
这是一个极其错误的真理……我们应该培养思考我们正在做什么的习惯。事实恰恰相反。文明的进步是通过扩展我们不需要思考就能完成的重要操作的数量。—— [Alfred North Whitehead](https://en.wikiquote.org/wiki/Alfred_North_Whitehead)，数学概论 (1911)

API 可能代表应用程序编程接口，但是人类也必须能够读取 API；必须有人进行编程。Django REST Framework 支持在请求 HTML 格式时为每个资源生成人性化的 HTML 输出。这些页面可以方便地浏览资源，以及使用 `POST`、`PUT` 和 `DELETE` 向资源提交数据的表单。

## URLs
如果您在资源输出中包含完全限定的 URL，它们将会被 “uralized”，并且可以被人们点击以方便浏览。`rest_framework` 包为此包含了 `reverse` 帮助。

## 格式 (Formats)
默认情况下，API 将返回标头指定的格式，在浏览器中是 HTML。可以在请求中使用 `?format=` 指定格式，因此您可以通过向 URL 添加 `?format=json` 来查看浏览器中的原始 JSON 响应。在 [Firefox](https://addons.mozilla.org/en-US/firefox/addon/jsonview/) 和 [Chrome](https://chrome.google.com/webstore/detail/chklaanhfefbnpoihckbnefhakgolnmc) 中有一些有用的扩展可以查看 JSON。

## 自定义 (Customizing)
可浏览的 API 使用 [Twitter 的 Bootstrap](https://getbootstrap.com/) (v3.3.5) 构建，使其易于自定义外观。

要自定义默认样式，请创建一个名为 `rest_framework/api.html` 的模板，该模板从 `rest_framework/base.html` 扩展而来。例如：

**templates/rest_framework/api.html**
```python
{% extends "rest_framework/base.html" %}

...  # 使用所需的自定义覆盖块
```

### 覆盖默认主题 (Overriding the default theme)
要替换默认主题，请将 `bootstrap_theme` 块添加到 `api.html` 并将 `link` 插入到所需 Bootstrap 主题 css 文件。这将完全取代包含的主题。
```python
{% block bootstrap_theme %}
    <link rel="stylesheet" href="/path/to/my/bootstrap.css" type="text/css">
{% endblock %}
```

在 [Bootswatch](https://bootswatch.com/) 可以找到合适的预先制作的替代主题。要使用任何 Bootswatch 主题，只需下载主题的 `bootstrap.min.css` 文件，将其添加到项目中，然后如上所述替换默认文件。

您还可以使用 `bootstrap_navbar_variant` 块更改导航栏变量，默认情况下为 `navbar-inverse`。空的 `{％ block bootstrap_navbar_variant ％}{％ endblock ％}` 将使用原始的 Bootstrap 导航栏样式。

完整示例：
```python
{% extends "rest_framework/base.html" %}

{% block bootstrap_theme %}
    <link rel="stylesheet" href="https://bootswatch.com/flatly/bootstrap.min.css" type="text/css">
{% endblock %}

{% block bootstrap_navbar_variant %}{% endblock %}
```

对于更具体的 CSS 调整，而不是简单地覆盖默认 bootstrap 主题，您可以覆盖 `style` 块。

***

![](https://www.django-rest-framework.org/img/cerulean.png)

*bootswatch 'Cerulean' 主题的屏幕截图*

***

![](https://www.django-rest-framework.org/img/slate.png)

*bootswatch 'Slate' 主题的屏幕截图*

***

### 块 (Blocks)
可浏览的 API 基本模板中可用的所有块都可以在您的 `api.html` 中使用。

- `body` - 整个 html `<body>`。
- `bodyclass` - `<body>` 标记的类属性，默认为空。
- `bootstrap_theme` - Bootstrap 主题的 CSS。
- `bootstrap_navbar_variant` - 导航栏的 CSS 类。
- `branding` - 导航栏的烙印部分，请参阅 [Bootstrap 组件](https://getbootstrap.com/2.3.2/components.html#navbar)。
- `breadcrumbs` - 链接显示资源嵌套，允许用户备份资源。建议保留这些，但可以使用 breadcrumbs 块覆盖它们。
- `script` - 页面的 JavaScript 文件。
- `style` - 页面的 CSS 样式表。
- `title` - 页面标题。
- `userlinks` - 这是标题右侧的链接列表，默认情况下包含登录/注销链接。要添加链接而不是替换，请使用 `{{ block.super }}` 来保留身份验证链接。

#### 组件 (Components)
所有标准的 [Bootstrap 组件](https://getbootstrap.com/2.3.2/components.html)都可用。

#### 工具提示 (Tooltips)
可浏览的 API 使用 Bootstrap 工具提示组件。带有 `js-tooltip` 类和 `title` 属性的任何元素，其标题内容都将显示悬停事件的工具提示。

### 登录模板 (Login Template)
要添加烙印并自定义登录模板的外观，请创建名为 `login.html` 的模板并将其添加到项目中，例如：`templates/rest_framework/login.html`。模板应该从 `rest_framework/login_base.html` 扩展。

您可以通过包含烙印块来添加您的网站名称或烙印：
```python
{% block branding %}
    <h3 style="margin: 0 0 20px;">My Site Name</h3>
{% endblock %}
```

您还可以通过添加类似于 `api.html` 的 `bootstrap_theme` 或 `style` 块来自定义样式。

### 高级定制 (Advanced Customization)
#### 上下文 (Context)
模板可用的上下文：

- `allowed_methods`：资源允许的方法列表
- `api_settings`：API设置
- `available_formats`：资源允许的格式列表
- `breadcrumblist`：嵌套资源链后面的链接列表
- `content`：API 响应的内容
- `description`：从文档字符串生成的资源的描述
- `name`：资源的名称
- `post_form`：POST 表单使用的表单实例 (如果允许的话)
- `put_form`：PUT 表单使用的表单实例 (如果允许的话)
- `display_edit_forms`：一个布尔值，指示是否显示 POST，PUT 和 PATCH 表单
- `request`：请求对象
- `response`：响应对象
- `version`：Django REST Framework 的版本
- `view`：处理请求的视图
- `FORMAT_PARAM`：视图可以接受格式覆盖
- `METHOD_PARAM`：视图可以接受方法覆盖

您可以覆盖 `BrowsableAPIRenderer.get_context()` 方法以自定义传递给模板的上下文。

#### 不使用 base.html (Not using base.html)
对于更高级的定制，例如没有 Bootstrap 基础或者没有与站点其他部分更紧密的集成，您可以简单地选择不使用 `api.html` 扩展 `base.html`。然后页面内容和功能完全取决于您。

#### 使用大量项处理 `ChoiceField` (Handling `ChoiceField` with large numbers of items.)
当关系或 `ChoiceField` 包含太多项时，渲染包含所有选项的小部件可能会变得非常慢，并导致可浏览的 API 渲染性能不佳。

在本例中，最简单的选项是用标准文本输入替换选择输入。例如：
```python
 author = serializers.HyperlinkedRelatedField(
    queryset=User.objects.all(),
    style={'base_template': 'input.html'}
)
```

#### 自动完成 (Autocomplete)
另一个更复杂的选项是用自动完成小部件替换输入，该小部件仅根据需要加载和渲染可用选项的子集。如果您需要这样做，您需要做一些工作来自己构建自定义自动完成 HTML 模板。

[自动完成小部件有多种软件包](https://www.djangopackages.com/grids/g/auto-complete/)，例如 [django-autocomplete-light](https://github.com/yourlabs/django-autocomplete-light)，您可能需要参考这些软件包。注意，您不能简单地将这些组件作为标准小部件包含，但是需要显式地编写 HTML 模板。这是因为 REST framework 3.0 不再支持 `widget` 关键字参数，因为它现在使用模板化 HTML 生成。

***
