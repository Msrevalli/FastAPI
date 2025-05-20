# **FastAPI Additional Status Codes **

FastAPI allows you to return **custom HTTP status codes** beyond the default ones. This is useful when you need to:
- Return different status codes based on conditions (e.g., `200 OK` vs. `201 Created`)
- Manually control responses for special cases (e.g., `404 Not Found`)

---

## **1. Default Behavior**
By default, FastAPI:
- Wraps your response in a `JSONResponse`.
- Uses the **default status code** (`200 OK` for most methods, `201 Created` for `POST`).
- You can override this with `status_code` in the decorator:
  ```python
  @app.post("/items/", status_code=201)
  ```

---

## **2. Returning Different Status Codes Dynamically**
Sometimes, you need to return **different status codes** based on logic (e.g., `200` for updates vs. `201` for creations).

### **Example: Upsert (Update or Create) Endpoint**
```python
from fastapi import FastAPI, Body, status
from fastapi.responses import JSONResponse

app = FastAPI()

items = {"foo": {"name": "Fighters", "size": 6}}

@app.put("/items/{item_id}")
async def upsert_item(
    item_id: str,
    name: str = Body(...),
    size: int = Body(...),
):
    if item_id in items:  # Update existing item → 200 OK
        items[item_id] = {"name": name, "size": size}
        return items[item_id]
    else:  # Create new item → 201 Created
        new_item = {"name": name, "size": size}
        items[item_id] = new_item
        return JSONResponse(
            status_code=status.HTTP_201_CREATED,
            content=new_item
        )
```
- **If item exists:** Returns `200 OK` (default for `PUT`).
- **If item is new:** Returns `201 Created` using `JSONResponse`.

---

## **3. Key Considerations**
### **Manually Returning Responses**
- When you return `JSONResponse`, **FastAPI skips automatic serialization**.
- Ensure the `content` is **valid JSON**.
- You can also use `PlainTextResponse`, `HTMLResponse`, etc.

### **Status Code Constants**
FastAPI provides `status` for HTTP codes (instead of hardcoding numbers):
```python
from fastapi import status

status.HTTP_200_OK      # 200
status.HTTP_201_CREATED # 201
status.HTTP_404_NOT_FOUND # 404
```

---

## **4. OpenAPI Documentation**
### **Problem**
- Manually returned responses **won’t appear in OpenAPI docs**.
- FastAPI can’t predict dynamic status codes.

### **Solution: Document Additional Responses**
Use `responses` in the decorator:
```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.put(
    "/items/{item_id}",
    responses={
        200: {"description": "Item updated successfully"},
        201: {"description": "Item created successfully"},
    }
)
async def upsert_item(item_id: str):
    if item_exists(item_id):
        return {"message": "Updated"}
    else:
        return JSONResponse(
            status_code=201,
            content={"message": "Created"}
        )
```
This ensures Swagger UI shows all possible responses.

---

## **5. Common Use Cases**
| Status Code | Scenario |
|-------------|----------|
| **200 OK** | Success (default for GET, PUT, DELETE) |
| **201 Created** | Resource created (POST/PUT) |
| **204 No Content** | Success with empty body (DELETE) |
| **400 Bad Request** | Invalid input |
| **404 Not Found** | Resource missing |
| **422 Unprocessable Entity** | Pydantic validation error |

---

## **6. Example: Full CRUD with Custom Status Codes**
```python
from fastapi import FastAPI, status, HTTPException
from fastapi.responses import JSONResponse

app = FastAPI()
items = {}

# Create (POST) → 201
@app.post("/items/", status_code=201)
async def create_item(item: dict):
    items[item["id"]] = item
    return item

# Read (GET) → 200 (or 404)
@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(404, detail="Item not found")
    return items[item_id]

# Update (PUT) → 200 or 201
@app.put("/items/{item_id}")
async def update_item(item_id: str, item: dict):
    if item_id in items:
        items[item_id] = item
        return item
    else:
        items[item_id] = item
        return JSONResponse(
            status_code=201,
            content={"message": "Item created"}
        )

# Delete (DELETE) → 204
@app.delete("/items/{item_id}", status_code=204)
async def delete_item(item_id: str):
    if item_id not in items:
        raise HTTPException(404, detail="Item not found")
    del items[item_id]
    return None  # 204 = No content
```

---

## **Summary**
| Technique | Usage |
|-----------|-------|
| **Default status_code** | Set in `@app` decorator |
| **JSONResponse** | Manually return responses |
| **status.HTTP_** | Predefined status codes |
| **responses={}** | Document in OpenAPI |

This gives you full control over HTTP responses while keeping API docs accurate. 🚀