## ⚠️ When Should You Handle Errors?

You send an error when the client:

1. 🔐 **Is not allowed** to do something
   *E.g., trying to access admin-only data.*

2. 🔍 **Requests something that doesn’t exist**
   *E.g., `GET /items/999` but item 999 isn’t in your database.*

3. ❌ **Sends bad data**
   *E.g., they send a string when a number is expected.*

4. 📜 **Breaks your rules**
   *E.g., you reject all orders containing the number 3.*

---

## 🧱 Basic Error with `HTTPException`

FastAPI’s built-in way to raise standard HTTP errors.

### 🧪 Example

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}

@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```

### 🧾 Result:

* ✅ `/items/foo` → `200 OK`, returns item
* ❌ `/items/bar` → `404 Not Found`, returns `{ "detail": "Item not found" }`

---

## 🛠️ Add Custom Headers to the Error Response

You can attach headers to your error:

```python
raise HTTPException(
    status_code=404,
    detail="Item not found",
    headers={"X-Error": "There goes my error"},
)
```

🧾 The client receives:

```http
X-Error: There goes my error
```

---

## 🦄 Custom Exception Class

You can define your **own error types** for unique cases.

```python
class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name
```

Then, tell FastAPI how to handle that error:

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse


# ✅ Define a custom exception
class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name


app = FastAPI()


# 🧠 Custom exception handler for UnicornException
@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )


# 🦄 Route that may raise the custom exception
@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```

### How it works:

* Access `/unicorns/yolo` → triggers the custom error.
* Access `/unicorns/sparkle` → returns the unicorn name normally.

---

## 🧼 Customizing FastAPI’s Built-in Error Handlers

FastAPI auto-validates data. If a request fails validation, it raises:

* `RequestValidationError`
* `StarletteHTTPException`

You can catch and customize those:

### 🔧 Validation Errors (like bad query/body/path data)

```python
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)
```

### 🔧 All HTTP Errors

```python
from starlette.exceptions import HTTPException as StarletteHTTPException

@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)
```

---

```python
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()


# 🛠️ Handle standard HTTP exceptions
@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)


# 🧼 Handle validation errors (e.g., invalid query/path/body data)
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)


# 🔍 Route that raises a custom error when item_id is 3
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

### Behavior:

* ✅ `/items/1` → returns `{"item_id": 1}`
* ❌ `/items/foo` → triggers `RequestValidationError`, returns plain text
* ❌ `/items/3` → triggers custom 418 error with message `"Nope! I don't like 3."`

Your FastAPI script is clean and handles validation errors in a customized way. Here's the breakdown:

---

```python
from fastapi import FastAPI, Request, status
from fastapi.encoders import jsonable_encoder
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel

app = FastAPI()

# 🧼 Custom validation error handler
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content=jsonable_encoder({
            "detail": exc.errors(),
            "body": exc.body
        }),
    )

# 📦 Pydantic model for input
class Item(BaseModel):
    title: str
    size: int

# 📥 POST endpoint
@app.post("/items/")
async def create_item(item: Item):
    return item
```

---

### 🔍 How it Works:

* **`Item` model** requires:

  * `title`: `str`
  * `size`: `int`

* When a client sends invalid data (e.g., wrong types or missing fields), your custom handler returns a JSON response showing:

  * Detailed error messages (`exc.errors()`)
  * The full body the client tried to send (`exc.body`)

---

### ✅ Example Success Request:

**Request:**

```json
POST /items/
{
  "title": "T-Shirt",
  "size": 42
}
```

**Response:**

```json
{
  "title": "T-Shirt",
  "size": 42
}
```

---

### ❌ Example Error Request:

**Request:**

```json
POST /items/
{
  "title": 123,
  "size": "big"
}
```

**Response:**

```json
{
  "detail": [
    {
      "loc": ["body", "title"],
      "msg": "str type expected",
      "type": "type_error.str"
    },
    {
      "loc": ["body", "size"],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ],
  "body": {
    "title": 123,
    "size": "big"
  }
}
```
---
```python
from fastapi import FastAPI, HTTPException
from fastapi.exception_handlers import (
    http_exception_handler,
    request_validation_exception_handler,
)
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

# 🔁 Custom HTTP error handler (reuses FastAPI's built-in handler with extra logging)
@app.exception_handler(StarletteHTTPException)
async def custom_http_exception_handler(request, exc):
    print(f"OMG! An HTTP error!: {repr(exc)}")
    return await http_exception_handler(request, exc)

# 🔁 Custom validation error handler (also logs before reusing FastAPI handler)
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    print(f"OMG! The client sent invalid data!: {exc}")
    return await request_validation_exception_handler(request, exc)

# 🧪 Simple route to trigger errors
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

---

### 🔍 Behavior

* `/items/2` → ✅ Returns `{ "item_id": 2 }`
* `/items/abc` → ❌ Triggers `RequestValidationError` (logs + returns 422)
* `/items/3` → ❌ Triggers `HTTPException` with 418 "I'm a teapot" (logs + returns 418)

---

### 🧠 Key Concepts

| Feature                                      | Purpose                                                                |
| -------------------------------------------- | ---------------------------------------------------------------------- |
| `@exception_handler(StarletteHTTPException)` | Handles **all HTTP errors** (like 404, 418) and logs them              |
| `@exception_handler(RequestValidationError)` | Handles **invalid input** (e.g., wrong data types, missing fields)     |
| `await http_exception_handler(...)`          | Delegates actual formatting to FastAPI's default behavior              |
| `print(...)`                                 | Adds custom logging — helpful for debugging or logging to a file later |

---

## 🚨 FastAPI vs Starlette HTTPException

| Feature                         | `FastAPI.HTTPException` | `Starlette.HTTPException`      |
| ------------------------------- | ----------------------- | ------------------------------ |
| Can return JSON `detail`        | ✅ Yes                   | ❌ Only strings                 |
| You raise in your routes        | ✅                       | ❌ (not typical)                |
| You catch in exception handlers | ✅ (indirectly)          | ✅ (used by FastAPI internally) |

---


