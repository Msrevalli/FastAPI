FastAPI is a modern, high-performance web framework for building APIs with Python 3.7+ based on standard Python type hints.

### Key Features of FastAPI:
- **Fast**: Very high performance, on par with Node.js and Go (thanks to Starlette and Pydantic under the hood).
- **Easy to Use**: Designed to be intuitive and easy to use, even for beginners.
- **Automatic Docs**: Automatically generates interactive API documentation (Swagger UI and ReDoc).
- **Type-Safe**: Leverages Python type hints to provide data validation and editor support (e.g., autocomplete, error checking).
- **Asynchronous Support**: Built with async/await for building concurrent applications.
- **Validation**: Uses Pydantic for request and response validation based on Python types.

---

### Example:
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "query": q}
```

When you run the above app, you get:
- API at `http://127.0.0.1:8000`
- Interactive docs at:
  - Swagger: `http://127.0.0.1:8000/docs`
  - ReDoc: `http://127.0.0.1:8000/redoc`

---

Exactly! You nailed it 👏 FastAPI uses **Python type hints** not just for *developer support*, but also for *runtime behavior*. Let’s break down what you mentioned with real-world context and examples.

---

## 🔹 1. **Editor Support & Type Checks**

Because FastAPI uses type hints, your editor (like VS Code or PyCharm) can:
- Autocomplete request/response models
- Warn about wrong types
- Help you navigate your code better

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/square")
def square(x: int) -> int:
    return x * x
```

✅ Editor knows `x` is an `int` and return is `int`.

---

## 🔹 2. **FastAPI Uses Type Hints to Define Requirements**

FastAPI understands how to **extract values** and **enforce types** from:
- **Path parameters**
- **Query parameters**
- **Request bodies**
- **Headers and cookies**
- **Dependencies**

Example:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "query": q}
```

- `item_id` is taken from the **path**, cast to `int`
- `q` is taken from the **query**, default is `None`

---

## 🔹 3. **Automatic Data Conversion & Validation**

FastAPI **parses and validates** incoming data using your type hints. If something doesn’t match, it returns a 422 error.

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

@app.post("/users/")
def create_user(user: User):
    return user
```

- JSON payload is validated automatically
- If `age` is a string or missing → automatic error
- If it passes, `user` is a real `User` object with editor support!

---

## 🔹 4. **Automatic Documentation (Swagger / ReDoc)**

All type hints are **reflected into OpenAPI docs**:
- FastAPI generates `/docs` (Swagger) and `/redoc`
- Request bodies, query params, responses, and errors are fully documented

![Swagger UI](https://fastapi.tiangolo.com/img/index/index-03-swagger-02.png)

---

## TL;DR – FastAPI’s Type Hint Superpowers

| Feature | What It Does |
|--------|--------------|
| **Editor Support** | Autocomplete, navigation, static analysis |
| **Validation** | Validates incoming data from query, path, body |
| **Conversion** | Converts strings/JSON into expected Python types |
| **Error Handling** | Sends automatic errors if data is invalid |
| **Documentation** | Generates full OpenAPI + interactive docs |

