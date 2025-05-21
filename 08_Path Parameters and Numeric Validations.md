## 🔹 **1. Basic Path Parameter**

### Concept:

A **path parameter** is part of the **URL path**, e.g. `/items/42`, where `42` is `item_id`.

### Code:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

### URL:

```
GET /items/42
```

### Output:

```json
{
  "item_id": 42
}
```

---

## 🔹 **2. Using `Path` for Metadata (like `title`)**

### Concept:

Use `Path()` to add **extra information** like a title or description for documentation.

### Code:

```python
from fastapi import FastAPI, Path
from typing import Annotated

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: Annotated[int, Path(title="The ID of the item to get")]):
    return {"item_id": item_id}
```

### Benefit:

* Adds a **title** to the path parameter in the **API docs** (`/docs`).

---

## 🔹 **3. Query Parameter with `Query` and Alias**

### Concept:

Query parameters are passed like `?q=value`. Use `Query()` to customize behavior (e.g., alias).

### Code:

```python
from fastapi import FastAPI, Query
from typing import Annotated

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(alias="item-query")] = None):
    return {"q": q}
```

### URL:

```
GET /items/?item-query=search_text
```

### Output:

```json
{
  "q": "search_text"
}
```

---

## 🔹 **4. Numeric Validation in Path Parameter: `ge` (greater than or equal)**

### Concept:

Use `ge=1` to ensure the number is **greater than or equal to 1**.

### Code:

```python
from fastapi import FastAPI, Path
from typing import Annotated

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(
    item_id: Annotated[int, Path(ge=1, title="The ID must be >= 1")]
):
    return {"item_id": item_id}
```

### Invalid Request:

```
GET /items/0 → 422 Unprocessable Entity
```

---

## 🔹 **5. Numeric Validation: `gt`, `le` (greater than, less than or equal)**

### Concept:

Set both upper and lower bounds for an integer.

### Code:

```python
from fastapi import FastAPI, Path
from typing import Annotated

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(
    item_id: Annotated[int, Path(gt=0, le=1000)]
):
    return {"item_id": item_id}
```

* Accepts values between `1` and `1000` (not `0`, not `1001`)

---

## 🔹 **6. Float Validation with `Query`**

### Concept:

Validating float query parameters with greater-than and less-than.

### Code:

```python
from fastapi import FastAPI, Path, Query
from typing import Annotated

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(
    item_id: Annotated[int, Path(ge=0, le=1000)],
    size: Annotated[float, Query(gt=0, lt=10.5)]
):
    return {"item_id": item_id, "size": size}
```

### URL:

```
GET /items/5?size=5.3 → ✅
GET /items/5?size=0 → ❌ (gt > 0 fails)
GET /items/5?size=11 → ❌ (lt < 10.5 fails)
```

---

## 🔹 **7. Using `*` to Force Keyword-Only Arguments (non-Annotated)**

### Concept:

In Python, use `*` in the function to make all parameters **keyword-only**, even if they don't have defaults.

### Code:

```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(
    *, item_id: int = Path(title="The ID"), q: str
):
    return {"item_id": item_id, "q": q}
```

### URL:

```
GET /items/10?q=hello
```

✅ This allows `q` to be required, without changing the parameter order.

---

## 🔹 **8. Recap: Validation Keywords**

| Validation | Meaning               | Example                  |
| ---------- | --------------------- | ------------------------ |
| `gt`       | Greater than          | `gt=0` → value > 0       |
| `ge`       | Greater than or equal | `ge=1` → value ≥ 1       |
| `lt`       | Less than             | `lt=100` → value < 100   |
| `le`       | Less than or equal    | `le=1000` → value ≤ 1000 |

---


