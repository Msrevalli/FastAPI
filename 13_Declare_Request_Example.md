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

Here's a detailed explanation of the **differences between the 5 types of examples** in FastAPI using Pydantic and OpenAPI — including when to use each and how they affect your API documentation:

---

## ✅ **1. Model-Level Example via `model_config` (`json_schema_extra`)**

### 🔍 Purpose

* Declares a **global example for the entire model**.
* Used automatically **anywhere that model is used** as input or output.

### ✅ Use When

* You want a **default example for documentation** reused across endpoints.

### 🧠 Notes

* Declared in the **Pydantic model itself**.
* Automatically shown in OpenAPI (Swagger UI) without needing to add it in `Body()`.

### 📌 Pro

* DRY: Define once, use anywhere the model is used.

### ⚠️ Con

* Not suitable for endpoints needing **different examples** for the same model.

---

## ✅ **2. Field-Level Examples via `Field(examples=[...])`**

### 🔍 Purpose

* Shows **per-field examples** in the docs.
* Helps users know expected data formats for each field.

### ✅ Use When

* You want to **guide users** with small, field-specific hints.

### 🧠 Notes

* Appears next to each field in Swagger UI.
* Works well with `description=...` in the same field.

### 📌 Pro

* Very detailed and fine-grained.

### ⚠️ Con

* Not visible as a full request example.

---

## ✅ **3. Body-Level Example via `Body(examples=[...])`**

### 🔍 Purpose

* Attach an example to a **single endpoint’s request body**.
* Allows overriding the model’s global example.

### ✅ Use When

* You need **different examples for the same model** depending on the route.

### 🧠 Notes

* Declared inline using `Body(...)` with `Annotated` or `Depends`.
* Only affects that specific endpoint.

### 📌 Pro

* Flexible and route-specific.

### ⚠️ Con

* Redundant if model-level example is sufficient.

---

## ✅ **4. Multiple Body Examples via `Body(examples=[...])`**

### 🔍 Purpose

* Shows **multiple unnamed examples** in Swagger.
* Great for showing valid, edge-case, and invalid formats.

### ✅ Use When

* You want to educate API users with **multiple possibilities**.

### 🧠 Notes

* Helpful for frontend/backend developers to understand input variety.
* Shown as a dropdown in Swagger UI.

### 📌 Pro

* Improves developer understanding with multiple test cases.

### ⚠️ Con

* Lacks summary/description for each case (see next type).

---

## ✅ **5. Named Examples via `openapi_examples`**

### 🔍 Purpose

* Adds **named, detailed, and described examples**.
* Shown with **title + description** in Swagger dropdown.

### ✅ Use When

* You want Swagger UI to show **rich, labeled examples** like:

  * “normal”
  * “converted”
  * “invalid”

### 🧠 Notes

* Best UX in documentation for APIs consumed by other devs.

### 📌 Pro

* Powerful, descriptive, and user-friendly.

### ⚠️ Con

* Slightly more verbose to write.

---

## 🔚 Summary Table

| Type                         | Scope        | Shown In Docs | Allows Multiple | Allows Descriptions |
| ---------------------------- | ------------ | ------------- | --------------- | ------------------- |
| `model_config`               | Model-wide   | ✅ Yes         | ❌ No            | ❌ No                |
| `Field(examples=...)`        | Per Field    | ✅ Yes         | ✅ Yes           | ❌ No                |
| `Body(examples=...)`         | Per Endpoint | ✅ Yes         | ✅ Yes           | ❌ No                |
| `Body(examples=[...])`       | Per Endpoint | ✅ Yes         | ✅ Yes           | ❌ No                |
| `Body(openapi_examples=...)` | Per Endpoint | ✅ Yes         | ✅ Yes           | ✅ Yes               |

---


