- Getting user credentials via form data using `OAuth2PasswordRequestForm`
- Simulating user authentication with a fake password hashing mechanism
- Protecting routes with a token using `OAuth2PasswordBearer`
- Validating the token (albeit in a "fake" way)
- Ensuring only active users get access

Here’s a summary of what’s going on and why it’s important:

---

### 🔐 `OAuth2PasswordBearer(tokenUrl="token")`
This is what declares your token-based authentication. It tells FastAPI where to expect the login (token) to happen — via a POST to `/token`.

### 📄 `OAuth2PasswordRequestForm`
This is the class that parses form-encoded data:
- `username`
- `password`
- `scope` (as a string, e.g. `"users:read users:write"`)

The form is required by the OAuth2 Password Flow spec — not JSON.

### 🧠 `fake_hash_password` and `fake_users_db`
You're mimicking hashing and a real database to simulate auth logic.

```python
def fake_hash_password(password: str):
    return "fakehashed" + password
```

In real life, you'd use something like `bcrypt` to hash and verify passwords securely.

### 🔐 `/token` route
This is where the user sends their `username` and `password`, and if correct, gets an "access token" back. You return the username as the access token (just for demo purposes).

```python
return {"access_token": user.username, "token_type": "bearer"}
```

### 📡 `get_current_user` and `get_current_active_user`
These are chained dependencies:
- `get_current_user`: gets the token, pretends to decode it, and returns a user
- `get_current_active_user`: makes sure that user is active (not disabled)

They’re used to protect routes like this one:

```python
@app.get("/users/me")
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)],
):
    return current_user
```

---

### ✅ What’s great about this example
- Fully aligns with OAuth2 Password Flow spec.
- Demonstrates form-based login.
- Uses layered dependencies to keep code modular and clear.
- Shows a clean separation between authentication (`get_current_user`) and authorization (`get_current_active_user`).

---

