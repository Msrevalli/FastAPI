`jsonable_encoder()` is **super handy** when you need to turn things like Pydantic models or datetime objects into **standard JSON-compatible Python data types**.

Let’s break it down even further with some **key points and a practical example**:

---

### 🔧 What `jsonable_encoder()` Does:

- Converts:
  - `Pydantic models` → `dict`
  - `datetime`, `UUID`, `Decimal`, etc. → `str` or other JSON-compatible formats
- Makes data compatible with:
  - `json.dumps()`
  - OR storing in a NoSQL database like MongoDB

---

### ✅ Example:

```python
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel
from datetime import datetime

app = FastAPI()

# fake database dictionary
fake_db = {}

# Pydantic model
class Item(BaseModel):
    title: str
    timestamp: datetime
    description: str | None = None

@app.put("/items/{id}")
def update_item(id: str, item: Item):
    # Convert to JSON-compatible dict
    json_compatible_item_data = jsonable_encoder(item)

    # Store in fake database
    fake_db[id] = json_compatible_item_data

    return {"msg": "Item updated", "data": json_compatible_item_data}
```

---

### 🧪 Output if you call with JSON like:

```json
{
  "title": "New Gadget",
  "timestamp": "2025-04-15T12:00:00",
  "description": "Latest update"
}
```

You'll store something like this in `fake_db`:

```python
{
  "title": "New Gadget",
  "timestamp": "2025-04-15T12:00:00",  # str, not datetime
  "description": "Latest update"
}
```

---

