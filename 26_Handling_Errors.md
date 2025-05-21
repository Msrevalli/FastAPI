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

## 🧪 Debugging: See What the Client Sent

Useful when you want to **see and return exactly what the user sent**:

```python
from fastapi.encoders import jsonable_encoder

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content=jsonable_encoder({
            "detail": exc.errors(),
            "body": exc.body
        }),
    )
```

---

## 🔁 Reuse Built-in Handlers (with logging)

```python
from fastapi.exception_handlers import (
    http_exception_handler,
    request_validation_exception_handler,
)

@app.exception_handler(StarletteHTTPException)
async def custom_http_exception_handler(request, exc):
    print(f"HTTP error: {repr(exc)}")
    return await http_exception_handler(request, exc)

@app.exception_handler(RequestValidationError)
async def custom_validation_exception_handler(request, exc):
    print(f"Validation error: {exc}")
    return await request_validation_exception_handler(request, exc)
```

---

## 🚨 FastAPI vs Starlette HTTPException

| Feature                         | `FastAPI.HTTPException` | `Starlette.HTTPException`      |
| ------------------------------- | ----------------------- | ------------------------------ |
| Can return JSON `detail`        | ✅ Yes                   | ❌ Only strings                 |
| You raise in your routes        | ✅                       | ❌ (not typical)                |
| You catch in exception handlers | ✅ (indirectly)          | ✅ (used by FastAPI internally) |

---


