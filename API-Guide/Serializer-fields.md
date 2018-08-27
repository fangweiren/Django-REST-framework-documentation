# 序列化器字段 (Serializer fields)
Form 类中的每个字段不仅负责验证数据，而且还负责“清理”它 — 将其规范化为一致的格式。—— [Django 文档](https://docs.djangoproject.com/en/stable/ref/forms/api/#django.forms.Form.cleaned_data)

序列化器字段处理原始值和内部数据类型之间的转换。它们还处理验证输入值，以及从父对象检索和设置值。

***

**注意**： 序列化器字段都声明在 `fields.py` 中，但按照惯例，应该使用 `from rest_framework import serializers` 导入它们，并引用字段作为 `serializers.<FieldName>`。

***

## 核心参数 (Core arguments)
每个序列化器字段类构造函数至少采用这些参数。某些 Field 类采用附加的、字段特定的参数，但应始终接受以下内容：

### `read_only`
只读字段包含在 API 输出中，但不应包含在创建或更新操作期间的输入中。任何不正确地包含在序列化器输入中的 'read_only' 字段将被忽略。

将此属性设置为 `True` 可确保在序列化表示时使用该字段，但在反序列化期间创建或更新实例时不使用该字段。

默认为 `False`

### `write_only`
将此属性设置为 `True` 可确保在更新或创建实例时可以使用该字段，但在序列化表示时不包括该字段。

默认为 `False`

### `required`
通常，如果在反序列化期间未提供字段，则会引发错误。如果在反序列化期间不需要此字段，则设置为 false。

将此属性设置为 `False` 还允许在序列化实例时从输出中省略对象属性或字典键。如果键不存在，它就不会包含在输出表示中。

默认为 `True`

### `default`
如果设置，则给出默认值，如果未提供输入值，将使用该字段。如果未设置，则默认行为是不填充该属性。

部分更新操作时不应用 `default`。在部分更新的情况下，只有传入数据中提供的字段将返回验证值。

可以设置为函数或其他可调用函数，在这种情况下，将在每次使用时评估该值。调用时，它不会收到任何参数。如果可调用函数具有 `set_context` 方法，那么每次在获取字段实例作为唯一参数的值之前都会调用该方法。这与[验证器](http://www.django-rest-framework.org/api-guide/validators/#using-set_context)的工作方式相同。

序列化实例时，如果实例中不存在对象属性或字典键，则将使用默认值。

请注意，设置默认值意味着该字段不是必须的。同时包括 `default` 和 `required` 的关键字参数都是无效的，并且会引发错误。

### `allow_null`
如果把 `None` 传递给序列化器字段，通常会引发错误。如果 `None` 应被视为有效值，则将此关键字参数设置为 `True`。

请注意，如果没有显式的默认值，将此参数设置为 `True` 将意味着将序列化输出的 `default` 为 `null`，但并不表示输入反序列化的默认值。

默认为 `False`

### `source`
将用于填充字段的属性的名称。可能是只接受 `self` 参数的方法，如 `URLField(source='get_absolute_url')`，或者使用点符号来遍历属性，如 `EmailField(source='user.email')`。在使用点符号时，如果在属性遍历期间任何对象不存在或为空，则可能需要提供 `default` 值。

`source='*'` 具有特殊含义，用于指示整个对象应该被传递到该字段。这对于创建嵌套表示或需要访问整个对象以确定输出表示的字段很有用。

默认为字段名称。

### `validators`
应用于传入字段输入的验证器函数列表，并且会引发验证错误或仅返回。验证器函数通常引发 `serializers.ValidationError`，但 Django 的内置 `ValidationError` 也支持与 Django 代码库或第三方 Django 包中定义的验证器兼容。

### `error_messages`
错误代码为键，错误消息为值的字典。

### `label`
一个简短的文本字符串，可用作 HTML 表单字段或其他描述性元素中字段的名称。

### `help_text`
一个文本字符串，可用作 HTML 表单字段或其他描述性元素中字段的描述。

### `initial`
用于预填充 HTML 表单字段值的值。您可以将 callable 传递给它，就像使用任何常规 Django `Field` 一样：
```python
import datetime
from rest_framework import serializers
class ExampleSerializer(serializers.Serializer):
    day = serializers.DateField(initial=datetime.date.today)
```

### `style`
用于控制渲染器如何渲染字段的键值对的字典。

这里有两个例子是 `'input_type'` 和 `'base_template'`：
```python
# 使用 <input type="password"> 作为输入。
password = serializers.CharField(
    style={'input_type': 'password'}
)

# 使用单选输入而不是选择输入。
color_channel = serializers.ChoiceField(
    choices=['red', 'green', 'blue'],
    style={'base_template': 'radio.html'}
)
```
有关更多详细信息，请参阅 [HTML 和表单](http://www.django-rest-framework.org/topics/html-and-forms/)文档。

***

# Boolean 字段 (Boolean fields)
## BooleanField
布尔表示。

当使用 HTML 编码表单输入时，注意省略总是被视为将字段设置为 `False` 的值，即使它指定了 `default=True` 选项。这是因为 HTML 复选框通过省略该值来表示未选中的状态，所以 REST framework 将省略看作是空的复选框输入。

注意，默认的 `BooleanField` 实例将使用 `required=False` 选项生成 (因为 Django `models.BooleanField` 始终为 `blank=True`)。如果想要更改此行为，请在序列化器类上显式声明 `BooleanField`。

对应于 `django.db.models.fields.BooleanField`。

**签名**：`BooleanField()`

## NullBooleanField
还接受 `None` 作为有效值的布尔表示。

对应于 `django.db.models.fields.NullBooleanField`。

**签名**：`NullBooleanField()`

***

# 字符串字段 (String fields)
## CharField
文本表示。(可选) 验证文本是否小于 `max_length` 且长于 `min_length`。

对应于 `django.db.models.fields.CharField` 或 `django.db.models.fields.TextField`。

**签名**：`CharField(max_length=None, min_length=None, allow_blank=False, trim_whitespace=True)`

- `max_length` —— 验证输入包含的字符数不超过此数目。
- `min_length` —— 验证输入包含的字符数不小于此数目。
- `allow_blank` —— 如果设置为 `True`，则空字符串应被视为有效值。如果设置为 `False`，那么空字符串被认为是无效的并会引发验证错误。默认为 `False`。
- `trim_whitespace` —— 如果设置为 `True`，则前后空格将被删除。默认为 `True`。

`allow_null` 选项也可用于字符串字段，但不鼓励使用 `allow_blank`。同时设置 `allow_blank=True` 和 `allow_null=True` 是有效的，但这样做意味着字符串表示允许有两种不同类型的空值，这可能导致数据不一致和微妙的应用程序错误。

## EmailField
文本表示，将文本验证为有效的电子邮件地址。

对应于 `django.db.models.fields.EmailField`

**签名**：`EmailField(max_length=None, min_length=None, allow_blank=False)`

## RegexField
文本表示，用于验证给定的值是否与某个正则表达式匹配。

对应于 `django.forms.fields.RegexField`

**签名**：`RegexField(regex, max_length=None, min_length=None, allow_blank=False)`

强制的 `regex` 参数可以是字符串，也可以是编译的 python 正则表达式对象。

使用 Django 的 `django.core.validators.RegexValidator` 进行验证。

## SlugField
根据模式 `[a-zA-Z0-9_-]+` 验证输入的 `RegexField`。

对应于 `django.db.models.fields.SlugField`

**签名**：`SlugField(max_length=50, min_length=None, allow_blank=False)`

## URLField
根据 URL 匹配模式验证输入的 `RegexField`。期望窗体的完全限定 URL 为 `http://<host>/<path>`。

对应于 `django.db.models.fields.URLField`。使用 Django 的 `django.core.validators.URLValidator` 进行验证。

**签名**：`URLField(max_length=200, min_length=None, allow_blank=False)`

## UUIDField
确保输入是有效 UUID 字符串的字段。`to_internal_value` 将返回 `uuid.UUID` 实例。在输出时，该字段将返回规范的连字符格式的字符串，例如：
```python
"de305d54-75b4-431b-adb2-eb6b9e546013"
```

**签名**：`UUIDField(format='hex_verbose')`

- `format`：确定 uuid 值的表示格式
  - `'hex_verbose'` —— 规范十六进制表示，包含连字符：`"5ce0e9a5-5ffa-654b-cee0-1238041fb31a"`
  - `'hex'` —— UUID 的紧凑的十六进制表示， 不包含连字符：`"5ce0e9a55ffa654bcee01238041fb31a"`
  - `'int'` —— UUID 的 128 位整数表示：`"123456789012312313134124512351145145114"`
  - `'urn'` —— UUID 的 RFC 4122 URN 表示：`"urn:uuid:5ce0e9a5-5ffa-654b-cee0-1238041fb31a"` 修改 `format` 参数只影响表示值。`to_internal_value` 接受所有格式。

## FilePathField
选择限于文件系统上某个目录中的文件名的字段。

应用于 `django.forms.fields.FilePathField`。

**签名**：`FilePathField(path, match=None, recursive=False, allow_files=True, allow_folders=False, required=None, **kwargs)`

- `path` —— FilePathField 应该从中选择的文件系统到目录的绝对路径。
- `match` —— 一个正则表达式，作为字符串，FilePathField 将用于过滤文件名。
- `recursive` —— 指定是否应该包含路径的所有子目录。默认值是 `False`。
- `allow_files` —— 指定是否应该包含指定位置的文件。默认值为 `True`。这个参数或 `allow_folders` 必须是 `True`。
- `allow_folders` —— 指定是否应该包含指定位置的文件夹。默认值是 `False`。这个参数或 `allow_files` 必须是 `True`。

## IPAddressField
确保输入是有效 IPv4 或 IPv6 字符串的字段。

对应于 `django.forms.fields.IPAddressField` 和 `django.forms.fields.GenericIPAddressField`。

**签名**：`IPAddressField(protocol='both', unpack_ipv4=False, **options)`

- `protocol` 限制指定协议的有效输入。可接受的值是 'both' (默认)，'IPv4' 或 'IPv6'。匹配不区分大小写。
- `unpack_ipv4` 解压 IPv4 映射地址，如 ::ffff:192.0.2.1。如果启用此选项，则该地址将解压到 192.0.2.1。默认为禁用。只能在 protocol 设置为 'both' 时使用。

***

# 数字字段 (Numeric fields)
## IntegerField
整数表示。

对应于 `django.db.models.fields.IntegerField`， `django.db.models.fields.SmallIntegerField`，`django.db.models.fields.PositiveIntegerField` 和 `django.db.models.fields.PositiveSmallIntegerField`。

**签名**：`IntegerField(max_value=None, min_value=None)`

- `max_value` 验证所提供的数字不大于这个值。
- `min_value` 验证所提供的数字不小于这个值。

## FloatField
浮点表示。

对应于 `django.db.models.fields.FloatField`。

**签名**：`FloatField(max_value=None, min_value=None)`

- `max_value` 验证所提供的数字不大于这个值。
- `min_value` 验证所提供的数字不小于这个值。

## DecimalField
十进制表示，由 `Decimal` 实例在 Python 中表示。

对应于 `django.db.models.fields.DecimalField`。

**签名**：`DecimalField(max_digits, decimal_places, coerce_to_string=None, max_value=None, min_value=None)`

- `max_digits` 数字中允许的最大位数。它必须是 `None` 或大于等于 `decimal_places` 的整数。
- `decimal_places` 以数字存储的小数位数。
- `coerce_to_string` 如果用于表示应返回字符串值，则设置为 `True`；如果应返回 `Decimal` 对象，则设置为 `False`。默认与 `COERCE_DECIMAL_TO_STRING` 设置中的键值相同，除非重写，否则将为 `True`。如果序列化器返回 `Decimal` 对象，则最终输出格式将由渲染器确定。请注意，设置 `localize` 会将值强制为 `True`。
- `max_value` 验证所提供的数字不大于这个值。
- `min_value` 验证所提供的数字不小于这个值。
- `localize` 设置为 `True` 以便基于当前区域启用输入和输出本地化。这也将强制 `coerce_to_string` 为 `True`。默认为 `False`。请注意，如果在设置文件中设置了 `USE_L10N = True`，则会启用数据格式化。
- `rounding` 设置量化到配置精度时使用的舍入模式。有效值是 [`decimal` 模块舍入模式](https://docs.python.org/3/library/decimal.html#rounding-modes)。默认为 `None`。

#### 用法示例 (Example usage)
要使用 2 位小数的精度验证最大为 999 的数字，您可以使用：
```python
serializers.DecimalField(max_digits=5, decimal_places=2)
```
并且以 10 位小数的精度验证最多不到 10 亿的数字：
```python
serializers.DecimalField(max_digits=19, decimal_places=10)
```
该字段还带有一个可选参数 `coerce_to_string`。如果设置为 `True`，则表示将以字符串形式输出。如果设置为 `False`，则表示将保留为 `Decimal` 实例，最终表示形式将由渲染器确定。

如果未设置，则默认与 `COERCE_DECIMAL_TO_STRING` 设置相同的值，除非另行设置，否则该值为 `True`。

***

# 日期和时间字段 (Date and time fields)
## DateTimeField
日期和时间表示。

对应于 `django.db.models.fields.DateTimeField`。

**签名**：`DateTimeField(format=api_settings.DATETIME_FORMAT, input_formats=None)`

- `format` —— 表示输出格式的字符串。如果未指定，则默认为与 `DATETIME_FORMAT` 设置键相同的值，除非设置，否则将为 `'iso-8601'`。设置为格式字符串表明 `to_representation` 返回值应该被强制为字符串输出。格式字符串描述如下。将此值设置为 `None` 表明应由 `to_representation` 返回 Python `datetime` 对象。在这种情况下，日期时间编码将由渲染器确定。
- `input_formats` —— 表示可用于解析日期的输入格式的字符串列表。如果未指定，则将使用 `DATETIME_INPUT_FORMATS` 设置，默认为 `['iso-8601']`。

#### `DateTimeField` 格式字符串。
格式化字符串可以是显式指定的 Python strftime 格式，也可以是表明使用 ISO 8601 样式的日期时间的特殊字符串 `iso-8601`。(例如 `'2013-01-29T12:34:56.000000Z'`)

当格式化日期时间对象使用 `None` 值时，`to_representation` 将返回，最终的输出表示将由渲染器类决定。

#### `auto_now` 和 `auto_now_add` 模型字段。
当使用 `ModelSerializer` 或 `HyperlinkedModelSerializer` 时，请注意，任何带有 `auto_now=True` 或 `auto_now_add=True` 的模型字段将默认使用 `read_only=True` 的序列化字段。

如果想重写此行为，则需要在序列化器中显式声明 `DateTimeField`。例如：
```python
class CommentSerializer(serializers.ModelSerializer):
    created = serializers.DateTimeField()

    class Meta:
        model = Comment
```

## DateField
日期表示。

对应于 `django.db.models.fields.DateField`

**签名**：`DateField(format=api_settings.DATE_FORMAT, input_formats=None)`

- `format` —— 表示输出格式的字符串。如果未指定，则默认与 `DATE_FORMAT` 设置的键相同的值，除非设置，否则将为 `'iso-8601'`。设置为格式化字符串则表明 `to_representation` 返回值应被强制为字符串输出。格式化字符串描述如下。将此值设置为 `None` 表示 Python `date` 对象应由 `to_representation` 返回。在这种情况下，日期编码将由渲染器确定。
- `input_formats` —— 表示可用于解析日期的输入格式的字符串列表。如果未指定，将使用 `DATE_INPUT_FORMATS` 设置，默认为 `['iso-8601']`。

#### `DateField` 格式化字符串
格式化字符串可以是显式指定的 [Python strftime 格式](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)，也可以是表明使用 [ISO 8601](https://www.w3.org/TR/NOTE-datetime) 样式的日期的特殊字符串 `iso-8601`。(例如 `'2013-01-29'`)

## TimeField
时间表示。

对应于 `django.db.models.fields.TimeField`

**签名**：`TimeField(format=api_settings.TIME_FORMAT, input_formats=None)`

- `format` —— 表示输出格式的字符串。如果未指定，则默认与 `TIME_FORMAT` 设置的键相同的值，除非设置，否则将为 `'iso-8601'`。设置为格式化字符串则表明 `to_representation` 返回值应被强制为字符串输出。格式化字符串描述如下。将此值设置为 `None` 表示 Python `time` 对象应由 `to_representation` 返回。在这种情况下，日期编码将由渲染器确定。
- `input_formats` —— 表示可用于解析日期的输入格式的字符串列表。如果未指定，将使用 `TIME_INPUT_FORMATS` 设置，默认为 `['iso-8601']`。

#### `TimeField` 格式化字符串
格式化字符串可以是显式指定的 [Python strftime 格式](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)，也可以是表明使用 [ISO 8601](https://www.w3.org/TR/NOTE-datetime) 样式的日期的特殊字符串 `iso-8601`。(例如 `'12:34:56.000000'`)

## DurationField
持续时间表示。对应于 `django.db.models.fields.DurationField`

这些字段的 `validated_data` 将包含 `datetime.timedelta` 实例。该表示形式是遵循格式 `'[DD] [HH:[MM:]]ss[.uuuuuu]'` 的字符串。

**签名**：`DurationField()`

***

# 选择字段 (Choice selection fields)
## ChoiceField
可以接受有限选项集中的值的字段。

如果相应的模型字段包含 `choices=…` 参数，则使用 `ModelSerializer` 自动生成字段。

**签名**：`ChoiceField(choices)`

- `choices` —— 有效值列表，或 `(key, display_name)` 元组列表。
- `allow_blank` —— 如果设置为 `True`，则空字符串应被视为有效值。如果设置为 `False`，那么空字符串被认为是无效的并会引发验证错误。默认是 `False`。
- `html_cutoff` —— 如果设置，这将是 HTML 选择下拉菜单中显示的选项的最大数量。可用于确保自动生成具有非常大的可能选择的 `ChoiceField`，而不会阻止模板的渲染。默认是 `None`。
- `html_cutoff_text` —— 如果设置，如果在 HTML 选择下拉菜单中截断了最大数量的项，则将显示文本指示符。默认为 `"More than {count} items…"`

`allow_blank` 和 `allow_null` 都是 `ChoiceField` 上的有效选项，但强烈建议只使用一个而不是两个都用。对于文本选择，`allow_blank` 应该是首选，而对于数字或其他非文本选择，`allow_null` 应该是首选。

## MultipleChoiceField
可以接受一组零个、一个或多个值的字段，从一组有限的选选项中选择。采用一个必填的参数。`to_internal_value` 返回包含选定值的 `set`。

**签名**：`MultipleChoiceField(choices)`

- `choices` —— 有效值列表，或 `(key, display_name)` 元组列表。
- `allow_blank` —— 如果设置为 `True`，则空字符串应被视为有效值。如果设置为 `False`，那么空字符串被认为是无效的并会引发验证错误。默认是 `False`。
- `html_cutoff` —— 如果设置，这将是 HTML 选择下拉菜单中显示的选项的最大数量。可用于确保自动生成具有非常大的可能选择的 `ChoiceField`，而不会阻止模板的渲染。默认是 `None`。
- `html_cutoff_text` —— 如果设置，如果在 HTML 选择下拉菜单中截断了最大数量的项，则将显示文本指示符。默认为 `"More than {count} items…"`

与 `ChoiceField` 一样，`allow_blank` 和 `allow_null` 都是 `ChoiceField` 选项都是有效的，但强烈建议只使用一个而不是两个都用。对于文本选择，`allow_blank` 应该是首选，而对于数字或其他非文本选择，`allow_null` 应该是首选。

***

# 文件上传字段 (File upload fields)
#### 解析器和文件上传 (Parsers and file uploads.)
`FileField` 和 `ImageField` 类只适用于 `MultiPartParser` 或 `FileUploadParser`。大多数解析器，例如，JSON 不支持文件上传。Django 的常规 `FILE_UPLOAD_HANDLERS` 用于处理上传的文件。

## FileField
文件表示。执行 Django 的标准 FileField 验证。

对应于 `django.forms.fields.FileField`

**签名**：`FileField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL)`

- `max_length` —— 指定文件名的最大长度。
- `allow_empty_file` —— 指定是否允许空文件。
- `use_url` —— 如果设置为 `True`，则 URL 字符串值将用于输出表示。如果设置为 `False`，则文件名字符串值将用于输出表示。默认为 `UPLOADED_FILES_USE_URL` 设置键的值，除非另有设置，否则为 `True`。

## ImageField
图像表示。验证上传的文件内容是否匹配已知的图像格式。

对应于 `django.forms.fields.ImageField`

**签名**：`ImageField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL)`

- `max_length` —— 指定文件名的最大长度。
- `allow_empty_file` —— 指定是否允许空文件。
- `use_url` —— 如果设置为 `True`，则 URL 字符串值将用于输出表示。如果设置为 `False`，则文件名字符串值将用于输出表示。默认为 `UPLOADED_FILES_USE_URL` 设置键的值，除非另有设置，否则为 `True`。

需要 `Pillow` 包或 `PIL` 包。建议使用 `Pillow` 包，因为不再主动维护 `PIL`。

***

# 复合字段 (Composite fields)
## ListField
验证对象列表的字段类。

**签名**：`ListField(child=<A_FIELD_INSTANCE>, min_length=None, max_length=None)`

- `child` —— 用于验证列表中的对象的字段实例。如果未提供此参数，则列表中的对象将不会被验证。
- `min_length` —— 验证列表中包含的元素数量不少于这个数。
- `max_length` —— 验证列表中包含的元素数量不超过这个数。

例如，要验证整数列表，可以使用以下内容：
```python
scores = serializers.ListField(
   child=serializers.IntegerField(min_value=0, max_value=100)
)
```

`ListField` 类还支持声明性样式，允许您编写可重用列表字段类。
```python
class StringListField(serializers.ListField):
    child = serializers.CharField()
```
我们现在可以在我们的应用程序中重新使用我们自定义的 `StringListField` 类，而无需为其提供 `child` 参数。

## DictField
验证对象字典的字段类。`DictField` 中的键始终假定为字符串值。

**签名**：`DictField(child=<A_FIELD_INSTANCE>)`

- `child` —— 用于验证字典中的值的字段实例。如果未提供此参数，则映射中的值将不会被验证。

例如，要创建一个验证字符串到字符串映射的字段，可以这样写：
```python
document = DictField(child=CharField())
```
您也可以使用声明式样式，与 `ListField` 一样。例如：
```python
class DocumentField(DictField):
    child = CharField()
```

## HStoreField
与 Django 的 postgres `HStoreField` 兼容的预配置的 `DictField`。

**签名**：`HStoreField(child=<A_FIELD_INSTANCE>)`

- `child` —— 用于验证字典中的值的字段实例。默认子字段接受空字符串和空值。

请注意，子字段**必须**是 `CharField` 的实例，因为 hstore 扩展将值存储为字符串。

## JSONField
验证传入数据结构由有效 JSON 基元组成的字段类。在它的交替二进制模式中，它将表示和验证 JSON 编码的二进制字符串。

**签名**：`JSONField(binary)`

- `binary` —— 如果设置为 `True`，那么该字段将输出并验证 JSON 编码的字符串，而不是原始数据结构。默认是 `False`。

***

# 各种各样的字段 (Miscellaneous fields)
## ReadOnlyField
只返回字段的值而不进行修改的字段类。

当包含与属性相关的字段名称而不是模型字段时，默认情况下，此字段与 `ModelSerializer` 一起使用。

**签名**：`ReadOnlyField()`

例如，如果 `has_expired` 是 `Account` 模型上的属性，则以下序列化器会自动将其生成为 `ReadOnlyField`：
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'has_expired')
```

## HiddenField
不根据用户输入获取值，而是从默认值或可调用值中获取值的字段类。

**签名**：`HiddenField()`

例如，要包含始终提供当前时间作为序列化器验证数据的一部分的字段，您将使用以下内容：
```python
modified = serializers.HiddenField(default=timezone.now)
```
如果需要运行某些基于预先提供的字段值的验证，通常只需要 `HiddenField` 类，而不是将所有这些字段公开给最终用户。

有关 `HiddenField` 的更多示例，请参阅[验证器](http://www.django-rest-framework.org/api-guide/validators/)文档。

## ModelField
可以绑定到任意模型字段的通用字段。`ModelField` 类将序列化/反序列化的任务委托给其关联的模型字段。该字段可用于为自定义模型字段创建序列化器字段，而无需创建新的自定义序列化器字段。

`ModelSerializer` 使用此字段来对应自定义模型字段类。

**签名**：`ModelField(model_field=<Django ModelField instance>)`

`ModelField` 类通常供内部使用，但如果需要，可以由 API 使用。为了正确实例化 `ModelField`，必须传递一个附属于实例化模型的字段。例如：`ModelField(model_field=MyModel()._meta.get_field('custom_field'))`

## SerializerMethodField
这是一个只读字段。它通过调用附属于序列化器类上的方法来获取其值。它可用于将任何类型的数据添加到对象的序列化表示中。

**签名**：`SerializerMethodField(method_name=None)`

- `method_name` —— 要调用序列化器上的方法名称。如果不包含，则默认为 `get_<field_name>`。

`method_name` 参数引用的序列化器方法应该接受一个参数 (除了 self )，这是被序列化的对象。它应该返回您想要包含在对象的序列化表示中的任何内容。例如：
```python
from django.contrib.auth.models import User
from django.utils.timezone import now
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    days_since_joined = serializers.SerializerMethodField()

    class Meta:
        model = User

    def get_days_since_joined(self, obj):
        return (now() - obj.date_joined).days
```

***

# 自定义字段 (Custom fields)
如果想要创建自定义字段，则需要子类化 `Field`，并重写 `.to_representation()` 和 `.to_internal_value()` 方法中的一个或两个。这两种方法用于在初始数据类型和原始可序列化数据类型之间进行转换。原始数据类型通常是数字，字符串，布尔值，`date`/`time`/`datetime` 或 `None`。它们也可以是任何仅包含其他原始对象的列表或字典。可能支持其他类型，具体取决于您使用的渲染器。

调用 `.to_representation()` 方法将初始数据类型转换为原始的可序列化数据类型。

调用 `to_internal_value()` 方法将原始数据类型恢复为其内部 python 表示。如果数据无效，此方法应该引发 `serializers.ValidationError` 异常。

请注意，2.x 版本中存在的 `WritableField` 类不再存在。 如果字段支持数据输入，则应该子类化 `Field` 并重写 `to_internal_value()`。

## 举个栗子
### 基本自定义字段 (A Basic Custom Field)
让我们来看一个序列化一个表示 RGB 颜色值的类的例子：
```python
class Color(object):
    """
    在 RGB 颜色空间中表示的颜色。
    """
    def __init__(self, red, green, blue):
        assert(red >= 0 and green >= 0 and blue >= 0)
        assert(red < 256 and green < 256 and blue < 256)
        self.red, self.green, self.blue = red, green, blue

class ColorField(serializers.Field):
    """
    颜色对象被序列化为 'rgb(#, #, #)' 表示法。
    """
    def to_representation(self, obj):
        return "rgb(%d, %d, %d)" % (obj.red, obj.green, obj.blue)

    def to_internal_value(self, data):
        data = data.strip('rgb(').rstrip(')')
        red, green, blue = [int(col) for col in data.split(',')]
        return Color(red, green, blue)
```
默认情况下，字段值被视为映射到对象上的属性。如果需要自定义访问和设置字段值的方式，则需要重写 `.get_attribute()` 和/或 `.get_value()`。

例如，让我们创建一个用于表示被序列化对象的类名的字段：
```python
class ClassNameField(serializers.Field):
    def get_attribute(self, obj):
        # We pass the object instance onto `to_representation`,
        # not just the field attribute.
        return obj

    def to_representation(self, obj):
        """
        Serialize the object's class name.
        """
        return obj.__class__.__name__
```

### 抛出验证错误 (Raising validation errors)
我们上面的 `ColorField` 类目前不执行任何数据验证。要指示无效数据，我们应引发 `serializers.ValidationError`，如下所示：
```python
def to_internal_value(self, data):
    if not isinstance(data, six.text_type):
        msg = 'Incorrect type. Expected a string, but got %s'
        raise ValidationError(msg % type(data).__name__)

    if not re.match(r'^rgb\([0-9]+,[0-9]+,[0-9]+\)$', data):
        raise ValidationError('Incorrect format. Expected `rgb(#,#,#)`.')

    data = data.strip('rgb(').rstrip(')')
    red, green, blue = [int(col) for col in data.split(',')]

    if any([col > 255 or col < 0 for col in (red, green, blue)]):
        raise ValidationError('Value out of range. Must be between 0 and 255.')

    return Color(red, green, blue)
```
`.fail()` 方法是引发 `ValidationError` 的快捷方式，它从 `error_messages` 字典中获取消息字符串。例如：
```python
default_error_messages = {
    'incorrect_type': 'Incorrect type. Expected a string, but got {input_type}',
    'incorrect_format': 'Incorrect format. Expected `rgb(#,#,#)`.',
    'out_of_range': 'Value out of range. Must be between 0 and 255.'
}

def to_internal_value(self, data):
    if not isinstance(data, six.text_type):
        self.fail('incorrect_type', input_type=type(data).__name__)

    if not re.match(r'^rgb\([0-9]+,[0-9]+,[0-9]+\)$', data):
        self.fail('incorrect_format')

    data = data.strip('rgb(').rstrip(')')
    red, green, blue = [int(col) for col in data.split(',')]

    if any([col > 255 or col < 0 for col in (red, green, blue)]):
        self.fail('out_of_range')

    return Color(red, green, blue)
```
这种样式使您的错误信息更清晰，并且代码更独立，应该是首选。

### 使用 `source='*'`
在这里，我们将以一个带有 `x_coordinate` 和 `y_coordinate` 属性的平面 `DataPoint` 模型为例。
```python
class DataPoint(models.Model):
    label = models.CharField(max_length=50)
    x_coordinate = models.SmallIntegerField()
    y_coordinate = models.SmallIntegerField()
```
使用自定义字段和 `source ='*'`，我们可以提供坐标对的嵌套表示：
```python
class CoordinateField(serializers.Field):

    def to_representation(self, obj):
        ret = {
            "x": obj.x_coordinate,
            "y": obj.y_coordinate
        }
        return ret

    def to_internal_value(self, data):
        ret = {
            "x_coordinate": data["x"],
            "y_coordinate": data["y"],
        }
        return ret


class DataPointSerializer(serializers.ModelSerializer):
    coordinates = CoordinateField(source='*')

    class Meta:
        model = DataPoint
        fields = ['label', 'coordinates']
```
请注意，此示例不处理验证。部分原因是，在真实的项目中，使用 `source ='*'` 的嵌套序列化器可以更好地处理坐标嵌套，并且带有两个 `IntegerField` 实例，每个实例都有自己的 `source` 指向相关字段。

不过，这个例子的关键点是：

- `to_representation` 传递整个 `DataPoint` 对象， 并且会映射到所需的输出。
```python
>>> instance = DataPoint(label='Example', x_coordinate=1, y_coordinate=2)
>>> out_serializer = DataPointSerializer(instance)
>>> out_serializer.data
ReturnDict([('label', 'testing'), ('coordinates', {'x': 1, 'y': 2})])
```
- 除非我们的字段是只读的，否则 `to_internal_value` 必须映射回适合更新目标对象的字典。使用 `source='*'`，`to_internal_value` 的返回将更新根验证的数据字典，而不是单个键。
```python
>>> data = {
...     "label": "Second Example",
...     "coordinates": {
...         "x": 3,
...         "y": 4,
...     }
... }
>>> in_serializer = DataPointSerializer(data=data)
>>> in_serializer.is_valid()
True
>>> in_serializer.validated_data
OrderedDict([('label', 'Second Example'),
             ('y_coordinate', 4),
             ('x_coordinate', 3)])
```
为了完整性，我们再次做同样的事情，但是使用上面建议的嵌套序列化方法：
```python
class NestedCoordinateSerializer(serializers.Serializer):
    x = serializers.IntegerField(source='x_coordinate')
    y = serializers.IntegerField(source='y_coordinate')


class DataPointSerializer(serializers.ModelSerializer):
    coordinates = NestedCoordinateSerializer(source='*')

    class Meta:
        model = DataPoint
        fields = ['label', 'coordinates']
```
在这里，目标和源属性对(`x` 和 `x_coordinate`，`y` 和 `y_coordinate`)之间的映射在 `IntegerField` 声明中处理。这是使用了 `source ='*'` 的 `NestedCoordinateSerializer`。

新的 `DataPointSerializer` 展现出与自定义字段方法相同的行为。

序列化：
```python
>>> out_serializer = DataPointSerializer(instance)
>>> out_serializer.data
ReturnDict([('label', 'testing'),
            ('coordinates', OrderedDict([('x', 1), ('y', 2)]))])
```
反序列化：
```python
>>> in_serializer = DataPointSerializer(data=data)
>>> in_serializer.is_valid()
True
>>> in_serializer.validated_data
OrderedDict([('label', 'still testing'),
             ('x_coordinate', 3),
             ('y_coordinate', 4)])
```
但我们也免费获得内置验证：
```python
>>> invalid_data = {
...     "label": "still testing",
...     "coordinates": {
...         "x": 'a',
...         "y": 'b',
...     }
... }
>>> invalid_serializer = DataPointSerializer(data=invalid_data)
>>> invalid_serializer.is_valid()
False
>>> invalid_serializer.errors
ReturnDict([('coordinates',
             {'x': ['A valid integer is required.'],
              'y': ['A valid integer is required.']})])
```
由于这个原因，嵌套的序列化器方法将是第一个尝试。当嵌套的序列化器变得不可行或过于复杂时，您将使用自定义字段方法。

# 第三方包 (Third party packages)
以下是可用的第三方包。

## DRF Compound Fields
[drf-compound-fields](https://drf-compound-fields.readthedocs.io/) 包提供“复合”序列化器字段，例如简单值列表，它可以由其他字段描述，而不是使用 `many = True` 选项的序列化器。还提供了类型化字典和值的字段，它们可以是特定类型或该类型的项列表。

## DRF Extra Fields
[drf-extra-fields](https://github.com/Hipo/drf-extra-fields) 包为 REST framework 提供了额外的序列化器字段，包括 `Base64ImageField` 和 `PointField` 类。

## djangorestframework-recursive
[djangorestframework-recursive](https://github.com/heywbj/django-rest-framework-recursive) 包为序列化和反序列化递归结构提供了 `RecursiveField`。

## django-rest-framework-gis
[django-rest-framework-gis](https://github.com/djangonauts/django-rest-framework-gis) 包为 django rest framework 提供了地理插件，如 `GeometryField` 字段和 GeoJSON 序列化器。

## django-rest-framework-hstore
[django-rest-framework-hstore](https://github.com/djangonauts/django-rest-framework-hstore) 包提供了 `HStoreField` 来支持 [django-hstore](https://github.com/djangonauts/django-hstore) `DictionaryField` 模型字段。
