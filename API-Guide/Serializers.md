# 序列化器 (Serializers)
扩展序列化器的有用性是我们想要解决的问题。然而，这不是一个微不足道的问题，它将需要一些严肃的设计工作。—— Russell Keith-Magee, Django 用户组

序列化器允许将复杂数据 (如查询集和模型实例) 转换为可以轻松渲染成 `JSON`，`XML` 或其他内容类型的原生 Python 数据类型。序列化器还提供反序列化，在验证传入的数据之后允许解析数据转换回复杂类型。

REST framework 中的序列化器与 Django 的 `Form` 和 `ModelForm` 类非常相似。我们提供了一个 `Serializer` 类，它为您提供了强大的、通用的方法来控制响应的输出，以及一个 `ModelSerializer` 类，它为创建用于处理模型实例和查询集的序列化器提供了有用的快捷实现方式。

## 声明序列化器 (Declaring Serializers)
让我们从创建一个我们可以用于示例目的的简单对象开始：
```python
from datetime import datetime

class Comment(object):
    def __init__(self, email, content, created=None):
        self.email = email
        self.content = content
        self.created = created or datetime.now()

comment = Comment(email='leila@example.com', content='foo bar')
```
我们将声明一个序列化器，我们可以使用它来序列化和反序列化与 `Comment` 对象相应的数据。

声明序列化器看起来与声明表单非常相似：
```python
from rest_framework import serializers

class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```

## 序列化对象 (Serializing objects)
我们现在可以使用 `CommentSerializer` 来序列化 comment 或 comment 列表。同样，使用 `Serializer` 类看起来很像使用 `Form` 类。
```python
serializer = CommentSerializer(comment)
serializer.data
# {'email': 'leila@example.com', 'content': 'foo bar', 'created': '2016-01-27T15:17:10.375877'}
```
此时，我们已将模型实例转换为 Python 原生的数据类型。为了完成序列化过程，我们将数据渲染为 `json`。
```python
from rest_framework.renderers import JSONRenderer

json = JSONRenderer().render(serializer.data)
json
# b'{"email":"leila@example.com","content":"foo bar","created":"2016-01-27T15:17:10.375877"}'
```

## 反序列化对象 (Deserializing objects)
反序列化是类似的。首先我们将一个流解析为 Python 原生的数据类型...
```python
from django.utils.six import BytesIO
from rest_framework.parsers import JSONParser

stream = BytesIO(json)
data = JSONParser().parse(stream)
```
...然后我们将这些原生数据类型恢复为验证数据的字典。
```python
serializer = CommentSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# {'content': 'foo bar', 'email': 'leila@example.com', 'created': datetime.datetime(2012, 08, 22, 16, 20, 09, 822243)}
```

## 保存实例 (Saving instances)
如果我们希望能够返回基于验证数据的完整对象实例，我们需要实现 `.create()` 和 `update()` 方法中的一个或全部。举个栗子：
```python
class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()

    def create(self, validated_data):
        return Comment(**validated_data)

    def update(self, instance, validated_data):
        instance.email = validated_data.get('email', instance.email)
        instance.content = validated_data.get('content', instance.content)
        instance.created = validated_data.get('created', instance.created)
        return instance
```
如果您的对象实例对应于 Django 模型，您还需要确保这些方法将对象保存到数据库。例如，如果 `Comment` 是 Django 模型，则方法可能如下所示：
```python
	def create(self, validated_data):
        return Comment.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.email = validated_data.get('email', instance.email)
        instance.content = validated_data.get('content', instance.content)
        instance.created = validated_data.get('created', instance.created)
        instance.save()
        return instance
```
现在，在反序列化数据时，根据验证的数据我们可以调用 `.save()` 返回一个对象实例。
```python
comment = serializer.save()
```
调用 `.save()` 方法将创建新实例或者更新现有实例，具体取决于实例化序列化器类时是否传递了现有实例：
```python
# .save() will create a new instance.
serializer = CommentSerializer(data=data)

# .save() will update the existing `comment` instance.
serializer = CommentSerializer(comment, data=data)
```
`.create()` 和 `.update()` 方法都是可选的。你可以根据你序列化器类的用例不实现、或实现其中一个或都实现。

#### 将附加属性传递给 `.save()` (Passing additional attributes to `.save()`)
有时您会希望您的视图代码能够在保存实例时注入额外的数据。此额外数据可能包括当前用户，当前时间或不是请求数据一部分的其他信息。

您可以通过在调用 `.save()` 时包含其他关键字参数来执行此操作。例如：
```python
serializer.save(owner=request.user)
```
在 `.create()` 或 `.update()` 被调用时，任何其他关键字参数将被包含在 `validated_data` 参数中。

#### 直接重写 `.save()` (Overriding `.save()` directly.)
在某些情况下，`.create()` 和 `.update()` 方法名称可能没有意义。例如在 contact form 中，我们可能不会创建新的实例，而是发送电子邮件或其他消息。

在这些情况下，您可以选择直接重写 `.save()`，因为这样更具有可读性和意义。

举个栗子：
```python
class ContactForm(serializers.Serializer):
    email = serializers.EmailField()
    message = serializers.CharField()

    def save(self):
        email = self.validated_data['email']
        message = self.validated_data['message']
        send_email(from=email, message=message)
```
请注意，在上面的情况下，我们现在必须直接访问序列化器 `.validated_data` 属性。

## 验证 (Validation)
在反序列化数据时，在尝试访问经过验证的数据或保存对象实例之前，总是需要调用 `is_valid()`。如果发生任何验证错误，`.errors` 属性将包含表示结果错误消息的字典。例如：
```python
serializer = CommentSerializer(data={'email': 'foobar', 'content': 'baz'})
serializer.is_valid()
# False
serializer.errors
# {'email': [u'Enter a valid e-mail address.'], 'created': [u'This field is required.']}
```
字典中的每个键都是字段名称，值是与该字段对应的任何错误消息的字符串列表。`non_field_errors` 键也可能存在，并将列出任何常规验证错误。可以使用 REST framework 设置中的 `NON_FIELD_ERRORS_KEY` 来自定义 `non_field_errors`  键的名称。

当反序列化项目列表时，错误将作为表示每个反序列化项目的字典列表返回。

#### 引发无效数据的异常 (Raising an exception on invalid data)
`.is_valid()` 方法使用可选的 `raise_exception` 标志，如果存在验证错误，将会抛出 `serializers.ValidationError` 异常。

这些异常由 REST framework 提供的默认异常处理程序自动处理，默认情况下将返回 `HTTP 400 Bad Request` 响应。
```python
# Return a 400 response if the data was invalid.
serializer.is_valid(raise_exception=True)
```

#### 字段级别验证 (Field-level validation)
您可以通过向您的 `Serializer` 子类中添加 `.validate_<field_name>` 方法来指定自定义字段级的验证。这些类似于 Django 表单中的 `.clean_<field_name>` 方法。

这些方法采用单个参数，即需要验证的字段值。

您的 `validate_<field_name>` 方法应该返回已验证的值或者抛出 `serializers.ValidationError` 异常。例如：
```python
from rest_framework import serializers

class BlogPostSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=100)
    content = serializers.CharField()

    def validate_title(self, value):
        """
        Check that the blog post is about Django.
        """
        if 'django' not in value.lower():
            raise serializers.ValidationError("Blog post is not about Django")
        return value
```

***

注意：如果在您的序列化器上声明了 `<field_name>` 的参数为 `required=False`，那么如果不包含该字段，则此验证步骤不会发生。

***

#### 对象级别验证 (Object-level validation)
要执行需要访问多个字段的任何其他验证，请添加名为 `.validate()` 的方法到您的 `Serializer` 子类中。此方法采用单个参数，该参数是字段值的字典。如果需要，它应该抛出 `ValidationError` 异常，或者只返回经过验证的值。例如：
```python
from rest_framework import serializers

class EventSerializer(serializers.Serializer):
    description = serializers.CharField(max_length=100)
    start = serializers.DateTimeField()
    finish = serializers.DateTimeField()

    def validate(self, data):
        """
        Check that the start is before the stop.
        """
        if data['start'] > data['finish']:
            raise serializers.ValidationError("finish must occur after start")
        return data
```

#### 验证器 (Validators)
序列化器上的各个字段都可以包含验证器，通过在字段实例上声明，例如：
```python
def multiple_of_ten(value):
    if value % 10 != 0:
        raise serializers.ValidationError('Not a multiple of ten')

class GameRecord(serializers.Serializer):
    score = IntegerField(validators=[multiple_of_ten])
    ...
```
序列化器类还可以包括应用于完整字段数据集的可重用验证器。通过在内部 `Meta` 类上声明来包含这些验证器，如下所示：
```python
class EventSerializer(serializers.Serializer):
    name = serializers.CharField()
    room_number = serializers.IntegerField(choices=[101, 102, 103, 201])
    date = serializers.DateField()

    class Meta:
        # Each room only has one event per day.
        validators = UniqueTogetherValidator(
            queryset=Event.objects.all(),
            fields=['room_number', 'date']
        )
```
有关更多信息，请参阅[验证器文档](http://www.django-rest-framework.org/api-guide/validators/)。

## 访问初始数据和实例 (Accessing the initial data and instance)
将初始化对象或者查询集传递给序列化器实例时，该对象将以 `.instance` 的形式提供。如果没有传递初始化对象，那么 `.instance` 属性将是 `None`。

将数据传递给序列化器实例时，未修改的数据将以 `.initial_data` 的形式提供。如果 data 关键字参数未被传递，那么 `.initial_data` 属性将不存在。

## 部分更新 (Partial updates)
默认情况下，序列化器必须传递所有必填字段的值，否则就会引发验证错误。您可以使用 `partial` 参数以允许部分更新。
```python
# Update `comment` with partial data
serializer = CommentSerializer(comment, data={'content': u'foo bar'}, partial=True)
```

## 处理嵌套对象 (Dealing with nested objects)
前面的示例适用于处理只有简单数据类型的对象，但有时我们还需要能够代表更复杂的物体，其中对象的某些属性可能不是字符串、日期、整数这样简单的数据类型。

`Serializer` 类本身也是一种 `Field`，并且可以用来表示一个对象类型嵌套在另一个对象类型中的关系。
```python
class UserSerializer(serializers.Serializer):
    email = serializers.EmailField()
    username = serializers.CharField(max_length=100)

class CommentSerializer(serializers.Serializer):
    user = UserSerializer()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```
如果嵌套表示可以可选地接受 `None` 值，则应将 `required=False` 标志传递给嵌套的序列化器。
```python
class CommentSerializer(serializers.Serializer):
    user = UserSerializer(required=False)  # May be an anonymous user.
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```
类似地，如果嵌套表示应该是项目列表，则应将 `many = True` 标志传递给嵌套的序列化器。
```python
class CommentSerializer(serializers.Serializer):
    user = UserSerializer(required=False)
    edits = EditItemSerializer(many=True)  # A nested list of 'edit' items.
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```

## 可写的嵌套表示 (Writable nested representations)
处理支持反序列化数据的嵌套表示时，嵌套对象的任何错误都将嵌套在嵌套对象的字段名称下。
```python
serializer = CommentSerializer(data={'user': {'email': 'foobar', 'username': 'doe'}, 'content': 'baz'})
serializer.is_valid()
# False
serializer.errors
# {'user': {'email': [u'Enter a valid e-mail address.']}, 'created': [u'This field is required.']}
```
类似的，`.validated_data` 属性将包含嵌套数据结构。

#### 为嵌套表示编写 `.create()` 方法 (Writing `.create()` methods for nested representations)
如果您支持可写的嵌套表示，则需要编写处理保存多个对象的 `.create()` 或 `.update()` 方法。

下面的示例演示了如何处理嵌套的配置文件对象创建用户。
```python
class UserSerializer(serializers.ModelSerializer):
    profile = ProfileSerializer()

    class Meta:
        model = User
        fields = ('username', 'email', 'profile')

    def create(self, validated_data):
        profile_data = validated_data.pop('profile')
        user = User.objects.create(**validated_data)
        Profile.objects.create(user=user, **profile_data)
        return user
```

#### 为嵌套表示编写 `.update()` 方法 (Writing `.update()` methods for nested representations)
对于更新，您需要仔细考虑如何处理关系更新。例如，如果关系的数据为 `None` 或未提供，则应发生以下哪种情况？

- 在数据库中将关系设置为 `NULL`。
- 删除关联的实例。
- 忽略数据并保留这个实例。
- 抛出验证错误。

这是我们之前 `UserSerializer` 类中 `update()` 方法的示例。
```python
    def update(self, instance, validated_data):
        profile_data = validated_data.pop('profile')
        # 除非应用程序正确地强制始终设置该字段，否则就应该抛出一个需要处理的`DoesNotExist`。
        profile = instance.profile

        instance.username = validated_data.get('username', instance.username)
        instance.email = validated_data.get('email', instance.email)
        instance.save()

        profile.is_premium_member = profile_data.get(
            'is_premium_member',
            profile.is_premium_member
        )
        profile.has_support_contract = profile_data.get(
            'has_support_contract',
            profile.has_support_contract
         )
        profile.save()

        return instance
```
因为嵌套创建和更新的行为可能不明确，并且可能需要相关模型之间的复杂依赖关系，REST framework 3 要求你始终显式的编写这些方法。默认的 `ModelSerializer` `.create()` 和 `.update()` 方法不包括对可写嵌套表示的支持。

但是，可用的第三方软件包 (如 [DRF Writable Nested](http://www.django-rest-framework.org/api-guide/serializers/#drf-writable-nested)) 支持自动可写嵌套表示。

#### 处理在模型管理类中保存关联实例 (Handling saving related instances in model manager classes)
在序列化器中保存多个相关实例的另一种方法是编写处理创建正确实例的自定义模型管理器类。

例如，假设我们希望确保User实例和Profile实例始终作为一对一起创建。我们可能会编写一个类似于下面的自定义管理器类：
```python
class UserManager(models.Manager):
    ...

    def create(self, username, email, is_premium_member=False, has_support_contract=False):
        user = User(username=username, email=email)
        user.save()
        profile = Profile(
            user=user,
            is_premium_member=is_premium_member,
            has_support_contract=has_support_contract
        )
        profile.save()
        return user
```
此管理器类现在更好地封装了 user 实例和 profile 实例始终同时创建。现在可以重写我们在序列化程序类上的 `.create()` 方法以使用新的管理器方法。
```python
def create(self, validated_data):
    return User.objects.create(
        username=validated_data['username'],
        email=validated_data['email']
        is_premium_member=validated_data['profile']['is_premium_member']
        has_support_contract=validated_data['profile']['has_support_contract']
    )
```
有关此方法的更多详细信息，请参阅[模型管理器](https://docs.djangoproject.com/en/stable/topics/db/managers/)上的 Django 文档，以及[使用模型和管理器类的相关博客](https://www.dabapps.com/blog/django-models-and-encapsulation/)。

## 处理多个对象 (Dealing with multiple objects)
`Serializer` 类还可以处理序列化或反序列化对象列表。

#### 序列化多个对象 (Serializing multiple objects)
要序列化查询集或对象列表而不是单个对象实例，应在实例化序列化器时传递 `many = True` 标志。然后，您可以传递要序列化的查询集或对象列表。
```python
queryset = Book.objects.all()
serializer = BookSerializer(queryset, many=True)
serializer.data
# [
#     {'id': 0, 'title': 'The electric kool-aid acid test', 'author': 'Tom Wolfe'},
#     {'id': 1, 'title': 'If this is a man', 'author': 'Primo Levi'},
#     {'id': 2, 'title': 'The wind-up bird chronicle', 'author': 'Haruki Murakami'}
# ]
```

#### 反序列化多个对象 (Deserializing multiple objects)
反序列化多个对象的默认行为是支持多个对象创建，但不支持多个对象更新。有关如何支持或自定义这些情况的更多信息，请参阅下面的 [ListSerializer](http://www.django-rest-framework.org/api-guide/serializers/#listserializer) 文档。

## 包括额外的上下文 (Including extra context)
在某些情况下，除了要序列化的对象之外，还需要为序列化器提供额外的上下文。一种常见的情况是，如果您正在使用包含超链接关系的序列化器，这需要序列化器能够访问当前的请求以便正确生成完全限定的 URL。

您可以通过在实例化序列化器时传递 `context` 参数来提供任意的附加上下文。例如：
```python
serializer = AccountSerializer(account, context={'request': request})
serializer.data
# {'id': 6, 'owner': u'denvercoder9', 'created': datetime.datetime(2013, 2, 12, 09, 44, 56, 678870), 'details': 'http://example.com/accounts/6/details'}
```
通过访问 `self.context` 属性，可以在任何序列化器字段逻辑中使用上下文字典，例如自定义的 `.to_representation()` 方法。

***

# ModelSerializer
通常，您会希望序列化器类紧密地映射到 Django 模型定义上。

`ModelSerializer` 类提供了一个快捷方式，可以自动创建具有与模型字段对应的字段的 `Serializer` 类。

**`ModelSerializer` 类与常规 `Serializer` 类相同，不同之处在于**：

- 它将根据模型自动为您生成一组字段。
- 它将自动为序列化器生成验证器，例如 unique_together 验证器。
- 它包含默认简单实现的 `.create()` 和 `.update()` 方法。

声明 `ModelSerializer` 如下所示：
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
```
默认情况下，类上的所有模型字段都将映射到相应的序列化器字段。

模型上的任何关系如 (外键) 都将映射到 `PrimaryKeyRelatedField`。默认情况下不包括反向关系，除非在[序列化关系](http://www.django-rest-framework.org/api-guide/relations/)文档中明确包含指定。

#### 检查 `ModelSerializer` (Inspecting a `ModelSerializer`)
序列化器类生成有用的详细表示字符串，允许您全面检查其字段的状态。在使用 `ModelSerializer` 时特别有用，因为您想确定为您自动创建了哪些字段和验证器。

为此，使用 `python manage.py shell` 打开 Django shell，然后导入序列化器类，实例化它，并打印对象的表示...
```python
>>> from myapp.serializers import AccountSerializer
>>> serializer = AccountSerializer()
>>> print(repr(serializer))
AccountSerializer():
    id = IntegerField(label='ID', read_only=True)
    name = CharField(allow_blank=True, max_length=100, required=False)
    owner = PrimaryKeyRelatedField(queryset=User.objects.all())
```

## 指定要包含的字段 (Specifying which fields to include)
如果您只想在模型序列化器中使用默认字段的子集，则可以使用 `fields` 或 `exclude` 选项，就像使用 `ModelForm` 一样。强烈建议您使用 `fields` 属性显式设置应序列化的所有字段。这将使得在模型更改时不太可能导致无意中暴露数据。

举个栗子：
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
```
你还可以将 `fields` 属性设置成 `'__all__'` 来表明使用模型中的所有字段。

举个栗子：
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = '__all__'
```
你可以将 `exclude` 属性设置为从序列化器中排除的字段列表。

举个栗子：
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        exclude = ('users',)
```
在上面的例子中，如果 `Account` 模型有三个字段 `account_name`，`users` 和 `created`，那么只有 `account_name` 和 `created` 会被序列化。

在 `fields` 和 `exclude` 属性中的名称，通常会映射到模型类中的模型字段。

或者，`fields` 选项中的名称可以映射到不包含模型类中存在的参数的属性或方法。

从版本 3.3.0 开始，必须提供其中一个属性 `fields` 或 `exclude`。

## 指定嵌套序列化 (Specifying nested serialization)
默认的 `ModelSerializer` 使用主键进行关联，但您也可以使用 `depth` 选项轻松的生成嵌套关联：
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
        depth = 1
```
`depth` 选项应设置为一个整数值，该值指示在恢复为平面表示之前应该遍历的关系深度。(后半句没看懂！！！)

如果要自定义序列化的方式，则需要自己定义字段。

## 显式指定字段 (Specifying fields explicitly)
您可以向 `ModelSerializer` 添加额外字段，或通过在类上声明字段来重写默认字段，就像对 `Serializer` 类一样。
```python
class AccountSerializer(serializers.ModelSerializer):
    url = serializers.CharField(source='get_absolute_url', read_only=True)
    groups = serializers.PrimaryKeyRelatedField(many=True)

    class Meta:
        model = Account
```
额外的字段可以对应模型上任何属性或可调用的 (字段)。

## 指定只读字段 (Specifying read only fields)
您可能希望将多个字段指定为只读。您可以使用快捷的 Meta 选项 `read_only_fields`，而不是使用 `read_only=True` 属性显式的添加每个字段。

该选项应该是字段名称的列表或元组，并声明如下：
```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
        read_only_fields = ('account_name',)
```
模型中已经设置 `editable=False` 的字段和默认就被设置为只读的 `AutoField` 字段都不需要添加到 `read_only_fields` 选项中。

***

**注意**：有一种特殊情况，其中只读字段是模型级别 `unique_together` 约束的一部分。在这种情况下，序列化器类需要该字段来验证约束，但也不能由用户编辑。

处理该问题的正确方法是在序列化器上显式指定该字段，同时提供 `read_only=True` 和 `default=…` 关键字参数。

其中一个例子是与当前已认证 `User` 的只读关系，它与另一个标识符是 `unique_together`。在这种情况下，您可以声明用户字段，如下所示：
```python
user = serializers.PrimaryKeyRelatedField(read_only=True, default=serializers.CurrentUserDefault())
```
有关 [UniqueTogetherValidator](http://www.django-rest-framework.org/api-guide/validators/#uniquetogethervalidator) 和 [CurrentUserDefault](http://www.django-rest-framework.org/api-guide/validators/#currentuserdefault) 类的详细文档，请查阅[验证器文档](http://www.django-rest-framework.org/api-guide/validators/)。

***

## 附加关键字参数 (Additional keyword arguments)
还有一个快捷方式允许您使用 `extra_kwargs` 选项在字段上指定任意附加关键字参数。与  `read_only_fields` 的情况一样，这意味着您不需要在序列化器中显式声明该字段。

此选项是一个字典，将字段名称映射到关键字参数的字典。例如：
```python
class CreateUserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('email', 'username', 'password')
        extra_kwargs = {'password': {'write_only': True}}

    def create(self, validated_data):
        user = User(
            email=validated_data['email'],
            username=validated_data['username']
        )
        user.set_password(validated_data['password'])
        user.save()
        return user
```

## 关系字段 (Relational fields)
序列化模型实例时，您可以选择多种不同的方式来表示关联关系。`ModelSerializer` 的默认表示是使用相关实例的主键。

替代表示方式包括使用超链接序列化，序列化完整的嵌套表示或者使用自定义表示的序列化。

有关详细信息，请参阅[序列化器关系](http://www.django-rest-framework.org/api-guide/relations/)文档。

## 自定义字段映射 (Customizing field mappings)
ModelSerializer 类还公开了一个可以重写的 API，以便在实例化序列化器时更改如何自动确定序列化器字段。

通常，如果 `ModelSerializer` 默认情况下没有生成你需要的字段，那么您应该将它们显式地添加到类中，或者简单地使用常规的 `Serializer` 类。但是在某些情况下，您可能需要创建一个新的基类，来定义如何为任意给定模型创建序列化字段。

`.serializer_field_mapping`

Django 模型类到 REST framework 序列化器类的映射。您可以重写此映射以更改应该用于每个模型类的默认序列化器类。

`.serializer_related_field`

此属性应是序列化器字段类，默认情况下用于关联字段。

对于 `ModelSerializer` 此属性默认是 `PrimaryKeyRelatedField`。

对于 `HyperlinkedModelSerializer` 此属性默认是 `serializers.HyperlinkedRelatedField`。

`serializer_url_field`

应该用于序列化器上任何 `url` 字段的序列化器字段类。

默认是 `serializers.HyperlinkedIdentityField`

`serializer_choice_field`

应用于序列化器上任何选择字段的序列化器字段类。

默认是 `serializers.ChoiceField`

### The field_class 和 field_kwargs API
调用下面的方法来确定应该自动包含在序列化器中每个字段的类和关键字参数。这些方法都应该返回 `(field_class, field_kwargs)` 元祖。

`.build_standard_field(self, field_name, model_field)`

调用以生成映射到标准模型字段的序列化器字段。

默认实现返回基于 `serializer_field_mapping` 属性的序列化器类。

`.build_relational_field(self, field_name, relation_info)`

调用以生成映射到关系模型字段的序列化器字段。

默认实现返回基于 `serializer_relational_field` 属性的序列化器类。

`relation_info` 参数是一个命名元祖，包含 `model_field`，`related_model`，`to_many` 和 `has_through_model` 属性。

`.build_nested_field(self, field_name, relation_info, nested_depth)`

当设置了 `depth` 选项时，调用以生成映射到关系模型字段的序列化器字段。

默认实现基于 `ModelSerializer` 或 `HyperlinkedModelSerializer` 动态创建嵌套的序列化器类。

`nested_depth` 的值是 `depth` 选项的值减 1。

`relation_info` 参数是一个命名元祖，包含 `model_field`，`related_model`，`to_many` 和 `has_through_model` 属性。

`.build_property_field(self, field_name, model_class)`

调用以生成映射到模型类中的属性或零参数方法的序列化器字段。

默认实现返回 `ReadOnlyField` 类。

`.build_url_field(self, field_name, model_class)`

调用为序列化器自己的 `url` 字段生成序列化器字段。默认实现返回 `HyperlinkedIdentityField` 类。

`.build_unknown_field(self, field_name, model_class)`

当字段名称未映射到任何模型字段或模型属性时调用。默认实现会引发错误，但子类可以自定义此行为。

***

# HyperlinkedModelSerializer
`HyperlinkedModelSerializer` 类类似于 `ModelSerializer` 类，不同之处在于它使用超链接来表示关联关系而不是主键。

默认情况下，序列化器将包含 `url` 字段而不是主键字段。

url 字段将使用 `HyperlinkedIdentityField` 序列化器字段表示，并且模型上的任何关系将使用 `HyperlinkedRelatedField` 序列化器字段表示。

您可以通过将主键添加到 `fields` 选项显式包含主键，例如：
```python
class AccountSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Account
        fields = ('url', 'id', 'account_name', 'users', 'created')
```

## 绝对和相对 URL (Absolute and relative URLs)
当实例化 `HyperlinkedModelSerializer` 时，必须在序列化器上下文中包含当前 `request`，例如：
```python
serializer = AccountSerializer(queryset, context={'request': request})
```
这样做将确保超链接可以包含适当的主机名，以便生成完全限定的 URL，例如：
```python
http://api.example.com/accounts/1/
```
而不是相对 URL，例如：
```python
/accounts/1/
```
如果您确实想使用相对 URL，应在序列化器上下文中显式传递 `{'request'：None}`。

## 如何确定超链接视图 (How hyperlinked views are determined)
需要有一种方法来确定应该将哪些视图用于超链接到模型实例。

默认情况下，超链接预期对应于匹配样式 `{model_name}-detail` 的视图名，并通过 `pk` 关键字参数查找实例。

你可以使用在 `extra_kwargs` 设置中的 `view_name` 和 `lookup_field` 选项中的一个或两个来重写 URL 字段视图名称和查询字段。如下所示：
```python
class AccountSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Account
        fields = ('account_url', 'account_name', 'users', 'created')
        extra_kwargs = {
            'url': {'view_name': 'accounts', 'lookup_field': 'account_name'},
            'users': {'lookup_field': 'username'}
        }
```
或者，您可以显式设置序列化器上的字段。例如：
```python
class AccountSerializer(serializers.HyperlinkedModelSerializer):
    url = serializers.HyperlinkedIdentityField(
        view_name='accounts',
        lookup_field='slug'
    )
    users = serializers.HyperlinkedRelatedField(
        view_name='user-detail',
        lookup_field='username',
        many=True,
        read_only=True
    )

    class Meta:
        model = Account
        fields = ('url', 'account_name', 'users', 'created')
```

***

**提示**：正确匹配超链接表示和你的URL配置有时可能会有点困难。打印 `HyperlinkedModelSerializer` 实例的 `repr` 是特别有用的方式来检查关联关系预映射到哪些视图名称和查询字段。

***

## 更改 URL 字段名称 (Changing the URL field name)
URL 字段的名称默认为 'url'。你可以通过使用 `URL_FIELD_NAME` 设置 (在 settings 文件) 进行全局覆盖。

***

# ListSerializer
`ListSerializer` 类提供同时序列化和验证多个对象的行为。您通常不需要直接使用 `ListSerializer`，而应该在实例化序列化器时简单地传递 `many=True`。

当序列化器被实例化并且 `many = True` 被传递时，`ListSerializer` 实例将被创建。然后，序列化器类将成为 `ListSerializer` 的子类。

以下参数也可以传递给 `ListSerializer` 字段或者被传递 `many=True` 参数的序列化器。

`allow_empty`

默认情况下为 `True`，但如果要禁止空列表作为有效输入，则可以设置为 `False`。

### 自定义 `ListSerializer` 行为 (Customizing `ListSerializer` behavior)
当您可能想要自定义 `ListSerializer` 行为时，有一些用例。例如：

- 你想要提供列表的特定验证，例如检查一个元素是否与列表中的另外一个元素冲突。
- 你想要自定义多个对象的创建或更新行为。

对于这些情况，您可以通过使用序列化器 `Meta` 类中的 `list_serializer_class` 选项来修改传递 `many=True` 时使用的类。

例如：
```python
class CustomListSerializer(serializers.ListSerializer):
    ...

class CustomSerializer(serializers.Serializer):
    ...
    class Meta:
        list_serializer_class = CustomListSerializer
```

#### 自定义多重创建 (Customizing multiple create)
多个对象的创建默认实现是简单地为列表中的每个项目调用 `.create()` 。如果想要自定义该行为，那么您需要自定义当被传递 `many=True` 参数时使用的 `ListSerializer` 类中的 `.create()` 方法。

例如：
```python
class BookListSerializer(serializers.ListSerializer):
    def create(self, validated_data):
        books = [Book(**item) for item in validated_data]
        return Book.objects.bulk_create(books)

class BookSerializer(serializers.Serializer):
    ...
    class Meta:
        list_serializer_class = BookListSerializer
```

#### 自定义多个更新 (Customizing multiple update)
默认情况下，`ListSerializer` 类不支持多个更新。这是因为插入和删除预期的行为不明确。

为了支持多个更新，您需要明确地进行这样的操作。编写多个更新代码时，请务必记住以下几点：

- 如何确定为数据列表中的每个 item 更新哪个实例？
- 如何处理插入？它们是无效的，还是创建新对象？
- 如何处理删除？它们是暗示删除对象还是删除关系？它们应该被忽略，还是无效？
- 如何处理排序？改变两个项目的位置是否意味着任何状态改变或被忽略？

你需要向实例序列化器显式添加 `id` 字段。默认隐式生成的 `id` 字段被标记为 `read_only`。这会导致它在更新时被删除。一旦你显式地声明它，它将在列表序列化器的 `update` 方法中可用。

以下是您可以选择实施多个更新的示例：
```python
class BookListSerializer(serializers.ListSerializer):
    def update(self, instance, validated_data):
        # id->instance 和 id->data item 的映射。
        book_mapping = {book.id: book for book in instance}
        data_mapping = {item['id']: item for item in validated_data}

        # 执行创建和更新。
        ret = []
        for book_id, data in data_mapping.items():
            book = book_mapping.get(book_id, None)
            if book is None:
                ret.append(self.child.create(data))
            else:
                ret.append(self.child.update(book, data))

        # 执行删除
        for book_id, book in book_mapping.items():
            if book_id not in data_mapping:
                book.delete()

        return ret

class BookSerializer(serializers.Serializer):
    # 我们需要使用主键识别列表中的元素，
    # 所以在这里使用一个可写字段，而不是默认的只读字段。
    id = serializers.IntegerField()
    ...

    class Meta:
        list_serializer_class = BookListSerializer
```
第三方包可能包括在 3.1 版本中，它为多个更新操作提供一些自动支持，类似于 REST framework 2 中存在的 `allow_add_remove` 行为。

#### 自定义 ListSerializer 初始化 (Customizing ListSerializer initialization)
当带有 `many=True` 的序列化器被实例化时，我们需要确定哪些参数和关键字参数应该被传递给子类 `Serializer` 和父类 `ListSerializer` 的 `.__init__()` 方法。

默认实现是将所有参数传递给两个类，除了 `validators` 和任何自定义关键字参数，这两个参数都假定是用于子类序列化器的。

偶尔，您可能需要显式的指定当被传递 `many=True` 参数时，子类和父类应该如何实例化。您可以使用 `many_init` 类方法执行此操作。
```python
    @classmethod
    def many_init(cls, *args, **kwargs):
        # Instantiate the child serializer.
        kwargs['child'] = cls()
        # Instantiate the parent list serializer.
        return CustomListSerializer(*args, **kwargs)
```

***

# BaseSerializer
`BaseSerialAlgisher` 类，可以用来方便地支持可选的序列化和反序列化样式。

该类实现与 `Serializer` 类相同的基本 API：

- `.data` —— 返回传出的基元表示。
- `.is_valid()` —— 反序列化并验证传入的数据。
- `.validated_data` —— 返回经过验证的传入数据。
- `.errors` —— 返回在验证期间的任何错误。
- `.save()` —— 将验证的数据保存到对象实例中。

可以重写四种方法，这取决于你希望序列化器类支持的功能：

- `.to_representation()` —— 重写它以支持序列化，用于读取操作。
- `.to_internal_value()` —— 重写它以支持反序列化，以用于写入操作。
- `.create()` 和 `.update()` —— 重写其中一个或两个以支持保存实例。

因为此类提供了与 `Serializer` 类相同的接口，所以您可以像使用常规的 `Serializer` 或 `ModelSerializer` 一样使用现有的通用的基于类的视图。

在执行此操作时您将注意到的唯一区别是 `BaseSerializer` 类不会在可浏览的 API 中生成 HTML 表单。这是因为它们返回的数据不包括允许每个字段被渲染成合适的 HTML 输入的所有字段信息。

##### 只读 `BaseSerializer` 类(Read-only `BaseSerializer` classes)
使用 `BaseSerializer` 类来实现只读序列化器，我们只需要重写 `.to_representation()` 方法。让我们看一个使用简单 Django 模型的示例：
```python
class HighScore(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    player_name = models.CharField(max_length=10)
    score = models.IntegerField()
```
创建只读序列化器将 `HighScore` 实例转换为原始数据类型很简单。
```python
class HighScoreSerializer(serializers.BaseSerializer):
    def to_representation(self, obj):
        return {
            'score': obj.score,
            'player_name': obj.player_name
        }
```
我们现在可以使用此类来序列化单个 `HighScore` 实例：
```python
@api_view(['GET'])
def high_score(request, pk):
    instance = HighScore.objects.get(pk=pk)
    serializer = HighScoreSerializer(instance)
    return Response(serializer.data)
```
或者使用它来序列化多个实例：
```python
@api_view(['GET'])
def all_high_scores(request):
    queryset = HighScore.objects.order_by('-score')
    serializer = HighScoreSerializer(queryset, many=True)
    return Response(serializer.data)
```

##### 读写 `BaseSerializer` 类 (Read-write `BaseSerializer` classes)
要创建读写序列化器，我们首先需要实现 `.to_internal_value()` 方法。此方法返回将用于构造对象实例的验证值，如果提供的数据格式不正确，则可能引发 `serializers.ValidationError`。

一旦实现 `.to_internal_value()`，基本验证 API 将在序列化器中可用，并且您将能够使用 `.is_valid()`，`.validated_data` 和 `.errors`。

如果还想支持 `.save()`，您还需要实现 `.create()` 和 `.update()` 方法中的一个或两个。

这是我们之前的 `HighScoreSerializer` 的完整示例，它已经更新以支持读写操作。
```python
class HighScoreSerializer(serializers.BaseSerializer):
    def to_internal_value(self, data):
        score = data.get('score')
        player_name = data.get('player_name')

        # 执行数据验证。
        if not score:
            raise serializers.ValidationError({
                'score': 'This field is required.'
            })
        if not player_name:
            raise serializers.ValidationError({
                'player_name': 'This field is required.'
            })
        if len(player_name) > 10:
            raise serializers.ValidationError({
                'player_name': 'May not be more than 10 characters.'
            })

        # 返回验证值。这将作为可用的 `.validated_data` 属性。
        return {
            'score': int(score),
            'player_name': player_name
        }

    def to_representation(self, obj):
        return {
            'score': obj.score,
            'player_name': obj.player_name
        }

    def create(self, validated_data):
        return HighScore.objects.create(**validated_data)
```

#### 创建新的基类 (Creating new base classes)
如果您想要实现新的通用序列化器类来处理特定的序列化样式，或者集成其他存储后端，那么 `BaseSerializer` 类也很有用。

以下类是可以处理将任意对象强制转换为基本表示的通用序列化器的示例。
```python
class ObjectSerializer(serializers.BaseSerializer):
    """
    任意复杂对象强制转换为原始表示的只读序列化器。
    """
    def to_representation(self, obj):
        for attribute_name in dir(obj):
            attribute = getattr(obj, attribute_name)
            if attribute_name('_'):
                # 忽略私有属性。
                pass
            elif hasattr(attribute, '__call__'):
                # 忽略方法和其他 callables。
                pass
            elif isinstance(attribute, (str, int, bool, float, type(None))):
                # 原始类型可以通过未修改的方式传递。
                output[attribute_name] = attribute
            elif isinstance(attribute, list):
                # 递归处理列表中的项。
                output[attribute_name] = [
                    self.to_representation(item) for item in attribute
                ]
            elif isinstance(attribute, dict):
                # 递归处理字典中的项。
                output[attribute_name] = {
                    str(key): self.to_representation(value)
                    for key, value in attribute.items()
                }
            else:
                # 将其他内容强制到其字符串表示形式。
                output[attribute_name] = str(attribute)
```

***

# 高级序列化器用法 (Advanced serializer usage)
## 重写序列化和反序列化行为 (Overriding serialization and deserialization behavior)
如果需要更改序列化器类的序列化或反序列化行为，可以通过重写 `.to_representation()` 或 `.to_internal_value()` 方法来实现。

可能有用的一些原因包括...

- 为新的序列化器基类添加新行为。
- 对现有类稍微修改行为。
- 提高频繁访问返回大量数据的 API 端点的序列化性能。

这些方法的明显特征如下：

`.to_representation(self, obj)`

获取需要序列化的对象实例，并返回原始表示。通常，这意味着返回内置 Python 数据类型的结构。可以处理的确切类型取决于为 API 配置的渲染器类。

可以重写以修改表示样式。例如：
```python
def to_representation(self, instance):
    """Convert `username` to lowercase."""
    ret = super().to_representation(instance)
    ret['username'] = ret['username'].lower()
    return ret
```

`.to_internal_value(self, data)`

将未经验证的传入数据作为输入，并返回可用的已验证数据作为 `serializer.validated_data` 。如果在序列化器类上调用 `.save()`，则返回值也将传递给 `.create()` 或 `.update()` 方法。

如果任何验证失败，则该方法应引发 `serializers.ValidationError(errors)`。`errors` 参数应该是将字段名称 (或 `settings.NON_FIELD_ERRORS_KEY` ) 映射到错误消息列表的字典。如果您不需要改变反序列化行为，而是想提供对象级验证，则建议改为重写 `.validate()` 方法。

传递给此方法的 `data` 参数通常是 `request.data` 的值，因此它提供的数据类型将取决于为 API 配置的解析器类。

## 序列化器继承 (Serializer Inheritance)
与 Django 表单类似，您可以通过继承来扩展和重用序列化器。这允许您在父类上声明一组通用的字段或方法，然后可以在多个序列化器中使用它们。例如，
```python
class MyBaseSerializer(Serializer):
    my_field = serializers.CharField()

    def validate_my_field(self):
        ...

class MySerializer(MyBaseSerializer):
    ...
```
与 Django 的 `Model` 和 `ModelForm` 类一样，序列化器内部 `Meta` 类不会隐式地继承它父类内部 `Meta` 类。如果您想要从父类继承 `Meta` 类，则必须明确地这样做。例如：
```python
class AccountSerializer(MyBaseSerializer):
    class Meta(MyBaseSerializer.Meta):
        model = Account
```
通常我们建议不要在内部 Meta 类上使用继承，而是显式声明所有选项。

此外，以下注意事项适用于序列化程序继承：

- 正常的 Python 名称解析规则适用。如果您有多个声明 `Meta` 内部类的基类，则只使用第一个类。这意味着子类的 `Meta` (如果存在)，否则是第一个父类的 `Meta`。
- 通过在子类上将名称设置为 `None`，声明性地删除继承自父类的 `Field` 是可能的。

```python
class MyBaseSerializer(ModelSerializer):
    my_field = serializers.CharField()

class MySerializer(MyBaseSerializer):
    my_field = None
```

但是，您只能使用这种技术选择从父类声明性定义的字段；它不会阻止 `ModelSerializer` 生成默认字段。要从默认字段中选择，请参阅[指定要包括的字段](http://www.django-rest-framework.org/api-guide/serializers/#specifying-which-fields-to-include)。

## 动态修改字段 (Dynamically modifying fields)
一旦序列化器已初始化，可以使用 `.field` 属性访问序列化器上设置的字段字典。访问和修改此属性允许您动态修改序列化器。

直接修改 `fields` 参数允许您做一些有趣的事情，例如在运行时更改序列化器字段上的参数，而不是在声明序列化器时。

### 举个栗子
例如，如果您希望能够设置序列化器在初始化时应使用哪些字段，您可以创建一个像这样的序列化器类：
```python
class DynamicFieldsModelSerializer(serializers.ModelSerializer):
    """
    ModelSerializer 获取额外的 `fields` 参数来控制哪些字段应该被显示。
    """

    def __init__(self, *args, **kwargs):
        # 不要将 'fields' 参数传递给超类
        fields = kwargs.pop('fields', None)

        # 通常实例化超类
        super(DynamicFieldsModelSerializer, self).__init__(*args, **kwargs)

        if fields is not None:
            # 删除`fields`参数中未指定的任何字段。
            allowed = set(fields)
            existing = set(self.fields)
            for field_name in existing - allowed:
                self.fields.pop(field_name)
```
这将允许您执行以下操作：
```python
>>> class UserSerializer(DynamicFieldsModelSerializer):
>>>     class Meta:
>>>         model = User
>>>         fields = ('id', 'username', 'email')
>>>
>>> print UserSerializer(user)
{'id': 2, 'username': 'jonwatts', 'email': 'jon@example.com'}
>>>
>>> print UserSerializer(user, fields=('id', 'email'))
{'id': 2, 'email': 'jon@example.com'}
```

## 自定义默认字段 (Customizing the default fields)
REST framework 2 提供了一个 API，允许开发人员重写如何自动生成默认字段集的 `ModelSerializer` 类。

该 API 包括 `.get_field()`，`. get_pk_field()` 和其他方法。

因为版本 3.0 序列化器已经从根本上重新设计了，这个 API 不再存在。您仍然可以修改已创建的字段，但您需要参考源代码，并且注意，如果所做的更改是针对 API 的私有位，那么它们可能会发生更改。

***

# 第三方包 (Third party packages)
以下是可用的第三方包。

## Django REST marshmallow
[django-rest-marshmallow](https://marshmallow-code.github.io/django-rest-marshmallow/) 包使用 python [marshmallow](https://marshmallow.readthedocs.io/en/latest/) 库为序列化器提供了另一种实现方式。它公开了与 REST framework 序列化器相同的 API，并且可以在某些用例中用作替代品。

## Serpy
[serpy](https://github.com/clarkduvall/serpy) 包是为速度而构建的序列化器的另一种实现。Serpy 将复杂数据类型序列化为简单的原生类型。原生类型可以轻松转换为 JSON 或任何其他所需格式。

## MongoengineModelSerializer
[django-rest-framework-mongoengine](https://github.com/umutbozkurt/django-rest-framework-mongoengine) 包提供了一个支持 MongoDB 作为 Django REST framework 的存储层的 `MongoEngineModelSerializer` 序列化器类。

## GeoFeatureModelSerializer
[django-rest-framework-gis](https://github.com/djangonauts/django-rest-framework-gis) 包提供了一个支持 GeoJSON 进行读写操作的 `GeoFeatureModelSerializer` 序列化器类。

## HStoreSerializer
[django-rest-framework-hstore](https://github.com/djangonauts/django-rest-framework-hstore) 包提供了一个 `HStoreSerializer` 来支持 [django-hstore](https://github.com/djangonauts/django-hstore) `DictionaryField` 模型字段及其 `schema-mode` 特点。

## Dynamic REST
[dynamic-rest](https://github.com/AltSchool/dynamic-rest) 包扩展了 ModelSerializer 和 ModelViewSet 接口，添加了 API 查询参数，用于过滤，排序和包含/排除序列化器定义的所有字段和关系。

## Dynamic Fields Mixin
drf-dynamic-fields 包提供了一个 mixin 来动态地限制每个序列化器的字段到 URL 参数指定的子集。

## DRF FlexFields
drf-flex-fields 包扩展了 ModelSerializer 和 ModelViewSet，以提供常用的功能，用于动态设置字段并将原始字段扩展到嵌套模型，它们都来自 URL 参数和序列化器类定义。

## Serializer Extensions
django-rest-framework-serializer-extensions 包提供了一系列工具来干扰序列化器，通过允许在每个视图/请求的基础上定义的字段。字段可以列入白名单，列入黑名单，并且可以选择的扩展子序列化器。

## HTML JSON Forms
[html-json-forms](https://github.com/wq/html-json-forms) 包提供了一个算法和序列化器，用于按照 (非活动) [HTML JSON 表单规范](https://www.w3.org/TR/html-json-forms/)处理 `<form>` 提交。序列化器有助于在 HTML 中处理任意嵌套的 JSON 结构。例如，`<input name="items[0][id]" value="5">` 将被解释为 `{"items": [{"id": "5"}]}`。

## DRF-Base64
[DRF-Base64](https://bitbucket.org/levit_scs/drf_base64) 提供了一组字段和模型序列化器，用于处理 base64 编码文件的上传。

## QueryFields
[djangorestframework-queryfields](https://djangorestframework-queryfields.readthedocs.io/) 允许 API 客户端通过包含/排除查询参数指定将在响应中发送的字段。

## DRF Writable Nested
[drf-writable-nested](https://github.com/beda-software/drf-writable-nested) 包提供可写的嵌套模型序列化器，允许使用嵌套的相关数据创建/更新模型。
