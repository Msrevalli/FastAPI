## ✅ **FastAPI Query Parameter Validation Overview**

FastAPI allows you to **validate query parameters** easily and expressively.

---

### 🧠 Base Example (Optional query param)

```python
@app.get("/items/")
async def read_items(q: str | None = None):
    ...
```

* `q` is optional (`str | None`)
* Default is `None`, so it's **not required**
* This enables editor/autocomplete support for optional logic

---

## 🧪 Adding Validation (e.g., max length)

Use `Query()` from FastAPI to add validation like `max_length=50`.

---

### ✅ Modern Way (Recommended): Using `Annotated`

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    ...
```

* `Annotated` wraps the type with extra metadata
* `Query(max_length=50)` tells FastAPI to:

  * Validate input
  * Show nice errors
  * Update OpenAPI docs

---

### 🛠️ Alternative (Legacy): Using `Query` as Default

```python
@app.get("/items/")
async def read_items(q: str | None = Query(default=None, max_length=50)):
    ...
```

* Still works (especially in **FastAPI < 0.95.0**)
* Less Pythonic than `Annotated`
* You *must* provide default inside `Query(default=None)`

---

## 🧨 Important Notes

### ❌ Don't do this:

```python
q: Annotated[str, Query(default="rick")] = "morty"
```

* Conflict between two defaults — ambiguous

---

### ✅ Do this:

```python
q: Annotated[str, Query()] = "rick"
# or the old way:
q: str = Query(default="rick")
```

---

## 🚀 Why `Annotated` Is Better

1. ✅ **Cleaner syntax** with Python 3.10+
2. ✅ **More intuitive default handling**
3. ✅ **Better reuse of functions (outside FastAPI)**
4. ✅ **Multiple metadata annotations** possible (e.g. security + validation)

---

## 📖 Final Annotated Example Recap:

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    return {"q": q}
```

Try `/items/?q=hello` — ✅
Try `/items/?q=` — ✅ (returns `null`)
Try `/items/?q=...<51 characters>` — ❌ error

---

Here's a **condensed reference guide** to make all this easier to recall and apply in practice:

## ✅ FastAPI Query Parameters — Reference & Examples

### 1. **Optional Query Parameter (Basic)**

```python
@app.get("/items/")
async def read_items(q: str | None = None):
    ...
```

---

### 2. **Add Validations with `Annotated` + `Query`**

```python
from typing import Annotated
from fastapi import Query

@app.get("/items/")
async def read_items(
    q: Annotated[str | None, Query(min_length=3, max_length=50)] = None,
):
    ...
```

---

### 3. **Regex / Pattern Matching**

```python
q: Annotated[str | None, Query(pattern="^fixedquery$")] = None
```

> Deprecated: `regex="..."` → use `pattern="..."` instead

---

### 4. **Default Values**

```python
q: Annotated[str, Query(min_length=3)] = "fixedquery"
```

---

### 5. **Required Query Parameter**

```python
q: Annotated[str, Query(min_length=3)]
```

---

### 6. **Required, but Can Be `None`**

```python
q: Annotated[str | None, Query(min_length=3)]
```

---

### 7. **Multiple Query Values (List)**

```python
q: Annotated[list[str] | None, Query()] = None
```

**Example request:**
`/items/?q=foo&q=bar`
→ `q = ["foo", "bar"]`

---

### 8. **List with Default Values**

```python
q: Annotated[list[str], Query()] = ["foo", "bar"]
```

---

### 9. **Generic List (No Validation)**

```python
q: Annotated[list, Query()] = []
```

> Less safe — no type enforcement on list elements.

---

### 10. **Metadata: Title, Description**

```python
q: Annotated[
    str | None,
    Query(
        title="Query string",
        description="Used to search for items in DB",
        min_length=3,
    ),
] = None
```

---

### 11. **Alias for Parameter Names**

```python
q: Annotated[str | None, Query(alias="item-query")] = None
```

> Accepts `/items/?item-query=foobar`
> `q = "foobar"`

---

### 🧠 Tip: Why Prefer `Annotated`?

* Keeps Python function signatures standard
* Safer when calling functions outside FastAPI
* Cleaner, more modular metadata support
* Avoids confusion with dual defaults (`Query(default=...)` vs Python default)

---
## ✅ Deprecating a Parameter

```python
q: Annotated[str | None, Query(deprecated=True)] = None
```

> Shows as "deprecated" in OpenAPI docs.

---

## ✅ Exclude Parameter from Schema (Docs)

```python
hidden_query: Annotated[str | None, Query(include_in_schema=False)] = None
```

> Hides parameter from docs but still usable in requests.

---

## ✅ Custom Validation with `AfterValidator`

### Use for logic like: must start with `"isbn-"` or `"imdb-"`

```python
from pydantic import AfterValidator

def check_valid_id(id: str):
    if not id.startswith(("isbn-", "imdb-")):
        raise ValueError('Invalid ID: must start with "isbn-" or "imdb-"')
    return id

id: Annotated[str | None, AfterValidator(check_valid_id)] = None
```

---

## ✅ Example Endpoint with All Combined:

```python
from typing import Annotated
from fastapi import FastAPI, Query
from pydantic import AfterValidator

app = FastAPI()

def validate_id(value: str):
    if not value.startswith(("isbn-", "imdb-")):
        raise ValueError("Must start with 'isbn-' or 'imdb-'")
    return value

@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None,
        Query(
            alias="item-query",
            title="Query string",
            description="Search string for items",
            min_length=3,
            max_length=50,
            pattern="^fixedquery$",
            deprecated=True,
        ),
    ] = None,
    hidden: Annotated[str | None, Query(include_in_schema=False)] = None,
    id: Annotated[str | None, AfterValidator(validate_id)] = None,
):
    return {"q": q, "hidden": hidden, "id": id}
```

---


