# REST, Hypermedia & HATEOAS
你一直用 “REST” 这个词。我不认为这意味着你认为它意味着什么。—— Mike Amundsen，[REST fest 2012主题演讲](https://vimeo.com/channels/restfest/page:2)。

首先，免责声明。“Django REST framework ” 这个名称早在 2011 年初就已经确定，之所以选择这个名称只是为了确保开发人员能够轻松找到该项目。在整个文档中，我们尝试使用更简单和技术上更正确的 “Web API” 术语。

如果您是认真设计超媒体 API 的，那么您应该查看本文档之外的参考资料，以帮助您进行设计选择。

以下属于“必读”类别。

- Roy Fielding 的论文 - [架构样式和基于网络的软件体系结构设计](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)。
- Roy Fielding 的 “[REST API 必须是超文本驱动](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)” 博客文章。
- Leonard Richardson 和 Mike Amundsen 的 [RESTful Web API](http://restfulwebapis.org/)。
- Mike Amundsen 的[使用 HTML5 和 Node 构建超媒体 API](https://www.amazon.com/Building-Hypermedia-APIs-HTML5-Node/dp/1449306578)。
- Steve Klabnik 的[设计超媒体 API](http://designinghypermediaapis.com/)。
- [Richardson 成熟度模型](https://martinfowler.com/articles/richardsonMaturityModel.html)

有关更全面的背景信息，请查看 Klabnik 的 [Hypermedia API 阅读列表](http://blog.steveklabnik.com/posts/2012-02-27-hypermedia-api-reading-list)。

## 使用 REST framework 构建超媒体 API (Building Hypermedia APIs with REST framework)
REST framework 是一个不可知的 Web API 工具包。它确实有助于指导您构建连接良好的 API，并且可以轻松设计适当的媒体类型，但它并不严格执行任何特定的设计风格。

## REST framework 提供了什么 (What REST framework provides.)
不言而喻，REST framework 使构建超媒体 API 成为可能。它提供的可浏览 API 是基于 HTML 构建的 - web 的超媒体语言。

REST framework 还包括[序列化](https://www.django-rest-framework.org/api-guide/serializers/)和[解析器](https://www.django-rest-framework.org/api-guide/parsers/)/[渲染器](https://www.django-rest-framework.org/api-guide/renderers/)组件，可以轻松构建适当的媒体类型，建立连接良好的系统的[超链接关系](https://www.django-rest-framework.org/api-guide/fields/)，以及对[内容协商](https://www.django-rest-framework.org/api-guide/content-negotiation/)的强大支持。

## REST framework 没有提供什么 (What REST framework doesn't provide.)
REST framework 没有做的是默认情况下为您提供机器可读的超媒体格式，如 [HAL](http://stateless.co/hal_specification.html)，[Collection + JSON](http://www.amundsen.com/media-types/collection/)，[JSON API](http://jsonapi.org/) 或 HTML [微格式](http://microformats.org/wiki/Main_Page)，或者能够自动神奇地创建完整的 HATEOAS 样式 API，其中包括基于超媒体的表单描述和语义标记的超链接。这样做将涉及对真正超出框架范围的 API 设计做出自圆其说的选择。
