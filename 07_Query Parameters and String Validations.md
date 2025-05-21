## ✅ **FastAPI Query Parameter Validation Overview**

FastAPI helps you **safely and clearly handle user input from URLs**. This prevents bugs, improves UX in Swagger docs, and reduces backend validation logic.

---

### 1. ✅ **Optional Query Parameter (Basic)**

```python
@app.get("/items/")
async def read_items(q: str | None = None):
    ...
```

* `q` is **optional**, because it's typed as `str | None` and defaulted to `None`.
* It works even if the user doesn’t pass it in the query string.
* 🚫 `/items/` → works
  ✅ `/items/?q=apple` → `q="apple"`

---

### 2. ✅ **Validation with `Annotated` + `Query`**

```python
q: Annotated[str | None, Query(min_length=3, max_length=50)] = None
```

* Adds automatic **min/max length** validation.
* If `q` is provided, it must be 3–50 characters long.
* 📘 Useful for things like search terms, usernames, etc.

---

### 3. ✅ **Pattern Matching (Regex)**

```python
q: Annotated[str | None, Query(pattern="^fixedquery$")] = None
```

* `q` must **exactly match** `"fixedquery"` or raise a validation error.
* Useful when you **only allow one exact value** for a parameter (e.g., a fixed legacy token).

This line:

```python
q: Annotated[str | None, Query(pattern="^fixedquery$")] = None
```

breaks down into several parts. Let’s go through it step by step:

---

### 🔹 1. **`q:`**

This defines a query parameter named `q`.

---

### 🔹 2. **`Annotated[...]`**

FastAPI uses `Annotated` to attach **extra metadata** to the type — in this case, validation logic via `Query(...)`.

Think of it like:
**“This variable has a type + some FastAPI instructions.”**

---

### 🔹 3. **`str | None`**

This means `q` can be:

* A `string` (`str`)
* Or `None` (i.e., not passed in the query string)

So it's an **optional query parameter**.

---

### 🔹 4. **`Query(pattern="^fixedquery$")`**

This is where **FastAPI adds validation**:

* `pattern="^fixedquery$"` is a **regex (regular expression)**.
* It means:

  * The query string **must match exactly** `"fixedquery"`.
  * No more, no less — it’s case-sensitive.

📌 Example:

* ✅ `/items/?q=fixedquery`
* ❌ `/items/?q=FixedQuery` → case mismatch
* ❌ `/items/?q=fixedquery123` → extra chars

---

### 🔹 5. **`= None`**

This means the query param `q` is **optional**.

If the user does **not** provide `q`, its value will be `None`.

---

### ✅ Summary (Plain English)

```python
q: Annotated[str | None, Query(pattern="^fixedquery$")] = None
```

> This declares an **optional query parameter** named `q`.
> If provided, it **must exactly equal** the string `"fixedquery"`.
> Otherwise, FastAPI will return a validation error.

---

### 4. ✅ **Default Values**

```python
q: Annotated[str, Query(min_length=3)] = "fixedquery"
```

* If user **doesn’t send `q`**, it defaults to `"fixedquery"`.
* Value still must pass validation rules.

---

### 5. ✅ **Required Query Parameter**

```python
q: Annotated[str, Query(min_length=3)]
```

* This is a **mandatory** query param (no default).
* FastAPI will return `422` error if `q` is missing.
* Use this for required filters, IDs, etc.

---

### 6. ✅ **Required but Can Be `None`**

```python
q: Annotated[str | None, Query(min_length=3)]
```

* Tricky edge case:

  * Still **optional**, but if included, it must be ≥3 chars.
  * Accepts `/items/?q=abc` ✅ or `/items/` ✅
  * But `/items/?q=ab` ❌ (fails validation)

---

### 7. ✅ **Multiple Query Values (List Input)**

```python
q: Annotated[list[str] | None, Query()] = None
```

* User can send repeated query params: `/items/?q=foo&q=bar`
* FastAPI parses it into a list: `q = ["foo", "bar"]`
* Optional — if not sent, `q = None`

---

### 8. ✅ **List with Default Values**

```python
q: Annotated[list[str], Query()] = ["foo", "bar"]
```

* Default list of values if nothing is passed.
* Good for **preloaded filters, categories**, etc.

---

### 9. ⚠️ **Generic List (No Type Safety)**

```python
q: Annotated[list, Query()] = []
```

* Accepts anything in the list — not recommended unless necessary.
* No validation on list items — could be mixed types.

---

### 10. ✅ **Metadata: Title, Description**

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

* Makes your **API docs clearer and more professional**.
* `title`: displayed above the field
* `description`: below the input box in Swagger UI

---

### 11. ✅ **Alias for Parameter Name**

```python
q: Annotated[str | None, Query(alias="item-query")] = None
```

* Accepts `/items/?item-query=value`
* Still uses Python name `q` in your code.
* Useful when API needs to keep legacy names or snake-case in code vs kebab-case in URL.

---

### 12. ✅ **Deprecating Parameters**

```python
q: Annotated[str | None, Query(deprecated=True)] = None
```

* Shows as **deprecated** in Swagger/OpenAPI docs.
* Still works, but this is a signal to users to stop using it.
* Helps during **API version upgrades**.

---

### 13. ✅ **Exclude Parameter from Schema**

```python
hidden_query: Annotated[str | None, Query(include_in_schema=False)] = None
```

* Hidden from OpenAPI docs but still processed by the server.
* Great for **debug params, internal tools, or secrets**.

---

### 14. ✅ **Custom Validation with `AfterValidator`**

```python
from pydantic import AfterValidator

def check_valid_id(id: str):
    if not id.startswith(("isbn-", "imdb-")):
        raise ValueError("Invalid ID")
    return id

id: Annotated[str | None, AfterValidator(check_valid_id)] = None
```

* Runs custom logic **after** default validation.
* Perfect for prefix checks, complex format rules.
* Keep in mind: only use this if validation doesn't need external systems (e.g., no DB calls).

---

### ✅ Combined Example

```python
@app.get("/items/")
async def read_items(
    q: Annotated[
        str | None,
        Query(
            alias="item-query",
            title="Search",
            description="Search items by keyword",
            min_length=3,
            max_length=50,
            pattern="^fixedquery$",
            deprecated=True
        )
    ] = None,
    hidden: Annotated[str | None, Query(include_in_schema=False)] = None,
    id: Annotated[str | None, AfterValidator(check_valid_id)] = None,
):
    return {"q": q, "hidden": hidden, "id": id}
```

* `q`: old, validated, fixed-format search param (deprecated)
* `hidden`: works but not shown in Swagger
* `id`: must start with `"isbn-"` or `"imdb-"`

---

## 🔁 Summary Table

| Feature             | Example                              | Use Case                     |                   |
| ------------------- | ------------------------------------ | ---------------------------- | ----------------- |
| Optional param      | \`q: str                             | None = None\`                | Free-form filters |
| Min/Max length      | `Query(min_length=3, max_length=50)` | Name/search constraints      |                   |
| Pattern match       | `pattern="^fixedquery$"`             | Exact-match filter           |                   |
| Default             | `= "fixedquery"`                     | Optional param with fallback |                   |
| Required param      | No default, no \`                    | None\`                       | Mandatory filters |
| List of values      | `q: list[str]`                       | Tags, filters, etc.          |                   |
| Title & description | `Query(title=..., description=...)`  | Swagger UI metadata          |                   |
| Alias               | `Query(alias="item-query")`          | Support legacy field names   |                   |
| Deprecate           | `Query(deprecated=True)`             | API migration                |                   |
| Exclude from docs   | `Query(include_in_schema=False)`     | Internal/debug params        |                   |
| Custom validator    | `AfterValidator(check_valid_id)`     | Prefix rules, manual checks  |                   |

---

