**FastAPI’s most powerful dependency patterns** — `yield`-based dependencies. These are **perfect for setup/teardown scenarios**, like managing a database session or acquiring/releasing a resource.

Let’s break it down clearly:

---

## ✅ What’s a `yield`-based Dependency?

It's a dependency that:
- Does something **before** the request (e.g., open a DB session),
- Then **yields a value** to be used during the request,
- And finally **does cleanup after** the response is sent (e.g., close DB session).

### 🧠 Think of it like:
```python
# Setup
resource = setup_resource()
try:
    yield resource   # Use this in your route
finally:
    # Teardown
    release_resource(resource)
```

---

## 💡 Example: Database Session Dependency

```python
async def get_db():
    db = DBSession()  # Setup: create DB session
    try:
        yield db       # Provide it to the route
    finally:
        db.close()     # Teardown: close it
```

### 👇 Usage

```python
from fastapi import Depends, FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(db: Annotated[DBSession, Depends(get_db)]):
    return db.query_items()
```

FastAPI will:
1. Call `get_db()`, run code *up to* the `yield`
2. Inject the `db` into your route
3. Wait for the route to finish (or error)
4. Then run the code *after* `yield`

---

## 🛡️ Why `try`/`finally`?

Because `finally` ensures the cleanup runs even if:
- Your route raises an exception
- A downstream dependency fails
- A timeout happens

You can even add custom error handling inside an `except`:

```python
async def get_db():
    db = DBSession()
    try:
        yield db
    except SomeDatabaseError:
        db.rollback()
        raise
    finally:
        db.close()
```

---

## 🔥 Bonus: Use `async` or regular

Both of these are valid:

```python
# Sync
def get_thing():
    thing = connect()
    try:
        yield thing
    finally:
        thing.disconnect()

# Async
async def get_async_thing():
    thing = await async_connect()
    try:
        yield thing
    finally:
        await thing.aclose()
```

FastAPI handles both transparently!

---

## 🚀 Real-World Use Cases

- Database sessions (SQLAlchemy, Tortoise, etc.)
- Redis or cache connections
- File or stream handles
- External API client sessions
- Locks / Mutex management
- Request-level profiling or logging

---

```python
from fastapi import Depends, FastAPI
from typing import Generator

app = FastAPI()

# Simulated database session class
class FakeDBSession:
    def __init__(self):
        print("📦 Opening DB session...")

    def query_items(self):
        return ["Portal Gun", "Plumbus", "Microverse Battery"]

    def close(self):
        print("🧹 Closing DB session...")


# Dependency with yield for setup and teardown
def get_db() -> Generator[FakeDBSession, None, None]:
    db = FakeDBSession()  # Setup
    try:
        yield db           # Provide the session
    finally:
        db.close()         # Teardown


@app.get("/items/")
def read_items(db: FakeDBSession = Depends(get_db)):
    items = db.query_items()
    return {"items": items}
```

---

### 🔍 What Happens When You Call `/items/`

**Console Output:**

```
📦 Opening DB session...
🧹 Closing DB session...
```

**HTTP Response:**

```json
{
  "items": ["Portal Gun", "Plumbus", "Microverse Battery"]
}
```

---

### 🧠 Recap: How This Works

* FastAPI **calls `get_db()`** before entering your route handler.
* It **runs the code up to the `yield`**, and **injects the returned `db`** into your route.
* After the route finishes (success or error), FastAPI **resumes and completes the `finally` block**, ensuring cleanup.

---
**Complete FastAPI script** with a custom `yield`-based dependency that handles an internal exception (`InternalError`) and raises it properly:

---

### ✅ Full Script: Custom Exception in a `yield`-based Dependency

```python
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException

app = FastAPI()

# --- Custom Exception ---
class InternalError(Exception):
    pass

# --- Dependency that yields a username ---
def get_username():
    try:
        yield "Rick"
    except InternalError:
        print("⚠️ We don't swallow the internal error here, we raise again 😎")
        raise

# --- Endpoint using the dependency ---
@app.get("/items/{item_id}")
def get_item(item_id: str, username: Annotated[str, Depends(get_username)]):
    if item_id == "portal-gun":
        raise InternalError(
            f"❌ The portal gun is too dangerous to be owned by {username}"
        )
    if item_id != "plumbus":
        raise HTTPException(
            status_code=404, detail="🔍 Item not found, there's only a plumbus here"
        )
    return {"item_id": item_id, "owner": username}
```

---

### 🧪 Example Requests & Responses

#### ✅ GET `/items/plumbus`

```json
{
  "item_id": "plumbus",
  "owner": "Rick"
}
```

#### ❌ GET `/items/portal-gun`

Raises `InternalError`, which will cause a 500 Internal Server Error (default behavior unless a custom handler is added).

Console Output:

```
We don't swallow the internal error here, we raise again 😎
```

#### ❌ GET `/items/anything-else`

```json
{
  "detail": "🔍 Item not found, there's only a plumbus here"
}
```

---

### 🛠 Want to add a custom handler for `InternalError`?

Add this to your script:

```python
from fastapi.requests import Request
from fastapi.responses import JSONResponse

@app.exception_handler(InternalError)
async def internal_error_handler(request: Request, exc: InternalError):
    return JSONResponse(
        status_code=500,
        content={"message": str(exc)},
    )
```

A `yield`-based dependency in FastAPI that wraps a **custom context manager** class (`MySuperContextManager`) to manage a database session. Here's the full explanation, correction, and usage example:

---

## ✅ Full Example with Custom Context Manager Dependency

```python
from fastapi import Depends, FastAPI
from typing import Generator

app = FastAPI()


# 🧪 Fake DBSession class for demonstration
class DBSession:
    def __init__(self):
        print("📦 DB session started")

    def close(self):
        print("🧹 DB session closed")

    def query_items(self):
        return [{"item": "Plumbus"}, {"item": "Portal Gun"}]


# ✅ Custom context manager
class MySuperContextManager:
    def __init__(self):
        self.db = DBSession()

    def __enter__(self):
        return self.db

    def __exit__(self, exc_type, exc_value, traceback):
        self.db.close()


# ✅ Dependency using yield
def get_db() -> Generator[DBSession, None, None]:
    with MySuperContextManager() as db:
        yield db


# ✅ Endpoint using the dependency
@app.get("/items/")
def read_items(db: DBSession = Depends(get_db)):
    return db.query_items()
```

---

## 🔄 What Happens Internally

1. FastAPI calls `get_db()` before the request.
2. The `MySuperContextManager` starts a DB session (`__enter__`).
3. FastAPI injects the `db` into the route handler.
4. After the route finishes, FastAPI runs `__exit__`, closing the session.

---

## 🧪 Example Output (Console)

When calling `GET /items/`, you'll see:

```
📦 DB session started
🧹 DB session closed
```

And the JSON response:

```json
[
  {"item": "Plumbus"},
  {"item": "Portal Gun"}
]
```

---




