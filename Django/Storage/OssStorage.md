1. `pip install boto3`
2.  修改 `settings.py`
Examples:
```python
@deconstructible
class TestOssStorage(OssStorage):
    location = "test"

STORAGE = TestOssStorage()

STORAGE.save(posixpath.join(tmp_path, file), io.BytesIO(json_data.encode()))
STORAGE.open(path, "r")
STORAGE.delete(path.as_posix())
STORAGE.listdir(tmp_path)
STORAGE.exists()
STORAGE.url()
```
源码:
```python title:<project>/settings.py
OSS_ACCESS_KEY = env("OSS_ACCESS_KEY")
OSS_SECRET_KEY = env("OSS_SECRET_KEY")
OSS_ENDPOINT = env("OSS_ENDPOINT")
OSS_INTERNAL_ENDPOINT = env("OSS_INTERNAL_ENDPOINT")
OSS_BUCKET_NAME = env("OSS_BUCKET_NAME")
OSS_EXPIRE_TIME = env("OSS_EXPIRE_TIME")
OSS_REGION = env("OSS_REGION")
```

```python title:storage/oss_storage.py
import posixpath
from datetime import datetime
from io import UnsupportedOperation
from tempfile import SpooledTemporaryFile
from typing import IO, Any, Optional

import boto3
from botocore.exceptions import ClientError
from django.conf import settings
from django.core.files import File
from django.core.files.storage import Storage
from django.utils.deconstruct import deconstructible
from django.utils.encoding import force_str


def _normalize_endpoint(endpoint):
    if not endpoint.startswith("http://") and not endpoint.startswith("https://"):
        return "https://" + endpoint
    else:
        return endpoint


@deconstructible
class OssStorage(Storage):
    location = ""

    def __init__(
            self,
            access_key=None,
            secret_key=None,
            endpoint=None,
            internal_endpoint=None,
            bucket_name=None,
            expire: int = 0,
            region=None
    ):
        self.access_key = access_key or settings.OSS_ACCESS_KEY
        self.secret_key = secret_key or settings.OSS_SECRET_KEY
        self.endpoint = _normalize_endpoint(endpoint or settings.OSS_ENDPOINT)
        self.internal_endpoint = _normalize_endpoint(
            internal_endpoint or settings.OSS_INTERNAL_ENDPOINT or self.endpoint
        )
        self.bucket_name = bucket_name or settings.OSS_BUCKET_NAME
        self.region = region or settings.OSS_REGION
        self.expire = int(expire or settings.OSS_EXPIRE_TIME)
        self._connection = None

    @property
    def connection(self):
        if self._connection is None:
            self._connection = boto3.Session(
                aws_access_key_id=self.access_key,
                aws_secret_access_key=self.secret_key,
                region_name=self.region
            ).resource("s3", endpoint_url=self.internal_endpoint)
        return self._connection

    @property
    def bucket(self):
        return self.connection.Bucket(self.bucket_name)


    @property
    def client(self):
        return self.connection.meta.client

    def _normalize_name(self, name: str) -> str:
        base_path = force_str(self.location)
        final_path = posixpath.normpath(posixpath.join(base_path, name))
        if not final_path.startswith(base_path):
            raise PermissionError("Attempted access to '%s' denied." % name)
        return final_path

    def _open(self, name: str, mode: str = "wb") -> File:
        if "w" in mode:
            raise UnsupportedOperation("Oss file is not readable")

        name = self._normalize_name(name)
        try:
            return OssStorageFile(name, mode, self)
        except ClientError as err:
            if err.response["ResponseMetadata"]["HTTPStatusCode"] == 404:
                raise FileNotFoundError(f"File does not exist: {name}") from err
            raise

    def _save(self, name: Optional[str], content: IO[Any]) -> str:
        name = self._normalize_name(name)
        obj = self.bucket.Object(name)
        obj.upload_fileobj(content)
        prefix = f"{self.location}/"
        return name[len(prefix):] if name.startswith(prefix) else name

    def exists(self, name: str) -> bool:
        name = self._normalize_name(name)
        try:
            self.client.head_object(Bucket=self.bucket_name, Key=name)
            return True
        except ClientError as error:
            if error.response["ResponseMetadata"]["HTTPStatusCode"] == 404:
                return False
            raise

    def size(self, name: str) -> int:
        name = self._normalize_name(name)
        return self.bucket.Object(name).content_length

    def delete(self, name: str) -> None:
        name = self._normalize_name(name)
        self.bucket.Object(name).delete()

    def url(self, name: Optional[str], expire=None) -> str:
        name = self._normalize_name(name)
        params = {
            "Bucket": self.bucket_name,
            "Key": name,
        }
        if expire is None:
            expire = self.expire
				return boto3.client(
				    "s3",
				    aws_access_key_id=self.access_key,
				    aws_secret_access_key=self.secret_key,
				    endpoint_url=self.endpoint,
				    region_name=self.region,
				).generate_presigned_url(
				    "get_object",
				    Params=params,
				    ExpiresIn=expire,
				)

    def get_modified_time(self, name: str) -> datetime:
        name = self._normalize_name(name)
        return self.bucket.Object(name).last_modified

    def listdir(self, path: str):
        prefix = self._normalize_name(path) + "/"
        response = self.client.list_objects(
            Bucket=self.bucket_name, Prefix=prefix, Delimiter="/"
        )
        # from lib.backports.str import String < Python 3.9
        # str.removeprefix was add in version Python 3.9
    
        directories = [
            str(obj["Prefix"]).removeprefix(prefix).rstrip("/")
            for obj in response.get("CommonPrefixes", [])
        ]
        files = [
            str(obj["Key"]).removeprefix(prefix).rstrip("/")
            for obj in response.get("Contents", [])
        ]
        return directories, files

    def delete(self, name: str) -> None:
        name = self._normalize_name(name)
        self.bucket.Object(name).delete()

		def upload_file(self, file_path, file_name):
		    file_name = self._normalize_name(file_name)
		    self.client.upload_file(file_path, self.bucket_name, file_name)

class OssStorageFile(File):
    def __init__(self, name, mode, storage) -> None:
        self.name = name
        self.mode = mode
        self._storage = storage
        self._get_file()

    def _get_file(self):
        self.file = SpooledTemporaryFile(max_size=settings.FILE_UPLOAD_MAX_MEMORY_SIZE)
        self._storage.bucket.Object(self.name).download_fileobj(self.file)
        self.file.seek(0)

    def read(self, num_bytes=None):
        data = self.file.read(num_bytes)

        if "b" not in self.mode:
            return force_str(data)
        return data

```
