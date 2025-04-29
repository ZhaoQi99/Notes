[\[Feature Request\] Make numpy scalar types JSON serializable · Issue #16432 · numpy/numpy · GitHub](https://github.com/numpy/numpy/issues/16432)

```python
from lib.utils.json import NumpyEncoder

json.dumps(arr, cls=NumpyEncoder)
```

```python title:lib/utils/json.py
import json
import numpy as np

class NumpyEncoder(json.JSONEncoder):
    def default(self, o):
        if isinstance(
            o,
            (
                np.int_,
                np.intc,
                np.intp,
                np.int8,
                np.int16,
                np.int32,
                np.int64,
                np.uint8,
                np.uint16,
                np.uint32,
                np.uint64,
            ),
        ):
            return int(o)
        elif isinstance(o, (np.float_, np.float16, np.float32, np.float64)):
            return float(o)
        elif isinstance(o, (np.complex_, np.complex64, np.complex128)):
            return {"real": o.real, "imag": o.imag}
        elif isinstance(o, (np.ndarray,)):
            return o.tolist()
        elif isinstance(o, (np.bool_)):
            return bool(o)
        elif isinstance(o, (np.void)):
            return None
        return super(NumpyEncoder, self).default(o)

```

> [!note] '-Infinity', 'Infinity' 或 'NaN'

```python
constant_map = {
    # keep exception for infinity and -infinity.
    "-Infinity": float("-Infinity"),
    "Infinity": float("Infinity"),
    "NaN": None,
}
json.loads(json_str, parse_constant=lambda x: constant_map[x])
```