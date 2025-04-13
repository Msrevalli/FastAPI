## 🧠 What is `Field`?

- `Field()` is used **inside Pydantic models** to:
  - Add **validation rules** (e.g., `gt=0`)
  - Add **metadata** (e.g., `title=`, `description=`)
- Think of it like `Query()`, `Path()`, or `Body()` — but for **model fields**.

---

## 🧾 How to Use `Field`

```python
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str
    description: str | None = Field(
        default=None,
        title="The description of the item",
        max_length=300
    )
    price: float = Field(
        gt=0,
        description="The price must be greater than zero"
    )
    tax: float | None = None
```

### 🔍 What’s going on:
- `name: str`: no extra validation or metadata
- `description`: optional (`None`), with a title and a max length
- `price`: must be greater than 0 (with a helpful description)
- `tax`: optional float, no extra rules

---

## 🧪 Example in an API Endpoint

```python
from typing import Annotated
from fastapi import FastAPI, Body
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = Field(default=None, title="The description of the item", max_length=300)
    price: float = Field(gt=0, description="The price must be greater than zero")
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    return {"item_id": item_id, "item": item}
```

### Request body expected:

```json
{
  "item": {
    "name": "Phone",
    "description": "A cool smartphone",
    "price": 699.99,
    "tax": 50.00
  }
}
```

---

## ⚠️ Important Notes

| Concept                          | Explanation |
|----------------------------------|-------------|
| `Field` comes from `pydantic`    | Not from `fastapi` |
| Used **inside models**           | While `Query`, `Path`, etc. are used in endpoint **parameters** |
| Works just like `Query()`        | Has `title`, `description`, `gt`, `lt`, etc. |
| Shows up in the docs             | FastAPI generates API docs based on this info |

---

## 🧬 Behind the Scenes

- `Field`, `Query`, `Path`, and `Body` are all based on the same core idea: **extra validation and metadata for parameters.**
- They all return subclasses of **Pydantic’s `FieldInfo`**.
- FastAPI uses that info to:
  - Validate inputs
  - Generate OpenAPI docs
  - Give helpful error messages

---

## 🧠 TL;DR Summary

| Feature          | Used For                             | Example                          |
|------------------|--------------------------------------|----------------------------------|
| `Field()`        | Inside models                        | `price: float = Field(gt=0)`     |
| `Query()`        | Query parameters                     | `q: str = Query(max_length=50)`  |
| `Path()`         | Path parameters                      | `id: int = Path(gt=0)`           |
| `Body()`         | Request body (non-model) params      | `importance: int = Body(gt=1)`   |

---
