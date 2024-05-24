1. `pip install user_agents`
2. Mac OS 11. x 及更高版本的 UA 是 Mac OS 10. x 的最新稳定版本 [216593 – \[macOS\] Limit reported macOS release to 10.15 series](https://bugs.webkit.org/show_bug.cgi?id=216593)

```python
from user_agents import parse


class RequestUtil:
    def __init__(self, request):
        self.request = request
        self.meta = self.request.META

    def get_ip(self):
        x_forwarded_for = self.meta.get("HTTP_X_FORWARDED_FOR", None)
        if x_forwarded_for:
            ip = x_forwarded_for.split(",")[-1].strip()
            return ip
        return self.meta.get("REMOTE_ADDR", "") or "unknown"

    def get_agent(self):
        return str(parse(self.meta.get("HTTP_USER_AGENT", "")))

    def get_browser(self):
        return parse(self.meta.get("HTTP_USER_AGENT", "")).get_browser()

    def get_os(self):
        return parse(self.meta.get("HTTP_USER_AGENT", "")).get_os()
```
