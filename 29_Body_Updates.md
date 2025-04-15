**full updates with `PUT`** and **partial updates with `PATCH`**, along with using `jsonable_encoder()` and Pydantic's `.model_dump()` and `.copy()` methods. Here's a quick **summary and cheat sheet** to make things crystal clear, plus some notes to help avoid common pitfalls 👇

---

## 🔁 Full Update with PUT

### ✅ Use Case:
When you want to **replace** the whole item.

### ⚠️ Gotcha:
If you omit a field in the input, it'll be **replaced with the default value** — not preserved from the existing item.

```python
@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    update_item_encoded = jsonable_encoder(item)
    items[item_id] = update_item_encoded
    return update_item_encoded
```

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

### ✅ Example:

```python
@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]  # step 1
    stored_item_model = Item(**stored_item_data)  # step 2
    update_data = item.model_dump(exclude_unset=True)  # step 3
    updated_item = stored_item_model.model_copy(update=update_data)  # step 4
    items[item_id] = jsonable_encoder(updated_item)  # step 5 & 6
    return updated_item
```

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

L