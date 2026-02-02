DEBUG 模式下仅允许携带有效 JWT 的活跃超级用户访问 DebugToolbar

访问：`http://xxx/__debug__` （需搭配 Nginx 使用）
## debug. py

依赖 `JWTAuthentication`： [[JSON Web Token]]

```python title:debug.py
from django.conf import settings
from django.core.exceptions import PermissionDenied

from lib.authentication import JWTAuthentication


class DebugToolbarMiddleware:
    COOKIE_NAME = "Token"

    def __init__(self, get_response) -> None:
        self.get_response = get_response

    def __call__(self, request):
        if request.path.startswith("/__debug__") and settings.DEBUG is True:
            jwt_token = request.COOKIES.get(self.COOKIE_NAME, "")
            try:
                user, _ = JWTAuthentication().authenticate_credentials(jwt_token)
                if user.is_superuser and user.is_active:
                    pass
                else:
                    raise PermissionDenied
            except:
                raise PermissionDenied

        return self.get_response(request)

```
## Settings. py
```python title:settings.py
# https://django-debug-toolbar.readthedocs.io/en/latest/installation.html
if DEBUG:
    INSTALLED_APPS += ["debug_toolbar"]
    MIDDLEWARE += [
        "debug_toolbar.middleware.DebugToolbarMiddleware",
    ]
DEBUG_TOOLBAR_PANELS = [
    "debug_toolbar.panels.history.HistoryPanel",
    "debug_toolbar.panels.timer.TimerPanel",
    "debug_toolbar.panels.headers.HeadersPanel",
    "debug_toolbar.panels.request.RequestPanel",
    "debug_toolbar.panels.sql.SQLPanel",
    "debug_toolbar.panels.cache.CachePanel",
    "debug_toolbar.panels.signals.SignalsPanel",
    # "debug_toolbar.panels.redirects.RedirectsPanel",
    # "debug_toolbar.panels.profiling.ProfilingPanel",
]

DEBUG_TOOLBAR_CONFIG = {
    "DISABLE_PANELS": {
        "debug_toolbar.panels.settings.SettingsPanel",
    },
    "TOOLBAR_LANGUAGE": "zh_CN",
    "SHOW_TOOLBAR_CALLBACK": lambda request: DEBUG,
}

```