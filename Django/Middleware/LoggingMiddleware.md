## 让某个接口不记录日志
1. 按路径忽略, `IGNORE_RECORD_PATHS = (reverse("account:login"),)`
2. 在对应的 View（或 ViewSet）上加属性：`ignore_logging_operation = True`
```python title:views.py
class LoginView(generics.CreateAPIView):
    ...
    ignore_logging_operation = True
    ...
```
## logging.py
```python title: logging.py
import contextlib
from http import HTTPStatus

from django.urls import reverse
from rest_framework.authentication import get_authorization_header

from account.models import Permission, User
from lib.authentication.access_key import AUTH_HEADER, AccessKey
from lib.authentication.jwt import Token
from lib.utils.http import HTTPMethod
from lib.utils.request import RequestUtil
from system.models import OperationLog

RECORD_METHODS = (
    HTTPMethod.POST.value,
    HTTPMethod.PUT.value,
    HTTPMethod.PATCH.value,
    HTTPMethod.DELETE.value,
)
IGNORE_RESPONSE_STATUS = (HTTPStatus.OK.value, HTTPStatus.CREATED.value)
IGNORE_RECORD_PATHS = (reverse("account:login"),)


class LoggingMiddleware:
    ignore_view_attribute = "ignore_logging_operation"

    def __init__(self, get_response):
        self.get_response = get_response

    def handle_body(self, request, body):
        if password := body.get("password", None):
            body["password"] = "*" * len(password)
        # 包含文件的请求，过滤掉上传文件字段
        if request.content_type != 'multipart/form-data':
            return body
        if not request.FILES:
            return body
        for key in request.FILES.keys():
            body.pop(key)
        return body

    def handle_response(self, response):
        if response.status_code in IGNORE_RESPONSE_STATUS:
            return None
        return response.data

    def get_operator(self, request):
        headers = request.headers
        if not request.user.is_authenticated:
            if _ := headers.get(AUTH_HEADER, None):
                return str(AccessKey(_))
            if header := get_authorization_header(request):
                with contextlib.suppress(Exception):
                    return Token(header.split()[1].decode(), False).get(
                        User.USERNAME_FIELD
                    )
                return "Anonymous"
        return str(request.user)

    def get_desc(self, view):
        if getattr(view, "basename", None):
            name = Permission.get_perm_name(view)
            if (
                permission := Permission.objects.filter(code_name=name).first()
                or Permission.objects.filter(code_name__contains=name).first()
            ):
                return "{} | {}".format(
                    permission.category.split("-")[-1], permission.name
                )
            return name
        return view.request._request.resolver_match.url_name

    def __call__(self, request):
        response = self.get_response(request)
        path = request.path
        if path.startswith("/api") and path not in IGNORE_RECORD_PATHS:
            if request.method in RECORD_METHODS:
                util = RequestUtil(request)
                renderer_context = getattr(response, "renderer_context", None)
				if renderer_context is None or getattr(
				    renderer_context["view"], self.ignore_view_attribute, False
				):
				    return response
                body = self.handle_body(request, renderer_context["request"].data)
                desc = self.get_desc(renderer_context["view"])
                kwargs = {
                    "path": request.path,
                    "method": request.method,
                    "status_code": response.status_code,
                    "operator": self.get_operator(request),
                    "ip": util.get_ip(),
                    "body": body,
                    "response": self.handle_response(response),
                    "desc": desc,
                }
                OperationLog.objects.create(**kwargs)
        return response

```