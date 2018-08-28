# 序列化器关系 (Serializer relations)
糟糕的程序员担心代码。好的程序员担心数据结构和它们的关系。—— [Linus Torvalds](https://lwn.net/Articles/193245/)

关系字段用于表示模型关系。它们可以应用于 `ForeignKey`，`ManyToManyField` 和 `OneToOneField` 关系，以及反向关系和自定义关系 (例如：`GenericForeignKey`)。

***

**注意**：关系字段在 `relations.py` 中声明，但按照惯例，你应该从 `serializers` 模块导入它们，使用 `from rest_framework import serializers` 并像 `serializers.<FieldName>` 这样引用字段。

***

#### 检查关系 (Inspecting relationships)
当使用 `ModelSerializer` 类时，将为您自动生成序列化器字段和关系。检查这些自动生成的字段可以作为确定如何定制关系样式的有用工具。

为此，使用 `python manage.py shell` 打开 Django shell，然后导入序列化器类，实例化它并打印对象表示...
```python
>>> from myapp.serializers import AccountSerializer
>>> serializer = AccountSerializer()
>>> print repr(serializer)  # Or `print(repr(serializer))` in Python 3.x.
AccountSerializer():
    id = IntegerField(label='ID', read_only=True)
    name = CharField(allow_blank=True, max_length=100, required=False)
    owner = PrimaryKeyRelatedField(queryset=User.objects.all())
```

# API 参考 (API Reference)
为了解释各种类型的关系字段，我们将为我们的示例使用一些简单的模型。我们的模型将用于音乐专辑，以及每个专辑中列出的歌曲。
```python
class Album(models.Model):
    album_name = models.CharField(max_length=100)
    artist = models.CharField(max_length=100)

class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    order = models.IntegerField()
    title = models.CharField(max_length=100)
    duration = models.IntegerField()

    class Meta:
        unique_together = ('album', 'order')
        ordering = ['order']

    def __unicode__(self):
        return '%d: %s' % (self.order, self.title)
```

## StringRelatedField
`StringRelatedField` 可用它的 `__unicode__` 方法来表示关系的目标。

例如，以下序列化器。
```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.StringRelatedField(many=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```
将序列化为以下表示。
```python
{
    'album_name': 'Things We Lost In The Fire',
    'artist': 'Low',
    'tracks': [
        '1: Sunflower',
        '2: Whitetail',
        '3: Dinosaur Act',
        ...
    ]
}
```
该字段是只读的。

**参数**：

- `many` - 如果是一对多的关系，就将此参数设置为 `True`。

## PrimaryKeyRelatedField
`PrimaryKeyRelatedField` 可用于使用其主键表示关系的目标。

例如，以下序列化器：
```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.PrimaryKeyRelatedField(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```
将序列化为这样的表示：
```python
{
    'album_name': 'Undun',
    'artist': 'The Roots',
    'tracks': [
        89,
        90,
        91,
        ...
    ]
}
```
默认情况下，此字段是读写的，但您可以使用 `read_only` 标志更改此行为。

**参数**：

- `queryset` - 验证字段输入时用于模型实例查询的查询集。关系必须显式地设置查询集，或设置 `read_only=True`。
- `many` - 如果应用于一对多关系，则应将此参数设置为 `True`。
- `allow_null` - 如果设置为 `True`，那么该字段将接受 `None` 值或可为空的关系的空字符串。默认为 `False`。
- `pk_field` - 设置字段来控制主键值的序列化/反序列化。例如， `pk_field=UUIDField(format='hex')` 会将 UUID 主键序列化为其紧凑的十六进制表示形式。

## HyperlinkedRelatedField
`HyperlinkedRelatedField` 可用于使用超链接表示关系的目标。

例如，以下序列化器：
```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.HyperlinkedRelatedField(
        many=True,
        read_only=True,
        view_name='track-detail'
    )

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```
将序列化为这样的表示：
```python
{
    'album_name': 'Graceland',
    'artist': 'Paul Simon',
    'tracks': [
        'http://www.example.com/api/tracks/45/',
        'http://www.example.com/api/tracks/46/',
        'http://www.example.com/api/tracks/47/',
        ...
    ]
}
```
默认情况下，此字段是读写的，但您可以使用 `read_only` 标志更改此行为。

***
**注意**：该字段设计用于映射到接受单个 URL 关键字参数的 URL 的对象，使用 `lookup_field` 和 `lookup_url_kwarg` 参数设置。

这适用于包含单个主键或 slug 参数作为 URL 一部分的 URL。

如果您需要更复杂的超链接表示，则需要自定义该字段，如下面的[自定义超链接字段](http://www.django-rest-framework.org/api-guide/relations/#custom-hyperlinked-fields)部分中所描述的。
***

**参数**：

- `view_name` - 用作关系目标的视图名称。如果你使用的是[标准路由器类](http://www.django-rest-framework.org/api-guide/routers#defaultrouter)，则这将是格式为 `<modelname>-detail` 的字符串。**必填**。
- `queryset` - 验证字段输入时用于模型实例查询的查询集。关系必须显式地设置查询集，或设置 `read_only=True`。
- `many` - 如果应用于一对多关系，则应将此参数设置为 `True`。
- `allow_null` - 如果设置为 `True`，那么该字段将接受 `None` 值或可为空的关系的空字符串。默认为 `False`。
- `lookup_field` - 用于查找的目标字段。对应于引用视图上的 URL 关键字参数。默认是 `'pk'`。
- `lookup_url_kwarg` - 与查找字段对应的 URL conf 中定义的关键字参数的名称。默认使用与 `lookup_field` 相同的值。
- `format` - 如果使用格式后缀，则超链接字段将使用与目标相同的格式后缀，除非使用 `format` 参数重写。

## SlugRelatedField
`SlugRelatedField` 可用于使用目标上的字段来表示关系的目标。

例如，以下序列化器：
```python
class AlbumSerializer(serializers.ModelSerializer):
    tracks = serializers.SlugRelatedField(
        many=True,
        read_only=True,
        slug_field='title'
     )

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```
将序列化为这样的表示：
```python
{
    'album_name': 'Dear John',
    'artist': 'Loney Dear',
    'tracks': [
        'Airport Surroundings',
        'Everything Turns to You',
        'I Was Only Going Out',
        ...
    ]
}
```
默认情况下，此字段是读写的，但您可以使用 `read_only` 标志更改此行为。

当使用 `SlugRelatedField` 作为读写字段时，通常需要确保 slug 字段与 `unique=True` 的模型字段相对应。

**参数**：

- `slug_field` - 目标上应该用来表示它的字段。这应该是唯一标识任何给定实例的字段。例如，`username`。必填。
- `queryset` - 验证字段输入时用于模型实例查询的查询集。关系必须显式地设置查询集，或设置 `read_only=True`。
- `many` - 如果应用于一对多关系，则应将此参数设置为 `True`。
- `allow_null` - 如果设置为 `True`，那么该字段将接受 `None` 值或可为空的关系的空字符串。默认为 `False`。

## HyperlinkedIdentityField
此字段可以作为标识关系应用，例如 HyperlinkedModelSerializer 上的 `'url'` 字段。它也可以用于对象的属性。例如，以下序列化器：
```python
class AlbumSerializer(serializers.HyperlinkedModelSerializer):
    track_listing = serializers.HyperlinkedIdentityField(view_name='track-list')

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'track_listing')
```
将序列化为这样的表示：
```python
{
    'album_name': 'The Eraser',
    'artist': 'Thom Yorke',
    'track_listing': 'http://www.example.com/api/track_list/12/',
}
```
该字段始终为只读。

**参数**：

- `view_name` - 用作关系目标的视图名称。如果你使用的是[标准路由器类](http://www.django-rest-framework.org/api-guide/routers#defaultrouter)，则这将是格式为 `<modelname>-detail` 的字符串。**必填**。
- `lookup_field` - 用于查找的目标字段。对应于引用视图上的 URL 关键字参数。默认是 `'pk'`。
- `lookup_url_kwarg` - 与查找字段对应的 URL conf 中定义的关键字参数的名称。默认使用与 `lookup_field` 相同的值。
- `format` - 如果使用格式后缀，则超链接字段将使用与目标相同的格式后缀，除非使用 `format` 参数重写。

***

# 嵌套关系 (Nested relationships)
可以使用序列化器作为字段来表示嵌套关系。

如果该字段用于表示一对多关系，则应将 `many = True` 标志添加到序列化器字段。

## 举个栗子
例如，以下序列化器：
```python
class TrackSerializer(serializers.ModelSerializer):
    class Meta:
        model = Track
        fields = ('order', 'title', 'duration')

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackSerializer(many=True, read_only=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```
将序列化为嵌套表示，如下所示：
```python
>>> album = Album.objects.create(album_name="The Grey Album", artist='Danger Mouse')
>>> Track.objects.create(album=album, order=1, title='Public Service Announcement', duration=245)
<Track: Track object>
>>> Track.objects.create(album=album, order=2, title='What More Can I Say', duration=264)
<Track: Track object>
>>> Track.objects.create(album=album, order=3, title='Encore', duration=159)
<Track: Track object>
>>> serializer = AlbumSerializer(instance=album)
>>> serializer.data
{
    'album_name': 'The Grey Album',
    'artist': 'Danger Mouse',
    'tracks': [
        {'order': 1, 'title': 'Public Service Announcement', 'duration': 245},
        {'order': 2, 'title': 'What More Can I Say', 'duration': 264},
        {'order': 3, 'title': 'Encore', 'duration': 159},
        ...
    ],
}
```

## 可写的嵌套序列化器 (Writable nested serializers)
默认情况下，嵌套序列化器是只读的。如果要支持对嵌套序列化器字段的写操作，则需要创建 `create()` 和/或 `update()` 方法，以便显式指定应如何保存子关系。
```python
class TrackSerializer(serializers.ModelSerializer):
    class Meta:
        model = Track
        fields = ('order', 'title', 'duration')

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackSerializer(many=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')

    def create(self, validated_data):
        tracks_data = validated_data.pop('tracks')
        album = Album.objects.create(**validated_data)
        for track_data in tracks_data:
            Track.objects.create(album=album, **track_data)
        return album

>>> data = {
    'album_name': 'The Grey Album',
    'artist': 'Danger Mouse',
    'tracks': [
        {'order': 1, 'title': 'Public Service Announcement', 'duration': 245},
        {'order': 2, 'title': 'What More Can I Say', 'duration': 264},
        {'order': 3, 'title': 'Encore', 'duration': 159},
    ],
}
>>> serializer = AlbumSerializer(data=data)
>>> serializer.is_valid()
True
>>> serializer.save()
<Album: Album object>
```

***

# 自定义关系字段 (Custom relational fields)
在极少数情况下，现有的关系样式都不适合您需要的表示，您可以实现一个完全自定义的关系字段，该字段准确地描述应该如何从模型实例生成输出表示。

要实现自定义关系字段，您应该重写 `RelatedField`，并实现 `.to_representation(self，value)` 方法。此方法将字段的目标作为 `value` 参数，并返回应用于序列化目标的表示。`value` 参数通常是模型实例。

如果想要实现读写关系字段，还必须实现 `.to_internal_value(self, data)` 方法。

要提供基于 `context` 的动态查询集，还可以重写 `.get_queryset(self)`，而不是在类上或初始化该字段时指定 `.queryset`。

## 举个栗子
例如，我们可以定义一个关系字段，使用它的顺序，标题和持续时间将音轨序列化为自定义字符串表示。
```python
import time

class TrackListingField(serializers.RelatedField):
    def to_representation(self, value):
        duration = time.strftime('%M:%S', time.gmtime(value.duration))
        return 'Track %d: %s (%s)' % (value.order, value.name, duration)

class AlbumSerializer(serializers.ModelSerializer):
    tracks = TrackListingField(many=True)

    class Meta:
        model = Album
        fields = ('album_name', 'artist', 'tracks')
```
然后，此自定义字段将序列化为以下表示形式。
```python
{
    'album_name': 'Sometimes I Wish We Were an Eagle',
    'artist': 'Bill Callahan',
    'tracks': [
        'Track 1: Jim Cain (04:39)',
        'Track 2: Eid Ma Clack Shaw (04:19)',
        'Track 3: The Wind and the Dove (04:34)',
        ...
    ]
}
```

***

# 自定义超链接字段 (Custom hyperlinked fields)
在某些情况下，您可能需要自定义超链接字段的行为，以表示需要多个查询字段的 URL。

您可以通过重写 `HyperlinkedRelatedField` 来实现。有两个方法重写：

**get_url(self, obj, view_name, request, format)**

`get_url` 方法用于将对象实例映射到其 URL 表示。

如果 `view_name` 和 `lookup_field` 属性未配置为正确匹配 URL conf，可能会引发 `NoReverseMatch`。

**get_object(self, queryset, view_name, view_args, view_kwargs)**

如果您想支持可写的超链接字段，那么您还需要重写 `get_object`，以便将传入的 URL 映射回它们表示的对象。对于只读的超链接字段，无需重写此方法。

此方法的返回值应该是与匹配的 URL conf 参数对应的对象。

可能会引发 `ObjectDoesNotExist` 异常。

## 举个栗子
假设我们有一个 customer 对象的 URL，它接受两个关键字参数，如下所示：
```python
/api/<organization_slug>/customers/<customer_pk>/
```
这不能用仅接受单个查找字段的默认实现来表示。

在这种情况下，我们需要重写 `HyperlinkedRelatedField` 以获得我们想要的行为：
```python
from rest_framework import serializers
from rest_framework.reverse import reverse

class CustomerHyperlink(serializers.HyperlinkedRelatedField):
    # 我们将它们定义为类属性，因此我们不需要将它们作为参数传递。
    view_name = 'customer-detail'
    queryset = Customer.objects.all()

    def get_url(self, obj, view_name, request, format):
        url_kwargs = {
            'organization_slug': obj.organization.slug,
            'customer_pk': obj.pk
        }
        return reverse(view_name, kwargs=url_kwargs, request=request, format=format)

    def get_object(self, view_name, view_args, view_kwargs):
        lookup_kwargs = {
           'organization__slug': view_kwargs['organization_slug'],
           'pk': view_kwargs['customer_pk']
        }
        return self.get_queryset().get(**lookup_kwargs)
```
请注意，如果您想将此样式与通用视图一起使用，那么您还需要在视图上重写 `.get_object` 以获得正确的查找行为。

通常，我们建议在可能的情况下使用 API ​​表示的平面样式，但适度使用嵌套的 URL 样式也是合理的。

***

# 进一步说明 (Further notes)
## `queryset` 参数 (The `queryset` argument)
`queryset` 参数只社用于可写关系字段，在这种情况下，它用于执行从原始用户输入映射到模型实例的模型实例查找。

在 2.x 版本中，如果正在使用 `ModelSerializer` 类，则序列化器有时会自动确定 `queryset` 参数。

此行为现在替换为始终为可写关系字段使用显式 `queryset` 参数。

这样做可以减少 `ModelSerializer` 提供的隐藏 “魔术” 数量，使字段的行为更加清晰，并确保在使用 `ModelSerializer` 快捷方式或使用完全显式的 `Serializer` 类之间转换是微不足道的。

## 自定义 HTML 显式 (Customizing the HTML display)
模型的内置 `__str__` 方法将用于生成用于填充 `choices` 属性的对象的字符串表示。这些 choices 用于填充在可浏览的 API 中选择的 HTML 输入。

要为此类输入提供自定义表示，请重写 `RelatedField` 子类的 `display_value()`。此方法将接收一个模型对象，并应返回一个适合表示它的字符串。例如：
```python
class TrackPrimaryKeyRelatedField(serializers.PrimaryKeyRelatedField):
    def display_value(self, instance):
        return 'Track: %s' % (instance.title)
```

## Select field cutoffs
当在可浏览的 API 中渲染时，关系字段将默认只显示最多 1000 个可选择的项。如果存在更多项，则会显示 "More than 1000 items…" 的禁用选项。

此行为旨在防止模板由于显示大量关系而无法在可接受的时间段中渲染。

有两个关键字参数可用于控制此行为：

- `html_cutoff` - 如果设置，这将是 HTML 选择下拉菜单中显示的选项的最大数量。设置为 `None` 可以禁用任何限制。默认为 `1000`。
- `html_cutoff_text` - 如果设置，如果在 HTML 选择下拉菜单中截断了最大数量的项，则将显示文本指示符。默认为 `"More than {count} items…"`

您还可以使用 settings 中的 `HTML_SELECT_CUTOFF` 和 `HTML_SELECT_CUTOFF_TEXT` 全局控制这些。

在强制执行截断的情况下，您可能希望使用 HTML 表单中的普通输入字段。您可以使用 `style` 关键字参数执行此操作。例如：
```python
assigned_to = serializers.SlugRelatedField(
   queryset=User.objects.all(),
   slug_field='username',
   style={'base_template': 'input.html'}
)
```

## 反向关系 (Reverse relations)
请注意，反向关系不会自动包含在 `ModelSerializer` 和 `HyperlinkedModelSerializer` 类中。要包含反向关系，您必须显式的将其添加到字段列表中。例如：
```python
class AlbumSerializer(serializers.ModelSerializer):
    class Meta:
        fields = ('tracks', ...)
```
您通常希望确保在关系上设置了适当的 `related_name` 参数，可以用作字段名称。例如：
```python
class Track(models.Model):
    album = models.ForeignKey(Album, related_name='tracks', on_delete=models.CASCADE)
    ...
```
如果您还没有为反向关系设置相关名称，则需要在 `fields` 参数中使用自动生成的相关名称。例如：
```python
class AlbumSerializer(serializers.ModelSerializer):
    class Meta:
        fields = ('track_set', ...)
```
有关更多详细信息，请参阅有关[反向关系](https://docs.djangoproject.com/en/stable/topics/db/queries/#following-relationships-backward)的 Django 文档。

## 通用关系 (Generic relationships)
如果要序列化通用外键，则需要定义一个自定义字段，以明确确定要如何序列化关系目标。

例如，给定标签的以下模型，它与其他任意模型的通用关系：
```python
class TaggedItem(models.Model):
    """
    使用通用关系的标签任意模型实例。

    See: https://docs.djangoproject.com/en/stable/ref/contrib/contenttypes/
    """
    tag_name = models.SlugField()
    content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    object_id = models.PositiveIntegerField()
    tagged_object = GenericForeignKey('content_type', 'object_id')

    def __unicode__(self):
        return self.tag_name
```
以下两个模型，可能有相关的标签：
```python
class Bookmark(models.Model):
    """
    A bookmark consists of a URL, and 0 or more descriptive tags.
    """
    url = models.URLField()
    tags = GenericRelation(TaggedItem)


class Note(models.Model):
    """
    A note consists of some text, and 0 or more descriptive tags.
    """
    text = models.CharField(max_length=1000)
    tags = GenericRelation(TaggedItem)
```
我们可以定义一个可用于序列化标签实例的自定义字段，使用每个实例的类型来确定它应该如何序列化。
```python
class TaggedObjectRelatedField(serializers.RelatedField):
    """
    A custom field to use for the `tagged_object` generic relationship.
    """

    def to_representation(self, value):
        """
        Serialize tagged objects to a simple textual representation.
        """
        if isinstance(value, Bookmark):
            return 'Bookmark: ' + value.url
        elif isinstance(value, Note):
            return 'Note: ' + value.text
        raise Exception('Unexpected type of tagged object')
```
如果需要关系的目标具有嵌套表示，则可以使用 `.to_representation()` 方法中所需的序列化器：
```python
    def to_representation(self, value):
        """
        Serialize bookmark instances using a bookmark serializer,
        and note instances using a note serializer.
        """
        if isinstance(value, Bookmark):
            serializer = BookmarkSerializer(value)
        elif isinstance(value, Note):
            serializer = NoteSerializer(value)
        else:
            raise Exception('Unexpected type of tagged object')

        return serializer.data
```
请注意，使用 `GenericRelation` 字段表示的反向通用键可以使用常规关系字段类型进行序列化，因为关系中目标的类型总是已知的。

有关更多信息，请参阅有关[通用关系的 Django 文档](https://docs.djangoproject.com/en/stable/ref/contrib/contenttypes/#id1)。

## ManyToManyFields with a Through Model
默认情况下，以指定 `through` 模型为目标的 `ManyToManyField` 的关系字段设置为只读。

如果你显式指定指向带有 through 模型的 `ManyToManyField` 的关系字段，请确保将 `read_only` 设置为 `True`。

***

# 第三方包 (Third party packages)
以下是可用的第三方包。

## DRF Nested Routers
[drf-nested-routers](https://github.com/alanjds/drf-nested-routers) 包提供了用于处理嵌套资源的路由器和关系字段。

## Rest Framework Generic Relations
[rest-framework-generic-relations](https://github.com/Ian-Foote/rest-framework-generic-relations) 库为通用外键提供读/写序列化。
