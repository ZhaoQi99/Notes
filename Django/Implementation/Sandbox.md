## How to use
```python 
import types
m = types.ModuleType("xxx", "The xxx module")
func = custom_exec(m, '<code>', '<name>')
```
## 实现
```python title:sandbox.py
import ast
import types
from typing import Callable

import structlog

# https://note.tonycrane.cc/ctf/misc/escapes/pysandbox
safe_dict = {
    "False": False,
    "True": True,
    "abs": abs,
    "all": all,
    "any": any,
    "ascii": ascii,
    "bin": bin,
    "bool": bool,
    "bytearray": bytearray,
    "bytes": bytes,
    "chr": chr,
    "complex": complex,
    "dict": dict,
    "dir": dir,
    "divmod": divmod,
    "enumerate": enumerate,
    "filter": filter,
    "float": float,
    "format": format,
    "hash": hash,
    "help": help,
    "hex": hex,
    "id": id,
    "int": int,
    "iter": iter,
    "len": len,
    "list": list,
    "map": map,
    "max": max,
    "min": min,
    "next": next,
    "oct": oct,
    "ord": ord,
    "pow": pow,
    "print": print,
    "range": range,
    "reversed": reversed,
    "round": round,
    "set": set,
    "slice": slice,
    "sorted": sorted,
    "str": str,
    "sum": sum,
    "tuple": tuple,
    "type": type,
    "zip": zip,
    "vars": vars,
    # Exception
    "__import__": None,
    "Exception": Exception,
    "ValueError": ValueError,
    "RuntimeError": RuntimeError,
    "isinstance": isinstance,
    "__build_class__": __build_class__,
}


ALLOWED_BUILTIN_IMPORTS = {
    "asyncio",
    "base64",
    "collections",
    "datetime",
    "hashlib",
    "hmac",
    "json",
    "math",
    "random",
    "re",
    "time",
    "traceback",
    "typing",
    "urllib.parse",
    "uuid",
    "pytz",
    "socket",
}

ALLOWED_THIRD_IMPORTS = {
    "sqlmodel",
    "structlog",
    "requests",
    "cryptography",
    "pycryptodome",
    "mysql.connector",
}
ALLOW_INTERNAL_IMPORTS = {
    # "lib.utils.xxx",
}

ALLOW_HIDDEN_IMPORTS = {
    "_strptime",
    "Crypto.Cipher",
    "Crypto.Hash",
    "Crypto.Random",
    "Crypto.Util",
    "Crypto.Protocol",
    "Crypto.PublicKey",
    "Crypto.Signature",
    "Crypto.IO",
    "Crypto.Math",
}
ALLOWED_IMPORTS = ALLOWED_BUILTIN_IMPORTS | ALLOWED_THIRD_IMPORTS | ALLOW_INTERNAL_IMPORTS | ALLOW_HIDDEN_IMPORTS


def __hook_import(name, *args, **kwargs):
    if name not in ALLOWED_IMPORTS:
        raise ImportError(f"Import {name} is not allowed.")
    return __import__(name, *args, **kwargs)


safe_dict["__import__"] = __hook_import


restricted_globals = {m: __import__(m) for m in ALLOWED_BUILTIN_IMPORTS} | {
    "__builtins__": safe_dict,
    "_strptime": __import__("_strptime"),  # datetime.strptime
    "datetime": __import__("datetime").datetime,  # Override built-in imported datetime
    "timedelta": __import__("datetime").timedelta,
    # typing
    "typing": __import__("typing"),
    "Optional": __import__("typing").Optional,
    "Union": __import__("typing").Union,
    "List": __import__("typing").List,
    "Dict": __import__("typing").Dict,
    "Annotated": __import__("typing").Annotated,
    "Tuple": __import__("typing").Tuple,
    "Deque": __import__("collections").deque,
    "Any": __import__("typing").Any,
    # Third package
    "requests": __import__("requests"),
}


def verify_py_code(code: str) -> tuple[bool, str | None]:
    try:
        code_obj = compile(code, "<inline>", "exec", flags=ast.PyCF_ONLY_AST)
    except SyntaxError as e:
        return False, (
            f"{e.__class__.__name__} in line {e.lineno}, col {e.offset}:\n"
            f"{e.text.rstrip()}\n"
            f"{max(e.offset - 1, 0) * ' '}^\n"
            f"错误信息: {e.msg}"
        )
    except Exception as e:
        return False, f"Code compile failed: {str(e)}"

    for x in ast.walk(code_obj):
        if type(x) is ast.Import | ast.ImportFrom:
            for module in x.names:
                if module.name not in ALLOWED_IMPORTS:
                    return False, f"Import module {module.name} is not allowed."
    return True, None


def custom_exec(module: types.ModuleType, code: str, func_name: str) -> Callable | None:
    code_obj = compile(code, "<custom-inline>", "exec")
    module.__dict__.update(restricted_globals)
    exec(code_obj, module.__dict__)
    return module.__dict__.get(func_name, None)

```