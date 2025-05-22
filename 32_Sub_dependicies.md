## sub-dependencies ##

## 🌱 What's a Sub-dependency?

A **sub-dependency** is a dependency that itself **depends on other dependencies**.

Think of it like a tree:
```
read_query (route)
└── query_or_cookie_extractor (depends on...)
    ├── query_extractor (depends on query parameter)
    └── Cookie (last_query)
```

FastAPI **resolves the entire tree automatically** and passes the correct values where they’re needed.

---

Here's your complete FastAPI script using `Depends`, `Annotated`, and a fallback from query to cookie:

```python
from typing import Annotated
from fastapi import Cookie, Depends, FastAPI

app = FastAPI()


def query_extractor(q: str | None = None):
    return q


def query_or_cookie_extractor(
    q: Annotated[str, Depends(query_extractor)],
    last_query: Annotated[str | None, Cookie()] = None,
):
    if not q:
        return last_query
    return q


@app.get("/items/")
async def read_query(
    query_or_default: Annotated[str, Depends(query_or_cookie_extractor)],
):
    return {"q_or_cookie": query_or_default}
```

This script allows `/items/` to return either the query parameter `q` or the cookie value `last_query` if `q` is not provided.

Your script is a great example of **chained dependencies** in FastAPI, combining **query parameters** and **cookies** to determine a value.

---

## ✅ What This Script Does

### Goal:

When the client calls `/items/`, return:

* The query parameter `q` if it exists
* Otherwise, fallback to the cookie `last_query`

---

## 🔍 Breakdown

### 1. **Dependency 1: `query_extractor`**

```python
def query_extractor(q: str | None = None):
    return q
```

This grabs `q` from the query string (e.g., `/items/?q=test`).

---

### 2. **Dependency 2: `query_or_cookie_extractor`**

```python
def query_or_cookie_extractor(
    q: Annotated[str, Depends(query_extractor)],
    last_query: Annotated[str | None, Cookie()] = None,
):
    if not q:
        return last_query
    return q
```

* Uses `Depends(query_extractor)` to try and get `q` from the query.
* If `q` is not present, it checks the `last_query` cookie.

---

### 3. **Route Handler**

```python
@app.get("/items/")
async def read_query(
    query_or_default: Annotated[str, Depends(query_or_cookie_extractor)],
):
    return {"q_or_cookie": query_or_default}
```

This endpoint returns a value depending on query or cookie.

---

## 🌐 Example Requests

### ✅ If `q` is in the query:

```
GET /items/?q=hello
```

**Response:**

```json
{"q_or_cookie": "hello"}
```

---

### ✅ If `q` is not present but cookie is:

```
GET /items/
Cookie: last_query=remembered
```

**Response:**

```json
{"q_or_cookie": "remembered"}
```

---

### ✅ If neither is present:

**Response:**

```json
{"q_or_cookie": null}
```

---

## 🔄 Summary

| Source                      | Value Returned      |
| --------------------------- | ------------------- |
| `q` present                 | Query param `q`     |
| `q` missing, cookie present | Cookie `last_query` |
| Both missing                | `null`              |

---

## 🧠 Key Concepts Recap

### ✅ Dependencies can **depend on other dependencies**.

- Nest as deep as you want.
- FastAPI handles it.

### ✅ Caching
If the **same dependency is used multiple times**, FastAPI **caches** the return value for that request.

```python
Depends(some_dependency)
Depends(some_dependency)  # not called twice
```

But if you want **fresh values**, you can override this:

```python
Depends(some_dependency, use_cache=False)
```

Useful in:
- Auditing/logging
- DB sessions that need to return new objects
- Random values or timestamps

---

## 🧠 When is this *really* useful?

Right now the example is simple, but here are **real-world scenarios**:

### 🔐 Authentication / Security

```python
def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    ...

def get_admin_user(user: Annotated[User, Depends(get_current_user)]):
    if not user.is_admin:
        raise HTTPException(...)
    return user

@app.get("/admin/")
def admin_dashboard(admin: Annotated[User, Depends(get_admin_user)]):
    ...
```

### 💾 Database Access

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

def get_user_repo(db: Annotated[Session, Depends(get_db)]):
    return UserRepository(db)

@app.get("/users/{user_id}")
def read_user(user_repo: Annotated[UserRepository, Depends(get_user_repo)]):
    return user_repo.get_by_id(user_id)
```

---

## 🌳 Visual Example of Dependency Tree

For:

```python
@app.get("/items/")
async def read_query(query_or_default: Annotated[str, Depends(query_or_cookie_extractor)]):
    ...
```

Dependency graph is:

```
read_query
└── query_or_cookie_extractor
    ├── query_extractor
    └── Cookie (last_query)
```

---

## ✅ Summary

| Concept | Description |
|--------|-------------|
| **Sub-dependency** | A dependency that depends on another dependency |
| **Resolution** | FastAPI auto-resolves dependencies recursively |
| **Caching** | Dependencies are called only once per request by default |
| **use_cache=False** | Forces FastAPI to call the dependency again |
| **Power** | Useful for authentication, DB access, shared logic, etc. |

---

