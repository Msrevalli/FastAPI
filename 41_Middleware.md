**FastAPI middleware**, which is a powerful way to customize the **behavior of every request and response** in your app. Let me break it down for you clearly:

---

## 💡 What is Middleware?

Middleware is like a “wrapper” around your request/response cycle.

It lets you:

- Run code **before** any endpoint is called
- Run code **after** the response is generated
- Modify the request or response
- Log, validate, inject headers, measure time, etc.

---

## 🧱 Structure of Middleware

Here’s the basic structure:

```python
@app.middleware("http")
async def my_middleware(request: Request, call_next):
    # 👈 Code before the request hits your endpoint
    response = await call_next(request)
    # 👈 Code after your endpoint returns a response
    return response
```

- `request`: The incoming HTTP request
- `call_next(request)`: Passes the request down the chain (to your endpoint)
- `response`: The result returned from your endpoint

---

## ⏱ Example: Measuring Process Time

This middleware adds a custom header `X-Process-Time` to each response:

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.perf_counter()         # start the timer
    response = await call_next(request)      # call the route handler
    process_time = time.perf_counter() - start_time  # calculate duration
    response.headers["X-Process-Time"] = str(process_time)  # add custom header
    return response
```

Now every response from your API will include:

```
X-Process-Time: 0.00156
```

---

## 🧪 Why Use Middleware?

Some common use cases:

| Use Case                          | Middleware? |
|----------------------------------|-------------|
| Logging request/response times   | ✅          |
| Adding custom headers            | ✅          |
| Global authentication check      | ✅ (but usually use dependencies) |
| CORS, GZip, Sessions             | ✅          |
| Database session management      | ✅          |

---

## ⚠️ Notes and Tips

- **Middleware runs before and after** the endpoint logic.
- **`call_next()` is async**, and it must be awaited.
- Use `time.perf_counter()` instead of `time.time()` for better precision.
- If you use **dependencies with `yield`**, their exit code runs **after** the middleware.
- If you use **background tasks**, they run **after** middleware finishes.

---

