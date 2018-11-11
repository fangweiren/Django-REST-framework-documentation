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

## 3.7.x 系列 (3.7.x series)
### 3.7.7
**日期**：2017年12月21日

- 修复将 *.mo 现场文件包含到打包中的拼写错误。[#5697](https://github.com/encode/django-rest-framework/issues/5697)，[#5695](https://github.com/encode/django-rest-framework/issues/5695)

### 3.7.6
**日期**：2017年12月21日

- 将缺少的 *.ico 图标文件添加到包装中。

### 3.7.5
**日期**：2017年12月21日

- 将缺少的 *.woff2 字体文件添加到打包中。[#5692](https://github.com/encode/django-rest-framework/issues/5692)
- 将缺少的 *.mo 语言环境文件添加到打包中。[#5695](https://github.com/encode/django-rest-framework/issues/5695)，[#5696](https://github.com/encode/django-rest-framework/issues/5696)

### 3.7.4
**日期**：2017年12月20日

- 模式：提取 `manual_fields` 处理的方法 [#5633](https://github.com/encode/django-rest-framework/issues/5633)

  允许更轻松地自定义 `manual_fields` 处理，例如，提供每个方法的手动字段。`AutoSchema` 添加了 `get_manual_fields` 作为预期的覆盖点，以及实用的方法 `update_fields`，用于处理列表中按名称字段替换，通常，您不应该覆盖该列表。

  注意：`AutoSchema.__init__` 现在确保 `manual_fields` 是一个列表。以前可能在内部存储为 `None`。

- 删除 ulrparse 兼容性垫片；用六个代替 [#5579](https://github.com/encode/django-rest-framework/issues/5579)
- 为 `TimeDelta.total_seconds()` 删除兼容包装器 [#5577](https://github.com/encode/django-rest-framework/issues/5577)
- 清理整个项目中的所有空白 [#5578](https://github.com/encode/django-rest-framework/issues/5578)
- 兼容清理 [#5581](https://github.com/encode/django-rest-framework/issues/5581)
- 在可浏览的 API 视图中添加 pygments CSS 块 [#5584](https://github.com/encode/django-rest-framework/issues/5584) [#5587](https://github.com/encode/django-rest-framework/issues/5587)
- 从兼容中删除 `set_rollback()` [#5591](https://github.com/encode/django-rest-framework/issues/5591)
- 修复请求主体/POST 访问 [#5590](https://github.com/encode/django-rest-framework/issues/5590)
- 重命名测试以引用正确的问题 [#5610](https://github.com/encode/django-rest-framework/issues/5610)
- 文档修复 [#5611](https://github.com/encode/django-rest-framework/issues/5611) [#5612](https://github.com/encode/django-rest-framework/issues/5612)
- 在文档和代码中删除对不受支持的 Django 版本的引用 [#5602](https://github.com/encode/django-rest-framework/issues/5602)
- 测试序列化器排除已声明的字段 [#5599](https://github.com/encode/django-rest-framework/issues/5599)
- 修复了过滤后端的模式生成 [#5613](https://github.com/encode/django-rest-framework/issues/5613)
- ModelSerializer 测试的小清理 [#5598](https://github.com/encode/django-rest-framework/issues/5598)
- 重新实现请求属性访问 w/`__getattr__` [#5617](https://github.com/encode/django-rest-framework/issues/5617)
- 修复了 SchemaJSRenderer 渲染无效的 Javascript [#5607](https://github.com/encode/django-rest-framework/issues/5607)
- 使 Django 2.0 支持官方/显式 [#5619](https://github.com/encode/django-rest-framework/issues/5619)
- 对传递的请求参数执行类型检查 [#5618](https://github.com/encode/django-rest-framework/issues/5618)
- 修复请求验证器上的 AttributeError 隐藏 [#5600](https://github.com/encode/django-rest-framework/issues/5600)
- 更新测试要求 [#5626](https://github.com/encode/django-rest-framework/issues/5626)
- 文档：`Serializer._declared_fields` 启用修改序列化器上的字段 [#5629](https://github.com/encode/django-rest-framework/issues/5629)
- 修复打包 [#5624](https://github.com/encode/django-rest-framework/issues/5624)
- 修复 PyPI 的 readme 渲染，向 CI 添加 readme 构造 [#5625](https://github.com/encode/django-rest-framework/issues/5625)
- 更新教程 [#5622](https://github.com/encode/django-rest-framework/issues/5622)
- 带有 `allow_null=True` 的非必需字段不应该表示默认值 [#5639](https://github.com/encode/django-rest-framework/issues/5639)
- 文档：添加 `allow_null` 序列化输出注释 [#5641](https://github.com/encode/django-rest-framework/issues/5641)
- 更新在 tox.ini 中使用 Django 2.0 版本 [#5645](https://github.com/encode/django-rest-framework/issues/5645)
- 修复 `Serializer.data` 在提供无效 `data` 时可浏览 API 渲染 [#5646](https://github.com/encode/django-rest-framework/issues/5646)
- 文档：注意裸露 APIView 上的 AutoSchema 限制 [#5649](https://github.com/encode/django-rest-framework/issues/5649)
- 将 `.basename` 和 `.reverse_action()` 添加到 ViewSet [#5648](https://github.com/encode/django-rest-framework/issues/5648)
- 文档：修复序列化器文档中的拼写错误 [#5652](https://github.com/encode/django-rest-framework/issues/5652)
- 修复 `override_settings` 兼容 [#5668](https://github.com/encode/django-rest-framework/issues/5668)
- 添加 DEFAULT_SCHEMA_CLASS 设置 [#5658](https://github.com/encode/django-rest-framework/issues/5658)
- 添加文档说明重新生成等于 `required=False` 的 BooleanField [#5665](https://github.com/encode/django-rest-framework/issues/5665)
- 添加 'dist' 构建 [#5656](https://github.com/encode/django-rest-framework/issues/5656)
- 修复 docstring 中的拼写错误 [#5678](https://github.com/encode/django-rest-framework/issues/5678)
- 文档：添加 `UNAUTHENTICATED_USER = None` 注释 [#5679](https://github.com/encode/django-rest-framework/issues/5679)
- 从 “记录您的 API” 更新 OPTIONS 示例 [#5680](https://github.com/encode/django-rest-framework/issues/5680)
- 文档：添加 FBVs 关于对象权限的说明 [#5681](https://github.com/encode/django-rest-framework/issues/5681)
- 文档：向 `to_representation` 文档添加示例 [#5682](https://github.com/encode/django-rest-framework/issues/5682)
- 在文档中添加链接到高级 DRF [#5683](https://github.com/encode/django-rest-framework/issues/5683)
- 文档 ViewSet.action [#5685](https://github.com/encode/django-rest-framework/issues/5685)
- 修复模式文档拼写错误 [#5687](https://github.com/encode/django-rest-framework/issues/5687)
- 修复模式生成中的 URL 模式解析 [#5689](https://github.com/encode/django-rest-framework/issues/5689)
- 添加使用 `source ='*'` 的示例到自定义字段文档。[#5688](https://github.com/encode/django-rest-framework/issues/5688)
- 修复 Django 2 path() 路由的 format_suffix_patterns 行为 [#5691](https://github.com/encode/django-rest-framework/issues/5691)

### 3.7.3
**日期**：2017年11月6日

- 修复从 contrib.auth 视图导入的 `AppRegistryNotReady` 错误 [#5567](https://github.com/encode/django-rest-framework/issues/5567)

### 3.7.2
**日期**：2017年11月6日

- 修复了因删除 django.contrib.auth.login()/logout() 视图而导致的 Django 2.1 兼容性问题。[#5510](https://github.com/encode/django-rest-framework/issues/5510)
- 为 TextLexer 添加缺少的导入。[#5512](https://github.com/encode/django-rest-framework/issues/5512)
- 添加缓存的示例和文档 [#5514](https://github.com/encode/django-rest-framework/issues/5514)
- 包括模式生成的日期和日期时间格式 [#5511](https://github.com/encode/django-rest-framework/issues/5511)
- 对 markdown 代码块使用三重反引号 [#5513](https://github.com/encode/django-rest-framework/issues/5513)
- 交互式文档 - 使底部边栏项成为粘性 [#5516](https://github.com/encode/django-rest-framework/issues/5516)
- 澄清分页系统检查 [#5524](https://github.com/encode/django-rest-framework/issues/5524)
- 停止 JSONBoundField 矫直无效的 JSON [#5527](https://github.com/encode/django-rest-framework/issues/5527)
- 让 JSONField 在可浏览 API 中渲染为 textarea [#5530](https://github.com/encode/django-rest-framework/issues/5530)
- 模式：排除 ViewSet 操作的 OPTIONS/HEAD [#5532](https://github.com/encode/django-rest-framework/issues/5532)
- 修复点源的排序 [#5533](https://github.com/encode/django-rest-framework/issues/5533)
- 修正：带有 `allow_null=True` 的字段应该意味着默认的序列化值 [#5518](https://github.com/encode/django-rest-framework/issues/5518)
- 确保 Location 标头是严格的 “str”，而不是子类。[#5544](https://github.com/encode/django-rest-framework/issues/5544)
- 向 api-guide/parsers 中的示例添加导入 [#5547](https://github.com/encode/django-rest-framework/issues/5547)
- 捕获“超出范围”日期时间的OverflowError [#5546](https://github.com/encode/django-rest-framework/issues/5546)
- 将 djangorestframework-quickjson 添加到第三方软件包 [#5549](https://github.com/encode/django-rest-framework/issues/5549)
- 增加 `drf_create_token` 命令的测试覆盖率 [#5550](https://github.com/encode/django-rest-framework/issues/5550)
- 添加 Python 3.6 支持的 trove 分类器。[#5555](https://github.com/encode/django-rest-framework/issues/5555)
- 将 pip 缓存支持添加到 Travis CI 配置 [#5556](https://github.com/encode/django-rest-framework/issues/5556)
- 将 [`wheel`] 部分重命名为 [`bdist_wheel`]，因为前者是遗留的 [#5557](https://github.com/encode/django-rest-framework/issues/5557)
- 修复无效的转义序列弃用警告 [#5560](https://github.com/encode/django-rest-framework/issues/5560)
- 添加交互式文档错误模板 [#5548](https://github.com/encode/django-rest-framework/issues/5548)
- 将舍入参数添加到 DecimalField [#5562](https://github.com/encode/django-rest-framework/issues/5562)
- 修复测试期间捕获的所有 BytesWarning [#5561](https://github.com/encode/django-rest-framework/issues/5561)
- 使用 dict 和 set 字面值，而不是对 dict() 和 set() 的调用 [#5559](https://github.com/encode/django-rest-framework/issues/5559)
- 更改 ImageField 验证模式，使用 DjangoImageField 中的验证器 [#5539](https://github.com/encode/django-rest-framework/issues/5539)
- 修复 Python 2 处理 query_string 中的 unicode 符号 [#5552](https://github.com/encode/django-rest-framework/issues/5552)

### 3.7.1
**日期**：2017年10月16日

- 修复交互式文档始终对请求中的布尔字段使用 false [#5492](https://github.com/encode/django-rest-framework/issues/5492)
- 改善与 Django 2.0 alpha 的兼容性。[#5500](https://github.com/encode/django-rest-framework/issues/5500) [#5503](https://github.com/encode/django-rest-framework/issues/5503)
- 模式命名冲突的改进处理 [#5486](https://github.com/encode/django-rest-framework/issues/5486)
- 添加了其他文档和测试，围绕为点 `source` 字段提供默认值 [#5489](https://github.com/encode/django-rest-framework/issues/5489)

### 3.7.0
**日期**：2017年10月6日

- 修复 `DjangoModelPermissions`，以确保在调用视图的 `get_queryset()` 方法之前进行用户身份验证。作为副作用，这会更改 HTTP 方法权限和身份验证检查的顺序，并且只有在进行身份验证时才会返回 405 响应。如果您想复制旧的行为，请参阅 PR 以获取详细信息。[#5376](https://github.com/encode/django-rest-framework/issues/5376)
- 弃用 `APIView` 和 `api_view` 装饰器上的 `exclude de_from_schema`。酌情地设置 `schema = None` 或 `@schema(None)`。[#5422](https://github.com/encode/django-rest-framework/issues/5422)
- 时区已知的 `DateTimeFields` 现在在序列化期间遵循活动或默认时区，而不总是使用 UTC。[#5435](https://github.com/encode/django-rest-framework/issues/5435)

  解决了不一致问题，实例被提供的 datetime 序列化以进行 `create`，但使用 UTC 进行 `retrieve`。[#3732](https://github.com/encode/django-rest-framework/issues/3732)
  
  如果依赖于 UTC 的 datetime 字符串，则**可能会出现向后兼容中断**。如果需要，让客户端解释 datetimes 或[将默认或活跃时区 (文档) 设置](https://docs.djangoproject.com/en/1.11/topics/i18n/timezones/#default-time-zone-and-current-time-zone)为 UTC。

- 删除了与弃用策略内联的 DjangoFilterBackend。请改用 `django_filters.rest_framework.FilterSet` 和/或 `django_filters.rest_framework.DjangoFilterBackend`。[#5273](https://github.com/encode/django-rest-framework/issues/5273)
- 不要在编码时剥离微秒。与 `datetime` 保持一致。**BC 变化**：以前只有毫秒被编码。[#5440](https://github.com/encode/django-rest-framework/issues/5440)
- 添加了 `STRICT_JSON` 设置 (默认为 `True`) 来引发 Python 的 `json` 模块所接受的扩展浮动值 (`nan`，`inf`，`-inf`) 的异常。**BC 变化**：以前，这些值将转换为相应的字符串。将 `STRICT_JSON` 设置为 `False` 以恢复先前的行为。[#5265](https://github.com/encode/django-rest-framework/issues/5265)
- 在 CursorPaginator 类中添加对 `page_size` 参数的支持 [#5250](https://github.com/encode/django-rest-framework/issues/5250)
- 默认情况下，让 `DEFAULT_PAGINATION_CLASS` 为 `None`。**BC 变化**：如果您**只**设置 `PAGE_SIZE` 来启用分页，则需要添加 `DEFAULT_PAGINATION_CLASS`。以前的默认值是 `rest_framework.pagination.PageNumberPagination`。有一个系统检查警告来捕捉这种情况。如果您在每个视图的基础上设置分页类，那么您可以对此保持沉默。[#5170](https://github.com/encode/django-rest-framework/issues/5170)
- 在模式生成中从 `get_serializer_fields` 捕获 `APIException`。[#5443](https://github.com/encode/django-rest-framework/issues/5443)
- 使用 `include_docs_urls` 时，允许自定义身份验证和权限类 [#5448](https://github.com/encode/django-rest-framework/issues/5448)
- 推迟在验证器上翻译字符串评估。[#5452](https://github.com/encode/django-rest-framework/issues/5452)
- 将 'detail' 参数的默认值添加到 'ValidationError' 异常中 [#5342](https://github.com/encode/django-rest-framework/issues/5342)
- 调整模式 get_filter_fields 规则以匹配框架 [#5454](https://github.com/encode/django-rest-framework/issues/5454)
- 更新测试矩阵以添加 Django 2.0 并删除 Django 1.8 和 1.9 **BC 变化**：这将从 Django REST Framework 支持的版本中删除 Django 1.8 和 Django 1.9。[#5457](https://github.com/encode/django-rest-framework/issues/5457)
- 修复了serializers.ModelField中的弃用警告 [#5058](https://github.com/encode/django-rest-framework/issues/5058)
- 当 `get_queryset` 返回 `None` 时，添加了更明确的错误消息 [#5348](https://github.com/encode/django-rest-framework/issues/5348)
- 修复响应 `data` 描述的文档 [#5361](https://github.com/encode/django-rest-framework/issues/5361)
- 修复 **pycache**/.pyc 在包装时排除 [#5373](https://github.com/encode/django-rest-framework/issues/5373)
- 修复点源的默认值处理 [#5375](https://github.com/encode/django-rest-framework/issues/5375)
- 当将空主体传递给 RequestFactory 时，确保设置了 content_type [#5351](https://github.com/encode/django-rest-framework/issues/5351)
- 修复 ErrorDetail 文档 [#5380](https://github.com/encode/django-rest-framework/issues/5380)
- 允许通用内容表单中的可选内容 [#5372](https://github.com/encode/django-rest-framework/issues/5372)
- 更新了 NullBooleanField 的受支持值 [#5387](https://github.com/encode/django-rest-framework/issues/5387)
- 模型上使用 source 修复 ModelSerializer 自定义命名字段 [#5388](https://github.com/encode/django-rest-framework/issues/5388)
- 修复 MultipleFieldLookupMixin 文档示例，以正确检查对象级权限 [#5398](https://github.com/encode/django-rest-framework/issues/5398)
- 更新 permissions.md 中的 get_object() 示例 [#5401](https://github.com/encode/django-rest-framework/issues/5401)
- 修正 authtoken 管理命令 [#5415](https://github.com/encode/django-rest-framework/issues/5415)
- 修复模式生成 markdown [#5421](https://github.com/encode/django-rest-framework/issues/5421)
- 允许动态设置 ChoiceField.choices [#5426](https://github.com/encode/django-rest-framework/issues/5426)
- 将项目布局添加到快速入门 [#5434](https://github.com/encode/django-rest-framework/issues/5434)
- 在 “render_markdown” templatetag 中重用 “apply_markdown” 函数 [#5469](https://github.com/encode/django-rest-framework/issues/5469)
- 在文档中添加链接到 drf-openapi 包 [#5470](https://github.com/encode/django-rest-framework/issues/5470)
- 添加了用 pygments 高亮显示的文档字符串代码 [#5462](https://github.com/encode/django-rest-framework/issues/5462)
- 修复了名为 `data` 的视图的文档渲染 [#5472](https://github.com/encode/django-rest-framework/issues/5472)
- 文档：澄清 “to_internal_value()” 验证行为 [#5466](https://github.com/encode/django-rest-framework/issues/5466)
- 修复 APIException.str 上丢失的 six.text_type() 调用 [#5476](https://github.com/encode/django-rest-framework/issues/5476)
- 文档 documentation.py [#5478](https://github.com/encode/django-rest-framework/issues/5478)
- 修复模式生成中的命名冲突 [#5464](https://github.com/encode/django-rest-framework/issues/5464)
- 使用请求对象调用 Django 的身份验证函数 [#5295](https://github.com/encode/django-rest-framework/issues/5295)
- 将 coreapi JS 更新为 0.1.1 [#5479](https://github.com/encode/django-rest-framework/issues/5479)
- `is_list_view` 是否能够识别 RetrieveModel... views [#5480](https://github.com/encode/django-rest-framework/issues/5480)
- 删除 Django 1.8 和 1.9 兼容性代码 [#5481](https://github.com/encode/django-rest-framework/issues/5481)
- 从 DefaultRouter 中删除已弃用的模式代码 [#5482](https://github.com/encode/django-rest-framework/issues/5482)
- 重构模式生成以允许按视图自定义。**BC 变化**：`SchemaGenerator.get_serializer_fields` 已被重构为 `AutoSchema.get_serializer_fields` 并删除了 `view` 参数

## 3.6.x 系列 (3.6.x series)
### 3.6.4
**日期**：2017年8月21日

- 忽略 OrderingFilter 的任何无效的查询参数。[#5131](https://github.com/encode/django-rest-framework/issues/5131)
- 在读取大型 JSON 请求时改善内存占用。[#5147](https://github.com/encode/django-rest-framework/issues/5147)
- 修复分页的模式生成。[#5161](https://github.com/encode/django-rest-framework/issues/5161)
- 修复 `HTML_CUTOFF` 设置为 `None` 时的异常。[#5174](https://github.com/encode/django-rest-framework/issues/5174)
- 修复可浏览的 API 不正确支持 `multipart/form-data`。[#5176](https://github.com/encode/django-rest-framework/issues/5176)
- 修复 `test_hyperlinked_related_lookup_url_encoded_exists`。[#5179](https://github.com/encode/django-rest-framework/issues/5179)
- 确保 max_length 在 FileField kwargs 中。[#5186](https://github.com/encode/django-rest-framework/issues/5186)
- 使用 kwargs 修复 `list_route` 和 `detail_route` 在 `url_path` 中包含大括号 [#5187](https://github.com/encode/django-rest-framework/issues/5187)
- 添加 Django 管理命令来创建 DRF 用户令牌。[#5188](https://github.com/encode/django-rest-framework/issues/5188)
- 确保 API 文档模板不检查用户身份验证 [#5162](https://github.com/encode/django-rest-framework/issues/5162)
- 修复 OneToOneField 也是主键的特殊情况。[#5192](https://github.com/encode/django-rest-framework/issues/5192)
- 在 base.html 中添加 aria-label 和用于可访问性目的的新区域 [#5196](https://github.com/encode/django-rest-framework/issues/5196)
- 引用 api.js 中的嵌套 API 参数 [#5214](https://github.com/encode/django-rest-framework/issues/5214)
- 调度前设置 ViewSet args/kwargs/请求。[#5229](https://github.com/encode/django-rest-framework/issues/5229)
- 为 SlugField 添加 unicode 支持。[#5231](https://github.com/encode/django-rest-framework/issues/5231)
- 修复 HiddenField 以原始数据形式呈现初始内容。[#5259](https://github.com/encode/django-rest-framework/issues/5259)
- 在无效时区解析时引发验证错误。[#5261](https://github.com/encode/django-rest-framework/issues/5261)
- 修复 SearchFilter 到多个行为/性能。[#5264](https://github.com/encode/django-rest-framework/issues/5264)
- 简化的链式比较和次要代码修复。[#5276](https://github.com/encode/django-rest-framework/issues/5276)
- RemoteUserAuthentication, docs, and tests. [#5306](https://github.com/encode/django-rest-framework/issues/5306)
- 还原 “缓存字段的根和上下文属性” [#5313](https://github.com/encode/django-rest-framework/issues/5313)
- 修复模式中列表字段的内省。[#5326](https://github.com/encode/django-rest-framework/issues/5326)
- 修复多个嵌套和额外方法的交互式文档。[#5334](https://github.com/encode/django-rest-framework/issues/5334)
- 修复/删除未定义的模板变量 “schema” [#5346](https://github.com/encode/django-rest-framework/issues/5346)

### 3.6.3
**日期**：2017年5月12日

- 如果 URL 查找导致 ValidationError，则引发 404 错误。[#5126](https://github.com/encode/django-rest-framework/issues/5126)
- 在生成 API 模式时，在基于类的视图上尊重 http_method_names。[#5085](https://github.com/encode/django-rest-framework/issues/5085)
- 允许在 LimitOffsetPagination 中覆盖 `get_limit` 以返回所有记录。[#4437](https://github.com/encode/django-rest-framework/issues/4437)
- 修复 ListSerializer 的部分更新。[#4222](https://github.com/encode/django-rest-framework/issues/4222)
- 在可浏览的 API 中正确渲染 JSONField 控件。[#4999](https://github.com/encode/django-rest-framework/issues/4999) [#5042](https://github.com/encode/django-rest-framework/issues/5042)
- 在给定时区中为无效的 datetime 引发验证错误。[#4987](https://github.com/encode/django-rest-framework/issues/4987)
- 支持限制文档和模式快捷方式到 url 子集。[#4979](https://github.com/encode/django-rest-framework/issues/4979)
- 使用没有 `page_size` 属性的分页器解决 SchemaGenerator 错误。[#5086](https://github.com/encode/django-rest-framework/issues/5086) [#3692](https://github.com/encode/django-rest-framework/issues/3692)
- 用 %20 而不是空格解决字符串上的 HyperlinkedRelatedField 异常。[#4748](https://github.com/encode/django-rest-framework/issues/4748) [#5078](https://github.com/encode/django-rest-framework/issues/5078)
- 可定制的模式生成器类。[#5082](https://github.com/encode/django-rest-framework/issues/5082)
- 更新响应中现有的变化标头，而不是覆盖它们。[#5047](https://github.com/encode/django-rest-framework/issues/5047)
- 支持将 `.as_view()` 传递给视图实例。[#5053](https://github.com/encode/django-rest-framework/issues/5053)
- 在视图上覆盖设置时使用正确的异常处理程序。[#5055](https://github.com/encode/django-rest-framework/issues/5055) [#5054](https://github.com/encode/django-rest-framework/issues/5054)
- 更新布尔字段以支持 “yes” 和 “no” 值。[#5038](https://github.com/encode/django-rest-framework/issues/5038)
- 修复 ChoiceField 的唯一验证器。[#5004](https://github.com/encode/django-rest-framework/issues/5004) [#5026](https://github.com/encode/django-rest-framework/issues/5026) [#5028](https://github.com/encode/django-rest-framework/issues/5028)
- API 文档中的 JavaScript 清理。[#5001](https://github.com/encode/django-rest-framework/issues/5001)
- 在有效的 API 模式中包含 URL 路径正则表达式。[#5014](https://github.com/encode/django-rest-framework/issues/5014)
- 正确设置 coreapi TokenAuthentication 方案。[#5000](https://github.com/encode/django-rest-framework/issues/5000) [#4994](https://github.com/encode/django-rest-framework/issues/4994)
- ViewSet 上的 HEAD 请求不应返回 405。[#4705](https://github.com/encode/django-rest-framework/issues/4705) [#4973](https://github.com/encode/django-rest-framework/issues/4973) [#4864](https://github.com/encode/django-rest-framework/issues/4864)
- 支持在 `extra_kwargs` 中使用 'source'。[#4688](https://github.com/encode/django-rest-framework/issues/4688)
- 修复 schema.js 的无效内容类型 [#4968](https://github.com/encode/django-rest-framework/issues/4968)
- 修复 DjangoFilterBackend 继承问题。[#5089](https://github.com/encode/django-rest-framework/issues/5089) [#5117](https://github.com/encode/django-rest-framework/issues/5117)

### 3.6.2
**日期**：2017年3月10日

- 在 API 文档中支持 Safari 和 IE。[#4959](https://github.com/encode/django-rest-framework/issues/4959) [#4961](https://github.com/encode/django-rest-framework/issues/4961)
- 在 API 文档模板标记中添加缺少的 `mark_safe`。[#4952](https://github.com/encode/django-rest-framework/issues/4952) [#4953](https://github.com/encode/django-rest-framework/issues/4953)
- 添加缺少的 glyphicon 字体。[#4950](https://github.com/encode/django-rest-framework/issues/4950) [#4951](https://github.com/encode/django-rest-framework/issues/4951)
- 修复 API 文档中的一对一字段。[#4955](https://github.com/encode/django-rest-framework/issues/4955) [#4956](https://github.com/encode/django-rest-framework/issues/4956)
- 测试清理。[#4949](https://github.com/encode/django-rest-framework/issues/4949)

### 3.6.1
**日期**：2017年3月9日

- 确保 `markdown` 依赖是可选的。[#4947](https://github.com/encode/django-rest-framework/issues/4947)

### 3.6.0
**日期**：2017年3月9日

- 请参阅[发布公告](https://www.django-rest-framework.org/community/3.6-announcement/)。

***

## 3.5.x 系列 (3.5.x series)
### 3.5.4
**日期**：2017年2月10日

- 为 ListField 添加 max_length 和 min_length 参数。[#4877](https://github.com/encode/django-rest-framework/issues/4877)
- 添加按照视图自定义异常处理程序支持。[#4753](https://github.com/encode/django-rest-framework/issues/4753)
- 支持禁用序列化器子类上声明的字段。[#4764](https://github.com/encode/django-rest-framework/issues/4764)
- 支持 `@list_route` 和 `@detail_route` 端点上的自定义视图名称。[#4821](https://github.com/encode/django-rest-framework/issues/4821)
- 使用自定义用户模型时，更正登录模板中字段的标签。[#4841](https://github.com/encode/django-rest-framework/issues/4841)
- 空格修复了从文档字符串生成的描述。[#4759](https://github.com/encode/django-rest-framework/issues/4759) [#4869](https://github.com/encode/django-rest-framework/issues/4869) [#4870](https://github.com/encode/django-rest-framework/issues/4870)
- 没有模式渲染的视图返回模式时，会更好的错误报告。[#4790](https://github.com/encode/django-rest-framework/issues/4790)
- 修复了使用 `prefetch_related` 时 `PUT` 请求的返回响应。[#4661](https://github.com/encode/django-rest-framework/issues/4661) [#4668](https://github.com/encode/django-rest-framework/issues/4668)
- 修复了痕迹视图名称。[#4750](https://github.com/encode/django-rest-framework/issues/4750)
- 修复 RequestsClient 以确保完全限定的 URL。[#4678](https://github.com/encode/django-rest-framework/issues/4678)
- 修复了某些情况下可写嵌套字段检查不正确的行为。[#4634](https://github.com/encode/django-rest-framework/issues/4634) [#4669](https://github.com/encode/django-rest-framework/issues/4669)
- 解决 Django 弃用警告。[#4712](https://github.com/encode/django-rest-framework/issues/4712)
- 各种测试用例的清理。

### 3.5.3
**日期**：2016年11月7日

- 不要引发不正确的 FilterSet 弃用警告。[#4660](https://github.com/encode/django-rest-framework/issues/4660) [#4643](https://github.com/encode/django-rest-framework/issues/4643) [#4644](https://github.com/encode/django-rest-framework/issues/4644)
- 在视图权限类执行时，模式生成不应该引发 404。[#4645](https://github.com/encode/django-rest-framework/issues/4645) [#4646](https://github.com/encode/django-rest-framework/issues/4646)
- 为输入控件添加 `autofocus` 支持。[#4650](https://github.com/encode/django-rest-framework/issues/4650)

### 3.5.2
**日期**：2016年11月1日

- 在 Python 2.7 中恢复异常回溯。[#4631](https://github.com/encode/django-rest-framework/issues/4631) [#4638](https://github.com/encode/django-rest-framework/issues/4638)
- 在管理控制台中正确显示 dicts。[#4532](https://github.com/encode/django-rest-framework/issues/4532) [#4636](https://github.com/encode/django-rest-framework/issues/4636)
- 修复了带有变量 args，kwargs 的 is_simple_callable。[#4622](https://github.com/encode/django-rest-framework/issues/4622) [#4602](https://github.com/encode/django-rest-framework/issues/4602)
- 使用 BooleanField 支持 'on'/'off' 文字。[#4640](https://github.com/encode/django-rest-framework/issues/4640) [#4624](https://github.com/encode/django-rest-framework/issues/4624)
- 启用值查询集的游标分页。[#4569](https://github.com/encode/django-rest-framework/issues/4569)
- 修复了对限流异常的 `get_full_details()`支持。[#4627](https://github.com/encode/django-rest-framework/issues/4627)
- 修复 FilterSet 代理。[#4620](https://github.com/encode/django-rest-framework/issues/4620)
- 使序列化器字段显式导入。[#4628](https://github.com/encode/django-rest-framework/issues/4628)
- 删除冗余请求适配器。[#4639](https://github.com/encode/django-rest-framework/issues/4639)

### 3.5.1
**日期**：2016年10月21日

- 创建 `rest_framework/compat.py` 导入。[#4612](https://github.com/encode/django-rest-framework/issues/4612) [#4608](https://github.com/encode/django-rest-framework/issues/4608) [#4601](https://github.com/encode/django-rest-framework/issues/4601)
- 修复模式基础路径生成中的 bug。[#4611](https://github.com/encode/django-rest-framework/issues/4611) [#4605](https://github.com/encode/django-rest-framework/issues/4605)
- 修复单一项的 ListSerializer 的破损情况。[#4609](https://github.com/encode/django-rest-framework/issues/4609) [#4606](https://github.com/encode/django-rest-framework/issues/4606)
- 删除 Python 3.5 兼容的空白 `raise`。[#4600](https://github.com/encode/django-rest-framework/issues/4600)

### 3.5.0
**日期**：2016年10月20日

***

## 3.4.x 系列 (3.4.x series)
### 3.4.7
**日期**：2016年9月21日

- request.POST 已经被访问时请求解析的回退行为。[#3951](https://github.com/encode/django-rest-framework/issues/3951) [#4500](https://github.com/encode/django-rest-framework/issues/4500)
- 修复 `RegexField` 的回归。[#4489](https://github.com/encode/django-rest-framework/issues/4489) [#4490](https://github.com/encode/django-rest-framework/issues/4490) [#2617](https://github.com/encode/django-rest-framework/issues/2617)
- `admin.html` 中缺少逗号导致 CSRF 错误。[#4472](https://github.com/encode/django-rest-framework/issues/4472) [#4473](https://github.com/encode/django-rest-framework/issues/4473)
- 修复带有空上下文的响应渲染。[#4495](https://github.com/encode/django-rest-framework/issues/4495)
- 修复 API 列表中的缩进回归。[#4493](https://github.com/encode/django-rest-framework/issues/4493)
- 修复了将错误值设置为 api_view 装饰视图的 `ResolverMatch.func_name` 的问题。[#4465](https://github.com/encode/django-rest-framework/issues/4465) [#4462](https://github.com/encode/django-rest-framework/issues/4462)
- 修复路径包含 unicode 参数时的 `APIClient.get()` [#4458](https://github.com/encode/django-rest-framework/issues/4458)

### 3.4.6
**日期**：2016年8月23日

- 修复可浏览 API 中难看的的 Javascript。[#4435](https://github.com/encode/django-rest-framework/issues/4435)
- 从模式字段中跳过 HiddenField。[#4425](https://github.com/encode/django-rest-framework/issues/4425) [#4429](https://github.com/encode/django-rest-framework/issues/4429)
- 改进 Create 以显示原始异常回溯。[#3508](https://github.com/encode/django-rest-framework/issues/3508)
- 修复 `AdminRenderer` 仅显示 PK 相关字段。[#4419](https://github.com/encode/django-rest-framework/issues/4419) [#4423](https://github.com/encode/django-rest-framework/issues/4423)


### 3.4.5
**日期**：2016年8月19日

- 改善调试错误处理。[#4416](https://github.com/encode/django-rest-framework/issues/4416) [#4409](https://github.com/encode/django-rest-framework/issues/4409)
- 允许自定义 CSRF_HEADER_NAME 设置。[#4415](https://github.com/encode/django-rest-framework/issues/4415) [#4410](https://github.com/encode/django-rest-framework/issues/4410)
- 生成模式时在视图集上包含 .action 属性。[#4408](https://github.com/encode/django-rest-framework/issues/4408) [#4398](https://github.com/encode/django-rest-framework/issues/4398)
- 请勿在 request.POST 中包含 request.FILES 项。[#4407](https://github.com/encode/django-rest-framework/issues/4407)
- 修复了多重复选框的渲染。[#4403](https://github.com/encode/django-rest-framework/issues/4403)
- 修复 Field.get_default 的文档字符串。[#4404](https://github.com/encode/django-rest-framework/issues/4404)
- 用 README 中的 ascii 对应字符替换 utf8 字符。[#4412](https://github.com/encode/django-rest-framework/issues/4412)

### 3.4.4
**日期**：2016年8月12日

- 确保在生成模式时完全初始化视图。[#4373](https://github.com/encode/django-rest-framework/issues/4373) [#4382](https://github.com/encode/django-rest-framework/issues/4382) [#4383](https://github.com/encode/django-rest-framework/issues/4383) [#4279](https://github.com/encode/django-rest-framework/issues/4279) [#4278](https://github.com/encode/django-rest-framework/issues/4278)
- 将表单字段描述添加到模式。[#4387](https://github.com/encode/django-rest-framework/issues/4387)
- 修复模式端点的类别生成。[#4391](https://github.com/encode/django-rest-framework/issues/4391) [#4394](https://github.com/encode/django-rest-framework/issues/4394) [#4390](https://github.com/encode/django-rest-framework/issues/4390) [#4386](https://github.com/encode/django-rest-framework/issues/4386) [#4376](https://github.com/encode/django-rest-framework/issues/4376) [#4329](https://github.com/encode/django-rest-framework/issues/4329)
- 分页时不要剥离空的查询参数。[#4392](https://github.com/encode/django-rest-framework/issues/4392) [#4393](https://github.com/encode/django-rest-framework/issues/4393) [#4260](https://github.com/encode/django-rest-framework/issues/4260)
- 不要对带有 LimitOffsetPagination 的空结果重新运行查询。[#4201](https://github.com/encode/django-rest-framework/issues/4201) [#4388](https://github.com/encode/django-rest-framework/issues/4388)
- 对 CharField 进行更严格的类型验证。[#4380](https://github.com/encode/django-rest-framework/issues/4380) [#3394](https://github.com/encode/django-rest-framework/issues/3394)
- RelatedField.choices 应保留非字符串值。[#4111](https://github.com/encode/django-rest-framework/issues/4111) [#4379](https://github.com/encode/django-rest-framework/issues/4379) [#3365](https://github.com/encode/django-rest-framework/issues/3365)
- 在垂直窗体样式中渲染复选框的测试用例。[#4378](https://github.com/encode/django-rest-framework/issues/4378) [#3868](https://github.com/encode/django-rest-framework/issues/3868)
- 在可浏览的 API 中显示错误回溯 HTML [#4042](https://github.com/encode/django-rest-framework/issues/4042) [#4172](https://github.com/encode/django-rest-framework/issues/4172)
- 修复 ALLOWED_VERSIONS 和没有 DEFAULT_VERSION 的处理。[#4370](https://github.com/encode/django-rest-framework/issues/4370)
- 在 DecimalField 上允许 `max_numbers=None`。[#4377](https://github.com/encode/django-rest-framework/issues/4377) [#4372](https://github.com/encode/django-rest-framework/issues/4372)
- 在渲染关系选择时限制查询集。[#4375](https://github.com/encode/django-rest-framework/issues/4375) [#4122](https://github.com/encode/django-rest-framework/issues/4122) [#3329](https://github.com/encode/django-rest-framework/issues/3329) [#3330](https://github.com/encode/django-rest-framework/issues/3330) [#3877](https://github.com/encode/django-rest-framework/issues/3877)
- 解决 ChoiceField、MultipleChoiceField 和非字符串选项表单显示。[#4374](https://github.com/encode/django-rest-framework/issues/4374) [#4119](https://github.com/encode/django-rest-framework/issues/4119) [#4121](https://github.com/encode/django-rest-framework/issues/4121) [#4137](https://github.com/encode/django-rest-framework/issues/4137) [#4120](https://github.com/encode/django-rest-framework/issues/4120)
- 修复对 TemplateHTMLRenderer.resolve_context() 回退方法的调用。[#4371](https://github.com/encode/django-rest-framework/issues/4371)

### 3.4.3
**日期**：2016年8月5日

- 包含旧版 TemplateHTMLRenderer 内部 API 用户的回退。[#4361](https://github.com/encode/django-rest-framework/issues/4361)

### 3.4.2
**日期**：2016年8月5日

- 在生成模式时将 kwargs 传递给 'as_view'。[#4359](https://github.com/encode/django-rest-framework/issues/4359) [#4330](https://github.com/encode/django-rest-framework/issues/4330) [#4331](https://github.com/encode/django-rest-framework/issues/4331)
- 在 Django 1.10+ 下访问 `request.user.is_authenticated` 作为属性而不是方法 [#4358](https://github.com/encode/django-rest-framework/issues/4358) [#4354](https://github.com/encode/django-rest-framework/issues/4354)
- 从模式中过滤出 HEAD。[#4357](https://github.com/encode/django-rest-framework/issues/4357)
- extra_kwargs 优先于唯一性 kwargs。[#4198](https://github.com/encode/django-rest-framework/issues/4198) [#4199](https://github.com/encode/django-rest-framework/issues/4199) [#4349](https://github.com/encode/django-rest-framework/issues/4349)
- 在代码缩进中使用制表符时的正确描述。[#4345](https://github.com/encode/django-rest-framework/issues/4345) [#4347](https://github.com/encode/django-rest-framework/issues/4347)
- 在 TemplateHTMLRenderer 中更改模板上下文生成。[#4236](https://github.com/encode/django-rest-framework/issues/4236)
- 序列化器默认值不应该包含在部分更新中。[#4346](https://github.com/encode/django-rest-framework/issues/4346) [#3565](https://github.com/encode/django-rest-framework/issues/3565)
- 当文件名未被包含时，从 FileUploadParser 始终如一的行为和描述错误。[#4340](https://github.com/encode/django-rest-framework/issues/4340) [#3610](https://github.com/encode/django-rest-framework/issues/3610) [#4292](https://github.com/encode/django-rest-framework/issues/4292) [#4296](https://github.com/encode/django-rest-framework/issues/4296)
- DecimalField 量化传入的数字。[#4339](https://github.com/encode/django-rest-framework/issues/4339) [#4318](https://github.com/encode/django-rest-framework/issues/4318)
- 处理 IP 字段的非字符串输入。[#4335](https://github.com/encode/django-rest-framework/issues/4335) [#4336](https://github.com/encode/django-rest-framework/issues/4336) [#4338](https://github.com/encode/django-rest-framework/issues/4338)
- 当模式生成包含根 URL 时，修复前导斜杠处理。[#4332](https://github.com/encode/django-rest-framework/issues/4332)
- 带有 allow_null 选项的 DictField 测试用例。[#4348](https://github.com/encode/django-rest-framework/issues/4348)
- 更新从 Django 1.10 beta 到 Django 1.10 的测试。[#4344](https://github.com/encode/django-rest-framework/issues/4344)

### 3.4.1
**日期**：2016年7月28日

- 向 `DefaultRouter` 添加 `root_renderers` 参数。[#4323](https://github.com/encode/django-rest-framework/issues/4323) [#4268](https://github.com/encode/django-rest-framework/issues/4268)
- 添加了 `url` 和 `schema_url` 参数。[#4321](https://github.com/encode/django-rest-framework/issues/4321) [#4308](https://github.com/encode/django-rest-framework/issues/4308) [#4305](https://github.com/encode/django-rest-framework/issues/4305)
- 唯一的一起检查应该适用于具有默认值的只读字段。[#4316](https://github.com/encode/django-rest-framework/issues/4316) [#4294](https://github.com/encode/django-rest-framework/issues/4294)
- 在模式生成器中设置 view.format_kwarg。[#4293](https://github.com/encode/django-rest-framework/issues/4293) [#4315](https://github.com/encode/django-rest-framework/issues/4315)
- 修复带有 `pagination_class = None` 的视图的模式生成器。[#4314](https://github.com/encode/django-rest-framework/issues/4314) [#4289](https://github.com/encode/django-rest-framework/issues/4289)
- 修复没有 `get_serializer_class` 的视图的模式生成器。[#4265](https://github.com/encode/django-rest-framework/issues/4265) [#4285](https://github.com/encode/django-rest-framework/issues/4285)
- 修复了 `Accept` 和 `Content-Type` 标头中的媒体类型参数。[#4287](https://github.com/encode/django-rest-framework/issues/4287) [#4313](https://github.com/encode/django-rest-framework/issues/4313) [#4281](https://github.com/encode/django-rest-framework/issues/4281)
- 在错误消息中使用 `verbose_name` 而不是 `object_name`。[#4299](https://github.com/encode/django-rest-framework/issues/4299)
- Twitter Bootstrap 的小版本更新。[#4307](https://github.com/encode/django-rest-framework/issues/4307)
- 使用相关字段时，SearchFilter 会引发错误。[#4302](https://github.com/encode/django-rest-framework/issues/4302) [#4303](https://github.com/encode/django-rest-framework/issues/4303) [#4298](https://github.com/encode/django-rest-framework/issues/4298)
- 添加对 RFC 4918 状态代码的支持。[#4291](https://github.com/encode/django-rest-framework/issues/4291)
- 将 LICENSE.md 添加到构建的 wheel 中。[#4270](https://github.com/encode/django-rest-framework/issues/4270)
- 序列化 “complex” 字段返回 None，而不是 3.4 之后的值 [#4272](https://github.com/encode/django-rest-framework/issues/4272) [#4273](https://github.com/encode/django-rest-framework/issues/4273) [#4288](https://github.com/encode/django-rest-framework/issues/4288)

### 3.4.0
**日期**：2016年7月14日

- 不要在 JSON 输出中剥离微秒。[#4256](https://github.com/encode/django-rest-framework/issues/4256)
- 两个稍微不同的 iso 8601 日期时间序列化。[#4255](https://github.com/encode/django-rest-framework/issues/4255)
- 解决不正确的包含媒体类型参数。[#4254](https://github.com/encode/django-rest-framework/issues/4254)
- 响应内容类型可能存在缺陷。[#4253](https://github.com/encode/django-rest-framework/issues/4253)
- 修复某些平台上的 setup.py 错误。[#4246](https://github.com/encode/django-rest-framework/issues/4246)
- 将 coreapi 中的替代格式移动到单独的包中。[#4244](https://github.com/encode/django-rest-framework/issues/4244)
- 将局部关键字参数添加到 `DecimalField`。[#4233](https://github.com/encode/django-rest-framework/issues/4233)
- 修复自定义列表路由和详细路由中的路由器问题。[#4229](https://github.com/encode/django-rest-framework/issues/4229)
- 具有嵌套命名空间的命名空间版本控制。[#4219](https://github.com/encode/django-rest-framework/issues/4219)
- 强大的唯一性检查。[#4217](https://github.com/encode/django-rest-framework/issues/4217)
- `must_call_distinct` 的轻微重构。[#4215](https://github.com/encode/django-rest-framework/issues/4215)
- CursorPagination 中的可覆盖偏移截止。[#4212](https://github.com/encode/django-rest-framework/issues/4212)
- 像日期/时间字段一样传递字符串。[#4196](https://github.com/encode/django-rest-framework/issues/4196)
- 添加测试确认 required=False 在关系字段上有效。[#4195](https://github.com/encode/django-rest-framework/issues/4195)
- 在 LimitOffsetPagination 中，`limit=0` 应恢复为默认限制。[#4194](https://github.com/encode/django-rest-framework/issues/4194)
- 从 unique_together 验证和添加文档中排除 read_only=True 字段。[#4192](https://github.com/encode/django-rest-framework/issues/4192)
- 处理 JSON 中的字节串。[#4191](https://github.com/encode/django-rest-framework/issues/4191)
- JSONField(binary=True) 表示使用二进制字符串，JSONRenderer 不支持二进制字符串。[#4187](https://github.com/encode/django-rest-framework/issues/4187) [#4185](https://github.com/encode/django-rest-framework/issues/4185)
- 可浏览 API 中更强大的表单渲染。[#4181](https://github.com/encode/django-rest-framework/issues/4181)
- `.validated_data` 和 `.errors` 的空用例作为 ListSerializer 的列表，而不是 dicts。[#4180](https://github.com/encode/django-rest-framework/issues/4180)
- 模式和客户端库。[#4179](https://github.com/encode/django-rest-framework/issues/4179)
- 删除了 `AUTH_USER_MODEL` 兼容属性。[#4176](https://github.com/encode/django-rest-framework/issues/4176)
- 清理现有的弃用警告。[#4166](https://github.com/encode/django-rest-framework/issues/4166)
- Django 1.10 支持。[#4158](https://github.com/encode/django-rest-framework/issues/4158)
- 将 jQuery 版本更新为 1.12.4。[#4157](https://github.com/encode/django-rest-framework/issues/4157)
- OrderingFilter 上更强大的默认行为。[#4156](https://github.com/encode/django-rest-framework/issues/4156)
- description.py 代码和测试删除。[#4153](https://github.com/encode/django-rest-framework/issues/4153)
- 在元组中包裹 guardian.VERSION。[#4149](https://github.com/encode/django-rest-framework/issues/4149)
- 为具有 kwargs 的字段优化验证器。[#4146](https://github.com/encode/django-rest-framework/issues/4146)
- 修复 ListField，DictField 的子节点中的无值表示。[#4118](https://github.com/encode/django-rest-framework/issues/4118)
- 解析午夜值的 TimeField 表示。[#4107](https://github.com/encode/django-rest-framework/issues/4107)
- 在 POST/DELETE 请求之后，在 AdminRenderer 中为重定向设置正确的状态码。[#4106](https://github.com/encode/django-rest-framework/issues/4106)
- TimeField 渲染返回 None 而不是 00:00:00。[#4105](https://github.com/encode/django-rest-framework/issues/4105)
- 修复错误命名的 zh-hans 和 zh-hant 区域设置路径。[#4103](https://github.com/encode/django-rest-framework/issues/4103)
- 限制为 0 时防止引发异常。[#4098](https://github.com/encode/django-rest-framework/issues/4098)
- TokenAuthentication：在标头中允许自定义关键字。[#4097](https://github.com/encode/django-rest-framework/issues/4097)
- 处理错误填充的 HTTP 基本认证头。[#4090](https://github.com/encode/django-rest-framework/issues/4090)
- LimitOffset 分页在 limit=0 时崩溃可浏览 API。[#4079](https://github.com/encode/django-rest-framework/issues/4079)
- 修复了 DecimalField 的任意精度支持。[#4075](https://github.com/encode/django-rest-framework/issues/4075)
- 添加了对自定义 CSRF cookie 名称的支持。[#4049](https://github.com/encode/django-rest-framework/issues/4049)
- 修复 #4035 引入的回归。[#4041](https://github.com/encode/django-rest-framework/issues/4041)
- 没有身份验证视图失败许可应引发 403。[#4040](https://github.com/encode/django-rest-framework/issues/4040)
- 修复 string_types / text_types 混淆。[#4025](https://github.com/encode/django-rest-framework/issues/4025)
- 不要在 OPTIONS 请求中列出相关的字段选项。[#4021](https://github.com/encode/django-rest-framework/issues/4021)
- 修复拼写错误。[#4008](https://github.com/encode/django-rest-framework/issues/4008)
- 重新排序初始化视图。[#4006](https://github.com/encode/django-rest-framework/issues/4006)
- 在 Python 3.4 上的 DjangoObjectPermissionsFilter 中类型错误。[#4005](https://github.com/encode/django-rest-framework/issues/4005)
- 修复使用已弃用的 Query.aggregates。[#4003](https://github.com/encode/django-rest-framework/issues/4003)
- 修复文档字符串周围的空白行。[#4002](https://github.com/encode/django-rest-framework/issues/4002)
- 修正了限制为 0 时的管理员分页。[#3990](https://github.com/encode/django-rest-framework/issues/3990)
- OrderingFilter 调整。[#3983](https://github.com/encode/django-rest-framework/issues/3983)
- 非必需的序列化器相关字段。[#3976](https://github.com/encode/django-rest-framework/issues/3976)
- 在教程中使用更安全的 “@api_view” 调用方式。[#3971](https://github.com/encode/django-rest-framework/issues/3971)
- ListSerializer 不处理 unique_together 约束。[#3970](https://github.com/encode/django-rest-framework/issues/3970)
- 添加缺少的迁移文件。[#3968](https://github.com/encode/django-rest-framework/issues/3968)
- `OrderingFilter` 应调用 `get_serializer_class()` 来确定默认字段。[#3964](https://github.com/encode/django-rest-framework/issues/3964)
- 从测试和兼容中删除旧的 Django 检查。[#3953](https://github.com/encode/django-rest-framework/issues/3953)
- 支持 callable 作为任何 `serializer.Field` 的 `initial` 值。[#3943](https://github.com/encode/django-rest-framework/issues/3943)
- 在 SearchFilter 中阻止不必要的 distinct() 调用。[#3938](https://github.com/encode/django-rest-framework/issues/3938)
- 修复 None UUID ForeignKey 序列化。[#3936](https://github.com/encode/django-rest-framework/issues/3936)
- 删掉 EOL Django 1.7。[#3933](https://github.com/encode/django-rest-framework/issues/3933)
- 在序列化器错误消息中添加丢失的空格。[#3926](https://github.com/encode/django-rest-framework/issues/3926)
- 修复 _force_text_recursive 拼写错误。[#3908](https://github.com/encode/django-rest-framework/issues/3908)
- 尝试处理 Django 2.0 与 `field.rel` 相关的弃用警告。[#3906](https://github.com/encode/django-rest-framework/issues/3906)
- 修复使用带列表的嵌套序列化器解析多部分数据。[#3820](https://github.com/encode/django-rest-framework/issues/3820)
- 将 API URL 解析为不同的命名空间。[#3816](https://github.com/encode/django-rest-framework/issues/3816)
- 不要在可浏览的 API 表单中使用 HTML 转义 `help_text`。[#3812](https://github.com/encode/django-rest-framework/issues/3812)
- OPTIONS 在选项字段中获取并显示所有可能的外键。[#3751](https://github.com/encode/django-rest-framework/issues/3751)
- Django 1.9 弃用警告 [#3729](https://github.com/encode/django-rest-framework/issues/3729)
- #3598 的测试用例 [#3710](https://github.com/encode/django-rest-framework/issues/3710)
- 添加对搜索过滤器的多个值的支持。[#3541](https://github.com/encode/django-rest-framework/issues/3541)
- 在排序过滤器中使用 `get_serializer_class`。[#3487](https://github.com/encode/django-rest-framework/issues/3487)
- 具有 many=True 的序列化器应返回空列表而不是空字典。[#3476](https://github.com/encode/django-rest-framework/issues/3476)
- LimitOffsetPagination limit=0 修复。[#3444](https://github.com/encode/django-rest-framework/issues/3444)
- 启用验证器延迟字符串评估并处理新的字符串格式。[#3438](https://github.com/encode/django-rest-framework/issues/3438)
- 如果字段无效，则唯一验证器被执行并断开。[#3381](https://github.com/encode/django-rest-framework/issues/3381)
- 不要忽略面包屑中覆盖的 View.get_view_name()。[#3273](https://github.com/encode/django-rest-framework/issues/3273)
- 当渲染序列化器失败时，重试表单渲染。[#3164](https://github.com/encode/django-rest-framework/issues/3164)
- 唯一约束防止嵌套的序列化器更新。[#2996](https://github.com/encode/django-rest-framework/issues/2996)
- 唯一性验证器不应该排除 (read_only) 字段运行。[#2848](https://github.com/encode/django-rest-framework/issues/2848)
- UniqueValidator 会引发嵌套对象的异常。[#2403](https://github.com/encode/django-rest-framework/issues/2403)
- `lookup_type` 已弃用，支持 `lookup_expr`。[#4259](https://github.com/encode/django-rest-framework/issues/4259)

***

## 3.3.x 系列 (3.3.x series)
### 3.3.3
**日期**：2016年3月14日。

- 从模板中删除版本字符串。感谢 @blag 的报告和修复。[#3878](https://github.com/encode/django-rest-framework/issues/3878) [#3913](https://github.com/encode/django-rest-framework/issues/3913) [#3912](https://github.com/encode/django-rest-framework/issues/3912)
- 修复 `BooleanField` 的垂直 html 布局。感谢 Mikalai Radchuk 的修复。[#3910](https://github.com/encode/django-rest-framework/issues/3910)
- Django 1.8 上的静默弃用警告。感谢 Simon Charette 的修复。[#3903](https://github.com/encode/django-rest-framework/issues/3903)
- authtoken 的国际化。感谢 Michael Nacharov 的修复。[#3887](https://github.com/encode/django-rest-framework/issues/3887) [#3968](https://github.com/encode/django-rest-framework/issues/3968)
- 当 authtoken 应用程序未声明时，将 `Token` 模型修正为 `abstract`。感谢 Adam Thomas 的报告。[#3860](https://github.com/encode/django-rest-framework/issues/3860) [#3858](https://github.com/encode/django-rest-framework/issues/3858)
- 改善 Markdown 版本兼容性。感谢 Michael J. Schultz 的修复。[#3604](https://github.com/encode/django-rest-framework/issues/3604) [#3842](https://github.com/encode/django-rest-framework/issues/3842)
- `QueryParameterVersioning` 不使用 `DEFAULT_VERSION` 设置。感谢 Brad Montgomery 的修复。[#3833](https://github.com/encode/django-rest-framework/issues/3833)
- 在模型上添加显式的 `on_delete`。感谢 Mads Jensen 的修复。[#3832](https://github.com/encode/django-rest-framework/issues/3832)
- 修复 `DateField.to_representation` 以便与 Python 2 unicode 一起工作。感谢 Mikalai Radchuk 的修复。[#3819](https://github.com/encode/django-rest-framework/issues/3819)
- 修复了 `TimeField` 不处理字符串时间。感谢 Areski Belaid 的修复。[#3809](https://github.com/encode/django-rest-framework/issues/3809)
- 避免更新 `Meta.extra_kwargs`。感谢 Kevin Massey 的报告和修复。[#3805](https://github.com/encode/django-rest-framework/issues/3805) [#3804](https://github.com/encode/django-rest-framework/issues/3804)
- 修复了嵌套验证错误被错误渲染。感谢 Craig de Stigter 的修复。[#3801](https://github.com/encode/django-rest-framework/issues/3801)
- 记录如何使用 `django-crispy-forms` 避免 CSRF 和缺少按钮问题。感谢 Emmanuelle Delescolle，JoséPadilla 和 Luis San Pablo 的报告，分析和修复。[#3787](https://github.com/encode/django-rest-framework/issues/3787) [#3636](https://github.com/encode/django-rest-framework/issues/3636) [#3637](https://github.com/encode/django-rest-framework/issues/3637)
- 改进 Rest Framework 设置文件建立时间。感谢 Miles Hutson 的报告和 Mads Jensen 的修复。[#3786](https://github.com/encode/django-rest-framework/issues/3786) [#3815](https://github.com/encode/django-rest-framework/issues/3815)
- 改进 Django 1.9 的 authtoken 兼容性。感谢 S. Andrew Sheppard 的修复。[#3785](https://github.com/encode/django-rest-framework/issues/3785)
- 修复模型的 `DecimalField` 中的 `Min/MaxValueValidator` 传输。感谢 Kevin Brown 的修复。[#3774](https://github.com/encode/django-rest-framework/issues/3774)
- 改进可浏览 API 中的 HTML 标题。感谢 Mike Lissner 的报告和修复。[#3769](https://github.com/encode/django-rest-framework/issues/3769)
- 修复 `AutoFilterSet` 以继承 `default_filter_set`。感谢 Tom Linford 的修复。[#3753](https://github.com/encode/django-rest-framework/issues/3753)
- 修复 transifex 配置以处理新的中文语言代码。感谢 @nypisces 的报告和修复。[#3739](https://github.com/encode/django-rest-framework/issues/3739)
- `DateTimeField` 无法正确处理空值。感谢 Mick Parker 的报告和修复。[#3731](https://github.com/encode/django-rest-framework/issues/3731) [#3726](https://github.com/encode/django-rest-framework/issues/3726)
- 设置删除的 rest_framework 设置时引发错误。感谢 Luis San Pablo 的修复。[#3715](https://github.com/encode/django-rest-framework/issues/3715)
- 在 AdminRenderer post 表单中添加缺少的 csrf_token。感谢 PiotrŚniegowski 的修复。[#3703](https://github.com/encode/django-rest-framework/issues/3703)
- 重构 `_get_reverse_relationships()` 以使用正确的 `to_field`。感谢 Benjamin Phillips 的修复。[#3696](https://github.com/encode/django-rest-framework/issues/3696)
- 记录有关 `RelatedField` 的 `get_queryset` 的使用。感谢 Ryan Hiebert 的修复。[#3605](https://github.com/encode/django-rest-framework/issues/3605)
- 修复 HyperlinkRelatedField.get_url 中的空 pk 检测。感谢 @jslang 的修复。[#3962](https://github.com/encode/django-rest-framework/issues/3962)

### 3.3.2
**日期**：2015年12月14日。

- `ListField` 强制输入是一个列表。[#3513](https://github.com/encode/django-rest-framework/issues/3513)
- 修复隐藏原始数据表单的回归。[#3600](https://github.com/encode/django-rest-framework/issues/3600) [#3578](https://github.com/encode/django-rest-framework/issues/3578)
- 修复 Python 3.5 兼容性。[#3534](https://github.com/encode/django-rest-framework/issues/3534) [#3626](https://github.com/encode/django-rest-framework/issues/3626)
- 允许在 `pagination.PageNumberPagination` 中设置自定义 Django Paginator。[#3631](https://github.com/encode/django-rest-framework/issues/3631) [#3684](https://github.com/encode/django-rest-framework/issues/3684)
- 修复没有 `to_fields` 属性的关系字段。[#3635](https://github.com/encode/django-rest-framework/issues/3635) [#3634](https://github.com/encode/django-rest-framework/issues/3634)
- 修复 Django 1.9 的 `template.render` 弃用警告。[#3654](https://github.com/encode/django-rest-framework/issues/3654)
- 在可浏览的 API 呈现器中对响应头进行排序。[#3655](https://github.com/encode/django-rest-framework/issues/3655)
- 对 Django 1.9+ 使用 related_objects api。[#3656](https://github.com/encode/django-rest-framework/issues/3656) [#3252](https://github.com/encode/django-rest-framework/issues/3252)
- 删除时添加确认模式。[#3228](https://github.com/encode/django-rest-framework/issues/3228) [#3662](https://github.com/encode/django-rest-framework/issues/3662)
- 在调用 has_[object_]permissions 时显示以前隐藏的 AttributeErrors 和 TypeErrors。[#3668](https://github.com/encode/django-rest-framework/issues/3668)
- 使 DRF 与 Django 1.8 中的多模板引擎兼容。[#3672](https://github.com/encode/django-rest-framework/issues/3672)
- 更新 `NestedBoundField` 以在渲染其表单时处理空字符串。[#3677](https://github.com/encode/django-rest-framework/issues/3677)
- 修复 UUID 验证以正确捕获无效输入类型。[#3687](https://github.com/encode/django-rest-framework/issues/3687) [#3679](https://github.com/encode/django-rest-framework/issues/3679)
- 修复缓存问题。[#3628](https://github.com/encode/django-rest-framework/issues/3628) [#3701](https://github.com/encode/django-rest-framework/issues/3701)
- 修复没有 filter_class 的视图的管理员和 API 浏览器。[#3705](https://github.com/encode/django-rest-framework/issues/3705) [#3596](https://github.com/encode/django-rest-framework/issues/3596) [#3597](https://github.com/encode/django-rest-framework/issues/3597)
- 将 app_name 添加到 rest_framework.urls。[#3714](https://github.com/encode/django-rest-framework/issues/3714)
- 改进 authtoken 的视图以支持 url 版本控制。[#3718](https://github.com/encode/django-rest-framework/issues/3718) [#3723](https://github.com/encode/django-rest-framework/issues/3723)

### 3.3.1
**日期**：2015年11月4日。

- 访问 `request.POST` 时解决解析错误 [#3592](https://github.com/encode/django-rest-framework/issues/3592)
- 正确处理引用主键的 `to_field`。[#3593](https://github.com/encode/django-rest-framework/issues/3593)
- 当没有定义 `filter_class` 时，允许过滤 HTML 去渲染。[#3560](https://github.com/encode/django-rest-framework/issues/3560)
- 修复管理员渲染问题。[#3564](https://github.com/encode/django-rest-framework/issues/3564) [#3556](https://github.com/encode/django-rest-framework/issues/3556)
- 修复了 DecimalValidator 的问题。[#3568](https://github.com/encode/django-rest-framework/issues/3568)

### 3.3.0
**日期**：2015年10月28日。

- 过滤器的 HTML 控件。[#3315](https://github.com/encode/django-rest-framework/issues/3315)
- Forms API。[#3475](https://github.com/encode/django-rest-framework/issues/3475)
- AJAX 可浏览的 API。[#3410](https://github.com/encode/django-rest-framework/issues/3410)
- 添加 JSONField。[#3454](https://github.com/encode/django-rest-framework/issues/3454)
- 在创建 `ModelSerializer` 关系字段时正确映射 `to_field`。[#3526](https://github.com/encode/django-rest-framework/issues/3526)
- 将 `FilePathField` 映射到序列化器字段时包含关键字参数。[#3536](https://github.com/encode/django-rest-framework/issues/3536)
- 在 `ModelSerializer` 唯一性约束上映射适当的模型 `error_messages`。[#3435](https://github.com/encode/django-rest-framework/issues/3435)
- 包括从 `TextField` 映射的 `ModelSerializer` 字段的 `max_length` 约束。[#3509](https://github.com/encode/django-rest-framework/issues/3509)
- 添加了对 Django 1.9 的支持。[#3450](https://github.com/encode/django-rest-framework/issues/3450) [#3525](https://github.com/encode/django-rest-framework/issues/3525)
- 删除了对 Django 1.5 和 1.6 的支持。[#3421](https://github.com/encode/django-rest-framework/issues/3421) [#3429](https://github.com/encode/django-rest-framework/issues/3429)
- 删除了 “south” 迁移。[#3495](https://github.com/encode/django-rest-framework/issues/3495)

***

## 3.2.x 系列 (3.2.x series)
### 3.2.5
**日期**：2015年10月27日。

- 在可选的注销标记中转义 `username`。[#3550](https://github.com/encode/django-rest-framework/issues/3550)

### 3.2.4
**日期**：2015年9月21日。

- 不要错过缺失的 `ViewSet.search_fields` 属性。[#3324](https://github.com/encode/django-rest-framework/issues/3324) [#3323](https://github.com/encode/django-rest-framework/issues/3323)
- 修复了 `allow_empty` 无法在带有 `many=True` 的序列化器上工作。[#3361](https://github.com/encode/django-rest-framework/issues/3361) [#3364](https://github.com/encode/django-rest-framework/issues/3364)
- 让 `DurationField` 接受整数。[#3359](https://github.com/encode/django-rest-framework/issues/3359)
- 多部分请求中不支持多级词典。[#3314](https://github.com/encode/django-rest-framework/issues/3314)
- 修复 HTTP PATCH 上的 `ListField` 截断 [#3415](https://github.com/encode/django-rest-framework/issues/3415) [#2761](https://github.com/encode/django-rest-framework/issues/2761)

### 3.2.3
**日期**：2015年8月24日。

- 为限制选择下拉菜单添加 `html_cutoff` 和 `html_cutoff_text`。[#3313](https://github.com/encode/django-rest-framework/issues/3313)
- 添加正则表达式样式到 `SearchFilter`。[#3316](https://github.com/encode/django-rest-framework/issues/3316)
- 解决设置空白 HTML 字段的问题。[#3318](https://github.com/encode/django-rest-framework/issues/3318) [#3321](https://github.com/encode/django-rest-framework/issues/3321)
- 在可浏览的 API 表单中正确显示现有的 “选择多个” 值。[#3290](https://github.com/encode/django-rest-framework/issues/3290)
- 解决 `IPAddressField` 的重复验证消息。[#3249](https://github.com/encode/django-rest-framework/issues/3249) [#3250](https://github.com/encode/django-rest-framework/issues/3250)
- 修复以确保在禁用分页时管理渲染器继续工作。[#3275](https://github.com/encode/django-rest-framework/issues/3275)
- 当 count=0，offset=0 时，解决与 `LimitOffsetPagination` 的错误 [#3303](https://github.com/encode/django-rest-framework/issues/3303)

### 3.2.2
**日期**：2015年8月13日。

- 添加 `display_value()` 方法，以便在显示关系字段选择输入时使用。[#3254](https://github.com/encode/django-rest-framework/issues/3254)
- 修复了 `BooleanField` 复选框错误显示为已选中的问题。[#3258](https://github.com/encode/django-rest-framework/issues/3258)
- 确保空复选框在所有情况下都将 `BooleanField` 正确设置为 `False`。[#2776](https://github.com/encode/django-rest-framework/issues/2776)
- 允许 `WSGIRequest.FILES` 属性，而不引发不正确的已弃用错误。[#3261](https://github.com/encode/django-rest-framework/issues/3261)
- 解决在表单中渲染嵌套序列化器的问题。[#3260](https://github.com/encode/django-rest-framework/issues/3260)
- 如果用户偶然地将序列化器实例传递给响应而不是数据，则引发错误。[#3241](https://github.com/encode/django-rest-framework/issues/3241)

### 3.2.1
**日期**：2015年8月7日。

- 修复了没有任何选项的关系选择小部件呈现。[#3237](https://github.com/encode/django-rest-framework/issues/3237)
- 在管理界面中修复 `1`，`0` 渲染为 `true`，`false`。[#3227](https://github.com/encode/django-rest-framework/issues/3227)
- 修复了在 HTML 表单输入中具有单​​个值的 ListFields。[#3238](https://github.com/encode/django-rest-framework/issues/3238)
- 允许 `request.FILES` 与 Django 的 `HTTPRequest` 类兼容。[#3239](https://github.com/encode/django-rest-framework/issues/3239)

### 3.2.0
**日期**：2015年8月6日。

- 添加 `AdminRenderer`。[#2926](https://github.com/encode/django-rest-framework/issues/2926)
- 添加 `FilePathField`。[#1854](https://github.com/encode/django-rest-framework/issues/1854)
- 将 `allow_empty` 添加到 `ListField`。[#2250](https://github.com/encode/django-rest-framework/issues/2250)
- 支持 django-guardian 1.3。[#3165](https://github.com/encode/django-rest-framework/issues/3165)
- 支持分组选择。[#3225](https://github.com/encode/django-rest-framework/issues/3225)
- 支持可浏览 API 中的错误表单。[#3024](https://github.com/encode/django-rest-framework/issues/3024)
- 允许权限类自定义错误消息。[#2539](https://github.com/encode/django-rest-framework/issues/2539)
- 支持超链接字段上 `source=<method>`。[#2690](https://github.com/encode/django-rest-framework/issues/2690)
- `ListField(allow_null=True)` 现在允许 null 作为列表值，而不是列表中的空项。[#2766](https://github.com/encode/django-rest-framework/issues/2766)
- `ManyToMany()` 映射到 `allow_empty=False`，`ManyToMany(blank=True)` 映射到 `allow_empty=True`。[#2804](https://github.com/encode/django-rest-framework/issues/2804)
- 支持主键字段的自定义序列化样式。[#2789](https://github.com/encode/django-rest-framework/issues/2789)
- `OPTIONS` 请求支持嵌套表示。[#2915](https://github.com/encode/django-rest-framework/issues/2915)
- 为具有 `OPTIONS` 请求的视图集设置 `view.action == "metadata"`。[#3115](https://github.com/encode/django-rest-framework/issues/3115)
- 支持 `UUIDField` 上的 `allow_blank`。
- 不显示包含 401 或 403 响应码的视图文档字符串。[#3216](https://github.com/encode/django-rest-framework/issues/3216)
- 解决 Django 1.8 弃用警告。[#2886](https://github.com/encode/django-rest-framework/issues/2886)
- 修复了 `DecimalField` 验证。[#3139](https://github.com/encode/django-rest-framework/issues/3139)
- 修复与 `trim_whitespace=True` 一起使用时 `allow_blank=False` 的行为。[#2712](https://github.com/encode/django-rest-framework/issues/2712)
- 修复了一些字段组合错误映射到无效 `allow_blank` 参数的问题。[#3011](https://github.com/encode/django-rest-framework/issues/3011)
- 修复具有预取和修改的查询集的输出表示。[#2704](https://github.com/encode/django-rest-framework/issues/2704) [#2727](https://github.com/encode/django-rest-framework/issues/2727)
- 修复了当 CursorPagination 被提供了某些无效的查询参数时的断言错误。[#2920](https://github.com/encode/django-rest-framework/issues/2920)
- 修复当在带有 `TokenAuthentication` 的标头中包含的无效字符时的 `UnicodeDecodeError`。[#2928](https://github.com/encode/django-rest-framework/issues/2928)
- 用 `@non_atomic_requests` 装饰器修复事务回滚。[#3016](https://github.com/encode/django-rest-framework/issues/3016)
- 使用 `SearchFilter` 修复 Oracle 数据库的重复结果问题。[#2935](https://github.com/encode/django-rest-framework/issues/2935)
- 修复可浏览 API 表单中的复选框对齐和呈现。[#2783](https://github.com/encode/django-rest-framework/issues/2783)
- 修复在表示中应该使用 `"url": null` 的未保存的文件对象。[#2759](https://github.com/encode/django-rest-framework/issues/2759)
- 修复可浏览 API 中的字段值渲染。[#2416](https://github.com/encode/django-rest-framework/issues/2416)
- 修复 `HStoreField` 以在 `DictField` 映射中包含 `allow_blank=True`。[#2659](https://github.com/encode/django-rest-framework/issues/2659)
- 许多其他的清理，错误消息的改进，私有 API 和小的修正。

***
