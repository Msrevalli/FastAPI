**structuring larger FastAPI applications**

## ✅ Project Structure Overview

```
.
├── app
│   ├── __init__.py
│   ├── main.py               # Entry point
│   ├── dependencies.py       # Shared dependencies
│   ├── routers/              # Modular routers (like Flask Blueprints)
│   │   ├── __init__.py
│   │   ├── users.py
│   │   └── items.py
│   └── internal/
│       ├── __init__.py
│       └── admin.py
```

Each directory with `__init__.py` is a **package** or **subpackage**, enabling import paths like:

```python
from app.routers import users
from app.internal import admin
```

---

## 🧩 `APIRouter`: Modular Routing

Instead of writing all routes in `main.py`, we create routers like this:

### `users.py` example

```python
# app/routers/users.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]

@router.get("/users/me", tags=["users"])
async def read_user_me():
    return {"username": "fakecurrentuser"}

@router.get("/users/{username}", tags=["users"])
async def read_user(username: str):
    return {"username": username}
```

> Think of this `router` as a mini-app. You'll later "plug" it into the main app.

---

## 🧷 Dependencies in a separate file

You might want to apply logic like **auth headers**, **tokens**, etc., across many routes:

### `dependencies.py`

```python
# app/dependencies.py
from typing import Annotated
from fastapi import Header, HTTPException

async def get_token_header(x_token: Annotated[str, Header()]):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")
```

---

## 🔁 Smart usage of `APIRouter` with common settings

### `items.py`

```python
# app/routers/items.py
from fastapi import APIRouter, Depends, HTTPException
from ..dependencies import get_token_header  # Relative import

router = APIRouter(
    prefix="/items",
    tags=["items"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)

fake_items_db = {"plumbus": {"name": "Plumbus"}, "gun": {"name": "Portal Gun"}}

@router.get("/")
async def read_items():
    return fake_items_db

@router.get("/{item_id}")
async def read_item(item_id: str):
    if item_id not in fake_items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"name": fake_items_db[item_id]["name"], "item_id": item_id}

@router.put("/{item_id}", tags=["custom"], responses={403: {"description": "Operation forbidden"}})
async def update_item(item_id: str):
    if item_id != "plumbus":
        raise HTTPException(status_code=403, detail="You can only update the item: plumbus")
    return {"item_id": item_id, "name": "The great Plumbus"}
```

> By defining `prefix`, `tags`, `dependencies`, and `responses` on the router, you avoid repetition in each route.

---

## 🔌 Putting it all together in `main.py`

```python
# app/main.py
from fastapi import FastAPI
from app.routers import users, items

app = FastAPI()

app.include_router(users.router)
app.include_router(items.router)
```

You now have a modular app where:
- Routes are logically separated.
- Dependencies are reusable.
- Docs are clean thanks to tags and response descriptions.

---
### ✅ **Relative Imports (the `.` and `..` syntax)**
- `from .routers import items, users`:  
  Starts from the current package (`app/`) and looks in the `routers/` subpackage.
- `from ..dependencies import get_token_header`:  
  Goes one level **up** from `app/routers/` to `app/` and grabs `dependencies.py`.
- `from ...dependencies import get_token_header`:  
  Would try to go **two levels up**—but if `app/` is your top-level package, you’ll get an import error.

So just like navigating folders in a terminal:
- `.` = current directory
- `..` = parent
- `...` = grandparent (not commonly used unless you're really nested)

---

### ✅ **How `APIRouter` Works**
- You define routers in separate modules (`items`, `users`, `admin`) to keep your app modular.
- Then, in `main.py`, you bring them all together with `app.include_router(...)`.
- You can customize:
  - `prefix`: All routes in this router get this prefix.
  - `tags`: For Swagger UI grouping.
  - `dependencies`: Automatically applied to every route in the router.
  - `responses`: Default documented responses (e.g., for 404s).

---

### ✅ **Avoiding Router Collisions**
You're importing the modules (`items`, `users`) instead of directly importing `router` from each to avoid overwriting:
```python
# GOOD
from .routers import items, users
app.include_router(items.router)
app.include_router(users.router)

# BAD (conflicts if done twice)
from .routers.items import router
from .routers.users import router
```

---

### ✅ **Global vs Local Dependencies**
- `app = FastAPI(dependencies=[Depends(get_query_token)])`:  
  This gets applied **globally**, to **all** routes.
- `APIRouter(..., dependencies=[Depends(get_token_header)])`:  
  Applies to just the routes inside that router.

---

### ✅ **Including APIRouter in Another APIRouter**
You can nest routers just like this:
```python
api_router = APIRouter()
api_router.include_router(users.router)
api_router.include_router(items.router)
app.include_router(api_router, prefix="/api")
```
Now everything under `/users` and `/items` becomes available under `/api/users`, `/api/items`, etc.

---

### ✅ **Same Router, Different Prefixes**
Totally valid:
```python
app.include_router(items.router, prefix="/v1/items")
app.include_router(items.router, prefix="/latest/items")
```
That way you can maintain backward compatibility while releasing new versions.

---
