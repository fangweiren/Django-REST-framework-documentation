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
