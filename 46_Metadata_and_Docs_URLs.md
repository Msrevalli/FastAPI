Here's a condensed and structured **cheat sheet** to help you quickly remember how to customize FastAPI's **metadata and docs URLs** — great for when you're polishing your API for others or for production.

---

## 🧠 FastAPI Metadata Cheat Sheet

### ✅ **Main API Metadata**
Set these when creating your `FastAPI()` app instance:

```python
from fastapi import FastAPI

description = """
Awesome API to do **cool stuff**. 🚀

## Users
* Create users
* Read users
"""

app = FastAPI(
    title="CoolApp",
    summary="Just a cool app.",
    description=description,
    version="1.0.0",
    terms_of_service="https://example.com/terms/",
    contact={
        "name": "Your Name",
        "url": "https://yourwebsite.com/contact/",
        "email": "you@example.com",
    },
    license_info={
        "name": "MIT",
        "url": "https://opensource.org/licenses/MIT",
        # or just use an identifier (OpenAPI 3.1+)
        # "identifier": "MIT"
    },
)
```

---

### 🏷️ **Tag Metadata**
Use `openapi_tags` to group and describe endpoints:

```python
tags_metadata = [
    {
        "name": "users",
        "description": "Operations related to users.",
    },
    {
        "name": "items",
        "description": "Manage your **fancy** items.",
        "externalDocs": {
            "description": "Find more info here",
            "url": "https://fastapi.tiangolo.com/",
        },
    },
]

app = FastAPI(openapi_tags=tags_metadata)
```

And then use tags in your endpoints:

```python
@app.get("/users/", tags=["users"])
async def get_users():
    return [{"name": "Alice"}]

@app.get("/items/", tags=["items"])
async def get_items():
    return [{"name": "Sword"}]
```

---

### 🌐 **Customize Docs URLs**

```python
app = FastAPI(
    openapi_url="/api/v1/openapi.json",   # OpenAPI schema
    docs_url="/documentation",            # Swagger UI
    redoc_url=None                        # Disable ReDoc
)
```

> To completely disable all docs:
```python
app = FastAPI(openapi_url=None, docs_url=None, redoc_url=None)
```

---

### 🔤 **Order of Tags**
The order in `openapi_tags` determines the section order in the docs UI.

---

### ✅ Summary
| Feature             | Parameter             | Example Value                            |
|---------------------|------------------------|-------------------------------------------|
| Title               | `title`               | `"Cool API"`                              |
| Summary             | `summary`             | `"Does cool things"`                      |
| Description         | `description`         | `"Multi-line Markdown description"`       |
| Version             | `version`             | `"1.0.0"`                                 |
| Terms of Service    | `terms_of_service`    | `"https://example.com/terms"`             |
| Contact             | `contact` (dict)      | `{"name": ..., "email": ..., "url": ...}` |
| License             | `license_info` (dict) | `{"name": ..., "url": ...}`               |
| Tags Metadata       | `openapi_tags`        | List of dicts (tags)                      |
| Swagger UI URL      | `docs_url`            | `"/documentation"`                        |
| ReDoc URL           | `redoc_url`           | `None` or `"/redoc"`                      |
| OpenAPI Schema URL  | `openapi_url`         | `"/openapi.json"` or custom path          |

---

