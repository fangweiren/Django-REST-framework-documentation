# Mozilla Grant
我们最近[获得了 Mozilla 的拨款](https://blog.mozilla.org/blog/2016/04/13/mozilla-open-source-support-moss-update-q1-2016/)，以资助 REST framework 的下一个主要版本。这项工作通过引入支持能够与 REST framework API 动态交互的客户端库，专注于无缝的客户端集成。该框架将提供超媒体或模式端点，这将公开与客户端库交互的可用接口。

此外，我们将构建 Django 频道提供的实时支持，支持和记录如何使用 REST framework 构建实时 API。同样，这将包括在相关客户端库中支持的工作，使构建丰富的交互式应用程序更加容易。

[Core API](https://www.coreapi.org/) 项目将为我们的客户端库支持提供基础，并允许我们支持使用广泛的模式和超媒体格式交互。值得注意的是，这些客户端库也不会紧密耦合到单独的 REST framework API，并且能够与任何公开受支持的模式或超媒体格式的 API 交互。

具体来说，工作包括：

## 客户端库 (Client libraries)
这项工作将包括内置模式和超媒体支持，允许动态客户端库与 API 交互。我还将发布 Python 和 Javascript 客户端库，以及命令行客户端、新的教程部分和进一步的文档。

- REST framework 中的客户端库支持。
- REST framework API 的模式和超媒体支持。
- 测试客户端，允许您编写模拟与 API 交互的客户端库的测试。
- 有关使用客户端库与 REST framework API 交互的新教程部分。
- Python 客户端库。
- JavaScript 客户端库。
- 命令行客户端。

## 实时 API (Realtime APIs)
下一个目标是在 Django 频道提供的实时支持的基础上，添加用于构建实时 API 端点的支持和文档。

- 使用 REST framework 和 Django 频道支持 API 订阅端点。
- 有关使用 REST framework 构建实时 API 端点的新教程部分。
- Python 和 Javascript 客户端库中的实时支持。

## 责任 (Accountability)
为了确保我能够完全专注于确保可持续且资金充足的开源业务，我将于 2016 年 5 月底离开我目前在 [DabApps](https://www.dabapps.com/) 的职位。

我已经成立了一家英国有限公司 [Encode](https://www.encode.io/)，它将作为 REST framework 背后的商业实体。我将每月发布来自 Encode 的进展报告，包括 Mozilla 的拨款，以及通过 [REST framework 付费计划](https://www.django-rest-framework.org/community/funding/)资助的开发时间。

## 保持与我们每月进度报告的同步…
