### ✅ **What are Cookie Parameters?**

Cookies are values stored in the user's browser and sent along with every request to the same domain. In FastAPI, you can access cookie values from incoming requests just like query or path parameters.

---

### ✅ **When to Use Cookies**

- Tracking user sessions (`session_id`)
- Storing preferences (`theme=dark`)
- Keeping non-sensitive auth tokens (`remember_me=true`)
- A/B testing or ads (`ads_id`)

---

### ✅ **How to Use `Cookie` in FastAPI**

Here’s a full minimal example:

```python
from typing import Annotated
from fastapi import Cookie, FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

#### 🔍 Breakdown:
- `ads_id` is a cookie, **not** a query or path parameter.
- If the client sends a cookie header like:
  ```
  Cookie: ads_id=abc123
  ```
  You’ll get `{"ads_id": "abc123"}` in the response.

---

### ✅ **Setting a Cookie (for completeness)**

FastAPI lets you read cookies, but if you want to **set** them in the response, you can do this:

```python
from fastapi import Response

@app.get("/set-cookie/")
async def set_cookie(response: Response):
    response.set_cookie(key="ads_id", value="cool_ad_456")
    return {"message": "Cookie set!"}
```

---

### 🧠 Pro Tip

- You can still use validation with cookies (like regex or value length).
- `Cookie()` has all the same options as `Query()` and `Path()`.

Example with extra options:
```python
ads_id: Annotated[str | None, Cookie(max_length=20, description="Ad session ID")] = None
```

---

