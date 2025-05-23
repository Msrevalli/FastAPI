FastAPI script that demonstrates OAuth2 password-based authentication using a token system with dependency injection and a fake user database:

```python
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Wonderson",
        "email": "alice@example.com",
        "hashed_password": "fakehashedsecret2",
        "disabled": True,
    },
}

app = FastAPI()


def fake_hash_password(password: str):
    return "fakehashed" + password


oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None


class UserInDB(User):
    hashed_password: str


def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)


def fake_decode_token(token):
    # This doesn't provide any security at all
    # Check the next version
    user = get_user(fake_users_db, token)
    return user


async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    user = fake_decode_token(token)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return user


async def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)],
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    user_dict = fake_users_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user = UserInDB(**user_dict)
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detail="Incorrect username or password")

    return {"access_token": user.username, "token_type": "bearer"}


@app.get("/users/me")
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)],
):
    return current_user
```

This script sets up:

* A mock user database.
* OAuth2 password-based token authentication.
* Token issuance via `/token`.
* A secured endpoint `/users/me` that only active users can access.

---

## ✅ What It Does

* **Provides a `/token` endpoint** for login using `OAuth2PasswordRequestForm` credentials.
* **Verifies user credentials** against a `fake_users_db`.
* **Issues a token** (just the username for this fake example).
* **Protects `/users/me` endpoint** and ensures:

  * The token is valid.
  * The user exists and is not disabled.

---

## 🔧 **Fake Database**

```python
fake_users_db = {
    "johndoe": {...},
    "alice": {...},
}
```

* Simulates user data.
* Includes fields like:

  * `username`
  * `email`
  * `hashed_password`
  * `disabled`

---

## 🧪 **Fake Password Hasher**

```python
def fake_hash_password(password: str):
    return "fakehashed" + password
```

* Mocks password hashing.
* You would replace this with something like `bcrypt` in production.

---

## 🔐 **OAuth2PasswordBearer**

```python
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
```

* Declares where clients get the token (via `/token` endpoint).
* Parses the `Authorization: Bearer <token>` header in incoming requests.

---

## 👤 **Pydantic Models**

```python
class User(BaseModel):
    ...
class UserInDB(User):
    hashed_password: str
```

* `User`: For output/public use (no password).
* `UserInDB`: For internal use (includes hashed password).

---

## 🧠 **User Lookup and Token Decoding**

```python
def get_user(db, username: str)
def fake_decode_token(token)
```

* `get_user()`: Pulls a user from the fake DB.
* `fake_decode_token()`: Treats the token as a username (insecure, but for demonstration).

---

## 🔒 **Authentication Dependencies**

### Get Current User

```python
async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)])
```

* Decodes the token (username).
* Fetches the user.
* If user not found → raises `401 Unauthorized`.

### Get Current Active User

```python
async def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)]
)
```

* Ensures the user is **not disabled**.
* Otherwise → raises `400 Bad Request`.

---

## 🔐 **Login Endpoint: `/token`**

```python
@app.post("/token")
async def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()])
```

* Validates username/password.
* Hashes the password and compares it to the stored one.
* If valid → returns an access token.

✅ Example Response:

```json
{
  "access_token": "johndoe",
  "token_type": "bearer"
}
```

---

## 🧾 **Protected Endpoint: `/users/me`**

```python
@app.get("/users/me")
async def read_users_me(current_user: Annotated[User, Depends(get_current_active_user)])
```

* Requires a valid token (in Authorization header).
* Returns the current user info if they’re active.

---

## 🧪 Test Flow Summary

1. **Login**:

   ```bash
   curl -X POST http://localhost:8000/token \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "username=johndoe&password=secret"
   ```

2. **Access Protected Route**:

   ```bash
   curl http://localhost:8000/users/me \
     -H "Authorization: Bearer johndoe"
   ```

---

## 🧪 Example Request Flow

### 1. Get a Token

```bash
curl -X POST http://127.0.0.1:8000/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=johndoe&password=secret"
```

✅ Response:

```json
{
  "access_token": "johndoe",
  "token_type": "bearer"
}
```

### 2. Use the Token to Access Protected Route

```bash
curl http://127.0.0.1:8000/users/me \
  -H "Authorization: Bearer johndoe"
```

✅ Response:

```json
{
  "username": "johndoe",
  "email": "johndoe@example.com",
  "full_name": "John Doe",
  "disabled": false
}
```

---

## 🔐 How Security Works

* **Token creation**: Just returns the username as the token.
* **Token decoding**: Looks up the username from the token in the `fake_users_db`.
* **Dependencies**:

  * `get_current_user` verifies the token maps to a user.
  * `get_current_active_user` checks if the user is not disabled.

---

## ✅ Realistic Improvements You Could Add

* Replace `fake_decode_token()` with real JWT handling using `pyjwt` or `python-jose`.
* Secure the password hash check using `passlib` or another library.
* Add scopes, roles, or permission-based control if needed.

Would you like an enhanced version using JWT and hashed passwords?
