### ✅ **1. `status_code`**
Sets the HTTP status code for the response.

```python
from fastapi import FastAPI, status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
    return item
```

🔹 You can use raw integers like `201`, but `status.HTTP_201_CREATED` is more descriptive and readable.

```python
from fastapi import FastAPI, status
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()


@app.post("/items/", response_model=Item, status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
    return item
```
* **Declares a Pydantic model `Item`** with several fields:

  * `name`: required `str`
  * `description`: optional `str`
  * `price`: required `float`
  * `tax`: optional `float`
  * `tags`: set of `str`, default empty
* **Creates a POST endpoint** at `/items/`:

  * **Accepts a request body** matching the `Item` model.
  * **Returns the same item as the response**, using `response_model=Item`.
  * **Sets HTTP status code 201 (Created)** for success.

---

### 🧾 **Example Request**

**POST** `/items/`

```json
{
  "name": "Laptop",
  "description": "Fast and light",
  "price": 999.99,
  "tax": 89.99,
  "tags": ["electronics", "computers"]
}
```

### ✅ **Response**

```json
{
  "name": "Laptop",
  "description": "Fast and light",
  "price": 999.99,
  "tax": 89.99,
  "tags": ["computers", "electronics"]  // Set may reorder
}
```

---

### 🧠 Key Concepts

| Feature                 | Explanation                                                            |                                                      |
| ----------------------- | ---------------------------------------------------------------------- | ---------------------------------------------------- |
| `BaseModel` (Pydantic)  | Used to define request and response data schemas                       |                                                      |
| `response_model=Item`   | Ensures consistent response output (includes validation + filtering)   |                                                      |
| `status_code=201`       | Tells FastAPI to return HTTP 201 Created instead of the default 200 OK |                                                      |
| `set[str]` in `tags`    | Ensures tags are unique (e.g., `["a", "a", "b"]` becomes `["a", "b"]`) |                                                      |
| Optional fields with \` | None\`                                                                 | Make the field optional in both request and response |

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
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()


@app.post("/items/", response_model=Item, tags=["items"])
async def create_item(item: Item):
    return item


@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]


@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]
```

### 🔍 What this does:

* `tags=["items"]` groups `/items/` endpoints together in the **Swagger UI**.
* `tags=["users"]` separates `/users/` under a different section.

---
### ✅ `POST /items/`

**Request Body:**

```json
{
  "name": "Laptop",
  "description": "A powerful machine",
  "price": 1500.99,
  "tax": 100.00,
  "tags": ["electronics", "computers"]
}
```

**Response:**

```json
{
  "name": "Laptop",
  "description": "A powerful machine",
  "price": 1500.99,
  "tax": 100.0,
  "tags": ["electronics", "computers"]
}
```

---

### ✅ `GET /items/`

**Response:**

```json
[
  {
    "name": "Foo",
    "price": 42
  }
]
```

---

### ✅ `GET /users/`

**Response:**

```json
[
  {
    "username": "johndoe"
  }
]
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

