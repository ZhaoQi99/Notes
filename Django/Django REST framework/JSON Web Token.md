1. settings. py
```python title: <project>/setting. py
TOKEN_LIFETIME = 3600
REST_FRAMEWORK = {
	"DEFAULT_AUTHENTICATION_CLASSES" : ["lib.authentication.JWTAuthentication"],
	....
}
```
2. JWTAuthentication
```python title:lib/authentication/jwt.py
import uuid
from calendar import timegm
from datetime import timedelta

import jwt
from django.conf import settings
from django.contrib.auth import get_user_model
from django.utils import timezone
from jwt.exceptions import ExpiredSignatureError, MissingRequiredClaimError
from rest_framework import authentication
from rest_framework.exceptions import AuthenticationFailed


User = get_user_model()


class Token:
    algorithm = "HS256"

    def __init__(self, token=None, verify=True):
        from system.utils.setting import SystemSetting

        self.payload = {}
        self.key = settings.SECRET_KEY
        self.current_time = timezone.now()
        self.lifetime = timedelta(
            seconds=SystemSetting.get_default(
                SystemSetting.Enum.TOKEN_LIFETIME.value,
                SystemSetting.dj_settings.TOKEN_LIFETIME,
            )
        )

        if token is None:
            self.set_exp(from_time=self.current_time, lifetime=self.lifetime)
            self.set_iat(at_time=self.current_time)
            self.set_jti()
        else:
            self.payload = jwt.decode(
                token,
                self.key,
                algorithms=[self.algorithm],
                options={"verify_signature": verify},
            )

    def __getitem__(self, key):
        return self.payload[key]

    def __setitem__(self, key, value):
        self.payload[key] = value

    def __delitem__(self, key):
        del self.payload[key]

    def get(self, key, default=None):
        return self.payload.get(key, default)

    def set_jti(self, claim="jti"):
        self.payload[claim] = uuid.uuid4().hex

    def set_iat(self, at_time=None, claim="iat"):
        if at_time is None:
            at_time = self.current_time
        self.payload[claim] = timegm(at_time.utctimetuple())

    def set_exp(self, lifetime=None, from_time=None, claim="exp"):
        if from_time is None:
            from_time = self.current_time
        if lifetime is None:
            lifetime = self.lifetime

        self.payload[claim] = timegm((from_time + lifetime).utctimetuple())

    def check_exp(self, current_time=None, claim="exp"):
        if current_time is None:
            current_time = self.current_time

        claim_time = self.payload.get(claim, None)
        if claim_time is None:
            raise MissingRequiredClaimError(claim)

        if claim_time <= timegm(current_time.utctimetuple()):
            raise ExpiredSignatureError(f"Token '{claim}' claim has expired")

    @classmethod
    def for_user(cls, user):
        username = getattr(user, User.USERNAME_FIELD)

        token = cls()
        token[User.USERNAME_FIELD] = username

        return token

    @classmethod
    def for_username(cls, username):
        token = cls()
        token["username"] = username

        return token

    def __str__(self) -> str:
        return jwt.encode(self.payload, self.key, algorithm=self.algorithm)

    def __repr__(self) -> str:
        return "<{}> {}".format(self.__class__.__name__, self.payload)


class JWTAuthentication(authentication.BaseAuthentication):
    www_authenticate_realm = "api"
    AUTH_HEADER_TYPE = "Bearer"

    def authenticate(self, request):
        if (header := self.get_header(request)) is None:
            return None

        if (raw_token := self.get_raw_token(header)) is None:
            return None

        return self.authenticate_credentials(raw_token)

    def authenticate_header(self, request):
        return '{} realm="{}"'.format(
            self.AUTH_HEADER_TYPE, self.www_authenticate_realm
        )

    def get_header(self, request):
        return authentication.get_authorization_header(request)

    def get_raw_token(self, header):
        parts = header.split()

        if len(parts) == 0 or parts[0] != self.AUTH_HEADER_TYPE.encode():
            return None

        if len(parts) != 2:
            raise AuthenticationFailed(
                "Authorization header must contain two space-delimited values"
            )

        return parts[1].decode()

    def authenticate_credentials(self, raw_token):
        try:
            token = Token(raw_token)
        except Exception as e:
            raise AuthenticationFailed(str(e)) from e

        if (username := token.get(User.USERNAME_FIELD)) is None:
            raise AuthenticationFailed(
                "Token contained no recognizable user identification"
            )

        if (
            user := User.objects.filter(**{User.USERNAME_FIELD: username}).first()
        ) is None:
            raise AuthenticationFailed("User not found")

        if not user.is_active:
            raise AuthenticationFailed("User is inactive")

        return (user, str(token))


```