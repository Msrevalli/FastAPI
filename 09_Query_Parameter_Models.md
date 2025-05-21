## 🔹 **What Are Query Parameter Models?**

Instead of writing many separate query parameters, you can **group them into a single Pydantic model**, making your code:

* Cleaner ✅
* Easier to reuse ✅
* Easier to validate ✅

---

## ✅ **Example 1: Grouping Query Parameters with a Pydantic Model**

```python
from typing import Annotated, Literal
from fastapi import FastAPI, Query
from pydantic import BaseModel, Field

app = FastAPI()

class FilterParams(BaseModel):
    limit: int = Field(100, gt=0, le=100)  # Default: 100, must be between 1 and 100
    offset: int = Field(0, ge=0)           # Default: 0, must be ≥ 0
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []                   # Optional list of tags (multiple ?tags=...)

@app.get("/items/")
async def read_items(filter_query: Annotated[FilterParams, Query()]):
    return filter_query
```

### 🧪 Try URLs Like:

```
GET /items/?limit=10&offset=5&order_by=updated_at&tags=books&tags=tech
```

### ✅ Response:

```json
{
  "limit": 10,
  "offset": 5,
  "order_by": "updated_at",
  "tags": ["books", "tech"]
}
```

---

## 🔐 **Example 2: Forbidding Extra Query Parameters**

If you want to **strictly control allowed query parameters**, use Pydantic’s `model_config = {"extra": "forbid"}`.

```python
class FilterParams(BaseModel):
    model_config = {"extra": "forbid"}  # 🚫 Disallow any unexpected fields

    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []

@app.get("/items/")
async def read_items(filter_query: Annotated[FilterParams, Query()]):
    return filter_query
```

### ❌ Invalid Request:

```
GET /items/?limit=10&tool=plumbus
```

### ❌ Response:

```json
{
  "detail": [
    {
      "type": "extra_forbidden",
      "loc": ["query", "tool"],
      "msg": "Extra inputs are not permitted",
      "input": "plumbus"
    }
  ]
}
```

---

## ✨ Benefits

| Feature         | Benefit                                             |
| --------------- | --------------------------------------------------- |
| ✅ Clean syntax  | Groups related query parameters into one model      |
| ✅ Reusability   | Use the same model in multiple endpoints            |
| ✅ Validation    | Automatically validates range, allowed values, etc. |
| ✅ Documentation | Auto-generates clean `/docs` UI                     |
| ✅ Security      | Optionally forbid unexpected query parameters       |

---

## 🧠 Bonus: Reusing Models Across Endpoints

You can use `FilterParams` in multiple endpoints:

```python
@app.get("/products/")
async def read_products(filter_query: Annotated[FilterParams, Query()]):
    return filter_query

@app.get("/articles/")
async def read_articles(filter_query: Annotated[FilterParams, Query()]):
    return filter_query
```

---


