## 🌐 What Are Query Parameters?

Query parameters are part of the **URL** and are used to pass optional or additional data to your API endpoints.

They come **after the `?`** in the URL and are separated by **`&`**:

```bash
http://127.0.0.1:8000/items/?skip=10&limit=5
```

In this example:
- `skip=10` and `limit=5` are **query parameters**.
- These are key-value pairs.

---

## 🚀 Declaring Query Parameters in FastAPI

Let’s start with a simple example:

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_items(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

### 🔍 Explanation:

- `/items/` is the **endpoint path**.
- `skip` and `limit` are **query parameters**.
- When you visit `http://127.0.0.1:8000/items/?skip=0&limit=10`, FastAPI reads those values from the URL.
- If they're not provided, FastAPI uses the **default values**: `skip=0`, `limit=10`.

---

## 🧠 How FastAPI Handles Query Parameters

### 1. **Type Conversion**
- FastAPI converts the query parameters to the types you declare.
  - Example: `skip: int` converts the string from the URL to an integer.
  - If the user passes something like `skip=abc`, FastAPI will return an error: `422 Unprocessable Entity`.

### 2. **Validation**
- If a required query parameter is missing or invalid, FastAPI automatically responds with an error and useful details.

### 3. **Documentation**
- FastAPI uses your function signature to automatically generate interactive API docs (via Swagger UI at `/docs`).

---

## 🔁 Default Values (Optional Parameters)

If a query parameter has a default value, it becomes optional:

```python
@app.get("/items/")
async def read_items(skip: int = 0, limit: int = 10):
    ...
```

- These can be omitted in the URL.
- `http://127.0.0.1:8000/items/` → uses `skip=0` and `limit=10`.

---

## ❓ Optional Parameters

To make a parameter **optional**, set its default to `None`:

```python
@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

- `item_id` is a **path parameter**.
- `q` is an **optional query parameter**.
- You can call the endpoint as:
  - `http://127.0.0.1:8000/items/foo` → no `q`
  - `http://127.0.0.1:8000/items/foo?q=bar` → with `q`

---

## ✅ Boolean Parameters

FastAPI can parse booleans from strings:

```python
@app.get("/items/{item_id}")
async def read_item(item_id: str, short: bool = False):
    ...
```

You can call this with:

- `/items/foo?short=true`
- `/items/foo?short=1`
- `/items/foo?short=on`
- `/items/foo?short=yes`

All are interpreted as `True`.

If none are provided or the value is not one of those, `short` is `False`.

---

## 📚 Multiple Path and Query Parameters

You can mix and match as needed — FastAPI detects by name and type:

```python
@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: str | None = None, short: bool = False
):
    ...
```

Here:
- `user_id` and `item_id` are **path parameters**.
- `q` and `short` are **query parameters**.

You can access this like:

```
/users/123/items/abc?q=hello&short=true
```

---

## ❗ Required Query Parameters

If you don’t set a default, a query parameter becomes **required**:

```python
@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    ...
```

- You **must** include `needy` in the URL.
- `/items/foo-item` → ❌ error
- `/items/foo-item?needy=value` → ✅ works

If it's missing, FastAPI returns:

```json
{
  "detail": [
    {
      "type": "missing",
      "loc": ["query", "needy"],
      "msg": "Field required",
      "input": null
    }
  ]
}
```

---

## 🧪 Combo Example

```python
@app.get("/items/{item_id}")
async def read_user_item(
    item_id: str,           # path (required)
    needy: str,             # required query parameter
    skip: int = 0,          # optional query with default
    limit: int | None = None  # optional query, default None
):
    return {"item_id": item_id, "needy": needy, "skip": skip, "limit": limit}
```

### Example URLs:

- `/items/abc?needy=yes` → ✅
- `/items/abc` → ❌ Missing required `needy`

---

## ✅ Summary Table

| Parameter Type | Example Declaration     | Required?     | From      |
|----------------|--------------------------|---------------|-----------|
| Path           | `item_id: str`           | Always        | Path      |
| Query (optional) | `q: str = None`        | No            | URL query |
| Query (required) | `needy: str`           | Yes           | URL query |
| Query (default)  | `skip: int = 0`        | No (default)  | URL query |
| Boolean Query    | `short: bool = False`  | No (default)  | URL query |

---
