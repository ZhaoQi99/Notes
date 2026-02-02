## 使用
1. 修改 `settings.py` 中的 `INSTALLED_APPS`
```python title:settings.py
INSTALLED_APPS = [
    # "django.contrib.admin",
    "<project>.apps.MyAdminConfig",
    ...
]
```
2. 搭配 [[AdminJWTMiddleware]] 一起使用
## 实现
```python title:admin.py
from django.contrib import admin
from django.core.handlers.wsgi import WSGIRequest
from django.template.response import TemplateResponse


class MyAdminSite(admin.AdminSite):
    COOKIE_NAME = "Token"

    def has_permission(self, request: WSGIRequest) -> bool:
        return request.user.is_active and request.user.is_superuser

    def get_app_list(self, request: WSGIRequest, app_label=None):
        if request.user.is_superuser is False:
            return []
        return super().get_app_list(request, app_label=app_label)

    def logout(self, request: WSGIRequest, extra_context=None) -> TemplateResponse:
        response = super().logout(request, extra_context)
        response.delete_cookie(self.COOKIE_NAME)
        return response


```