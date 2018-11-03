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

## 3.8.x 系列 (3.8.x series)
### 3.8.2
**日期**：2018年4月6日

- 修复 `read_only` + `default` `unique_together` 验证。[#5922](https://github.com/encode/django-rest-framework/issues/5922)
- authtoken.views 从 rest_framework.compat 导入 coreapi，而不是直接导入。[#5921](https://github.com/encode/django-rest-framework/issues/5921)
- 文档：向 Route 添加缺失的参数 'detail' [#5920](https://github.com/encode/django-rest-framework/issues/5920)

### 3.8.1
**日期**：2018年4月4日

- 在路由装饰器中使用旧的 `url_name` 行为 [#5915](https://github.com/encode/django-rest-framework/issues/5915)
- 对于 `list_route` 和 `detail_route`，维护 `url_name` 的旧行为，基于 `url_path` 而不是函数名。

### 3.8.0
**日期**：2018年4月3日

- **突破更改**：改变 `read_only` 加上 `default` 行为。[#5886](https://github.com/encode/django-rest-framework/issues/5886)

  `read_only` 字段现在将**始终**从可写字段中排除。

  以前带有 `default` 值的 `read_only` 字段将使用 `default` 值来创建和更新操作。

  为了维护旧行为，您可能需要在视图中调用 `save()` 时传递 `read_only` 字段的值：
  ```python
  def perform_create(self, serializer):
      serializer.save(owner=self.request.user)
  ```

  或者，您可以根据需要在序列化器上重写 `save()` 或 `create()` 或 `update()`。

- 当 required=False 时纠正 allow_null 行为 [#5888](https://github.com/encode/django-rest-framework/issues/5888)

  没有显式的 `default`，`allow_null` 意味着输出序列化的默认值为 `null`。以前这些字段在只读或不需要时被跳过。

  如果您依赖于从传出表示中排除的此类字段，则**可能向后兼容性中断**。为了恢复旧行为，您可以覆盖 `data` 以便在 `None` 时排除该字段。

  例如：
  ```python
  @property
  def data(self):
      """
      Drop `maybe_none` field if None.
      """
      data = super().data
      if 'maybe_none' in data and data['maybe_none'] is None:
          del data['maybe_none']
      return data
  ```

- 重构动态路由生成，提高视图集动作自省能力。[#5705](https://github.com/encode/django-rest-framework/issues/5705)
  `ViewSets` 已经被提供了新的属性和方法，允许它自省它的动作集和当前动作的细节。
	- 将 `list_route` 和 `detail_route` 合并为 `action` 装饰器。
	- 使用 `.get_extra_actions()` 在 `ViewSet` 上获取所有额外的操作。
	- 额外操作现在在装饰方法上设置 `url_name` 和 `url_path`。
	- `url_name` 现在基于函数名而不是 `url_path`，因为路径并不总是合适的 (例如，捕获路径中的参数)。
	- 通过 `.reverse_action()` 方法启用操作 url 反转 (在 3.7.4 中添加)
	- 反向调用示例：`self.reverse_action(self.custom_action.url_name)`
	- 添加 `detail` initkwarg，以指示当前操作是否在集合或单个实例上操作。

  额外的变化：
	- 弃用 `list_route` 和 `detail_route` 以使用带有 `detail` 布尔值的 `action` 装饰器。
	- 弃用动态列表/详细信息路由变体，支持带有 `detail` 布尔值的 `DynamicRoute`。
	- 重构路由器的动态路由生成。
	- `list_route` 和 `detail_route` 维护 `url_name` 的旧行为，基于 `url_path` 而不是函数名。

- 修复 3.7.4 发行说明的格式 [#5704](https://github.com/encode/django-rest-framework/issues/5704)
- 文档：更新 DRF 可写嵌套序列化器参考 [#5711](https://github.com/encode/django-rest-framework/issues/5711)
- 文档：修复认证 URL 示例中的拼写错误。[#5713](https://github.com/encode/django-rest-framework/issues/5713)
- 改进复合字段子错误 [#5655](https://github.com/encode/django-rest-framework/issues/5655)
- 禁用 dict/list 字段的 HTML 输入 [#5702](https://github.com/encode/django-rest-framework/issues/5702)
- 修复 HostNameVersioning 文档中的拼写错误 [#5709](https://github.com/encode/django-rest-framework/issues/5709)
- 使用 rsplit 获取导入的模块和类名 [#5712](https://github.com/encode/django-rest-framework/issues/5712)
- 形式化 URLPatternsTestCase [#5703](https://github.com/encode/django-rest-framework/issues/5703)
- 添加异常翻译测试 [#5700](https://github.com/encode/django-rest-framework/issues/5700)
- 测试静态文件 [#5701](https://github.com/encode/django-rest-framework/issues/5701)
- 将 drf-yasg 添加到文档和架构第三方软件包中 [#5720](https://github.com/encode/django-rest-framework/issues/5720)
- 删除未使用的 `compat._resolve_model()` [#5733](https://github.com/encode/django-rest-framework/issues/5733)
- 终止不支持 Python 3.2 的兼容解决方法 [#5734](https://github.com/encode/django-rest-framework/issues/5734)
- 喜欢 `iter(dict)` 胜过 `iter(dict.keys())` [#5736](https://github.com/encode/django-rest-framework/issues/5736)
- 将 `python_requires` 参数传递给 setuptools [#5739](https://github.com/encode/django-rest-framework/issues/5739)
- 从文档中删除未使用的链接 [#5735](https://github.com/encode/django-rest-framework/issues/5735)
- 当可用的链接在文档，首选 https 协议 [#5729](https://github.com/encode/django-rest-framework/issues/5729)
- 添加 HStoreField，postgres 字段测试 [#5654](https://github.com/encode/django-rest-framework/issues/5654)
- 始终完全限定文档中的 ValidationError [#5751](https://github.com/encode/django-rest-framework/issues/5751)
- 从 ManualSchema 中删除无法访问的代码 [#5766](https://github.com/encode/django-rest-framework/issues/5766)
- 允许自定义 API 文档代码示例 [#5752](https://github.com/encode/django-rest-framework/issues/5752)
- 更新文档以使用 `pip show` [#5757](https://github.com/encode/django-rest-framework/issues/5757)
- 在模板中加载 'static' 而不是 'staticfiles' [#5773](https://github.com/encode/django-rest-framework/issues/5773)
- 修复了 `fields` 文档中的拼写错误 [#5783](https://github.com/encode/django-rest-framework/issues/5783)
- 在文档中请参考 “NamespaceVersioning” 而不是 “NamespacedVersioning” [#5754](https://github.com/encode/django-rest-framework/issues/5754)
- ErrorDetail：添加 `__eq__`/`__ne__` 和 `__repr__` [#5787](https://github.com/encode/django-rest-framework/issues/5787)
- 更换文档中 `background-attachment: fixed` [#5777](https://github.com/encode/django-rest-framework/issues/5777)
- 使 404 和 403 响应与 `exceptions.APIException` 输出一致 [#5763](https://github.com/encode/django-rest-framework/issues/5763)
- 对 API 文档的小修正：模式 [#5796](https://github.com/encode/django-rest-framework/issues/5796)
- 修复 PrimaryKeyRelatedField 的模式生成 [#5764](https://github.com/encode/django-rest-framework/issues/5764)
- 将序列化器 DictField 表示为模式中的对象 [#5765](https://github.com/encode/django-rest-framework/issues/5765)
- 添加了文档示例重新实现 ObtainAuthToken [#5802](https://github.com/encode/django-rest-framework/issues/5802)
- 将模式添加到 ObtainAuthToken 视图 [#5676](https://github.com/encode/django-rest-framework/issues/5676)
- 修复请求表单数据处理 [#5800](https://github.com/encode/django-rest-framework/issues/5800)
- 修正 authtoken 视图导入 [#5818](https://github.com/encode/django-rest-framework/issues/5818)
- 更新 pytest，isort [#5815](https://github.com/encode/django-rest-framework/issues/5815) [#5817](https://github.com/encode/django-rest-framework/issues/5817) [#5894](https://github.com/encode/django-rest-framework/issues/5894)
- 修复了对非 ISO8601 日期时间的活跃时区处理。[#5833](https://github.com/encode/django-rest-framework/issues/5833)
- 当值为 0 时，使 TemplateHTMLRenderer 渲染 IntegerField 输入。[#5834](https://github.com/encode/django-rest-framework/issues/5834)
- 修正了教程说明中的端点 [#5835](https://github.com/encode/django-rest-framework/issues/5835)
- 将 Django Rest Framework 角色过滤器添加到第三方软件包 [#5809](https://github.com/encode/django-rest-framework/issues/5809)
- 使用静态资产的单一副本。更新 jQuery [#5823](https://github.com/encode/django-rest-framework/issues/5823)
- 将三元条件更改为符合 PEP308 [#5827](https://github.com/encode/django-rest-framework/issues/5827)
- 添加链接到 'A Todo List API with React' 和 'Blog API' 教程 [#5837](https://github.com/encode/django-rest-framework/issues/5837)
- 修复 ModelSerializer 中的注释错误 [#5844](https://github.com/encode/django-rest-framework/issues/5844)
- 将 admin 添加到已安装的应用程序以避免测试失败。[#5870](https://github.com/encode/django-rest-framework/issues/5870)
- 修复了 SimpleMetadata 中 UUIDField 的模式。[#5872](https://github.com/encode/django-rest-framework/issues/5872)
- 修正路由器上的文档包括命名空间。[#5843](https://github.com/encode/django-rest-framework/issues/5843)
- 使用模型对象测试虚线源默认值。[#5880](https://github.com/encode/django-rest-framework/issues/5880)
- 允许遍历可空相关字段 [#5849](https://github.com/encode/django-rest-framework/issues/5849)
- 补充：教程：使用 React 的 Django REST (Django 2.0) [#5891](https://github.com/encode/django-rest-framework/issues/5891)
- 添加 `LimitOffsetPagination.get_count` 以允许方法覆盖 [#5846](https://github.com/encode/django-rest-framework/issues/5846)
- 不要在元数据中显示隐藏字段 [#5854](https://github.com/encode/django-rest-framework/issues/5854)
- 启用 OrderingFilter 以处理 “ordering” 字段的空元组 (或列表)。[#5899](https://github.com/encode/django-rest-framework/issues/5899)
- 添加了通用的 500 和 400 JSON 错误处理程序。[#5904](https://github.com/encode/django-rest-framework/issues/5904)
