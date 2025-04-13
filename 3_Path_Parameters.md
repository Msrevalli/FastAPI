---

### 🔹 What are Path Parameters?

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

### 🔹 Path Parameters with Paths Inside

To allow parameters that include slashes, like file paths, use the `:path` converter:

```python
@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

This route matches `/files/home/user/data.txt` and captures the whole path after `/files/`.

Note: URLs will look like `/files//home/user/data.txt` if your file path starts with `/`.

---

### ✅ Why It’s Powerful

By just using Python's type hints:
- You get request parsing
- Input validation
- Auto-generated documentation (Swagger at `/docs`, ReDoc at `/redoc`)
- Helpful error messages
- Code that’s easy to read and maintain

---

