Examples:
```python title:views.py
from django_filters import rest_framework as filters

from lib.rest_framework import filters as lib_filters

class TestFilterSet(filters.FilterSet):
	field_a = lib_filters.CharInFilter(field_name='field_a')
	field_b = lib_filters.MultipleChoiceFilter(choices=CHOICES, method="field_b")

	def filter_field_b(self, queryset, name, value):
		pass

```
*  `localhost:8000/test/?field=a,b,c`
* `localhost:8000/test/?field=a`
```python title:filters.py
import django_filters

class CharInFilter(django_filters.BaseInFilter, django_filters.CharFilter):
    pass


class MultipleChoiceFilter(django_filters.BaseInFilter, django_filters.ChoiceFilter):
    pass
```
