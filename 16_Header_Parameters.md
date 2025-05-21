#### **Header parameters** are used to extract data from the HTTP **headers** of incoming requests.

These are often used for:

* Authentication (e.g., `Authorization` header)
* API versioning (e.g., `X-API-Version`)
* Custom metadata (e.g., `X-Request-ID`)

---

### ✅ How to Declare Header Parameters

You use `Header` from `fastapi`:

```python
from fastapi import FastAPI, Header
from typing import Annotated

app = FastAPI()

@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

#### Notes:

* The header name will be automatically converted from the parameter name (`user_agent` → `User-Agent`).
* `Header()` is used just like `Query()`, `Path()`, or `Body()`.

---

### 🌐 Sending Header Example

```http
GET /items/ HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
```

---

### ✅ **Basic Header Example**

```python
from typing import Annotated
from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

If the client sends:

```
User-Agent: Mozilla/5.0
```

Response:

```json
{
  "User-Agent": "Mozilla/5.0"
}
```

---

### ✅ **Header Name Conversion**

FastAPI **automatically converts underscores (`_`) to hyphens (`-`)** because HTTP headers often use hyphens.

So:
```python
x_token: Annotated[str | None, Header()] = None
```

...will match the `X-Token` header.

But if you **don’t** want that behavior:

```python
strange_header: Annotated[str | None, Header(convert_underscores=False)] = None
```

---

### ✅ **Multiple Values in One Header (Duplicate Headers)**

You can accept **multiple headers with the same name**:

```python
x_token: Annotated[list[str] | None, Header()] = None
```

Request headers:

```
X-Token: foo
X-Token: bar
```

Response:

```json
{
  "X-Token values": ["bar", "foo"]
}
```

---

### 🧠 Pro Tips

- Use headers for auth tokens, versioning, or metadata like `X-Request-ID`.
- Remember: headers are case-insensitive and hyphen-based, but you write them in Pythonic `snake_case`.
- Combine `Header` with dependencies for auth systems (like reading `Authorization` headers).

---
