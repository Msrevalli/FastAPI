Here’s a **clear explanation** of how **dependency overrides** work in FastAPI for testing:

---

## ✅ Why Override Dependencies in Tests?

Sometimes, real dependencies:

* Make network/database calls 💸
* Are slow or flaky 😩
* Aren't needed for the test logic 🤷‍♂️

So during tests, you replace them with **mock or fake versions** that are:

* Fast ⚡
* Stable ✅
* Controlled 🧪

---

## 🧪 Example Use Case

### Suppose you have this real dependency:

```python
async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}
```

It's used like this:

```python
@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return {"message": "Hello Items!", "params": commons}
```

---

## 🧰 Override it in Tests

Instead of running the original dependency, we override it like this:

```python
async def override_dependency(q: str | None = None):
    return {"q": q, "skip": 5, "limit": 10}

app.dependency_overrides[common_parameters] = override_dependency
```

This tells FastAPI:

> “When you see `Depends(common_parameters)`, use `override_dependency` instead.”

---

## 🧪 Test Example

```python
def test_override_in_items():
    response = client.get("/items/")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Hello Items!",
        "params": {"q": None, "skip": 5, "limit": 10},
    }
```

Even though the request didn't pass `skip=5` or `limit=10`, the **override forced those values**, which makes it predictable for testing.

---

## 🔄 Resetting the Overrides

After your test, if you're running multiple tests or using pytest, **reset the overrides**:

```python
app.dependency_overrides = {}
```

Otherwise, it might interfere with other tests unintentionally.

---

## ✅ Summary Table

| Feature                         | Purpose                                           |
| ------------------------------- | ------------------------------------------------- |
| `app.dependency_overrides[...]` | Replaces the real dependency with a mock/fake one |
| Useful for                      | Skipping slow/external dependencies in test       |
| Overrides apply to              | Any route or nested dependency that uses it       |
| Reset with                      | `app.dependency_overrides = {}` after the test    |

---


You can override **dependencies from routers**, sub-apps, and even those injected via `Depends` in `APIRouter.include_router(...)`. The override mechanism is global within the app.

---

