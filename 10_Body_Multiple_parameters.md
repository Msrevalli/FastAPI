## 🚀 What Are Body Parameters?

When you send **JSON data in a request** (typically in POST or PUT requests), that data is called the **request body**.

In FastAPI, you can declare **Pydantic models** to handle and validate this data.

---

**FastAPI's request body handling** 

---

### ✅ 1. **Mix Path, Query and Body Parameters**

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
    item: Item | None = None,
):
    results = {"item_id": item_id}
    if q:
        results["q"] = q
    if item:
        results["item"] = item
    return results
```

**Request body (optional):**

```json
{
  "name": "Shoes",
  "price": 59.99
}
```

---

### ✅ 2. **Multiple Body Parameters**

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

class User(BaseModel):
    username: str

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    return {"item_id": item_id, "item": item, "user": user}
```

**Request body:**

```json
{
  "item": {
    "name": "Guitar",
    "price": 699.99
  },
  "user": {
    "username": "dave"
  }
}
```

---

### ✅ 3. **Singular Values in Body (Using `Body()`)**

```python
from typing import Annotated
from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

class User(BaseModel):
    username: str

@app.put("/items/{item_id}")
async def update_item(
    item_id: int,
    item: Item,
    user: User,
    importance: Annotated[int, Body()]
):
    return {"item_id": item_id, "item": item, "user": user, "importance": importance}
```

**Request body:**

```json
{
  "item": {
    "name": "Mic",
    "price": 120.0
  },
  "user": {
    "username": "alice"
  },
  "importance": 10
}
```

---

### ✅ 4. **Multiple Body Params and Query**

```python
from typing import Annotated
from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

class User(BaseModel):
    username: str

@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int,
    item: Item,
    user: User,
    importance: Annotated[int, Body(gt=0)],
    q: str | None = None,
):
    result = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    if q:
        result["q"] = q
    return result
```

**Request body:**

```json
{
  "item": {
    "name": "Amp",
    "price": 300.0
  },
  "user": {
    "username": "bob"
  },
  "importance": 2
}
```

**Query parameter (optional):** `?q=loud`

---

### ✅ 5. **Embed a Single Body Parameter**

```python
from typing import Annotated
from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    return {"item_id": item_id, "item": item}
```

**Request body:**

```json
{
  "item": {
    "name": "Drumsticks",
    "price": 15.0
  }
}
```

Without `embed=True`, it would expect just the `Item` fields at the top level.

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

