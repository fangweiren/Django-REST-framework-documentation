# API 客户端 (API Clients)
API 客户端处理网络请求是如何发出的以及如何解码响应的底层细节。它们为开发人员提供了一个应用程序接口来工作，而不是直接与网络接口一起工作。

此处记录的 API 客户端不限于使用 Django REST framework 构建的 API。它们可以与任何公开支持的模式格式的 API 一起使用。

例如，[Heroku 平台 API](https://devcenter.heroku.com/categories/platform-api) 以 JSON 超模式格式公开模式。因此，Core API 命令行客户端和 Python 客户端库可[用于与 Heroku API 交互](http://www.coreapi.org/tools-and-resources/example-services/#heroku-json-hyper-schema)。

## 客户端核心 API (Client-side Core API)
[核心 API](http://www.coreapi.org/) 是一种可用于描述 API 的文档规范。它既可以在服务器端使用，就像 REST framework 的[模式生成](http://www.django-rest-framework.org/api-guide/schemas/)一样，也可以在客户端使用，如此处所述。

在客户端使用时，Core API 允许动态驱动的客户端库可以与任何公开支持的模式或超媒体格式的 API 进行交互。

与通过直接构建 HTTP 请求与 API 交互相比，使用动态驱动的客户端具有许多优势。

#### 更有意义的交互 (More meaningful interaction)
API 交互以更有意义的方式呈现。您正在应用程序接口层工作，而不是网络接口层。

#### 弹性和进化性 (Resilience & evolvability)
客户端确定哪些端点可用，针对每个特定端点存在哪些参数，以及如何形成 HTTP 请求。

这还允许一定程度的 API 可进化性。URL 可以在不破坏现有客户端的情况下进行修改，或者可以在线使用更高效的编码，并且客户端可以透明地升级。

#### 自描述 API (Self-descriptive APIs)
动态驱动的客户端能够向最终用户提供 API 上的文档。该文档允许用户发现可用的端点和参数，并更好地了解他们正在使用的 API。

因为此文档是由 API 模式驱动的，所以它总是与最近部署的服务版本完全同步。

***

# 命令行客户端 (Command line client)
命令行客户端允许您检查和交互任何公开支持的模式格式的 API。

## 入门 (Getting started)
要安装 Core API 命令行客户端，请使用 `pip`。

请注意，命令行客户端是 python 客户端库的单独包。确保安装 `coreapi-cli`。
```python
$ pip install coreapi-cli
```

要开始检查和与 API 交互，必须首先从网络加载模式。
```python
$ coreapi get http://api.example.org/
<Pastebin API "http://127.0.0.1:8000/">
snippets: {
    create(code, [title], [linenos], [language], [style])
    destroy(pk)
    highlight(pk)
    list([page])
    partial_update(pk, [title], [code], [linenos], [language], [style])
    retrieve(pk)
    update(pk, code, [title], [linenos], [language], [style])
}
users: {
    list([page])
    retrieve(pk)
}
```

然后，这将加载模式，显示生成的 `Document`。`Document` 包含可能针对 API 进行的所有可用交互。

要与 API 交互，请使用 `action` 命令。此命令需要用于索引到链接的键列表。
```python
$ coreapi action users list
[
    {
        "url": "http://127.0.0.1:8000/users/2/",
        "id": 2,
        "username": "aziz",
        "snippets": []
    },
    ...
]
```

要检查底层 HTTP 请求和响应，请使用 `--debug` 标志。
```python
$ coreapi action users list --debug
> GET /users/ HTTP/1.1
> Accept: application/vnd.coreapi+json, */*
> Authorization: Basic bWF4Om1heA==
> Host: 127.0.0.1
> User-Agent: coreapi
< 200 OK
< Allow: GET, HEAD, OPTIONS
< Content-Type: application/json
< Date: Thu, 30 Jun 2016 10:51:46 GMT
< Server: WSGIServer/0.1 Python/2.7.10
< Vary: Accept, Cookie
<
< [{"url":"http://127.0.0.1/users/2/","id":2,"username":"aziz","snippets":[]},{"url":"http://127.0.0.1/users/3/","id":3,"username":"amy","snippets":["http://127.0.0.1/snippets/3/"]},{"url":"http://127.0.0.1/users/4/","id":4,"username":"max","snippets":["http://127.0.0.1/snippets/4/","http://127.0.0.1/snippets/5/","http://127.0.0.1/snippets/6/","http://127.0.0.1/snippets/7/"]},{"url":"http://127.0.0.1/users/5/","id":5,"username":"jose","snippets":[]},{"url":"http://127.0.0.1/users/6/","id":6,"username":"admin","snippets":["http://127.0.0.1/snippets/1/","http://127.0.0.1/snippets/2/"]}]

[
    ...
]
```

某些操作可能包括可选或必需的参数。
```python
$ coreapi action users create --param username=example
```

使用 `--param` 时，将自动确定输入的类型。

如果您希望更明确地参数类型，那么对任何空值、数值、布尔值、列表或对象输入使用 `--data`，对字符串输入使用 `--string`。
```python
$ coreapi action users edit --string username=tomchristie --data is_admin=true
```

## 认证和标头 (Authentication & headers)
`credentials` 命令用于管理请求 `Authentication:` 标头。添加的任何凭证总是链接到特定域名，以确保凭证不会在不同 API 之间泄漏。

添加新凭据的格式为：
```python
$ coreapi credentials add <domain> <credentials string>
```

例如：
```python
$ coreapi credentials add api.example.org "Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b"
```

可选的 `--auth` 标志还允许您添加特定类型的身份验证，为您处理编码。目前，此处仅支持 `“basic”` 作为选项。例如：
```python
$ coreapi credentials add api.example.org tomchristie:foobar --auth basic
```

您还可以使用 `headers` 命令添加特定的请求标头：
```python
$ coreapi headers add api.example.org x-api-version 2
```

有关更多信息和可用子命令的列表，请使用 `coreapi credentials --help` 或 `coreapi headers --help`。

## 编解码器 (Codecs)
默认情况下，命令行客户端仅包括对读取 Core JSON 模式的支持，但它包含用于安装其他编解码器的插件系统。
```python
$ pip install openapi-codec jsonhyperschema-codec hal-codec
$ coreapi codecs show
Codecs
corejson        application/vnd.coreapi+json encoding, decoding
hal             application/hal+json         encoding, decoding
openapi         application/openapi+json     encoding, decoding
jsonhyperschema application/schema+json      decoding
json            application/json             data
text            text/*                       data
```

## Utilities
命令行客户端包括以令人难忘的名称为 API URL 添加书签的功能。例如，您可以为现有API添加书签，像这样...
```python
$ coreapi bookmarks add accountmanagement
```

还有用于向前或向后导航访问API URL的历史记录的功能。
```python
$ coreapi history show
$ coreapi history back
```

有关更多信息和可用子命令的列表，请使用 `coreapi bookmarks --help` 或 `coreapi history --help`。

## 其他命令 (Other commands)
要显示当前 `Document`：
```python
$ coreapi show
```

要从网络重新加载当前 `Document`：
```python
$ coreapi reload
```

要从磁盘加载模式文件：
```python
$ coreapi load my-api-schema.json --format corejson
```

以给定的格式将当前文档转储到控制台：
```python
$ coreapi dump --format openapi
```

要删除当前文档以及所有当前保存的历史记录，凭据，标头和书签：
```python
$ coreapi clear
```

***

# Python 客户端库 (Python client library)
`coreapi` Python 包允许您以编程方式与任何公开支持的模式格式的 API 进行交互。

## 入门 (Getting started)
在开始之前，您需要使用 `pip` 安装 `coreapi` 软件包。
```python
$ pip install coreapi
```

为了开始使用 API​​，我们首先需要一个 `Client` 实例。客户端保存与 API 交互时支持编解码器和传输的任何配置，这允许您提供更高级的行为。
```python
import coreapi
client = coreapi.Client()
```

一旦我们有了 `Client` 实例，我们就可以从网络中获取 API 模式。
```python
schema = client.get('https://api.example.org/')
```

从此调用返回的对象将是 `Document` 实例，它是 API 模式的表示。

## 认证 (Authentication)
通常，您还需要在实例化客户端时提供一些身份验证凭据。

#### 令牌认证 (Token authentication)
`TokenAuthentication` 类可用于支持 REST framework 的内置 `TokenAuthentication`，以及 OAuth 和 JWT 模式。
```python
auth = coreapi.auth.TokenAuthentication(
    scheme='JWT'
    token='<token>'
)
client = coreapi.Client(auth=auth)
```

使用 `TokenAuthentication` 时，您可能需要使用 CoreAPI 客户端实现登录流程。

建议模式最初是发送未经过身份验证的客户端请求到 "获取令牌" 端点。

例如，使用 “Django REST framework JWT” 包。
```python
client = coreapi.Client()
schema = client.get('https://api.example.org/')

action = ['api-token-auth', 'obtain-token']
params = {username: "example", email: "example@example.com"}
result = client.action(schema, action, params)

auth = coreapi.auth.TokenAuthentication(
    scheme='JWT',
    token=result['token']
)
client = coreapi.Client(auth=auth)
```

#### 基本认证 (Basic authentication)
`BasicAuthentication` 类可用于支持 HTTP 基本身份验证。
```python
auth = coreapi.auth.BasicAuthentication(
    username='<username>',
    password='<password>'
)
client = coreapi.Client(auth=auth)
```

## 与 API 交互 (Interacting with the API)
现在我们有了一个客户端并且已经获取了我们的模式 `Document`，现在我们可以开始与 API 交互了：
```python
users = client.action(schema, ['users', 'list'])
```

某些端点可能包含命名参数，这些参数可能是可选的或必需的：
```python
new_user = client.action(schema, ['users', 'create'], params={"username": "max"})
```

## 编解码器 (Codecs)
编解码器负责编码或解码文档。

客户端使用解码过程来获取 API 模式定义的字节串，并返回表示该接口的 Core API 文档。

编解码器应与特定媒体类型相关联，例如 `'application/coreapi+json'`。

服务器在响应 `Content-Type` 标头中使用此媒体类型，以指示响应中返回的数据类型。

#### 配置编解码器 (Configuring codecs)
在实例化客户端时可以配置可用的编解码器。这里使用的关键字参数是 `decoders`，因为在客户端的上下文中，编解码器仅用于解码响应。

在以下示例中，我们将客户端配置为仅接受 `Core JSON` 和 `JSON` 响应。这将允许我们接收和解码 Core JSON 模式，然后接收针对 API 做出的 JSON 响应。
```python
from coreapi import codecs, Client

decoders = [codecs.CoreJSONCodec(), codecs.JSONCodec()]
client = Client(decoders=decoders)
```

#### 加载和保存模式 (Loading and saving schemas)
您可以直接使用编解码器，以加载现有的模式定义，并返回生成的 `Document`。
```python
input_file = open('my-api-schema.json', 'rb')
schema_definition = input_file.read()
codec = codecs.CoreJSONCodec()
schema = codec.load(schema_definition)
```

您还可以在给定 `Document` 实例的情况下直接使用编解码器生成模式定义：
```python
schema_definition = codec.dump(schema)
output_file = open('my-api-schema.json', 'rb')
output_file.write(schema_definition)
```

## 传输 (Transports)
传输负责发出网络请求。客户端已安装的传输集确定了它能够支持的网络协议。

目前，`coreapi` 库仅包含 HTTP/HTTPS 传输，但也可以支持其他协议。

#### 配置传输 (Configuring transports)
可以通过配置客户端实例化的传输来自定义网络层的行为。
```python
import requests
from coreapi import transports, Client

credentials = {'api.example.org': 'Token 3bd44a009d16ff'}
transports = transports.HTTPTransport(credentials=credentials)
client = Client(transports=transports)
```

还可以实现更复杂的自定义，例如修改底层的 `requests.Session` 实例以附加修改传出请求的[传输适配器](http://docs.python-requests.org/en/master/user/advanced/#transport-adapters)。

***

# JavaScript 客户端库 (JavaScript Client Library)
JavaScript 客户端库允许您通过浏览器或使用节点与 API 进行交互。

## 安装 JavaScript 客户端 (Installing the JavaScript client)
为了使用 JavaScript 客户端库，您需要在 HTML 页面中包含两个单独的 JavaScript 资源。这些是一个静态 `coreapi.js` 文件，它包含动态客户端库的代码，以及一个模板化的 `schema.js` 资源，它公开了您的 API 模式。

首先，安装 API 文档视图。这些将包括模式资源，它允许您直接从 HTML 页面加载模式，而无需进行异步 AJAX 调用。
```python
from rest_framework.documentation import include_docs_urls

urlpatterns = [
    ...
    url(r'^docs/', include_docs_urls(title='My API service'))
]
```

安装 API 文档 URL 后，您将能够包含所需的 JavaScript 资源。请注意，这两行的排序很重要，因为模式加载需要已安装 CoreAPI。
```python
<!--
    Load the CoreAPI library and the API schema.

    /static/rest_framework/js/coreapi-0.1.1.js
    /docs/schema.js
-->
{% load static %}
<script src="{% static 'rest_framework/js/coreapi-0.1.1.js' %}"></script>
<script src="{% url 'api-docs:schema-js' %}"></script>
```

`coreapi` 库和 `schema` 对象现在都可以在 `window` 实例上使用。
```python
const coreapi = window.coreapi
const schema = window.schema
```

## 实例化客户端 (Instantiating a client)
为了与API交互，您需要一个客户端实例。
```python
var client = new coreapi.Client()
```

通常，您还需要在实例化客户端时提供一些身份验证凭据。

#### 会话认证 (Session authentication)
`SessionAuthentication` 类允许会话 cookie 提供用户身份验证。您需要提供标准的 HTML 登录流程，以允许用户登录，然后使用会话身份验证实例化客户端：
```python
let auth = new coreapi.auth.SessionAuthentication({
    csrfCookieName: 'csrftoken',
    csrfHeaderName: 'X-CSRFToken'
})
let client = new coreapi.Client({auth: auth})
```

认证方案将处理包括在任何不安全的 HTTP 方法的输出请求中的 CSRF 标头。

#### 令牌认证 (Token authentication)
`TokenAuthentication` 类可用于支持 REST framework 的内置 `TokenAuthentication`，以及 OAuth 和 JWT 模式。
```python
let auth = new coreapi.auth.TokenAuthentication({
    scheme: 'JWT'
    token: '<token>'
})
let client = new coreapi.Client({auth: auth})
```

使用 `TokenAuthentication` 时，您可能需要使用 CoreAPI 客户端实现登录流程。

建议模式最初是发送未经过身份验证的客户端请求到 "获取令牌" 端点。

例如，使用 “Django REST framework JWT” 包。
```python
// 设置一些全局可访问状态
window.client = new coreapi.Client()
window.loggedIn = false

function loginUser(username, password) {
    let action = ["api-token-auth", "obtain-token"]
    let params = {username: "example", email: "example@example.com"}
    client.action(schema, action, params).then(function(result) {
        // 成功时，实例化经过身份验证的客户端。
        let auth = window.coreapi.auth.TokenAuthentication({
            scheme: 'JWT',
            token: result['token']
        })
        window.client = coreapi.Client({auth: auth})
        window.loggedIn = true
    }).catch(function (error) {
        // 处理错误情况，例如：用户提供不正确的凭据
    })
}
```

#### 基本认证 (Basic authentication)
`BasicAuthentication` 类可用于支持 HTTP 基本身份验证。
```python
let auth = new coreapi.auth.BasicAuthentication({
    username: '<username>',
    password: '<password>'
})
let client = new coreapi.Client({auth: auth})
```

## 使用客户端 (Using the client)
发出请求：
```python
let action = ["users", "list"]
client.action(schema, action).then(function(result) {
    // Return value is in 'result'
})
```

包括参数：
```python
let action = ["users", "create"]
let params = {username: "example", email: "example@example.com"}
client.action(schema, action, params).then(function(result) {
    // Return value is in 'result'
})
```

处理错误：
```python
client.action(schema, action, params).then(function(result) {
    // Return value is in 'result'
}).catch(function (error) {
    // Error value is in 'error'
})
```

## 使用节点安装 (Installation with node)
`coreapi` 包在 NPM 上可用。
```python
$ npm install coreapi
$ node
const coreapi = require('coreapi')
```

您可能希望直接在代码库中包含 API 模式，方法是从 `schema.js` 资源中复制它，或者异步加载模式。例如：
```python
let client = new coreapi.Client()
let schema = null
client.get("https://api.example.org/").then(function(data) {
    // Load a CoreJSON API schema.
    schema = data
    console.log('schema loaded')
})
```
