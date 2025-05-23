## ✅ OAuth2PasswordBearer Dependency

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

# Declare where the token can be obtained from (usually /token endpoint)
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


@app.get("/items/")
async def read_items(token: Annotated[str, Depends(oauth2_scheme)]):
    return {"token": token}
```

---

## 💬 Example HTTP Request

You must send the token in an `Authorization` header like this:

```
GET /items/ HTTP/1.1
Host: 127.0.0.1:8000
Authorization: Bearer mysecrettoken123
```

---

## 📦 Example Response

```json
{
  "token": "mysecrettoken123"
}
```

---

## ⚙️ What FastAPI Does

1. Extracts the `Authorization` header.
2. Validates that it starts with `Bearer `.
3. Passes the token (e.g., `mysecrettoken123`) to your route handler as the `token` argument.

---

## 🛠 Next Step (usually required)

This only extracts the token. You still need to:

* Validate the token (e.g., with JWT)
* Decode user identity
* Enforce scopes/roles

Let me know if you want an example with token validation or `/token` endpoint setup!

```
---

Here’s a teaser of what’s coming:

```python
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordRequestForm

@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    username = form_data.username
    password = form_data.password
    # Verify credentials, generate token...
    return {"access_token": "some-token", "token_type": "bearer"}
```

---

## 🧠 Summary

| Concept               | What It Does                                 |
|------------------------|-----------------------------------------------|
| `OAuth2PasswordBearer` | Dependency to extract Bearer token            |
| `tokenUrl="token"`     | URL where clients can request a token         |
| `/items/`              | Protected route that expects a Bearer token   |

---
## 🔐 What You Have Now

You've implemented this:

```python
from fastapi import FastAPI, Depends
from fastapi.security import OAuth2PasswordBearer
from typing import Annotated

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/items/")
async def read_items(token: Annotated[str, Depends(oauth2_scheme)]):
    return {"token": token}
```

This does:

- Defines a **security scheme** that expects an `Authorization: Bearer <token>` header.
- Integrates into FastAPI's OpenAPI docs (try `/docs` or `/redoc` to see it).
- Automatically returns **401 Unauthorized** if the header is missing or invalid format.
- Provides the `token` string to your route.

🧠 **Key takeaway**: FastAPI is already protecting your endpoint and injecting the Bearer token automatically!

---

## 🚧 What’s Missing (And What’s Next)

Right now, this token could be anything — it's not being **validated** or **generated**. Your API isn't checking whether it's real, expired, or tied to a user.

So next steps are:

### ✅ 1. Create the `/token` route
This will:
- Accept `username` and `password` as form data
- Verify credentials
- Return an access token (usually JWT)

### 🧾 2. Use `OAuth2PasswordRequestForm`
FastAPI has a built-in dependency that parses the form.

### 🔑 3. Generate a JWT token
You'll use `python-jose` or `PyJWT` to create secure tokens.

### 🧍 4. Decode the token and retrieve the user in other routes
Create a `get_current_user` dependency that decodes the token and returns user info.

---

### 🧪 Want a Quick Preview?

Here's what your `/token` route might look like:

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from typing import Annotated

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    if form_data.username != "admin" or form_data.password != "secret":
        raise HTTPException(status_code=400, detail="Incorrect credentials")
    
    return {"access_token": "fake-jwt-token", "token_type": "bearer"}
```

🔐 This lets you get a fake token from `/token`, and use it in the `/items/` endpoint.

Your script defines a minimal OAuth2 password flow using FastAPI. Here's a breakdown of what it does and what to expect when you use it.

---

## 📤 Example Request

Make a `POST` request to `/token` with `x-www-form-urlencoded` data:

### Request:

```
POST /token
Content-Type: application/x-www-form-urlencoded

username=admin&password=secret
```

---

## 📥 Example Response

```json
{
  "access_token": "fake-jwt-token",
  "token_type": "bearer"
}
```

---

## 🔒 How to Use This Token

Once you get the token, you can make authenticated requests to endpoints that depend on `oauth2_scheme` like this:

```http
GET /items/
Authorization: Bearer fake-jwt-token
```


