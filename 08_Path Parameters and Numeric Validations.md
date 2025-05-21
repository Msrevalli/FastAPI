**Path parameters with numeric validations in FastAPI**:

---

## ✅ **1. Path Parameters with Metadata & Validation**

### ✅ Why `Path()`?

* Used to validate **path parameters**, like `/items/{item_id}`
* Similar to `Query()` but for the **URL path**
* Always **required**

---

### ✅ How to Use `Path()` with `Annotated` (Python 3.9+ / 3.10+)

```python
from typing import Annotated
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get")]
):
    return {"item_id": item_id}
```

✅ Path parameter `item_id`:

* Must be present in URL
* Must be an `int`
* Will show title in the OpenAPI docs

---

## 📏 **2. Numeric Validations with `Path()`**

### Greater than or equal to (ge):

```python
item_id: Annotated[int, Path(ge=1)]
```

* Must be `≥ 1`

### Greater than (gt) and Less than or equal (le):

```python
item_id: Annotated[int, Path(gt=0, le=1000)]
```

* Must be `> 0` and `≤ 1000`

### Float validation with `Query()`:

```python
size: Annotated[float, Query(gt=0, lt=10.5)]
```

* Must be a float `> 0` and `< 10.5`

---

## 🔄 **3. Annotated vs Non-Annotated Syntax**

### ✅ Preferred: Using `Annotated`

```python
@app.get("/items/{item_id}")
async def read_items(
    item_id: Annotated[int, Path(title="ID", ge=1)],
    q: Annotated[str | None, Query(alias="item-query")] = None,
):
    ...
```

### 🔁 Still works: Non-Annotated (Python 3.8+)

```python
@app.get("/items/{item_id}")
async def read_items(q: str, item_id: int = Path(title="ID")):
    ...
```

📌 But this forces you to reorder parameters to avoid Python errors (non-default before default).

---

## 🛠️ **4. Forcing Keyword-Only Parameters with `*`**

```python
@app.get("/items/{item_id}")
async def read_items(
    *, item_id: int = Path(title="ID"), q: str
):
    ...
```

✅ Use `*` to force all arguments to be passed by keyword.
✅ Avoids reordering hassle in non-Annotated syntax.

---

## 📘 Recap: Path vs Query

| Feature            | `Path()`                              | `Query()`            |
| ------------------ | ------------------------------------- | -------------------- |
| Used for           | URL path parameters                   | URL query parameters |
| Required?          | Always required                       | Optional by default  |
| Validations        | `gt`, `ge`, `lt`, `le`, `title`, etc. | Same                 |
| Preferred Syntax   | `Annotated` (Python 3.9+/3.10+)       | Same                 |
| Keyword-only trick | Use `*` in function signature         | Same applies         |

---

## 💡 Extra: Why use `Path()` and `Query()` at all?

Because they allow:

* **Validation** (e.g., only accept values between 1 and 1000)
* **Metadata** (e.g., descriptions/titles in OpenAPI docs)
* **Alias naming** (e.g., use a different external param name)

---


