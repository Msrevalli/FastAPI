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

