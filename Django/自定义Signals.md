## 定义Signal
```python title:account/signals.py
from django.dispatch import Signal

user_logged_in = Signal()
user_login_failed = Signal()

```

## 监听Signal
### 使用`connect`方法
```python title:account/apps.py
from django.apps import AppConfig

from .signals import user_logged_in

class AccountConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name = "account"

    def ready(self) -> None:
        from django.contrib.auth.models import update_last_login

        user_logged_in.connect(update_last_login, dispatch_uid="update_last_login")

```
### 使用`receiver`装饰器
```python
from account.signals import user_logged_in
from django.dispatch import receiver

@receiver(user_logged_in)
def save_logged_in_log(sender, request, user, **kwargs):
    pass

```

## 发送Signal
```python title:account/serializers.py
class LoginSerializer(serializers.Serializer):
    def create(self, attrs):
        ...
        user_logged_in.send(
            sender=LoginSerializer.__class__, user=user, request=self.context["request"]
        )
        ...
```