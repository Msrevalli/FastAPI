## 📦 Header Parameters with Pydantic Models

### Why?
Grouping related headers into a Pydantic model gives you:
- Reusability
- Clean code
- Built-in validation
- Auto docs
- Control over which headers are accepted

---

### ✅ Example: Basic Header Model

```python
from typing import Annotated
from fastapi import FastAPI, Header
from pydantic import BaseModel

app = FastAPI()

class CommonHeaders(BaseModel):
    host: str
    save_data: bool
    if_modified_since: str | None = None
    traceparent: str | None = None
    x_tag: list[str] = []

@app.get("/items/")
async def read_items(headers: Annotated[CommonHeaders, Header()]):
    return headers
```

If a request sends headers like:

```
Host: example.com
Save-Data: true
X-Tag: alpha
X-Tag: beta
```

The response would look like:

```json
{
  "host": "example.com",
  "save_data": true,
  "if_modified_since": null,
  "traceparent": null,
  "x_tag": ["alpha", "beta"]
}
```

---

### ⛔ Forbidding Extra Headers

Want to **strictly allow only certain headers**? Use `extra="forbid"`:

```python
class CommonHeaders(BaseModel):
    model_config = {"extra": "forbid"}

    host: str
    save_data: bool
    if_modified_since: str | None = None
    traceparent: str | None = None
    x_tag: list[str] = []
```

If a request includes something like:

```
Tool: plumbus
```

You’ll get:

```json
{
  "detail": [
    {
      "type": "extra_forbidden",
      "loc": ["header", "tool"],
      "msg": "Extra inputs are not permitted",
      "input": "plumbus"
    }
  ]
}
```

---

### 🚫 Disable Underscore Conversion

By default, FastAPI converts underscores in field names to hyphens in headers.

So this field:
```python
save_data: bool
```

...expects the header:

```
Save-Data: true
```

To disable this conversion:

```python
@app.get("/items/")
async def read_items(
    headers: Annotated[CommonHeaders, Header(convert_underscores=False)],
):
    return headers
```

⚠️ Warning: Some proxies don’t support headers with underscores, so use this carefully.

---

### 💡 Pro Tip: Combine with Dependency Injection

You can even inject header models as dependencies, just like Pydantic `Body` models:

```python
from fastapi import Depends

@app.get("/secure/")
async def secure_route(headers: Annotated[CommonHeaders, Depends()]):
    if headers.save_data:
        # handle logic
        pass
    return headers
```

