## 🛡️ What Is HTTP Basic Auth?

* It's the simplest way to authenticate HTTP requests.
* The client sends an `Authorization: Basic <base64-encoded-username:password>` header.
* If missing or wrong, the server responds with:

  ```http
  401 Unauthorized
  WWW-Authenticate: Basic realm="..."
  ```
* Browsers will then prompt the user for username/password.

✅ It's built into HTTP — no token handling or cookies needed.

---

## ✅ Basic Implementation in FastAPI

### 1. 📦 Import Modules and Set Up

```python
from fastapi.security import HTTPBasic, HTTPBasicCredentials
from fastapi import FastAPI, Depends
from typing import Annotated
```

* `HTTPBasic` creates the security scheme.
* `HTTPBasicCredentials` will hold the `username` and `password` provided by the client.

---

### 2. 🔐 Define Security Scheme

```python
security = HTTPBasic()
```

This will:

* Register the "Basic" scheme in the OpenAPI docs.
* Parse the `Authorization: Basic ...` header.

---

### 3. 📥 Get Credentials Securely

```python
def get_current_username(credentials: Annotated[HTTPBasicCredentials, Depends(security)]):
```

* FastAPI uses this to automatically extract the username/password from the HTTP header.
* Returns a `HTTPBasicCredentials` object with `.username` and `.password`.

---

### 4. 🔒 Secure Username & Password Check

```python
import secrets

current_username_bytes = credentials.username.encode("utf8")
correct_username_bytes = b"stanleyjobson"

is_correct_username = secrets.compare_digest(current_username_bytes, correct_username_bytes)
```

> Why not just `credentials.username == "stanleyjobson"`?

* Because that’s **vulnerable to timing attacks**.
* `secrets.compare_digest()` takes **constant time**, regardless of how much of the string is correct.
* This prevents attackers from guessing your credentials one character at a time based on response timing.

✅ Same logic applies for password checking.

---

### 5. 🚫 Raise Error if Invalid

```python
if not (is_correct_username and is_correct_password):
    raise HTTPException(
        status_code=401,
        detail="Incorrect username or password",
        headers={"WWW-Authenticate": "Basic"},
    )
```

* Returns `401 Unauthorized`
* Adds the `WWW-Authenticate` header so browsers prompt again.

---

### 6. ✅ Use It in Endpoint

```python
@app.get("/users/me")
def read_current_user(username: Annotated[str, Depends(get_current_username)]):
    return {"username": username}
```

* If credentials are valid, it returns the username.
* If not, the user is blocked with `401 Unauthorized`.

---

## 📊 How the Full Flow Works

```plaintext
[1] Client makes request → GET /users/me
[2] Server responds:
    401 Unauthorized
    WWW-Authenticate: Basic
[3] Browser prompts: "Username & Password?"
[4] User enters: stanleyjobson / swordfish
[5] Browser sends header:
    Authorization: Basic c3RhbmxleWpvYnNvbjpzd29yZGZpc2g=
[6] FastAPI extracts credentials via Depends(HTTPBasic)
[7] Checks them using secrets.compare_digest
[8] Returns {"username": "stanleyjobson"} if correct
```

---

## 🔒 Timing Attacks Explained Visually

If you use:

```python
credentials.username == "admin"
```

The server might:

* Fail fast if the first character doesn’t match.
* Take longer if more characters are correct.

An attacker can measure response time and **guess one character at a time**.

With:

```python
secrets.compare_digest()
```

* Every comparison takes the same time.
* No timing difference.
* Much harder (practically infeasible) to attack.

---

## 🧠 Takeaways

* **HTTP Basic Auth** is useful for:

  * Internal tools
  * Dev-only routes
  * Simple admin panels

* Don’t store passwords in code in real apps. Instead:

  * Hash passwords (e.g., with `bcrypt`)
  * Validate them securely
  * Store credentials in environment variables or secrets managers

---

## 🚀 Bonus: How to Add Role Checks or Scope-Like Logic

You can extend the `get_current_username` function like this:

```python
def get_current_username(credentials: Annotated[HTTPBasicCredentials, Depends(security)]):
    ...
    if credentials.username == "admin":
        return {"username": credentials.username, "role": "admin"}
    return {"username": credentials.username, "role": "user"}
```

Then in your route:

```python
@app.get("/admin")
def read_admin(user: Annotated[dict, Depends(get_current_username)]):
    if user["role"] != "admin":
        raise HTTPException(status_code=403, detail="Access forbidden")
    return {"message": "Welcome admin"}
```

---

