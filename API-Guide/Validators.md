# 验证器 (Validators)
验证器对于在不同类型的字段之间重用验证逻辑非常有用。—— [Django 文档](https://docs.djangoproject.com/en/stable/ref/validators/)

大多数情况下，您在 REST framework 中处理验证时，只需依赖默认的字段验证，或者在序列化器或字段类上编写显式的验证方法。

但是，有时您需要将验证逻辑放入可重用的组件中，以便可以在整个代码库中轻松地重用它。这可以通过使用验证器函数和验证器类来实现。

# REST framework 中的验证 (Validation in REST framework)
Django REST framework 序列化器中的验证与 Django 的 `ModelForm` 类中的验证工作方式略有不同。

使用 `ModelForm`，验证一部分在表单上执行，一部分在模型实例上执行。使用 REST framework ，验证完全在序列化器上执行。这是有利的，原因如下：

- 它引入了适当的关注点分离，使您的代码行为更加明显。
- 使用快捷的 `ModelSerializer` 类和使用显式的 `Serializer` 类可以轻松切换。任何用于 `ModelSerializer` 的验证行为都很容易复制。
- 打印序列化器实例的 `repr` 将准确显示它应用的验证规则。在模型实例上没有额外的隐藏验证行为被调用。

当您使用 `ModelSerializer` 时，所有这些都会自动为您处理。如果您想转而使用 `Serializer` 类，那么需要显式地定义验证规则。

#### 举个栗子
作为 REST framework 如何使用显式验证的一个示例，我们将使用一个具有唯一性约束的字段的简单模型类。
```python
class CustomerReportRecord(models.Model):
    time_raised = models.DateTimeField(default=timezone.now, editable=False)
    reference = models.CharField(unique=True, max_length=20)
    description = models.TextField()
```
这是一个基本的 `ModelSerializer`，我们可以用它来创建或更新 `CustomerReportRecord` 实例：
```python
class CustomerReportSerializer(serializers.ModelSerializer):
    class Meta:
        model = CustomerReportRecord
```
如果我们使用 `manage.py shell` 打开 Django shell，我们现在可以
```python
>>> from project.example.serializers import CustomerReportSerializer
>>> serializer = CustomerReportSerializer()
>>> print(repr(serializer))
CustomerReportSerializer():
    id = IntegerField(label='ID', read_only=True)
    time_raised = DateTimeField(read_only=True)
    reference = CharField(max_length=20, validators=[<UniqueValidator(queryset=CustomerReportRecord.objects.all())>])
    description = CharField(style={'type': 'textarea'})
```
这里有趣的一点是 `reference` 字段。我们可以看到，序列化器字段上的验证器显式强制执行唯一性约束。

由于这种更显式的样式，REST framework 包含了一些在核心 Django 中不可用的验证器类。下面详细介绍这些类。

***

## UniqueValidator
该验证器可用于在模型字段上强制实施 `unique=True` 约束。它需要一个必需的参数和一个可选的 `messages` 参数：

- `queryset` 必须 - 这是强制执行唯一性的查询集。
- `message` - 验证失败时使用的错误消息。
- `lookup` - lookup 用于查找具有验证值的现有实例。默认为 `'exact'`。

此验证器应该应用于序列化器字段，如下所示：
```python
from rest_framework.validators import UniqueValidator

slug = SlugField(
    max_length=100,
    validators=[UniqueValidator(queryset=BlogPost.objects.all())]
)
```

## UniqueTogetherValidator
此验证器可用于在模型实例上强制执行 `unique_together` 约束。它有两个必需的参数和一个可选的 `messages` 参数：

- `queryset` 必须 - 这是强制执行唯一性的查询集。
- `fields` 必须 - 生成唯一集合的字段名称的列表或元组。这些必须作为序列化器类的字段存在。
- `message` - 验证失败时使用的错误消息。

此验证器应该应用于序列化器类，如下所示：
```python
from rest_framework.validators import UniqueTogetherValidator

class ExampleSerializer(serializers.Serializer):
    # ...
    class Meta:
        # ToDo 项属于父列表，并具有由 'position' 字段定义的排序。给定列表中没有两个项目可以共享相同的位置。
        validators = [
            UniqueTogetherValidator(
                queryset=ToDoItem.objects.all(),
                fields=('list', 'position')
            )
        ]
```

***

**注意**：`UniqueTogetherValidation` 类始终施加一个隐式约束，即它所应用的所有字段都是按必须处理的。具有 `default` 值的字段是个例外，因为它们总是提供一个值，即使在用户输入中省略。

***

## UniqueForDateValidator
## UniqueForMonthValidator
## UniqueForYearValidator
这些验证器可用于在模型实例上强制执行 `unique_for_date`，`unique_for_month` 和 `unique_for_year` 约束。他们有以下参数：

- `queryset` 必须 - 这是强制执行唯一性的查询集。
- `fields` 必须 - 将验证给定日期范围中唯一性的字段名。这必须作为序列化类中的字段存在。
- `date_field` 必须 - 将用于确定唯一性约束的日期范围的字段名称。这必须作为序列化器类中的字段存在。
- `message` - 验证失败时使用的错误消息。

此验证器应该应用于序列化器类，如下所示：
```python
from rest_framework.validators import UniqueForYearValidator

class ExampleSerializer(serializers.Serializer):
    # ...
    class Meta:
        # Blog posts should have a slug that is unique for the current year.
        validators = [
            UniqueForYearValidator(
                queryset=BlogPostItem.objects.all(),
                field='slug',
                date_field='published'
            )
        ]
```
用于验证的日期字段始终需要出现在序列化器类中。不能只依赖模型类的 `default=…`，因为要用于默认值的值在验证运行之后才会生成。

你可能需要使用几种样式，具体取决于您希望 API 如何展现。如果您使用的是 `ModelSerializer` ，可能只需依赖 REST framework 为您生成的默认值，但如果你使用的是 `Serializer` 或只想更明确的控制，请使用下面演示的样式。

#### 使用可写日期字段 (Using with a writable date field.)
如果您希望日期字段可写，唯一值得注意的是您应确保输入数据始终可用，或者通过设置 `default` 参数，或者设置 `required = True`。
```python
published = serializers.DateTimeField(required=True)
```

#### 使用只读日期字段 (Using with a read-only date field.)
如果你希望日期字段可见，但用户无法编辑，请设置 `read_only=True` 并另外设置 `default=...` 参数。
```python
published = serializers.DateTimeField(read_only=True, default=timezone.now)
```
该字段对用户不可写，但默认值仍将传递给 `validated_data`。

#### 使用隐藏日期字段 (Using with a hidden date field.)
如果您希望日期字段对用户完全隐藏，请使用 `HiddenField`。该字段类型不接受用户输入，而是始终将其默认值返回到序列化器中的 `validated_data`。
```python
published = serializers.HiddenField(default=timezone.now)
```

***
**注意**：`UniqueFor<Range>Validation` 类强加一个隐式约束，即它所应用的字段都按必须处理的。具有 `default` 值的字段是个例外，因为它们总是提供一个值，即使在用户输入中省略。
***

# 高级字段默认值 (Advanced field defaults)
在序列化器中跨多个字段应用的验证器有时需要 API 客户端不应提供的字段输入，但可用作验证器的输入。

您可能希望用于此类验证的两种模式包括：

- 使用 `HiddenField`。该字段将出现在 `validated_data` 中，但不会在序列化器输出表示中使用。
- 使用 `read_only=True` 的标准字段，但也包含 `default=...` 参数。该字段将用于序列化器输出表示，但不能由用户直接设置。

REST framework 包含一些在此上下文中可能有用的默认值。

#### CurrentUserDefault
可用于表示当前用户的默认类。为了使用它，在实例化序列化器时，'request' 必须作为上下文字典的一部分提供。
```python
owner = serializers.HiddenField(
    default=serializers.CurrentUserDefault()
)
```

#### CreateOnlyDefault
可用于在创建操作期间仅设置默认参数的默认类。在更新期间，该字段被省略。

它接受一个参数，这是在创建操作期间应该使用的默认值或可调用参数。
```python
created_at = serializers.DateTimeField(
    default=serializers.CreateOnlyDefault(timezone.now)
)
```

***

# 验证器的限制 (Limitations of validators)
有一些模棱两可的情况，您需要显式处理验证，而不是依赖于 `ModelSerializer` 生成的默认序列化器类。

在这些情况下，您可能希望通过为序列化器 `Meta.validators` 属性指定一个空列表来禁用自动生成的验证器。

## 可选字段 (Optional fields)
默认情况下 "unique together" 验证强制所有字段都是 `required=True`。在某些情况下，您可能希望显式的将 `required=False` 应用到其中一个字段，在这种情况下，期望的验证行为是不明确的。

在这种情况下，您通常需要从序列化器类中排除验证器，而不是显式地编写任何验证逻辑，无论是在 `.validate()` 方法中，还是在视图中。

举个栗子：
```python
class BillingRecordSerializer(serializers.ModelSerializer):
    def validate(self, data):
        # 在这里或视图中应用自定义验证。

    class Meta:
        fields = ('client', 'date', 'amount')
        extra_kwargs = {'client': {'required': False}}
        validators = []  # 删除默认的 "unique together" 约束。
```

## 更新嵌套的序列化器 (Updating nested serializers)
将更新应用于现有实例时，唯一性验证器将从唯一性检查中排除当前实例。当前实例在唯一性检查的上下文中可用，因为它作为序列化器中的属性存在，最初在实例化序列化器时使用 `instance=...` 传递。

在嵌套序列化器上进行更新操作时，无法应用此排除，因为该实例不可用。

同样，您可能希望从序列化器类中显式删除验证器，并在 `.validate()` 方法或视图中显式地为验证约束编写代码。

## 调试复杂案例 (Debugging complex cases)
如果您不确定 `ModelSerializer` 类将生成什么行为，那么运行 `manage.py shell` 通常是个好主意，并打印序列化器的实例，以便您可以检查自动生成的字段和验证器。
```python
>>> serializer = MyComplexModelSerializer()
>>> print(serializer)
class MyComplexModelSerializer:
    my_fields = ...
```
还要记住，对于复杂的情况，通常最好显式地定义序列化器类，而不是依赖于默认的 `ModelSerializer` 行为。这涉及更多代码，但确保结果行为更透明。

***

# 编写自定义验证器 (Writing custom validators)
您可以使用任何 Django 现有的验证器，也可以编写自己的自定义验证器。

## 基于函数 (Function based)
验证器可以是任何可调用函数，在失败时引发 `serializers.ValidationError`。
```python
def even_number(value):
    if value % 2 != 0:
        raise serializers.ValidationError('This field must be an even number.')
```

#### 字段级验证 (Field-level validation)
您可以通过将 `.validate_<field_name>` 方法添加到 `Serializer` 子类来指定自定义字段级验证。这在 [Serializer 文档](http://www.django-rest-framework.org/api-guide/serializers/#field-level-validation)中有记录

## 基于类 (Class-based)
要编写基于类的验证器，请使用 `__call__` 方法。基于类的验证器非常有用，因为它们允许您参数化和重用行为。
```python
class MultipleOf(object):
    def __init__(self, base):
        self.base = base

    def __call__(self, value):
        if value % self.base != 0:
            message = 'This field must be a multiple of %d.' % self.base
            raise serializers.ValidationError(message)
```

#### 使用 `set_context()` (Using `set_context()`)
在某些高级情况下，您可能希望将验证器传递给作为附加上下文使用的序列化器字段。您可以通过在基于类的验证器上声明 `set_context` 方法来实现。
```python
def set_context(self, serializer_field):
    # 确定这是更新还是创建操作。
    # 在 `__call__` 中，我们可以使用该信息来修改验证行为。
    self.is_update = serializer_field.parent.instance is not None
```
