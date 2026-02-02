```python title:admin.py
@admin.register(Model)
class ModelAdmin(admin.ModelAdmin):
    readonly_fields = ("pretty_config",)

    def pretty_config(self, obj):
        result = json.dumps(
            obj.config, indent=2, sort_keys=True, ensure_ascii=False
        )
        return mark_safe(f"<pre>{result}</pre>")
```