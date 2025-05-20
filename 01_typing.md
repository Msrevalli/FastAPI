### FastAPI and Typing
The `typing` library in Python is used to provide **type hints**—a way to hint what type of data a variable, function argument, or return value is expected to have. This helps with:

- Code readability
- Editor autocomplete & error checking
- Better collaboration
- Fewer bugs

Type hints **don’t enforce types at runtime** (unless you use a separate library like `pydantic`, `mypy`, etc.)—they're mainly for tooling and static analysis.

---

### Basic Usage

```python
from typing import List, Dict, Tuple, Union, Optional

def greet(name: str) -> str:
    return f"Hello, {name}"

def get_numbers() -> List[int]:
    return [1, 2, 3]

def get_user() -> Dict[str, str]:
    return {"name": "Alice", "email": "alice@example.com"}

def divide(a: float, b: float) -> Union[float, str]:
    if b == 0:
        return "Cannot divide by zero"
    return a / b

def get_name(user: Dict[str, str]) -> Optional[str]:
    return user.get("name")  # could return None
```

---

### Common `typing` Types

| Type | Description |
|------|-------------|
| `List[X]` | A list of items of type `X` |
| `Dict[X, Y]` | A dictionary with keys of type `X` and values of type `Y` |
| `Tuple[X, Y]` | A tuple with specific types in each position |
| `Union[X, Y]` | A value that could be type `X` or type `Y` |
| `Optional[X]` | Either type `X` or `None` (shortcut for `Union[X, None]`) |
| `Any` | Any type at all (no checking) |
| `Callable` | A function with specific arguments and return type |

---
FastAPI relies heavily on the `typing` module to define request and response schemas and validate data automatically.
