## 🧠 What is this about?

Sometimes, you have **a bunch of query parameters** that all go together. Instead of listing them one by one, you can **group them into a single Pydantic model**.

This:
```python
@app.get("/items/")
async def read_items(limit: int = 100, offset: int = 0, tags: list[str] = []):
    ...
```

Becomes this:
```python
@app.get("/items/")
async def read_items(filter_query: FilterParams):
    ...
```

---

## ✅ Step-by-Step Guide

### Step 1: Define a Pydantic Model for Your Query Parameters

```python
from pydantic import BaseModel, Field
from typing import Literal

class FilterParams(BaseModel):
    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []
```

### What’s happening here?

- `limit`: default is `100`, must be between 1 and 100
- `offset`: default is `0`, must be ≥ 0
- `order_by`: must be `"created_at"` or `"updated_at"`
- `tags`: optional list of strings

---

### Step 2: Use That Model in Your Endpoint

```python
from fastapi import FastAPI, Query
from typing import Annotated

app = FastAPI()

@app.get("/items/")
async def read_items(
    filter_query: Annotated[FilterParams, Query()]
):
    return filter_query
```

✅ FastAPI will:
- Automatically read query parameters into the model
- Validate them using the model rules
- Show everything nicely in the docs at `/docs`

---

## 💥 Example URL

```
GET /items/?limit=10&offset=5&order_by=created_at&tags=fastapi&tags=python
```

FastAPI will parse this into:

```json
{
  "limit": 10,
  "offset": 5,
  "order_by": "created_at",
  "tags": ["fastapi", "python"]
}
```

---

## 🔒 Forbid Extra Query Parameters (optional)

Want to reject any query parameter not defined in your model? Add this to your model:

```python
class FilterParams(BaseModel):
    model_config = {"extra": "forbid"}  # 👈 magic line

    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []
```

If someone tries:

```
GET /items/?limit=10&tool=plumbus
```

They’ll get:

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

## 🔁 Summary

| Feature | Benefit |
|--------|---------|
| ✅ Group query parameters | Cleaner code, reusable |
| ✅ Use validation | Just like request body models |
| ✅ Great docs | Auto-generated in `/docs` |
| 🔒 Optional strict mode | Reject unknown params with `extra="forbid"` |

---

## 🎯 Goal

Instead of writing lots of query parameters one by one, you **group them into a Pydantic model**, which makes your code:

✅ Cleaner  
✅ Easier to reuse  
✅ Automatically validated  
✅ Well-documented in Swagger UI (`/docs`)

---

## 🧱 Example Without a Model (Messy Version)

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(limit: int = 100, offset: int = 0, tags: list[str] = []):
    return {"limit": limit, "offset": offset, "tags": tags}
```

This works fine, but it gets messy if there are many query parameters or you want to validate them more carefully.

---

## ✅ Clean Version: Using a Pydantic Model

### 1. **Create a model** for the related query parameters:

```python
from pydantic import BaseModel, Field
from typing import Literal

class FilterParams(BaseModel):
    limit: int = Field(100, gt=0, le=100)  # Must be 1–100
    offset: int = Field(0, ge=0)           # Must be ≥ 0
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []                   # List of strings
```

---

### 2. **Use it in the endpoint**:

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(filter_query: Annotated[FilterParams, Query()]):
    return filter_query
```

---

### ✅ What Happens?

If a user visits this URL:

```
/items/?limit=10&offset=5&order_by=created_at&tags=fastapi&tags=python
```

FastAPI will automatically:

- Read and validate the query parameters
- Convert them to a `FilterParams` model
- Give you a `filter_query` object like:

```json
{
  "limit": 10,
  "offset": 5,
  "order_by": "created_at",
  "tags": ["fastapi", "python"]
}
```

---

## 🔒 BONUS: Reject Extra Parameters

If you don’t want users to send any unknown query parameters:

### 1. Add this to your model:

```python
class FilterParams(BaseModel):
    model_config = {"extra": "forbid"}  # 👈 this line rejects extra params

    limit: int = Field(100, gt=0, le=100)
    offset: int = Field(0, ge=0)
    order_by: Literal["created_at", "updated_at"] = "created_at"
    tags: list[str] = []
```

### 2. If user sends something like:

```
/items/?limit=10&tool=plumbus
```

They’ll get an error like:

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

## 🔁 Recap

| Concept                     | Description |
|----------------------------|-------------|
| ✅ Use a model              | Makes grouped query parameters clean and reusable |
| ✅ Use `Field()`            | Adds validation rules to query parameters |
| ✅ Use `Query()`            | Tells FastAPI it's a query parameter, not a body |
| ✅ Use `extra="forbid"`     | Rejects any extra unknown query parameters |

---

