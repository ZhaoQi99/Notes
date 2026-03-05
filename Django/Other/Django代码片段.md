1.  获取Model所有字段
```python
ALL_FIELDS = set(x.attname for x in Model._meta.local_fields)
```
