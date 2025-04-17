## 🔐 What You're Building

You're creating a **secure login system** with:

1. ✅ Username & password login
2. ✅ JWT token generation
3. ✅ Token-based authentication (Bearer)
4. ✅ Password hashing (so passwords aren’t stored in plain text)
5. ✅ Protected routes that only logged-in users can access

---

## 🧱 How It Works (Step by Step)

### 1. **Fake Database**

You define a fake user database with hashed password (bcrypt):

```python
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        ...
        "hashed_password": "...",  # bcrypt hash of "secret"
        "disabled": False,
    }
}
```

---

### 2. **Password Hashing and Checking**

- `get_password_hash(password)` → Converts a plain password to a hashed one
- `verify_password(plain, hashed)` → Verifies if the plain password matches the hashed one

These are secure thanks to **bcrypt**.

---

### 3. **User Models**

You define different models for:

- `User` → Public info (what you send to clients)
- `UserInDB` → Internal use (includes the hashed password)
- `Token` → What your login returns (JWT token)
- `TokenData` → What you extract from the JWT token

---

### 4. **OAuth2 & JWT Setup**

You set up OAuth2 with:

```python
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
```

This says: “Get the token from the `Authorization: Bearer <token>` header, and users must log in at `/token`.”

---

### 5. **Login Endpoint `/token`**

This route:

- Accepts username + password (via form)
- Verifies the user and password
- Creates a **JWT token** if login is successful

```python
@app.post("/token")
async def login_for_access_token(...):
    user = authenticate_user(...)
    access_token = create_access_token(data={"sub": user.username})
    return {"access_token": access_token, "token_type": "bearer"}
```

📝 The JWT contains `sub` (subject = username) and `exp` (expiry time).

---

### 6. **Creating the JWT Token**

```python
def create_access_token(data, expires_delta):
    data["exp"] = datetime.utcnow() + expires_delta
    return jwt.encode(data, SECRET_KEY, algorithm="HS256")
```

This creates a signed token. It is:
- Not encrypted (anyone can read it),
- But **tamper-proof** (signed with a secret key).

---

### 7. **Get the Current User from Token**

This function:

- Decodes the JWT token
- Checks that it’s valid and not expired
- Gets the user from the database

```python
async def get_current_user(token: Depends(oauth2_scheme)):
    payload = jwt.decode(token, SECRET_KEY)
    username = payload.get("sub")
    user = get_user(fake_users_db, username)
    return user
```

If anything fails → raise `401 Unauthorized`.

---

### 8. **Protecting Routes**

You use `Depends(get_current_active_user)` to protect routes:

```python
@app.get("/users/me")
async def read_users_me(current_user: Depends(get_current_active_user)):
    return current_user
```

This only works **if the user has a valid token**.

---

## ✅ What You Have Built

| Feature                     | Done |
|----------------------------|------|
| Secure password hashing    | ✅   |
| Login with username+password | ✅ |
| JWT token creation         | ✅   |
| Token-based authentication | ✅   |
| Protected routes           | ✅   |

---

