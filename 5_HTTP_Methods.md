---

## 🧠 HTTP Methods Overview

| Method | Action | Real-World Analogy | Used For |
|--------|--------|--------------------|----------|
| `GET`    | Read data       | "Read me that file"        | Fetching data |
| `POST`   | Create new data | "Add a new file to the cabinet" | Creating something |
| `PUT`    | Replace data    | "Replace this file entirely" | Replacing something |
| `PATCH`  | Update data     | "Make a small change to the file" | Partially updating something |
| `DELETE` | Remove data     | "Remove that file from the cabinet" | Deleting something |

---

## 🔍 Detailed Breakdown

---

### 🟢 `GET` — **Read/FETCH data**

- **Used to**: Retrieve data from the server.
- **Does not** modify anything.
- **Safe** and **idempotent** (no matter how many times you call it, the result is the same).
- **No request body** (usually).

#### Example:
```http
GET /users/123
```

🔁 Returns info about the user with ID 123.

---

### 🟡 `POST` — **Create data**

- **Used to**: Submit new data to be created on the server.
- Typically includes a **request body** (with the data).
- Not idempotent: calling it multiple times usually creates **multiple entries**.

#### Example:
```http
POST /users
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com"
}
```

🧾 Creates a new user.

---

### 🟠 `PUT` — **Replace/Update all**

- **Used to**: Replace an existing resource **entirely**.
- Requires **full data** in the request.
- Idempotent: calling it again doesn’t create duplicates — it replaces the same data.

#### Example:
```http
PUT /users/123
Content-Type: application/json

{
  "name": "Alice Updated",
  "email": "alice@newmail.com"
}
```

✏️ Completely replaces the existing user’s info.

---

### 🟣 `PATCH` — **Partial Update**

- **Used to**: Update part of a resource.
- Only send the fields you want to change.
- Also **idempotent**.

#### Example:
```http
PATCH /users/123
Content-Type: application/json

{
  "email": "new@email.com"
}
```

✂️ Only updates the email, leaves everything else the same.

---

### 🔴 `DELETE` — **Remove resource**

- **Used to**: Delete a resource from the server.
- May or may not return data.
- **Idempotent** — deleting the same thing twice has the same effect (it’s just gone).

#### Example:
```http
DELETE /users/123
```

🗑️ Deletes the user with ID 123.

---

## 🧪 Quick Comparison Table

| Method  | Purpose            | Sends Body? | Idempotent? | Safe? |
|---------|--------------------|-------------|-------------|--------|
| `GET`    | Read data          | ❌ No        | ✅ Yes      | ✅ Yes |
| `POST`   | Create data        | ✅ Yes       | ❌ No       | ❌ No  |
| `PUT`    | Replace data       | ✅ Yes       | ✅ Yes      | ❌ No  |
| `PATCH`  | Partial update     | ✅ Yes       | ✅ Yes      | ❌ No  |
| `DELETE` | Remove data        | ❌ (or yes)  | ✅ Yes      | ❌ No  |

---

## 🎯 Use in FastAPI

In FastAPI, you choose the method like this:

```python
@app.get("/items/")
async def read_items():
    return {"message": "GET request"}

@app.post("/items/")
async def create_item(item: Item):
    return {"message": "POST request"}

@app.put("/items/{item_id}")
async def replace_item(item_id: int, item: Item):
    return {"message": "PUT request"}

@app.patch("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"message": "PATCH request"}

@app.delete("/items/{item_id}")
async def delete_item(item_id: int):
    return {"message": "DELETE request"}
```

---

## 🧠 Summary

| You want to...                 | Use this method |
|-------------------------------|------------------|
| Get data                      | `GET`            |
| Add a new record              | `POST`           |
| Completely replace a record   | `PUT`            |
| Partially update a record     | `PATCH`          |
| Delete a record               | `DELETE`         |

---

