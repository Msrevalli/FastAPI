## 🧠 What Are Path and Query Parameters?

- **Path parameter** is part of the URL:  
  `/items/{item_id}` → `item_id` is a path parameter

- **Query parameter** is optional and passed after `?`:  
  `/items/42?q=hello` → `q` is a query parameter

---

## ✅ Validating Parameters with `Annotated`, `Path`, and `Query`

### Step 1: Basic Example

```python
from typing import Annotated
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get")],  # path param
    q: Annotated[str | None, Query(alias="item-query")] = None         # query param
):
    results = {"item_id": item_id}
    if q:
        results["q"] = q
    return results
```

### ✅ Explanation

- `item_id`: a **required path parameter** with metadata (`title`)
- `q`: an **optional query parameter** that will appear as `item-query` in the docs

---

## ✨ Add Number Validations to Path Parameters

You can use:
- `gt=...`: greater than
- `ge=...`: greater than or equal
- `lt=...`: less than
- `le=...`: less than or equal

### Example with Path Validation

```python
@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="Item ID", ge=1, le=1000)],  # Must be 1–1000
    q: str
):
    return {"item_id": item_id, "q": q}
```

✅ Now `item_id` must be between `1` and `1000`.

---

## 🔢 Float Validation with Query

You can also validate floats in query parameters:

```python
@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(ge=1, le=1000)],
    size: Annotated[float, Query(gt=0.0, lt=10.5)],  # Must be 0 < size < 10.5
    q: str
):
    return {"item_id": item_id, "q": q, "size": size}
```

📌 This makes sure:
- `item_id` is between 1 and 1000
- `size` is a float > 0 and < 10.5

---

## 🎓 Bonus: `*` Trick for Function Parameters

In Python, you can use `*` to force **all parameters to be keyword-only**:

```python
@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: Annotated[int, Path(title="Item ID", ge=1)],
    q: str
):
    return {"item_id": item_id, "q": q}
```

✅ This helps avoid the "default before non-default" error in non-Annotated functions.

---

## 📦 Recap

| Feature | What it does |
|--------|---------------|
| `Path(...)` | Validates and adds metadata to path parameters |
| `Query(...)` | Validates and adds metadata to query parameters |
| `Annotated` | Cleaner, modern way to declare types and validation |
| `gt`, `ge`, `lt`, `le` | Number validations (greater than, less than, etc.) |
| `*` in params | Forces keyword arguments, avoids function order issues |

---

## 🎯 What are Path Parameters?

Path parameters are values that **are part of the URL path**.

Example:
```
GET /items/42
```
Here, `42` is a **path parameter**. It's captured like this:

```python
@app.get("/items/{item_id}")
async def read_items(item_id: int):
    return {"item_id": item_id}
```

---

## 🎯 What are Query Parameters?

Query parameters are optional values added **after the `?` in the URL**.

Example:
```
GET /items/42?q=hello
```
Here, `q=hello` is a **query parameter**.

---

## ✅ Declaring Path Parameters with Validation

We use `Path()` to **add rules** or **metadata** to path parameters.

### Example:

```python
from fastapi import FastAPI, Path
from typing import Annotated

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get", ge=1, le=1000)]
):
    return {"item_id": item_id}
```

### 🔍 Explanation:
- `item_id` must be an integer between `1` and `1000`
- `title="..."` shows up in the documentation
- `ge=1` means **greater than or equal to 1**
- `le=1000` means **less than or equal to 1000**

---

## ✅ Declaring Query Parameters with Validation

Use `Query()` to do the same for query parameters.

### Example:

```python
from fastapi import FastAPI, Query
from typing import Annotated

app = FastAPI()

@app.get("/search")
async def search_items(
    q: Annotated[str, Query(min_length=3, max_length=50)]
):
    return {"q": q}
```

### 🔍 Explanation:
- `q` is a **query parameter**
- `min_length=3` → must be at least 3 characters
- `max_length=50` → must be at most 50 characters

---

## ✅ Float Number Validation Example

You can also use these rules for **float numbers**.

```python
@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(ge=1)],
    size: Annotated[float, Query(gt=0, lt=10.5)]
):
    return {"item_id": item_id, "size": size}
```

### 🔍 Explanation:
- `size` must be **greater than 0** and **less than 10.5**

---

## ✅ Using `*` for Keyword-Only Arguments

In Python, sometimes you get an error if you write a default value before a required one.

This gives an error:

```python
def read_items(item_id: int = Path(...), q: str):  # ❌ will error
```

✅ Fix it using `*`:

```python
@app.get("/items/{item_id}")
async def read_items(
    *, 
    item_id: int = Path(..., title="The ID"), 
    q: str
):
    return {"item_id": item_id, "q": q}
```

---

## 💡 Use `Annotated` to Keep It Clean

Instead of doing this:

```python
item_id: int = Path(title="The ID", ge=1)
```

You do this (cleaner):

```python
item_id: Annotated[int, Path(title="The ID", ge=1)]
```

✅ This is the **modern recommended way** (FastAPI 0.95.0+).

---

## 🧠 Summary

| Concept | Tool | Example |
|--------|------|---------|
| Path parameter | `Path()` | `item_id: Annotated[int, Path(ge=1)]` |
| Query parameter | `Query()` | `q: Annotated[str, Query(max_length=50)]` |
| Integer limits | `ge`, `le`, `gt`, `lt` | `ge=1`, `lt=10.5` |
| Cleaner syntax | `Annotated` | Use with `Path` or `Query` |
| Fix default order issues | `*` | `def read_items(*, item_id, q)` |

---

