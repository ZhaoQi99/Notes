## admin. py
依赖 JWTAuthentication：[[JSON Web Token]]
```python title:admin.py
from aiops.admin import MyAdminSite
from lib.authentication import JWTAuthentication


class AdminJWTMiddleware:
    COOKIE_NAME = ""

    def __init__(self, get_response) -> None:
        self.get_response = get_response

    def __call__(self, request):
        if request.path.startswith("/admin") and request.user.is_authenticated is False:
            jwt_token = request.COOKIES.get(self.COOKIE_NAME, "")
            try:
                user, _ = JWTAuthentication().authenticate_credentials(jwt_token)
                request.user = user
            except:
                pass
        return self.get_response(request)

```