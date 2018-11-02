# 发行说明 (Release Notes)
尽早发布，经常发布 — Eric S. Raymond, [The Cathedral and the Bazaar](http://www.catb.org/~esr/writings/cathedral-bazaar/cathedral-bazaar/ar01s04.html)

## 版本 (Versioning)
次要版本号 (0.0.x) 用于 API 兼容的更改。您应该能够在小点版本之间进行升级，而无需更改任何其他代码。

中等版本号 (0.x.0) 可能包含 API 更改，这与[弃用策略](https://www.django-rest-framework.org/community/release-notes/#deprecation-policy)一致。在中点版本之间进行升级之前，您应该仔细阅读发布说明。

主要版本号 (x.0.0) 保留给重要的项目里程碑。

## 弃用策略 (Deprecation policy)
REST framework 发布遵循正式的弃用策略，这与 Django 的弃用策略一致。

1.0 版本中出现的功能的弃用时间表如下：

- 版本 1.1 将保持与 1.0 完全向后兼容，但如果您使用将被弃用的功能，则会引发 `PendingDeprecationWarning` 警告。默认情况下，这些警告是静默的，但是当您准备开始迁移任何所需的更改时，可以显式启用这些警告。例如，如果您使用 `python -Wd manage.py test` 开始运行测试，则您将收到有关您需要进行的任何API更改的警告。
- 版本 1.2 会将这些警告升级为 `DeprecationWarning`，默认情况下声音很大。
- 版本 1.3 将完全删除被弃用的 API。

注意，按照 Django 的策略，文档中没有提到的框架的任何部分通常都应该被视为私有 API，并且可能会发生更改。

## 升级 (Upgrading)
要将 Django REST framework 升级到最新版本，请使用 pip：
```python
pip install -U djangorestframework
```

您可以使用 `pip show` 确定当前安装的版本：
```python
pip show djangorestframework
```

***

## 3.9.x 系列 (3.9.x series)
### 3.9.0
**日期**：2018年10月18日

- ViewSet 额外操作的改进 [#5605](https://github.com/encode/django-rest-framework/issues/5605)
- 修复 ViewSet 后缀的 `action` 支持 [#6081](https://github.com/encode/django-rest-framework/issues/6081)
- 允许 `action` 文档部分 [#6060](https://github.com/encode/django-rest-framework/issues/6060)
- 弃用 `Router.register` `base_name` 参数以支持 `basename`。[#5990](https://github.com/encode/django-rest-framework/issues/5990)
- 弃用 `Router.get_default_base_name` 方法以支持 `Router.get_default_basename`。[#5990](https://github.com/encode/django-rest-framework/issues/5990)
- 将 `CharField` 更改为不允许空字节。[#6073](https://github.com/encode/django-rest-framework/issues/6073) 要恢复旧的行为，子类化 `CharField` 并从验证器中删除 `ProhibitNullCharactersValidator`。`python class NullableCharField(serializers.CharField): def __init__(self, *args, **kwargs): super().__init__(*args, **kwargs) self.validators = [v for v in self.validators if not isinstance(v, ProhibitNullCharactersValidator)]`
- 添加 `OpenAPIRenderer` 和 `generate_schema` 管理命令。[#6229](https://github.com/encode/django-rest-framework/issues/6229)
- 默认情况下添加 OpenAPIRenderer，并添加模式文档。[#6233](https://github.com/encode/django-rest-framework/issues/6233)
- 允许组合权限 [#5753](https://github.com/encode/django-rest-framework/issues/5753)
- 在 Django 2.1 中允许可空的 BooleanField [#6183](https://github.com/encode/django-rest-framework/issues/6183)
- 添加 Python 3.7 支持的测试 [#6141](https://github.com/encode/django-rest-framework/issues/6141)
- 使用 Django 2.1 最终版本进行测试。[#6109](https://github.com/encode/django-rest-framework/issues/6109)
- 将 djangorestframework-datatables 添加到第三方软件包 [#5931](https://github.com/encode/django-rest-framework/issues/5931)
- 更改 ISO 8601 日期格式以排除年/月 [#5936](https://github.com/encode/django-rest-framework/issues/5936)
- 将所有 pypi.python.org 网址更新到 pypi.org [#5942](https://github.com/encode/django-rest-framework/issues/5942)
- 确保 html 表单 (多部分表单数据) 尊重可选字段 [#5927](https://github.com/encode/django-rest-framework/issues/5927)
- 允许哈希 ErrorDetail。[#5932](https://github.com/encode/django-rest-framework/issues/5932)
- 解析 JSONField 的正确模式 [#5878](https://github.com/encode/django-rest-framework/issues/5878)
- 使用安全渲染描述 (来自 help_text) [#5869](https://github.com/encode/django-rest-framework/issues/5869)
- 从 deault_error_message 中删除了输入值 [#5881](https://github.com/encode/django-rest-framework/issues/5881)
- 在 DurationField 中添加 min_value/max_value 支持 [#5643](https://github.com/encode/django-rest-framework/issues/5643)
- 修复实例被覆盖在 pk-only 优化 try/except 块 [#5747](https://github.com/encode/django-rest-framework/issues/5747)
- 当值为 None 时，修复了项过滤器中的 AttributeError [#5981](https://github.com/encode/django-rest-framework/issues/5981)
- 修复了 Javascript `e.indexOf` 不是函数错误 [#5982](https://github.com/encode/django-rest-framework/issues/5982)
- 修复额外操作的模式 [#5992](https://github.com/encode/django-rest-framework/issues/5992)
- 改进了 get_error_detail 以使用 error_dict/error_list [#5785](https://github.com/encode/django-rest-framework/issues/5785)
- Admin 渲染器中的改进 URL [#5988](https://github.com/encode/django-rest-framework/issues/5988)
- 将 “社区” 部分添加到文档，进行小清理 [#5993](https://github.com/encode/django-rest-framework/issues/5993)
- 将监护人的进口从兼容性移出 [#6054](https://github.com/encode/django-rest-framework/issues/6054)
- 弃用 `DjangoObjectPermissionsFilter` 类，移动到 `djangorestframework-guardian` 包。[#6075](https://github.com/encode/django-rest-framework/issues/6075)
- 放弃 Django 1.10 支持 [#5657](https://github.com/encode/django-rest-framework/issues/5657)
- 只捕获对象查找的 TypeError/ValueError [#6028](https://github.com/encode/django-rest-framework/issues/6028)
- 在 ModelSerializer 中处理没有 .objects 管理器的模型。[#6111](https://github.com/encode/django-rest-framework/issues/6111)
- 改进 ModelSerializer.create() 错误消息。[#6112](https://github.com/encode/django-rest-framework/issues/6112)
- 修复使用 django 1.11.6+ 会话认证时 CSRF cookie 检查失败 [#6113](https://github.com/encode/django-rest-framework/issues/6113)
- 更新 JWT 文档。[#6138](https://github.com/encode/django-rest-framework/issues/6138)
- 修复 autoescape 没有传递到 urlize_quoted_links 过滤器 [#6191](https://github.com/encode/django-rest-framework/issues/6191)
