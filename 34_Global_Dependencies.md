## 🌐 Global Dependencies

### ✅ Apply dependencies to every single route in your app

```python
from fastapi import Depends, FastAPI, Header, HTTPException
from typing_extensions import Annotated

# Global dependency 1
async def verify_token(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")

# Global dependency 2
async def verify_key(x_key: Annotated[str, Header()]):
    if x_key != "fake-super-secret-key":
        raise HTTPException(status_code=400, detail="X-Key header invalid")
    return x_key

# ✅ Declare app-wide dependencies here
app = FastAPI(
    dependencies=[
        Depends(verify_token),
        Depends(verify_key)
    ]
)

@app.get("/items/")
async def read_items():
    return [{"item": "Portal Gun"}, {"item": "Plumbus"}]

@app.get("/users/")
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]
```

### 🚀 Result:
- Every request to any route will **trigger both `verify_token` and `verify_key`**.
- Their return values are **not passed to the route handlers**, just enforced.

---

## ✅ Use Cases

- App-wide auth check
- Logging or request timing
- Maintenance mode checks
- Rate limiting or throttling
- Global feature flag gate

---

## ⚠️ Pro Tip

You can **combine global dependencies** with:
- Route-level dependencies (more specific checks)
- Sub-dependencies (deep trees of logic)

FastAPI solves them all efficiently and **caches results per request**, unless you explicitly disable caching (`use_cache=False`).

---


