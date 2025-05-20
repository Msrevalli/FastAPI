### 🔹 What are Path Parameters?
In **FastAPI**, **path parameters** are values passed **in the URL path** that your API can extract and use in route functions. They are a key part of defining dynamic routes and are typically used to access a specific resource (e.g., a user by ID).

---

### 🔧 Example:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"user_id": user_id}
```

* Here, `/users/{user_id}` is the path.
* `{user_id}` is a **path parameter**.
* If you request `/users/123`, FastAPI extracts `123` and passes it as an integer to the `user_id` parameter.

---

### ✅ Features:

1. **Automatic type conversion and validation**:

   * If you declare `user_id: int`, FastAPI will validate the path value and return a 422 error if it isn’t an integer.

2. **Used for resource identification**:

   * e.g., `/items/{item_id}`, `/posts/{slug}`, `/orders/{order_id}`.

3. **Ordering matters**:

   * Path parameters must appear in the function signature in the order they appear in the path.

---

In FastAPI, **path parameters** allow you to make parts of the URL dynamic. For example:

```python
@app.get("/items/{item_id}")
async def read_item(item_id: str):
    return {"item_id": item_id}
```

If you visit `/items/foo`, FastAPI captures `foo` and sends it into the `read_item` function as `item_id`.

---

### 🔹 Type Validation and Conversion

FastAPI automatically **converts and validates** types using the type hints in your function.

Example:

```python
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

If you visit `/items/3`, FastAPI converts the path parameter to an integer.

If you visit `/items/foo`, it returns a 422 error with a clear explanation because `"foo"` isn’t a valid integer.

---

### 🔹 Order Matters in Route Definitions

If you define this:

```python
@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}

@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

✅ Good: `/users/me` is matched before the more general `/users/{user_id}`

If reversed, `/users/me` would be incorrectly captured as `user_id = "me"`.

---

### 🔹 Enumerations for Fixed Parameters

To restrict path parameters to predefined values, use Enums:

```python
from enum import Enum

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    return {"model_name": model_name}
```

Now only `/models/alexnet`, `/models/resnet`, and `/models/lenet` are valid.

Benefits:
- Autocompletion in docs
- Better validation
- Cleaner control logic

---
### 🔄 **Path Converter: `:path`**

Normally, a path parameter stops at the first `/`. But if you want to capture a full file path (which can contain slashes), you can use the `:path` converter.

---

### ✅ Example

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

#### How it works:

* Route: `/files/{file_path:path}`
* The `:path` converter tells FastAPI to **match the remainder of the URL**, including slashes.
* This is useful for file systems, nested routes, or hierarchical data.

---

### 🔍 Example URL Call

Request:

```
GET /files/home/johndoe/myfile.txt
```

Response:

```json
{
  "file_path": "home/johndoe/myfile.txt"
}
```

---

### ⚠️ Leading Slash Tip

If your input has a **leading slash**, e.g., `/home/johndoe/myfile.txt`, then:

You must call:

```
GET /files//home/johndoe/myfile.txt
```

Note the **double slash**:

* First `/` is from the fixed `/files/`
* Second `/` starts the actual file path (`/home/...`)

---

### ✅ Why It’s Powerful

By just using Python's type hints:
- You get request parsing
- Input validation
- Auto-generated documentation (Swagger at `/docs`, ReDoc at `/redoc`)
- Helpful error messages
- Code that’s easy to read and maintain

---

