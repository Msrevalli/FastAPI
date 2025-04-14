### ✅ **1. Examples via Pydantic Model Config (JSON Schema Extra)**

**Best for:** Global model-level examples (shown in docs and OpenAPI schema)

```python
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
```

---

### ✅ **2. Examples via `Field()`**

**Best for:** Field-level examples

```python
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str = Field(examples=["Foo"])
    description: str | None = Field(default=None, examples=["A very nice Item"])
    price: float = Field(examples=[35.4])
    tax: float | None = Field(default=None, examples=[3.2])
```

---

### ✅ **3. Examples via `Body(examples=...)`**

**Best for:** Parameter-level examples for a request body

```python
from typing import Annotated
from fastapi import Body

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
    ...
```

---

### ✅ **4. Multiple Body Examples using `Body(examples=[...])`**

**Best for:** Demonstrating good, invalid, or alternative examples

```python
item: Annotated[
    Item,
    Body(
        examples=[
            {"name": "Foo", "description": "Nice", "price": 35.4, "tax": 3.2},
            {"name": "Bar", "price": "35.4"},  # Auto conversion
            {"name": "Baz", "price": "thirty five"},  # Invalid
        ]
    )
]
```

---

### ✅ **5. OpenAPI-Specific Examples via `openapi_examples`**

**Best for:** Adding detailed, named examples with descriptions in the Swagger UI

```python
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
```

---

### ⚙️ Version Notes

- **Use FastAPI ≥ 0.99.0** to benefit from OpenAPI 3.1 and JSON Schema `examples`.
- **Use FastAPI ≥ 0.103.0** to use `openapi_examples`.

---
