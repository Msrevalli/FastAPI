## ✅ **1. Model-Level Example via `model_config` (`json_schema_extra`)**

### 🔧 Code

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                }
            ]
        }
    }

@app.post("/items/")
async def create_item(item: Item):
    return item
```

### 📤 Example Request (JSON)

```json
{
  "name": "Foo",
  "description": "A very nice Item",
  "price": 35.4,
  "tax": 3.2
}
```

---

## ✅ **2. Field-Level Examples via `Field(examples=[...])`**

### 🔧 Code

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str = Field(examples=["Foo"])
    description: str | None = Field(default=None, examples=["A very nice Item"])
    price: float = Field(examples=[35.4])
    tax: float | None = Field(default=None, examples=[3.2])

@app.post("/items/")
async def create_item(item: Item):
    return item
```

### 📤 Example Request (JSON)

Same as above. These examples are shown **per field** in the docs:

```json
{
  "name": "Foo",
  "description": "A very nice Item",
  "price": 35.4,
  "tax": 3.2
}
```

---

## ✅ **3. Body-Level Examples via `Body(examples=[...])`**

### 🔧 Code

```python
from fastapi import FastAPI, Body
from pydantic import BaseModel
from typing import Annotated

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.put("/items/{item_id}")
async def update_item(
    item_id: int,
    item: Annotated[
        Item,
        Body(
            examples=[
                {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                }
            ]
        )
    ]
):
    return {"item_id": item_id, "item": item}
```

### 📤 Example Request (JSON)

```json
{
  "name": "Foo",
  "description": "A very nice Item",
  "price": 35.4,
  "tax": 3.2
}
```

---

## ✅ **4. Multiple Body Examples via `Body(examples=[...])`**

### 🔧 Code

```python
@app.put("/items/{item_id}/multi")
async def update_item_multi_example(
    item_id: int,
    item: Annotated[
        Item,
        Body(
            examples=[
                {"name": "Foo", "description": "Nice", "price": 35.4, "tax": 3.2},
                {"name": "Bar", "price": "35.4"},  # Auto converted
                {"name": "Baz", "price": "thirty five"}  # Invalid
            ]
        )
    ]
):
    return {"item_id": item_id, "item": item}
```

### 📤 Example Requests (JSON)

✅ Valid:

```json
{
  "name": "Foo",
  "description": "Nice",
  "price": 35.4,
  "tax": 3.2
}
```

✅ Valid with auto-type conversion:

```json
{
  "name": "Bar",
  "price": "35.4"
}
```

❌ Invalid:

```json
{
  "name": "Baz",
  "price": "thirty five"
}
```

---

## ✅ **5. Named Examples via `openapi_examples`**

### 🔧 Code

```python
@app.put("/items/{item_id}/named")
async def update_item_named_examples(
    item_id: int,
    item: Annotated[
        Item,
        Body(
            openapi_examples={
                "normal": {
                    "summary": "A normal example",
                    "description": "A **normal** item",
                    "value": {
                        "name": "Foo",
                        "description": "A very nice Item",
                        "price": 35.4,
                        "tax": 3.2,
                    },
                },
                "converted": {
                    "summary": "String price auto-converts",
                    "value": {"name": "Bar", "price": "35.4"},
                },
                "invalid": {
                    "summary": "Invalid price",
                    "value": {"name": "Baz", "price": "thirty five"},
                },
            }
        )
    ]
):
    return {"item_id": item_id, "item": item}
```

### 📤 Example Request (Valid: `normal`)

```json
{
  "name": "Foo",
  "description": "A very nice Item",
  "price": 35.4,
  "tax": 3.2
}
```

### 📤 Example Request (Invalid)

```json
{
  "name": "Baz",
  "price": "thirty five"
}
```

---


