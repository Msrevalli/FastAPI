Here’s a clear breakdown of how to **test FastAPI apps using asynchronous test functions** with `httpx.AsyncClient` and `pytest`:

---

## 🚀 Why Async Tests?

FastAPI is **async by nature**, and you may:

* Use async database clients like `asyncpg`, `Databases`, `SQLModel`, etc.
* Want to test async business logic
* Need to await functions like `await db.fetch(...)`

So, writing **async test functions** makes sense.

---

## 🧪 The Problem with `TestClient`

`TestClient` is **synchronous**, so it:

* Can’t be used inside `async def test_...()`
* Will raise runtime errors if you try

---

## ✅ Solution: Use `httpx.AsyncClient` with `ASGITransport`

### Example file structure:

```
.
├── app/
│   ├── main.py
│   └── test_main.py
```

---

## 📄 `main.py`

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Tomato"}
```

---

## 📄 `test_main.py`

```python
import pytest
from httpx import AsyncClient, ASGITransport
from app.main import app  # adjust import if needed

@pytest.mark.anyio  # 🔁 tells pytest to run this as async
async def test_root():
    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test"
    ) as client:
        response = await client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Tomato"}
```

---

## 🏃 Run Tests

```bash
pytest
```

And pytest will handle async correctly thanks to `@pytest.mark.anyio`.

---

## 🧠 Key Concepts

| Concept                  | Explanation                                                  |
| ------------------------ | ------------------------------------------------------------ |
| `@pytest.mark.anyio`     | Tells pytest to allow async test functions                   |
| `AsyncClient`            | Like `TestClient` but async — used to send requests          |
| `ASGITransport(app=app)` | Makes the client send requests to your FastAPI app in memory |
| `base_url="http://test"` | Required placeholder; not actually used for networking       |
| `.get(...)`              | Works the same as `requests.get`, just `await`ed             |

---

## ⚠️ Lifespan Events (Startup/Shutdown)

If your app uses `@app.on_event("startup")` or FastAPI 0.95+ `lifespan=...` handlers:

* **Problem**: `AsyncClient` alone won’t trigger these
* **Solution**: Use [`asgi-lifespan`](https://github.com/florimondmanca/asgi-lifespan)

```bash
pip install asgi-lifespan
```

### Then wrap your app:

```python
from asgi_lifespan import LifespanManager

@pytest.mark.anyio
async def test_root():
    async with LifespanManager(app):
        async with AsyncClient(
            transport=ASGITransport(app=app), base_url="http://test"
        ) as client:
            response = await client.get("/")
        assert response.json() == {"message": "Tomato"}
```

---

## 🧪 When to Use Async Tests

Use async tests when you:

* Use async DBs (e.g., SQLAlchemy 2.0 async, Tortoise ORM, etc.)
* Call other async dependencies or services in tests
* Want more control over concurrency and context

---


