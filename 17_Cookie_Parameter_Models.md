## 🍪 Cookie Parameters with Pydantic Models

FastAPI lets you model a group of cookies as a Pydantic class. This gives you:

✅ Reusability  
✅ Validation  
✅ Cleaner route functions  
✅ Schema docs generation

---

### ✅ Basic Cookie Model Example

```python
from typing import Annotated
from fastapi import Cookie, FastAPI
from pydantic import BaseModel

app = FastAPI()

class Cookies(BaseModel):
    session_id: str
    fatebook_tracker: str | None = None
    googall_tracker: str | None = None

@app.get("/items/")
async def read_items(cookies: Annotated[Cookies, Cookie()]):
    return cookies
```

If the request includes these cookies:

```
session_id=abc123; fatebook_tracker=track42
```

You’ll get:

```json
{
  "session_id": "abc123",
  "fatebook_tracker": "track42",
  "googall_tracker": null
}
```

---

### ⛔ Forbidding Extra Cookies

Wanna lock it down and reject unexpected cookies? Use `extra="forbid"` in your model config:

```python
class Cookies(BaseModel):
    model_config = {"extra": "forbid"}

    session_id: str
    fatebook_tracker: str | None = None
    googall_tracker: str | None = None
```

Now if someone sends:

```
session_id=abc123; santa_tracker=good-list-please
```

FastAPI will return:

```json
{
  "detail": [
    {
      "type": "extra_forbidden",
      "loc": ["cookie", "santa_tracker"],
      "msg": "Extra inputs are not permitted",
      "input": "good-list-please"
    }
  ]
}
```

---

### 📘 Bonus Tips

- Use this pattern for **complex auth cookies**, **tracking cookies**, or **preferences**.
- You can combine cookie models with `Depends` for injection into multiple routes.
- Also works for **headers** and **query parameters**!

---

