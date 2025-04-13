## ✅ 1. **Simple Body with a Model**

Basic usage — model fields define the request structure:

```python
class Item(BaseModel):
    name: str
    price: float
```

Used in route:

```python
@app.post("/items/")
async def create_item(item: Item):
    return item
```

---

## ✅ 2. **Fields with Validation & Metadata**

Use `Field(...)` inside your model to add:
- Defaults
- Descriptions
- Constraints like `gt=0`, `max_length=300`, etc.

```python
from pydantic import Field

class Item(BaseModel):
    name: str
    description: str | None = Field(default=None, max_length=300)
    price: float = Field(gt=0, description="Must be greater than zero")
```

---

## ✅ 3. **Nested Models**

You can nest models inside models:

```python
class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str
    image: Image | None = None
```

Request body would look like:

```json
{
  "name": "Camera",
  "image": {
    "url": "http://example.com/image.jpg",
    "name": "Front View"
  }
}
```

---

## ✅ 4. **Lists / Sets of Values or Models**

You can define attributes as collections:
- `list[str]` or `List[str]`
- `set[str]` for **unique items**
- `list[SubModel]` for nested lists of models

```python
class Item(BaseModel):
    tags: list[str]
    images: list[Image]
```

---

## ✅ 5. **Deeply Nested Models**

Example: `Offer` → list of `Item` → list of `Image`

```python
class Offer(BaseModel):
    name: str
    items: list[Item]
```

Request:

```json
{
  "name": "Big Sale",
  "items": [
    {
      "name": "Item A",
      "images": [
        {"url": "http://img.com/a.jpg", "name": "A"}
      ]
    }
  ]
}
```

---

## ✅ 6. **Top-Level List as Body**

The whole request body can be a list:

```python
@app.post("/images/")
async def create_images(images: list[Image]):
    return images
```

Request:

```json
[
  {"url": "http://img.com/a.jpg", "name": "A"},
  {"url": "http://img.com/b.jpg", "name": "B"}
]
```

---

## ✅ 7. **Top-Level Dict as Body**

Accept a dynamic dict body with unknown keys:

```python
@app.post("/index-weights/")
async def create_index_weights(weights: dict[int, float]):
    return weights
```

Request:

```json
{
  "1": 0.9,
  "2": 0.75
}
```

- Even though keys are strings (JSON), Pydantic converts to `int`.

---

## 🎁 Recap of Benefits

With **Pydantic + FastAPI**:

✅ Automatic request validation  
✅ Clear request structure with type hints  
✅ Rich, interactive API docs  
✅ Editor autocomplete — even deep inside nested models  
✅ Auto conversion (e.g., `str` → `int`)

---

