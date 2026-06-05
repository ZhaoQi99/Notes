> 自定义 DRF 异常处理函数。
## 主要功能

- 对 ValidationError 进行标准化处理。
- 记录未被 DRF 默认处理的异常日志。
- DEBUG 模式下直接返回异常信息，便于开发调试。
- 生产环境异常告警通知。
## exception_handler. py
```python title:exception_handler.py
import logging
import socket
import traceback
from datetime import datetime

from django.conf import settings
from rest_framework import exceptions, serializers, status
from rest_framework.response import Response
from rest_framework.views import exception_handler

from system.utils.setting import SystemSetting

LOGGER = logging.getLogger("server")


def custom_exception_handler(exc, context):
    if isinstance(exc, exceptions.ValidationError):
        exc = exceptions.ValidationError(detail=serializers.as_serializer_error(exc))

    response = exception_handler(exc, context)

    if response is None:
        request = context["request"]
        LOGGER.error(
            """"\
Processing exception: %s
%s %s
User: %s
%s""",
            exc,
            request.method,
            request.get_full_path(),
            request.user,
            traceback.format_exc(),
        )
        if settings.DEBUG is True:
            return Response(
                {"detail": str(exc)}, status=status.HTTP_500_INTERNAL_SERVER_ERROR
            )
        # if SystemSetting.get_default(SystemSetting.Enum.SYSTEM_MONITOR.value, False):
        if 需要发送告警:

            exc_type = exc.__class__.__name__
            now = datetime.now().astimezone()
            subject = f"[Themis Next Status] {exc_type} {request.path}"
            kwargs = {
                "exc_type": exc_type,
                "message": str(exc).replace("\n", " "),
                "time": now.strftime("%Y-%m-%d %H:%M:%S"),
                "username": str(request.user),
                "method": request.method.upper(),
                "path": request.get_full_path(),
                "trace": traceback.format_exc().strip(),
                "ip": socket.gethostbyname(socket.gethostname()),
            }

            # fmt:off
            content = """\
异常类型: {exc_type}
异常信息: {message}
用户: {username}
时间: {time}
请求路径: {method} {path}
IP: {ip}
堆栈: {trace}
            """.format(**kwargs)
            # fmt:on
        return Response(
            {"detail": "Server Error (500)"},
            status=status.HTTP_500_INTERNAL_SERVER_ERROR,
        )

    return response

```