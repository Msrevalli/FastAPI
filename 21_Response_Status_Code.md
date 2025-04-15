## 🔢 What Is `status_code`?

Every HTTP response includes a **status code**, a 3-digit number that tells the client **what happened** with the request.

In FastAPI, you can **specify the status code** by using the `status_code` parameter in the route decorator (not in the function!).

### ✅ Example

```python
from fastapi import FastAPI

app = FastAPI()

@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```

- The endpoint will return:
  - HTTP status **201 Created**
  - JSON body: `{"name": "some-name"}`

---

## 📚 Status Code Categories

| Range        | Category        | Notes |
|--------------|------------------|-------|
| 100–199      | Informational    | Rarely used |
| 200–299      | ✅ Success        | Most common in APIs |
| 300–399      | Redirection      | Less common in APIs |
| 400–499      | ❌ Client Error   | For invalid inputs |
| 500–599      | 💥 Server Error   | Internal issues, usually automatic |

---

## ✍️ Commonly Used Status Codes

| Code | Name          | Use Case |
|------|---------------|----------|
| `200` | OK            | Default for `GET`, successful response |
| `201` | Created       | After creating a resource (POST) |
| `204` | No Content    | Successful but no content to return |
| `400` | Bad Request   | Validation or request error |
| `404` | Not Found     | Resource not found |
| `422` | Unprocessable Entity | Auto-used by FastAPI when validation fails |
| `500` | Internal Server Error | Not usually set manually |

---

## 🤓 Using `status` Enum for Autocomplete

You don’t need to memorize all status codes. FastAPI (via Starlette) provides named constants for each code.

### ✅ Example with Enum

```python
from fastapi import FastAPI, status

app = FastAPI()

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}
```

Benefits:
- More readable
- Autocomplete in editors like VS Code
- Less prone to mistakes (like typing `210` instead of `201`)

---

## ⚠️ Note on `status_code`

- It must be passed to the **decorator**, not to the function:
  
  ❌ This is incorrect:
  ```python
  async def create_item(name: str, status_code=201):
  ```

  ✅ Correct way:
  ```python
  @app.post("/items/", status_code=201)
  ```

---

## 🧪 Special Case: `204 No Content`

When you use:

```python
@app.delete("/items/{item_id}", status_code=204)
```

FastAPI knows this code **should not return a body**, and will:
- Document it in the OpenAPI schema
- Automatically prevent sending a body in the response

---

## 🧩 Alternative: Use Python's `http.HTTPStatus`

```python
from fastapi import FastAPI
from http import HTTPStatus

app = FastAPI()

@app.post("/items/", status_code=HTTPStatus.CREATED)
async def create_item(name: str):
    return {"name": name}
```

It works the same way! FastAPI accepts either:
- Integer (e.g., `201`)
- Enum (e.g., `HTTPStatus.CREATED` or `status.HTTP_201_CREATED`)

---

## ✅ TL;DR Recap

- `status_code` lets you set the correct HTTP status for your route.
- Use integer codes (`201`) or enums (`status.HTTP_201_CREATED`) for readability.
- Decorate your path functions with the right status to follow HTTP standards.
- Helps clients understand what happened (e.g., created, no content, not found).
- OpenAPI docs will show correct response codes and bodies automatically.

---

