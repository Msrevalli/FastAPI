Great — you’re now learning about **Sub Applications** (a.k.a. **Mounts**) in FastAPI, a powerful feature for building **modular** and **scalable** APIs. Here’s a breakdown of how it works and when you might want to use it:

---

## ✅ What Is a Mounted Sub-Application?

Mounting a sub-application means:

> "Embedding" an entire, **independent FastAPI app** under a specific path of a larger **main app**.

Each app:

* Has its **own routes**
* Has its **own docs (Swagger & ReDoc)**
* Can have its own **middleware**, **dependencies**, and **exceptions**

---

## 🧱 Use Case Example

Imagine you’re building:

* `Main App`: `/app`, admin panel
* `Sub App`: `/subapi`, internal API or third-party service integration

---

## 🧪 Code Example

```python
# main.py
from fastapi import FastAPI

# Main app
app = FastAPI()

@app.get("/app")
def read_main():
    return {"message": "Hello World from main app"}

# Sub app
subapi = FastAPI()

@subapi.get("/sub")
def read_sub():
    return {"message": "Hello World from sub API"}

# Mount sub app at /subapi
app.mount("/subapi", subapi)
```

---

## 🔍 What Happens Internally?

When you mount `subapi` at `/subapi`, FastAPI ensures:

* All sub-routes are **prefixed** correctly (e.g. `/subapi/sub`)
* The sub-app's documentation and OpenAPI schema are available at `/subapi/docs`, `/subapi/openapi.json`
* The ASGI `root_path` is set for correct routing

---

## 🌐 Available Routes

| Path            | App Type | Description         |
| --------------- | -------- | ------------------- |
| `/app`          | Main App | Hello from main app |
| `/docs`         | Main App | Main app Swagger UI |
| `/subapi/sub`   | Sub App  | Hello from sub API  |
| `/subapi/docs`  | Sub App  | Sub app Swagger UI  |
| `/subapi/redoc` | Sub App  | Sub app ReDoc UI    |

---

## 🧠 Tips & Best Practices

### ✅ Modular Design

Organize each sub-application in its own Python module or package:

```plaintext
.
├── main.py
├── admin/
│   └── app.py  # FastAPI instance
├── api/
│   └── app.py
```

In `main.py`:

```python
from admin.app import app as admin_app
from api.app import app as api_app

app.mount("/admin", admin_app)
app.mount("/api", api_app)
```

### ✅ Independent Configuration

Each sub-app can have:

* Its own `dependencies`
* Custom `middleware`
* Its own error handlers

### ⚠️ Middleware Scope

Middleware added to `main_app` does **not** affect `sub_app` (and vice versa).

---

## 🚀 Summary

| Feature                     | Main App  | Sub App          |
| --------------------------- | --------- | ---------------- |
| Independent routing         | ✅         | ✅                |
| Own docs UI                 | ✅ `/docs` | ✅ `/subapi/docs` |
| Own middleware/dependencies | ✅         | ✅                |
| Shares server               | ✅         | ✅ (via mount)    |

---

