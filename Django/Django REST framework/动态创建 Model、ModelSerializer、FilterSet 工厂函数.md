## 模型类 -> model_factory
```python title:models.py
from django.db import models

def model_factory(name, __module__, fields, bases=None, attrs=None, meta_attrs=None):
    meta = type("Meta", (), {**(meta_attrs or {})})
    model_class = type(
        name,
        (*(bases or ()), models.Model),
        {"Meta": meta, "__module__": __module__, **fields, **(attrs or {})},
    )
    return model_classÏ
```
Examples:
```python
from django.db import models

fields = {
    "username": models.CharField(max_length=255, verbose_name="用户名"),
}

# 简单用法
model = model_factory("User", __module__="account.models.user", fields=fields)

# 高级用法
class BaseModel(models.Model):
    create_dt = models.DateTimeField("创建时间", auto_now_add=True)
    update_dt = models.DateTimeField("更新时间", auto_now=True)

    class Meta:
        abstract = True
        default_manager_name = "objects"

meta_attrs = {
    "db_table": "user",
    "verbose_name": "用户",
    "verbose_name_plural": "用户",
}
attrs = {
    "__str__": lambda self: self.username,
}
model = model_factory(
    "DynamicUser",
    __module__="account.models.user",
    fields=fields,
    bases=(BaseModel,),
    attrs=attrs,
    meta_attrs=meta_attrs,
)

# QuerySet
model.objecs.all()
```
## 序列化器类 -> serializer_factory
```python title:serializers.py
from rest_framework import serializers

ALL_FIELDS = "__all__"

def serializer_factory(
    model, fields=ALL_FIELDS, exclude=None, bases=None, attrs=None, name_prefix=""
):
    meta_attrs = {"model": model}
    if exclude:
        meta_attrs["exclude"] = exclude
    elif fields:
        meta_attrs["fields"] = fields

    meta = type("Meta", (), meta_attrs)
    serializer = type(
        "{prefix}{model}Serializer".format(
            prefix=name_prefix, model=model._meta.object_name
        ),
        (*(bases or ()), serializers.ModelSerializer),
        {"Meta": meta, **(attrs or {})},
    )
    return serializer
```
Examples:
```python
from lib.rest_framework import serializers as lib_serializers

# 简单用法
serializer_class = lib_serializers.serializer_factory(User, exclude=("password",))

# 高级用法
attrs = {
    "nickname": serializers.SerializerMethodField(),
    "get_nickname": lambda self, obj: obj.username.upper(),
}
serializer_class = lib_serializers.serializer_factory(
    User,
    exclude=("deleted",),
    name_prefix="List",
    attrs=attrs,
)

```

### 过滤集类 -> filterset_factory
```python title:filterset.py
from django_filters.constants import ALL_FIELDS
from django_filters import filterset

def filterset_factory(model, fields=ALL_FIELDS, bases=None, attrs=None):
    meta = type(str("Meta"), (object,), {"model": model, "fields": fields})
    filterset = type(
        str("%sFilterSet" % model._meta.object_name),
        (*(bases or ()), filterset.FilterSet),
        {"Meta": meta, **(attrs or {})},
    )
    return filterset
```
Examples:
```python
from lib.rest_framework import filterset as lib_filterset
from django_filters import rest_framework as filters

queryset = User.objects.all()
fields = ('is_active',)

# 简单用法
filterset_class = lib_filterset.filterset_factory(User, fields=fields)

# 高级用法
class BaseUserFilterSet(filters.FilterSet):
    custom = filters.ChoiceFilter(choices=CUSTOM_CHOICES, field_name="<field>", lookup_expr="in")

attrs = {
    "is_superuser": filters.BooleanFilter(field_name="is_superuser", lookup_expr="exact")
}
filterset_class = lib_filterset.filterset_factory(
	queryset.model,
	fields=fields,
	bases=(BaseUserFilterSet,)
	attrs=attrs
)
```