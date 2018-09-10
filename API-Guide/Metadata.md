# 元数据 (Metadata)
[`OPTIONS`] 方法允许客户端在无暗示资源动作或发起资源检索的情况下确定与资源相关的选项和/或要求，或服务器的能力。—— [RFC7231, Section 4.3.7.](https://tools.ietf.org/html/rfc7231#section-4.3.7)

REST framework 包括一个用于确定您的 API 应该如何响应 `OPTIONS` 请求可配置的机制。这允许您返回 API 模式或其他资源信息。

目前还没有任何广泛采用的约定来确定对于 HTTP `OPTIONS` 请求应该返回什么类型的响应，因此我们提供了一个特别的样式，它返回一些有用的信息。

下面是一个演示默认情况下返回的信息的示例响应。
```python
HTTP 200 OK
Allow: GET, POST, HEAD, OPTIONS
Content-Type: application/json

{
    "name": "To Do List",
    "description": "List existing 'To Do' items, or create a new item.",
    "renders": [
        "application/json",
        "text/html"
    ],
    "parses": [
        "application/json",
        "application/x-www-form-urlencoded",
        "multipart/form-data"
    ],
    "actions": {
        "POST": {
            "note": {
                "type": "string",
                "required": false,
                "read_only": false,
                "label": "title",
                "max_length": 100
            }
        }
    }
}
```

## 设置元数据方案 (Setting the metadata scheme)
您可以使用 `'DEFAULT_METADATA_CLASS'` 设置键全局设置元数据类：
```python
REST_FRAMEWORK = {
    'DEFAULT_METADATA_CLASS': 'rest_framework.metadata.SimpleMetadata'
}
```

或者，您可以为视图单独设置元数据类：
```python
class APIRoot(APIView):
    metadata_class = APIRootMetadata

    def get(self, request, format=None):
        return Response({
            ...
        })
```

REST framework 包只包含一个名为 `SimpleMetadata` 的元数据类实现。如果您想使用另一种样式，则需要实现一个自定义的元数据类。

## 创建模式端点 (Creating schema endpoints)
如果您对创建使用常规 `GET` 请求访问的模式端点有特定的要求，您可考虑重新使用元数据 API 来进行这样的操作。

例如，可以在视图集上使用以下附加路由来提供可链接的模式端点。
```python
@action(methods=['GET'], detail=False)
def schema(self, request):
    meta = self.metadata_class()
    data = meta.determine_metadata(request, self)
    return Response(data)
```

这有几个您可能会选择采用此方法的原因，包括 `OPTIONS` 响应[不可缓存](https://www.mnot.net/blog/2012/10/29/NO_OPTIONS)。

***

# 自定义元数据类 (Custom metadata classes)
如果您想提供一个自定义的元数据类，您应该继承 `BaseMetadata` 并且实现 `determine_metadata(self, request, view)` 方法。

您可能想要做的有用的事情包括返回模式信息，使用 [JSON 模式](http://json-schema.org/)等格式，或将调试信息返回给管理员用户。

## 举个栗子
以下类可用于限制返回到 `OPTIONS` 请求的信息。
```python
class MinimalMetadata(BaseMetadata):
    """
    不要为 “OPTIONS” 请求包含字段和其他信息。
    只需返回名称和说明即可。
    """
    def determine_metadata(self, request, view):
        return {
            'name': view.get_view_name(),
            'description': view.get_view_description()
        }
```

然后配置您的设置以使用此自定义类：
```python
REST_FRAMEWORK = {
    'DEFAULT_METADATA_CLASS': 'myproject.apps.core.MinimalMetadata'
}
```

# 第三方包 (Third party packages)
以下第三方包提供其他元数据实现。

## DRF-schema-adapter
[drf-schema-adapter](https://github.com/drf-forms/drf-schema-adapter) 是可以更轻松地为前端框架和库提供模式信息一组工具。它提供了元数据混合以及 2 个元数据类和适合于生成 [JSON 模式](http://json-schema.org/)的多个适配器以及各种库可读的模式信息。

您也可以编写自己的适配器来处理特定的前端。如果您希望这样做，它还提供了一个可以将这些模式信息导出到 json 文件的导出器。
