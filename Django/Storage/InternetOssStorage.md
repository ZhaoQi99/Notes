[[OssStorage]]
```python title:<project>/settings.py
INTERNET_OSS_ACCESS_KEY = env("INTERNET_OSS_ACCESS_KEY")
INTERNET_OSS_SECRET_KEY = env("INTERNET_OSS_SECRET_KEY")
INTERNET_OSS_ENDPOINT = env("INTERNET_OSS_ENDPOINT")
INTERNET_OSS_INTERNAL_ENDPOINT = env("INTERNET_OSS_INTERNAL_ENDPOINT")
INTERNET_OSS_BUCKET_NAME = env("INTERNET_OSS_BUCKET_NAME")
INTERNET_OSS_EXPIRE_TIME = env("INTERNET_OSS_EXPIRE_TIME")
INTERNET_OSS_REGION = env("INTERNET_OSS_REGION")
```
```python
class InternetOssStorage(OssStorage):
    def __init__(
        self,
        access_key=None,
        secret_key=None,
        endpoint=None,
        internal_endpoint=None,
        bucket_name=None,
        expire: int = 0,
        region=None,
    ):
        super().__init__(
            access_key or settings.INTERNET_OSS_ACCESS_KEY,
            secret_key or settings.INTERNET_OSS_SECRET_KEY,
            endpoint or settings.INTERNET_OSS_ENDPOINT,
            internal_endpoint or settings.INTERNET_OSS_INTERNAL_ENDPOINT,
            bucket_name or settings.INTERNET_OSS_BUCKET_NAME,
            expire or settings.INTERNET_OSS_EXPIRE_TIME,
            region or settings.INTERNET_OSS_REGION,
        )
```