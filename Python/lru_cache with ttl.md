## How to use
```python
from functools import lru_cache

@lru_cache (maxsize=32)
def func (x, y, ttl=None):
    print ('func 2', ttl)
    return x+y
    
func(1, 2, ttl_hash=get_ttl_hash())
```
## Implementation
```python
import time
def get_ttl_hash (seconds=60):
    """Return the same value withing `seconds` time period"""
    return round (time.time () / seconds)


```