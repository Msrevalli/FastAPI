**Full updates with `PUT`** and **partial updates with `PATCH`**, along with using `jsonable_encoder()` and Pydantic's `.model_dump()` and `.copy()` methods. Here's a quick **summary and cheat sheet** to make things crystal clear, plus some notes to help avoid common pitfalls 👇

---

## 🔁 Full Update with PUT

### ✅ Use Case:
When you want to **replace** the whole item.

### ⚠️ Gotcha:
If you omit a field in the input, it'll be **replaced with the default value** — not preserved from the existing item.

```python
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str | None = None
    description: str | None = None
    price: float | None = None
    tax: float = 10.5
    tags: list[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]


@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    update_item_encoded = jsonable_encoder(item)
    items[item_id] = update_item_encoded
    return update_item_encoded
```
---

### ✅ `GET /items/foo`

**Request:**

```http
GET /items/foo
```

**Response:**

```json
{
  "name": "Foo",
  "description": null,
  "price": 50.2,
  "tax": 10.5,
  "tags": []
}
```

---

### ✅ `PUT /items/foo`

**Request:**

```http
PUT /items/foo
Content-Type: application/json

{
  "name": "Updated Foo",
  "description": "A new Foo",
  "price": 60.0,
  "tax": 15.0,
  "tags": ["updated", "important"]
}
```

**Response:**

```json
{
  "name": "Updated Foo",
  "description": "A new Foo",
  "price": 60.0,
  "tax": 15.0,
  "tags": ["updated", "important"]
}
```

---

### ✅ `GET /items/bar`

**Response:**

```json
{
  "name": "Bar",
  "description": "The bartenders",
  "price": 62.0,
  "tax": 20.2,
  "tags": []
}
```

---

If you request a non-existent item like `/items/xyz`, you'll currently get a **500 Internal Server Error**, because there's no error handling for missing keys. 

---

## ⚠️ The Risk with `PUT` and Default Values

In your model:

```python
class Item(BaseModel):
    name: str | None = None
    description: str | None = None
    price: float | None = None
    tax: float = 10.5  # <- default value
    tags: list[str] = []
```

And in your `PUT` endpoint:

```python
@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    update_item_encoded = jsonable_encoder(item)
    items[item_id] = update_item_encoded
    return update_item_encoded
```

---

### 🧪 If You Send:

```json
{
  "name": "Barz",
  "price": 3,
  "description": null
}
```

* You **did not include** `tax`.
* Pydantic will assign the default: `tax = 10.5`.
* Your old value `tax = 20.2` is now **overwritten**!

---

### 🧨 Result in `items["bar"]`:

```json
{
  "name": "Barz",
  "description": null,
  "price": 3,
  "tax": 10.5,       ← ⚠️ NOT the original 20.2
  "tags": []
}
```

---

## ✅ Safer Alternatives

### Option 1: Use `PATCH` instead of `PUT`

* `PUT` means **replace** the entire resource.
* `PATCH` means **partially update** only the fields provided.

You’d define a `PartialItem` model where all fields are optional, and update only what's provided.

### Option 2: Manually merge fields

```python
@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    update_data = item.dict(exclude_unset=True)
    stored_item_data.update(update_data)
    items[item_id] = jsonable_encoder(stored_item_data)
    return items[item_id]
```

* `exclude_unset=True` ensures only fields sent in the request are updated.
* Fields not sent (like `tax`) remain unchanged.

---

## 🧩 Partial Update with PATCH

### ✅ Use Case:
When you want to update only certain fields, and **leave others untouched**.

### 🔑 Steps:
1. Get existing item from DB
2. Create a model from it
3. Get the fields from the request using `.model_dump(exclude_unset=True)`
4. Merge with original using `.model_copy(update=...)`
5. Encode with `jsonable_encoder`
6. Save and return

```python
@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```

You are now:

1. **Loading** the existing item from memory.
2. **Extracting only the fields** that were sent in the request using `exclude_unset=True`.
3. **Merging** the new fields into the existing model using `.copy(update=...)`.
4. **Encoding** and saving the merged result.

---

## ✅ Example Input

```http
PATCH /items/bar
Content-Type: application/json

{
  "name": "Barz",
  "price": 3,
  "description": null
}
```

### ✅ Output:

```json
{
  "name": "Barz",
  "description": null,
  "price": 3,
  "tax": 20.2,         // 🔒 Preserved!
  "tags": []           // 🧼 Also preserved!
}
```

---

## 🧠 Summary

* ✅ Safer than `PUT` for partial updates.
* ✅ Prevents accidental overwriting of fields like `tax`.
* ✅ Ideal when clients may only want to update a few fields.

---

## 🆚 PUT vs PATCH

| Feature                  | PUT                                 | PATCH                                       |
|--------------------------|--------------------------------------|---------------------------------------------|
| Use Case                 | Replace full item                   | Update only specific fields                 |
| Input Requirements       | All fields required or defaulted    | All fields optional                         |
| Preserves Existing Data? | ❌ No — replaced with default values | ✅ Yes — if fields not provided              |
| FastAPI Restriction?     | ❌ None                              | ❌ None                                      |
| Best Practice            | Use for full replace                | Use for partial update                      |

---

## 🧠 Extra Pro Tip

To avoid mixing update and create logic, you can use **two different models**:

```python
class ItemCreate(BaseModel):
    name: str
    price: float
    # required fields only

class ItemUpdate(BaseModel):
    name: str | None = None
    price: float | None = None
    # all fields optional
```

This is the **"Extra Models"** idea mentioned at the end.

---
