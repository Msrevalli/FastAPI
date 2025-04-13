## 🚀 What Are Body Parameters?

When you send **JSON data in a request** (typically in POST or PUT requests), that data is called the **request body**.

In FastAPI, you can declare **Pydantic models** to handle and validate this data.

---

## 🧩 Mixing Path, Query, and Body Parameters

You can combine them however you like:

### ✅ Example:

```python
from typing import Annotated
from fastapi import FastAPI, Path
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=0, le=1000)],
    q: str | None = None,
    item: Item | None = None,  # 👈 Optional request body
):
    results = {"item_id": item_id}
    if q:
        results["q"] = q
    if item:
        results["item"] = item
    return results
```

📝 This mixes:
- `item_id`: from path
- `q`: from query
- `item`: from request body (optional)

---

## 👥 Multiple Body Parameters

If you need to send **multiple chunks of JSON**, just define multiple models.

### ✅ Example:

```python
class User(BaseModel):
    username: str
    full_name: str | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    return {"item_id": item_id, "item": item, "user": user}
```

### Body expected:

```json
{
  "item": {
    "name": "Foo",
    "description": "Bar",
    "price": 50.0,
    "tax": 5.0
  },
  "user": {
    "username": "alice",
    "full_name": "Alice Smith"
  }
}
```

🧠 FastAPI automatically **matches each model to its part** of the JSON!

---

## 🔢 Body with Singular Values (Not a Model)

If you want to include a simple value (like a number or string) in the body, use `Body()`:

```python
from fastapi import Body

@app.put("/items/{item_id}")
async def update_item(
    item_id: int,
    item: Item,
    user: User,
    importance: Annotated[int, Body(gt=0)],  # 👈 This is a body value, not query
):
    return {
        "item_id": item_id,
        "item": item,
        "user": user,
        "importance": importance
    }
```

### Body expected:

```json
{
  "item": { ... },
  "user": { ... },
  "importance": 5
}
```

🔎 If you don’t use `Body()`, FastAPI assumes simple values come from the **query string**, not the body.

---

## ❓ Add Extra Query Parameters Too

You can still include query parameters alongside body data:

```python
@app.put("/items/{item_id}")
async def update_item(
    item_id: int,
    item: Item,
    user: User,
    importance: Annotated[int, Body(gt=0)],
    q: str | None = None,  # 👈 query parameter
):
    return {"q": q, "item_id": item_id, "item": item, "user": user, "importance": importance}
```

---

## 📦 Embedding a Single Body Parameter

If you want to wrap a **single body model** under a key (like `"item": {...}`), use `Body(embed=True)`:

```python
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    return {"item_id": item_id, "item": item}
```

### Body expected:

```json
{
  "item": {
    "name": "Foo",
    "description": "Bar",
    "price": 50.0,
    "tax": 5.0
  }
}
```

🧠 Without `embed=True`, it would expect just the object:

```json
{
  "name": "Foo",
  "description": "Bar",
  "price": 50.0,
  "tax": 5.0
}
```

---

## ✅ Recap

| Concept                        | Summary |
|-------------------------------|---------|
| Combine query/path/body       | FastAPI handles all at once |
| Optional body parameter       | Use `item: Item | None = None` |
| Multiple body models          | FastAPI expects a JSON with each model under a key |
| Simple values in body         | Use `Body()` |
| Embed a model in body         | Use `Body(embed=True)` |
| Add query alongside body      | Just declare normal params like `q: str | None = None` |

---

