### ✅ **1. `status_code`**
Sets the HTTP status code for the response.

```python
from fastapi import FastAPI, status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
    return item
```

🔹 You can use raw integers like `201`, but `status.HTTP_201_CREATED` is more descriptive and readable.

---

### ✅ **2. `tags`**
Used to organize operations in the interactive docs.

```python
@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]
```

🔹 You can use Enums for consistency in large apps:

```python
from enum import Enum
class Tags(Enum):
    users = "users"

@app.get("/users/", tags=[Tags.users])
```

---

### ✅ **3. `summary` and `description`**
Provide metadata for docs. Use for clearer OpenAPI docs.

```python
@app.post(
    "/items/",
    summary="Create an item",
    description="Create an item with all fields like name, description, price, etc.",
)
```

🔹 Or use a docstring for `description`:

```python
@app.post("/items/", summary="Create an item")
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: must have a name
    - **price**: required
    - **tags**: optional set of tags
    """
    return item
```

---

### ✅ **4. `response_description`**
Provides a description specifically for the **response**.

```python
@app.post(
    "/items/",
    response_description="The created item",
    response_model=Item
)
```

🔹 If you don't provide it, FastAPI defaults to `"Successful Response"`.

---

### ✅ **5. `deprecated`**
Marks an endpoint as deprecated in the docs (but still usable).

```python
@app.get("/elements/", tags=["items"], deprecated=True)
async def read_elements():
    return [{"item_id": "Foo"}]
```

---

