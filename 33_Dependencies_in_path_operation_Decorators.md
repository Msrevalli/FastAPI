**Declaring dependencies at the decorator level**, especially useful when you **don’t need the return value** in your route handler but **still want to enforce checks or run side effects** (like validation or auth).

---

## 🔍 What is this feature about?

### 👉 Usually, dependencies look like this:

```python
@app.get("/items/")
def read_items(x_token: Annotated[str, Depends(verify_token)]):
    return ...
```

But what if you're not going to use `x_token` in the function? Your editor might complain about an unused variable, and it's not super clean.

---

## ✅ Cleaner alternative: Decorator-level `dependencies`

```python
@app.get("/items/", dependencies=[Depends(verify_token)])
def read_items():
    return ...
```

This **executes `verify_token`**, and if it raises an exception, the request is stopped. But **no value is passed to `read_items()`**.

---

### 📦 Full Example

```python
from typing import Annotated
from fastapi import Depends, FastAPI, Header, HTTPException

app = FastAPI()

# Dependency that checks the token
async def verify_token(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")

# Dependency that checks the key
async def verify_key(x_key: Annotated[str, Header()]):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key  # This will be ignored if declared in `dependencies=[]`

# Route that uses both checks but doesn't need their values
@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```
---

### ✅ **Valid Headers (Access Granted)**

**Request:**

```http
GET /items/ HTTP/1.1
Host: localhost:8000
X-Token: fake-super-secret-token
X-Key: fake-super-secret-key
```

**Response:**

```json
[
  {"item": "Foo"},
  {"item": "Bar"}
]
```

**Status Code:** `200 OK`

---

### ❌ **Missing X-Token Header**

**Request:**

```http
GET /items/ HTTP/1.1
Host: localhost:8000
X-Key: fake-super-secret-key
```

**Response:**

```json
{
  "detail": "X-Token header invalid"
}
```

**Status Code:** `400 Bad Request`

---

### ❌ **Invalid X-Key Header**

**Request:**

```http
GET /items/ HTTP/1.1
Host: localhost:8000
X-Token: fake-super-secret-token
X-Key: wrong-key
```

**Response:**

```json
{
  "detail": "X-Key header invalid"
}
```

**Status Code:** `400 Bad Request`

---

### ❌ **No Headers at All**

**Request:**

```http
GET /items/ HTTP/1.1
Host: localhost:8000
```

**Response:**

```json
{
  "detail": "X-Token header invalid"
}
```

**Status Code:** `400 Bad Request`

---

### 🛡️ Behavior

- FastAPI **runs the dependencies before the route**.
- If either one fails (e.g. wrong header), the route won’t be executed.
- Clean route signature with **no unused parameters**.

---

## 💡 Use Cases

- **Auth/Access control** (e.g., `verify_token`)
- **Logging or analytics**
- **Rate limiting**
- **Feature flags**
- **Conditionally blocking traffic** based on headers, IP, etc.

---

## 🔄 Reusability Bonus

You can **reuse dependencies that return values**, even if you don’t use those values in some routes:

```python
@app.get("/ping", dependencies=[Depends(verify_token)])
def ping():
    return {"status": "ok"}
```

And elsewhere:

```python
@app.get("/admin")
def admin_dashboard(key: Annotated[str, Depends(verify_key)]):
    return {"access": "granted", "key": key}
```

Same function, used differently — super DRY.

---

## 🔁 Shared/Global Dependencies (Sneak Peek)

You'll later learn how to attach dependencies to:
- **Router groups** (via `APIRouter`)
- **The entire app** (`app = FastAPI(dependencies=[...])`)

Useful for applying a dependency to:
- All admin routes
- All API routes under a version prefix
- Everything in the whole app

---

## 🔚 TL;DR

| Feature | Summary |
|--------|---------|
| **`dependencies=[]`** | Used in decorators to run dependencies without needing their return values |
| **Cleaner code** | No unused function parameters |
| **Still full power** | Can validate headers, raise exceptions, log, etc. |
| **Reuses logic** | Works with existing dependencies that do return values |
| **Great for** | Auth, side-effects, logging, preconditions |

---

