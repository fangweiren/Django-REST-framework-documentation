# 项目管理 (Project management)
“没有人可以吹响交响乐；它需要整个管弦乐队演奏它” —— Halford E. Luccock

本文档概述了 REST framework 的项目管理流程。

其目的是确保项目具有较高的 [“公共要素”](https://en.wikipedia.org/wiki/Bus_factor)，并且在可预见的未来可以继续得到良好的支持。欢迎提出改进我们流程的建议。

***

## 维护团队 (Maintenance team)
我们有一个季度维护周期，新成员可以加入维护团队。我们目前将团队规模限制为5名成员，并且可能会鼓励人们离开团队一段时间以允许新成员参与。

#### 现在的团队 (Current team)
[2015 年第四季度维护团队](https://github.com/encode/django-rest-framework/issues/2190)：

- [@tomchristie](https://github.com/encode/)
- [@xordoquy](https://github.com/xordoquy/) (发布经理。)
- [@carltongibson](https://github.com/carltongibson/)
- [@kevin-brown](https://github.com/kevin-brown/)
- [@jpadilla](https://github.com/jpadilla/)

#### 维护周期 (Maintenance cycles)
每个维护周期都由使用 `Process` 标签打开的问题启动。

- 要考虑维护者角色，只需对该问题进行评论。
- 现有成员必须通过勾选自己的名字明确地选择进入下一个周期。
- 关于即将到来的团队的最终决定将由 `@tomchristie` 做出。

维护团队的成员将作为协作者添加到存储库中。

以下模板应用于问题描述，并作为选择团队的正式流程。
```python
This issue is for determining the maintenance team for the *** period.

Please see the [Project management](http://www.django-rest-framework.org/topics/project-management/) section of our documentation for more details.

---

#### Renewing existing members.

The following people are the current maintenance team. Please checkmark your name if you wish to continue to have write permission on the repository for the *** period.

- [ ] @***
- [ ] @***
- [ ] @***
- [ ] @***
- [ ] @***

---

#### New members.

If you wish to be considered for this or a future date, please comment against this or subsequent issues.

To modify this process for future maintenance cycles make a pull request to the [project management](http://www.django-rest-framework.org/topics/project-management/) documentation.
```

#### 团队成员的责任 (Responsibilities of team members)
团队成员有以下责任。

- 关闭无效或已解决的票证。
- 在票据中添加分类标签和里程碑。
- 合并最终的拉取请求。
- 使用 `mkdocs gh-deploy` 构建和部署文档。
- 构建和更新包含的翻译包。

维护者的进一步说明：

- 代码更改应以拉取请求的形式出现 - 不要直接推送到 master。
- 维护者通常不应合并他们自己的拉取请求。
- 每个问题/拉取请求在分类后应该只有一个标签。
- 使用 [is:open no:label](https://github.com/encode/django-rest-framework/issues?q=is%3Aopen+no%3Alabel) 搜索未分类的问题。

应该注意的是，积极参与 REST framework 项目显然**不需要成为维护团队的一部分**。几乎问题分类和项目改进的每个导入部分都可以被积极地处理，而不管您在存储库中的协作者状态如何。

***

## 发布过程 (Release process)
每个季度维护周期都会选择发布管理员。

- 管理员应该由 `@tomchristie` 选中。
- 然后，管理员将把维护者角色添加到 PyPI 包中。
- 之前的管理员将从 PyPI 包中删除维护者角色。

我们的 PyPI 版本将由当前版本管理员或 `@tomchristie` 处理。每个版本都应该有一个标记有 `Release` 标签的开放问题，并标记为相应的里程碑。

应使用以下模板来描述问题，并将其用作发布清单。
```python
Release manager is @***.
Pull request is #***.

During development cycle:

- [ ] Upload the new content to be translated to [transifex](http://www.django-rest-framework.org/topics/project-management/#translations).


Checklist:

- [ ] Create pull request for [release notes](https://github.com/encode/django-rest-framework/blob/master/docs/topics/release-notes.md) based on the [*.*.* milestone](https://github.com/encode/django-rest-framework/milestones/***).
- [ ] Update supported versions:
    - [ ] `setup.py` `python_requires` list
    - [ ] `setup.py` Python & Django version trove classifiers
    - [ ] `README` Python & Django versions
    - [ ] `docs` Python & Django versions
- [ ] Update the translations from [transifex](http://www.django-rest-framework.org/topics/project-management/#translations).
- [ ] Ensure the pull request increments the version to `*.*.*` in [`restframework/__init__.py`](https://github.com/encode/django-rest-framework/blob/master/rest_framework/__init__.py).
- [ ] Confirm with @tomchristie that release is finalized and ready to go.
- [ ] Ensure that release date is included in pull request.
- [ ] Merge the release pull request.
- [ ] Push the package to PyPI with `./setup.py publish`.
- [ ] Tag the release, with `git tag -a *.*.* -m 'version *.*.*'; git push --tags`.
- [ ] Deploy the documentation with `mkdocs gh-deploy`.
- [ ] Make a release announcement on the [discussion group](https://groups.google.com/forum/?fromgroups#!forum/django-rest-framework).
- [ ] Make a release announcement on twitter.
- [ ] Close the milestone on GitHub.

To modify this process for future releases make a pull request to the [project management](http://www.django-rest-framework.org/topics/project-management/) documentation.
```

当将版本推到 PyPI 时，请确保您的环境已经从我们的开发 `requirement.txt` 中安装了。以便文档和 PyPI 安装始终是根据固定的一组软件包构建的。

***

## 翻译 (Translations)
维护团队负责管理 REST framework 中的翻译包。通过 [transifex 服务](https://www.transifex.com/projects/p/django-rest-framework/)管理将源字符串翻译成多种语言。

### 管理 Transifex (Managing Transifex)
[官方 Transifex 客户端](https://pypi.org/project/transifex-client/)用于上传和下载翻译到 Transifex。客户端使用 pip 安装：
```python
pip install transifex-client
```

要使用它，您需要登录 Transifex 并输入密码，并且您需要拥有 Transifex 项目的管理访问权限。您需要创建一个包含您的凭据的 `~/.transifexrc` 文件。
```python
[https://www.transifex.com]
username = ***
token = ***
password = ***
hostname = https://www.transifex.com
```

### 上传新的源文件 (Upload new source files)
当任何用户可见字符串发生更改时，应将其上传到 Transifex，以便翻译人员可以开始翻译它们。要做到这一点，只需运行：
```python
# 1. 更新源 django.po 文件，这是美国英语版本。
cd rest_framework
django-admin.py makemessages -l en_US
# 2. 将源 django.po 文件推送到 Transifex。
cd ..
tx push -s
```

推送源文件时，Transifex 将更新资源的源字符串以匹配新源文件中的源字符串。

以下是如何处理旧源文件和新源文件之间的差异：

- 将添加新字符串。
- 修改后的字符串也会被添加。
- 新源文件中不存在的字符串及其翻译将从数据库中删除。如果稍后重新添加源字符串，则 [Transifex 翻译记忆库](http://docs.transifex.com/guides/tm#let-tm-automatically-populate-translations)将自动包含翻译字符串。

### 下载翻译 (Download translations)
翻译人员完成翻译后，他们的工作需要从 Transifex 下载到 REST framework 存储库中。为此，请运行：
```python
# 3. 从 Transifex 中提取已翻译的 django.po 文件。
tx pull -a --minimum-perc 10
cd rest_framework
# 4. 编译所有支持语言的二进制 .mo 文件。
django-admin.py compilemessages
```

***

## 项目要求 (Project requirements)
我们所有的测试要求都固定在精确的版本上，以确保我们的测试运行具有可重复性。我们维护 `requirements` 目录中的需求。需求文件从 `tox.ini` 配置文件中引用，确保我们在测试中使用的包版本具有单一的事实来源。

通常应将包升级视为隔离拉取请求。您可以使用 `pip list --outdated` 来检查是否有更新版本的软件包可用。

***

## 项目所有权 (Project ownership)
PyPI 包由 `@tomchristie` 拥有。作为备份，`@j4mie` 还拥有该软件包的所有权。

如果 `@tomchristie` 不再参与这个项目，那么 `@j4mie` 有责任移交所有权。

#### 未解决的管理和所有权问题 (Outstanding management & ownership issues)
以下问题仍需解决：

- [考虑将 repo 移动到适当的 GitHub 组织中](https://github.com/encode/django-rest-framework/issues/2162)。
- 确保 `@jamie` 具有对 `django-rest-framework.org` 域设置和管理员的备份访问权限。
- [生动的示例](https://restframework.herokuapp.com/) API 的文档所有权。
- [邮件列表](https://groups.google.com/forum/#!forum/django-rest-framework)和 IRC 通道的文档所有权。
- 安全邮件列表的文档所有权和管理。
