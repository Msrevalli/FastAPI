---

## ⚠️ When to Handle Errors
You’ll want to send an error response to the client when:
- 🔐 They don’t have permission
- 🔍 They ask for something that doesn’t exist
- ❌ They send invalid data
- 📜 You want to enforce business logic (e.g. “I don’t like 3” 😄)

---

## 🧱 Basic Error: `HTTPException`

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

### 🧾 Response
- ✅ `/items/foo` → 200 + item data
- ❌ `/items/bar` → 404 + `{ "detail": "Item not found" }`

---

## 🛠️ Add Custom Headers to Error

```python
raise HTTPException(
    status_code=404,
    detail="Item not found",
    headers={"X-Error": "There goes my error"},
)
```

---

## 🦄 Custom Exception Handling

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name

app = FastAPI()

@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )

@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```

---

## 🧼 Customizing FastAPI’s Built-in Error Handlers

### Handle invalid path/query/body (`RequestValidationError`)

```python
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)
```

### Handle all HTTP errors (`StarletteHTTPException`)

```python
from starlette.exceptions import HTTPException as StarletteHTTPException

@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)
```

---

## 🧪 Bonus: See What the Client Sent (for debugging)

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

## 🔁 Reuse FastAPI's Built-In Handlers

```python
from fastapi.exception_handlers import (
    http_exception_handler,
    request_validation_exception_handler,
)

@app.exception_handler(StarletteHTTPException)
async def custom_http_exception_handler(request, exc):
    print(f"OMG! An HTTP error!: {repr(exc)}")
    return await http_exception_handler(request, exc)

@app.exception_handler(RequestValidationError)
async def custom_validation_exception_handler(request, exc):
    print(f"OMG! Client sent bad data: {exc}")
    return await request_validation_exception_handler(request, exc)
```

---

## 🚨 `FastAPI.HTTPException` vs `Starlette.HTTPException`

| Feature                      | FastAPI version                          | Starlette version                         |
|-----------------------------|------------------------------------------|-------------------------------------------|
| Accepts JSON for `detail`   | ✅ Yes                                    | ❌ Only strings                            |
| Raise this in your code     | ✅ Use FastAPI’s `HTTPException`         |                                           |
| Catch with custom handler   | ✅ Use `StarletteHTTPException` in `@app.exception_handler(...)` |

---

