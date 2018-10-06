# 使用 AJAX，CSRF 和 CORS (Working with AJAX, CSRF & CORS)
“仔细查看您自己网站上可能存在的 CSRF / XSRF 漏洞。它们是最糟糕的一种漏洞——很容易被攻击者利用，但对软件开发人员来说却不那么直观易懂，至少在您被攻击之前是这样。” —— [Jeff Atwood](https://blog.codinghorror.com/preventing-csrf-and-xsrf-attacks/)

## Javascript 客户端 (Javascript clients)
如果您正在构建 JavaScript 客户端来与 Web API 进行交互，那么您需要考虑客户端是否可以使用与网站其他部分相同的身份验证策略，并确定是否需要使用 CSRF 令牌或 CORS 标头。

在与它们交互的 API 相同的上下文中发出的 AJAX 请求通常使用 `SessionAuthentication`。这确保了一旦用户登录，就可以使用与网站其余部分相同的基于会话的身份验证来对发出的任何 AJAX 请求进行身份验证。

在与其通信的 API 不同的站点上进行的 AJAX 请求通常需要使用非基于会话的身份验证方案，例如 `TokenAuthentication`。

## CSRF 防护 (CSRF protection)
[跨站点请求伪造](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))防护是一种防范特定类型攻击的机制，当用户未注销网站并继续拥有有效会话时，可能会发生此类攻击。在这种情况下，恶意站点可能能够在登录会话的上下文内对目标站点执行操作。

为了防范这类攻击，您需要做两件事：

1. 确保 “安全” 的 HTTP 操作 (如 `GET`、`HEAD` 和 `OPTIONS`) 不能用于更改任何服务器端状态。
2. 确保任何 “不安全” 的 HTTP 操作 (如 `POST`、`PUT`、`PATCH` 和 `DELETE`) 始终需要有效的 CSRF 令牌。

如果您正在使用 `SessionAuthentication`，则需要为任何 `POST`，`PUT`，`PATCH` 或 `DELETE` 操作包含有效的 CSRF 令牌。

为了发出 AJAX 请求，您需要在 HTTP 标头中包含 CSRF 令牌，如 [Django 文档中所述](https://docs.djangoproject.com/en/stable/ref/csrf/#ajax)。

## CORS
[跨域资源共享](https://www.w3.org/TR/cors/)是一种允许客户端与托管在不同域上的 API 进行交互的机制。CORS 的工作原理是要求服务器包含一组特定的标头，允许浏览器确定是否以及何时允许跨域请求。

在 REST framework 中处理 CORS 的最佳方法是在中间件中添加所需的响应标头。这确保了透明地支持 CORS，而无需更改视图中的任何行为。

[Otto Yiu](https://github.com/ottoyiu/) 维护着 [django-cors-headers](https://github.com/ottoyiu/django-cors-headers/) 包，已知它可以与 REST framework API 一起正常工作。
