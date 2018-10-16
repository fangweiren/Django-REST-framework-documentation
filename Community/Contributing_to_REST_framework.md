# 对 REST framework 的贡献 (Contributing to REST framework)
世界真的只能一次改变一件事。艺术正在挑选那件作品。—— [Tim Berners-Lee](https://www.w3.org/People/Berners-Lee/FAQ.html)

您可以通过多种方式为 Django REST framework 做出贡献。我们希望它是一个由社区主导的项目，所以请参与并帮助塑造项目的未来。

## 社区 (Community)
您可以做的最重要的事情是尽可能积极地参与推动 REST framework 项目。代码贡献作为参与项目的主要方式经常被高估，我们认为情况并非如此。

如果您使用 REST framework，我们希望您能够畅所欲言地讲述您的经验 - 您可以考虑撰写关于使用 REST framework 的博客文章，或者发布关于使用特定 JavaScript 框架构建项目的教程。初学者的经验可能特别有用，因为您将处于最佳位置，以评估哪些 REST framework 更难以理解和使用。

您可以帮助社区向前发展的其他非常棒的方法包括帮助回答讨论组的问题，或者在 StackOverflow 上设置一个电子邮件警告，这样您就可以得到关于 `django-rest-framework` 标签的任何新问题的通知。

在回答问题时，请确保通过超链接尽可能地帮助未来的贡献者找到相关的线程和票据，并在相关时包含这些项目的反向链接。

## 行为守则 (Code of conduct)
请保持礼貌和专业的语气。对于一些用户来说，关于REST framework 邮件列表或票据跟踪器的讨论可能是他们与开源社区的首次接触。第一印象很重要，所以让我们试着让每个人都感到受欢迎。

注意你选择的语言。例如，在一个男性占主导地位的环境中，以 “嘿，伙伴” 开头的帖子可能会被无意中排除。在这种情况下，使用中性语言同样容易，也更加包容。

[Django 行为准则](https://www.djangoproject.com/conduct/)为参与社区论坛提供了更全面的指导方针。

# 问题 (Issues)
如果你能确保在正确的频道上解决问题，这真的很有帮助。使用问题应针对讨论组。应在 GitHub [问题跟踪器](https://github.com/encode/django-rest-framework/issues?state=open)上引发功能请求、错误报告和其他问题。

关于良好问题报告的一些提示：

- 在描述问题时，尝试根据您认为需要更改的行为而不是您认为需要更改的代码来表达您的标签。
- 首先搜索问题列表以查找相关项目，并确保在报告问题之前运行最新版本的 REST framework。
- 如果报告错误，则尝试包含带有失败测试用例的 pull 请求。这将有助于我们快速确定是否存在有效问题，并确保在有问题时更快地解决问题。
- 功能请求通常会被关闭，建议它们在核心 REST framework 库之外实现。保持作为第三方库实现的新功能请求允许我们降低 REST framework 的维护开销，因此重点可以放在持续稳定性，错误修正和优秀文档上。
- 结束问题并不一定意味着讨论的结束。如果您认为您的问题已被错误关闭，请解释原因，我们会考虑是否需要重新打开。

## 分类问题 (Triaging issues)
参与分类传入问题是开始贡献的好方法。进入票据跟踪器的每一张票都需要进行审查，以确定下一步应该采取什么步骤。任何人都可以帮忙解决这个问题，你只需要愿意

- 读完这张票 - 它是否有意义，是否缺少任何能更好地解释它的上下文？
- 票据是否在正确的地方报告，是否更适合在讨论组进行讨论？
- 如果票据是错误报告，你能复制它吗？您是否能够编写一个演示该问题且可以作为拉取请求提交的失败测试用例？
- 如果票据是功能请求，您是否同意，并且功能请求是否可以实现为第三方程序包？
- 如果一张票没有多少活动，并且它解决了您需要的一些问题，那么请在这张票上进行评论，并尝试找出让它再次运行所需要的东西。

# 发展 (Development)
要开始开发 Django REST framework，首先在 GitHub 上从 [Django REST Framework repo](https://github.com/encode/django-rest-framework) 创建一个 Fork。

然后克隆您的 fork。clone 命令将如下所示，使用您的 GitHub 用户名而不是 YOUR-USERNAME：
```python
git clone https://github.com/YOUR-USERNAME/Spoon-Knife
```

有关更多帮助，请参阅 GitHub 的 [Fork a Repo](https://help.github.com/articles/fork-a-repo/) 指南。

更改应该大致遵循 [PEP 8](https://www.python.org/dev/peps/pep-0008/) 样式约定，我们建议您设置编辑器以自动指示不符合的样式。

## 测试 (Testing)
要运行测试，请克隆存储库，然后：
```python
# Setup the virtual environment
virtualenv env
source env/bin/activate
pip install django
pip install -r requirements.txt

# Run the tests
./runtests.py
```

### 测试选项 (Test options)
使用更简洁的输出样式运行。
```python
./runtests.py -q
```

使用更简洁的输出样式运行测试，没有覆盖，没有 flake8。
```python
./runtests.py --fast
```

不要运行 flake8 代码检查。
```python
./runtests.py --nolint
```

只运行 flake8 代码检查，不要运行测试。
```python
./runtests.py --lintonly
```

为给定的测试用例运行测试。
```python
./runtests.py MyTestCase
```

为给定的测试方法运行测试。
```python
./runtests.py MyTestCase.test_this_method
```

为给定的测试方法以较短形式运行测试。
```python
./runtests.py test_this_method
```

注意：测试用例和测试方法匹配是模糊的，有时会运行包含与给定命令行输入的部分字符串匹配的其他测试。

### 针对多种环境运行 (Running against multiple environments)
您还可以使用优秀的 [tox](https://tox.readthedocs.io/en/latest/) 测试工具对所有受支持的 Python 和 Django 版本运行测试。全局安装 `tox`，然后运行：
```python
tox
```

## 拉取请求 (Pull requests)
尽早发出拉取请求是个好主意。拉取请求代表讨论的开始，并不一定需要是最终的完成提交。

在开始处理拉取请求之前，最好先建立一个新分支。这意味着您以后可以切换回处理另一个单独的问题，而不会干扰正在进行的拉取请求。

记住，如果你有一个未完成的拉取请求，那么将新的提交推送到你的GitHub仓库也会自动更新拉取请求。

GitHub 有关处理拉取请求的文档可在[此处获得](https://help.github.com/articles/using-pull-requests)。

总是在提交拉取请求之前运行测试，理想情况下运行 `tox`，以检查您的修改是否兼容 Python 2 和 Python 3，以及它们是否在所有受支持的 Django 版本上正确运行。

完成拉取请求后，请查看 GitHub 界面中的 Travis 构建状态，并确保测试按预期运行。

<div align=center><img src="https://www.django-rest-framework.org/img/travis-status.png"/></div>

*上图：Travis 构建通知*

## 管理兼容性问题 (Managing compatibility issues)
有时，为了确保您的代码适用于各种不同版本的 Django，Python 或第三方库，您需要根据环境运行稍微不同的代码。以这种方式分支的任何代码都应该隔离到 `compat.py` 模块中，并且应该提供代码库的其余部分可以使用的单个公共接口。

# 文档 (Documentation)
REST framework 的文档是从 [docs 目录](https://github.com/encode/django-rest-framework/tree/master/docs)中的 [Markdown](https://daringfireball.net/projects/markdown/basics) 源文件构建的。

有许多伟大的 Markdown 编辑器使得使用文档非常容易。[Mac 的 Mou 编辑器](http://mouapp.com/)是强烈推荐的编辑器之一。

## 构建文档 (Building the documentation)
要构建文档，请使用 `pip install mkdocs` 安装 MkDocs，然后运行以下命令。
```python
mkdocs build
```

这将把文档构建到 `site` 目录中。

您可以使用 `serve` 命令构建文档并在浏览器窗口中打开预览。
```python
mkdocs serve
```

## 语言样式 (Language style)
文档应采用美式英语。文档的基调是非常重要的 - 尽可能坚持简单、朴素、客观和平衡的风格。

其他一些提示：

- 保持段落相当短。
- 不要使用 'e.g.' 等缩写词而是使用长形式，例如 'For example'。

## Markdown 样式 (Markdown style)
在处理文档时，您应遵循几个约定。

##### 1.Headers
标头应该使用散列样式。例如：
```python
### Some important topic
```

不应使用下划线样式。**不要这样做：**
```python
Some important topic
====================
```

##### 2. Links
链接应该始终使用引用样式，并将引用的超链接保留在文档的末尾。
```python
Here is a link to [some other thing][other-thing].

More text...

[other-thing]: http://example.com/other/thing
```

此样式有助于保持文档源的一致性和可读性。

如果要超链接到另一个 REST framework 文档，则应使用相对链接，并链接到 `.md` 后缀。例如：
```python
[authentication]: ../api-guide/authentication.md
```

以此样式链接意味着您可以单击 Markdown 编辑器中的超链接以打开引用的文档。构建文档后，这些链接将转换为 HTML 页面的常规链接。

##### 3. Notes 
如果你想吸引注意或警告，使用一对封闭线，像这样：
```python
---

**Note:** A useful documentation note.

---
```
