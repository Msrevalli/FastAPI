# FastAPI OAuth2 Scopes Implementation Summary

This example demonstrates how to implement OAuth2 scopes in FastAPI for fine-grained permission control. Here are the key components:

## Key Concepts

1. **OAuth2 Scopes**: Strings that declare specific permissions (e.g., `me` for user info, `items` for item access)
2. **Security Scheme**: Configured with available scopes and descriptions
3. **JWT Tokens**: Include granted scopes in the token payload
4. **Scope Validation**: Checks if token has required scopes for each endpoint

## Main Components

### Security Configuration
```python
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={
        "me": "Read information about the current user.",
        "items": "Read items."
    },
)
```

### Token Generation
```python
@app.post("/token")
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()],
) -> Token:
    user = authenticate_user(...)
    access_token = create_access_token(
        data={"sub": user.username, "scopes": form_data.scopes},
        expires_delta=...
    )
    return Token(access_token=access_token, token_type="bearer")
```

### Scope Validation
```python
async def get_current_user(
    security_scopes: SecurityScopes, 
    token: Annotated[str, Depends(oauth2_scheme)]
):
    # Decode token and verify scopes
    for scope in security_scopes.scopes:
        if scope not in token_data.scopes:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Not enough permissions",
                headers={"WWW-Authenticate": authenticate_value},
            )
```

### Protected Endpoints
```python
@app.get("/users/me/", response_model=User)
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)],
):
    return current_user

@app.get("/users/me/items/")
async def read_own_items(
    current_user: Annotated[User, Security(get_current_active_user, scopes=["items"])],
):
    return [{"item_id": "Foo", "owner": current_user.username}]
```

## Important Notes

1. **Security Warning**: In production, validate requested scopes against what the user is actually allowed to request
2. **Scope Format**: Can be any string without spaces (e.g., `users:read`, `https://api.example.com/auth`)
3. **API Docs Integration**: Scopes will appear in the interactive docs for authorization

This implementation follows the same pattern used by major providers like Google, Facebook, and GitHub for their OAuth2 integrations.

I'll provide a clearer explanation with focused code examples that show exactly how OAuth2 scopes work in FastAPI.

# FastAPI OAuth2 Scopes - Step by Step Implementation

## 1. Define Your OAuth2 Scheme

```python
from fastapi.security import OAuth2PasswordBearer

# Define available scopes and where to get tokens
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={
        "me": "Read information about the current user.",
        "items": "Read items."
    },
)
```

This creates the authentication scheme with two possible permission scopes.

## 2. Create the Token Endpoint

```python
from fastapi import Depends, FastAPI, HTTPException
from fastapi.security import OAuth2PasswordRequestForm
from datetime import timedelta

@app.post("/token")
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()]
):
    # Validate username and password
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    
    # Create token with the requested scopes
    access_token = create_access_token(
        data={
            "sub": user.username,
            "scopes": form_data.scopes  # The scopes the client requested
        },
        expires_delta=timedelta(minutes=30)
    )
    
    return {"access_token": access_token, "token_type": "bearer"}
```

The client requests specific scopes during authentication, which get stored in the token.

## 3. Token Creation Function

```python
import jwt
from datetime import datetime, timezone

# Secret key should be kept secure in production
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"

def create_access_token(data: dict, expires_delta: timedelta):
    # Copy the data to avoid modifying the original
    payload = data.copy()
    
    # Set expiration time
    expire = datetime.now(timezone.utc) + expires_delta
    payload.update({"exp": expire})
    
    # Create the JWT token
    encoded_jwt = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt
```

This function creates a JWT token containing the user identity and their granted scopes.

## 4. Token Validation with Scope Checking

```python
from fastapi import Security, status
from fastapi.security import SecurityScopes
from pydantic import ValidationError
from jwt.exceptions import InvalidTokenError

async def get_current_user(
    security_scopes: SecurityScopes,  # Automatically receives required scopes
    token: Annotated[str, Depends(oauth2_scheme)]
):
    # Create appropriate authentication error
    if security_scopes.scopes:
        authenticate_value = f'Bearer scope="{security_scopes.scope_str}"'
    else:
        authenticate_value = "Bearer"
        
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": authenticate_value},
    )
    
    try:
        # Decode the token
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        
        # Extract username and scopes
        username = payload.get("sub")
        if username is None:
            raise credentials_exception
            
        token_scopes = payload.get("scopes", [])
        
    except (InvalidTokenError, ValidationError):
        raise credentials_exception
        
    # Get the user from database
    user = get_user(fake_users_db, username=username)
    if user is None:
        raise credentials_exception
    
    # Check if token has all required scopes
    for scope in security_scopes.scopes:
        if scope not in token_scopes:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Not enough permissions",
                headers={"WWW-Authenticate": authenticate_value},
            )
    
    return user
```

This function:
1. Receives the token and the required scopes via `SecurityScopes`
2. Validates the token and extracts the user and granted scopes
3. Checks if all required scopes are present in the token

## 5. Building Security Layers with Scopes

```python
# First security layer - valid token with any scope
async def get_current_user(
    security_scopes: SecurityScopes,
    token: Annotated[str, Depends(oauth2_scheme)]
):
    # Token validation (as above)
    ...

# Second security layer - requires "me" scope and active user
async def get_current_active_user(
    current_user: Annotated[User, Security(get_current_user, scopes=["me"])]
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user
```

The `Security` class allows specifying which scopes are required for this dependency.

## 6. Applying Scopes to Endpoints

```python
# Endpoint 1: Requires only "me" scope (from get_current_active_user)
@app.get("/users/me/")
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)]
):
    return current_user

# Endpoint 2: Requires both "me" AND "items" scopes
@app.get("/users/me/items/")
async def read_own_items(
    current_user: Annotated[User, Security(get_current_active_user, scopes=["items"])]
):
    return [{"item_id": "Foo", "owner": current_user.username}]

# Endpoint 3: Requires just a valid token, no specific scopes
@app.get("/status/")
async def read_system_status(
    current_user: Annotated[User, Depends(get_current_user)]
):
    return {"status": "ok"}
```

## How Scopes Combine: A Clear Example

Let's see exactly how scopes stack in the `/users/me/items/` endpoint:

```python
@app.get("/users/me/items/")
async def read_own_items(
    # This line requires both "me" AND "items" scopes:
    current_user: Annotated[User, Security(get_current_active_user, scopes=["items"])]
):
    return [{"item_id": "Foo", "owner": current_user.username}]
```

Tracing the scope requirements:
1. The endpoint directly requires the "items" scope
2. It depends on `get_current_active_user`, which requires the "me" scope
3. The token must have BOTH scopes to access this endpoint

## Testing with curl

Here's how you would test this API:

```bash
# 1. Get a token with both scopes
curl -X POST "http://localhost:8000/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=johndoe&password=secret&scope=me+items"

# 2. Use the token to access the protected endpoint
curl -X GET "http://localhost:8000/users/me/items/" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

If your token doesn't have both required scopes, you'll get a 401 Unauthorized error with a message indicating "Not enough permissions".

# FastAPI OAuth2 with Scopes: Code Explanation

Let me walk through this FastAPI OAuth2 implementation step by step, explaining each component:

## 1. Imports and Setup

```python
from datetime import datetime, timedelta, timezone
from typing import Annotated

import jwt
from fastapi import Depends, FastAPI, HTTPException, Security, status
from fastapi.security import (
    OAuth2PasswordBearer,
    OAuth2PasswordRequestForm,
    SecurityScopes,
)
from jwt.exceptions import InvalidTokenError
from passlib.context import CryptContext
from pydantic import BaseModel, ValidationError
```

- **datetime modules**: Handle token expiration
- **jwt**: JSON Web Token encoding/decoding
- **fastapi.security**: OAuth2 authentication tools
- **passlib.context**: Password hashing
- **pydantic**: Data validation

## 2. Configuration Constants

```python
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30
```

- **SECRET_KEY**: Used to sign JWT tokens (should be kept secret in production)
- **ALGORITHM**: HS256 is the JWT signing algorithm
- **ACCESS_TOKEN_EXPIRE_MINUTES**: Token validity period

## 3. Mock User Database

```python
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",
        "disabled": False,
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Chains",
        "email": "alicechains@example.com",
        "hashed_password": "$2b$12$gSvqqUPvlXP2tfVFaWK1Be7DlH.PKZbv5H8KnzzVgXXbVxpva.pFm",
        "disabled": True,
    },
}
```

A dictionary simulating a user database with hashed passwords.

## 4. Data Models

```python
class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: str | None = None
    scopes: list[str] = []

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None

class UserInDB(User):
    hashed_password: str
```

Pydantic models for:
- **Token**: Response data for the token endpoint
- **TokenData**: JWT payload structure
- **User**: User information returned to clients
- **UserInDB**: User information with password hash (internal use)

## 5. Password and Authentication Setup

```python
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={"me": "Read information about the current user.", "items": "Read items."},
)

app = FastAPI()
```

- **pwd_context**: Password hashing configuration using bcrypt
- **oauth2_scheme**: OAuth2 flow configuration with defined scopes
  - `tokenUrl="token"`: Endpoint for token acquisition 
  - `scopes={"me": "...", "items": "..."}`: Available permission scopes

## 6. Password Utility Functions

```python
def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password):
    return pwd_context.hash(password)
```

Helper functions for password verification and hashing.

## 7. User Authentication Functions

```python
def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user
```

- **get_user**: Retrieves user from DB by username
- **authenticate_user**: Validates username/password combination

## 8. Token Creation

```python
def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt
```

Creates a JWT token with:
- User data (typically username and scopes)
- Expiration time
- Signed with the secret key

## 9. Token Validation and Scope Checking

```python
async def get_current_user(
    security_scopes: SecurityScopes, token: Annotated[str, Depends(oauth2_scheme)]
):
    if security_scopes.scopes:
        authenticate_value = f'Bearer scope="{security_scopes.scope_str}"'
    else:
        authenticate_value = "Bearer"
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": authenticate_value},
    )
    try:
        # Decode JWT token
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")
        if username is None:
            raise credentials_exception
        # Extract scopes from token
        token_scopes = payload.get("scopes", [])
        token_data = TokenData(scopes=token_scopes, username=username)
    except (InvalidTokenError, ValidationError):
        raise credentials_exception
    
    # Get user from database
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    
    # Check if token has all required scopes
    for scope in security_scopes.scopes:
        if scope not in token_data.scopes:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Not enough permissions",
                headers={"WWW-Authenticate": authenticate_value},
            )
    return user
```

This function:
1. Receives required scopes via `SecurityScopes` parameter
2. Creates appropriate authentication error responses
3. Decodes and validates the JWT token
4. Extracts username and scopes from token
5. Retrieves the user from the database
6. Verifies the token has all required scopes
7. Returns the user if all checks pass

## 10. Active User Check

```python
async def get_current_active_user(
    current_user: Annotated[User, Security(get_current_user, scopes=["me"])],
):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user
```

This adds another layer of security:
1. Requires the `"me"` scope using `Security(get_current_user, scopes=["me"])`
2. Checks if the user is disabled
3. Returns the active user if checks pass

## 11. Token Endpoint

```python
@app.post("/token")
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()],
) -> Token:
    # Authenticate user
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    
    # Create access token with requested scopes
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username, "scopes": form_data.scopes},
        expires_delta=access_token_expires,
    )
    return Token(access_token=access_token, token_type="bearer")
```

The token endpoint:
1. Validates username and password
2. Creates a token with the requested scopes
3. Returns the token in the response

## 12. Protected Endpoints with Different Scope Requirements

```python
@app.get("/users/me/", response_model=User)
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)],
):
    return current_user

@app.get("/users/me/items/")
async def read_own_items(
    current_user: Annotated[User, Security(get_current_active_user, scopes=["items"])],
):
    return [{"item_id": "Foo", "owner": current_user.username}]

@app.get("/status/")
async def read_system_status(current_user: Annotated[User, Depends(get_current_user)]):
    return {"status": "ok"}
```

Three different endpoints with varying scope requirements:

1. `/users/me/`:
   - Uses `Depends(get_current_active_user)` 
   - Requires the "me" scope (inherited from `get_current_active_user`)
   - Returns the current user info

2. `/users/me/items/`:
   - Uses `Security(get_current_active_user, scopes=["items"])`
   - Requires both "me" (inherited) and "items" scopes
   - Returns a list of items owned by the user

3. `/status/`:
   - Uses `Depends(get_current_user)`
   - Only requires a valid token, no specific scopes
   - Returns system status

## Key Security Concepts Illustrated:

1. **Layered Security**: Dependencies build on each other, combining scope requirements
2. **Scope Stacking**: The `/users/me/items/` endpoint requires both "me" and "items" scopes
3. **Granular Permissions**: Different endpoints require different scopes based on sensitivity
4. **JWT Token**: Securely carries user identity and permissions between requests

This implementation demonstrates how to create a comprehensive OAuth2 authentication system with granular permission controls in FastAPI.

Certainly! Let’s walk through your FastAPI OAuth2 scopes example **step-by-step** with **detailed explanation**, including how scopes work, how they are encoded in JWTs, how FastAPI enforces them, and why `SecurityScopes` is critical.

---

## 🧰 Concepts Involved

* **OAuth2 with Password Flow**
* **JWT (JSON Web Tokens)**
* **Scopes** – Used to limit what a token allows
* **Security dependency (`Security`)**
* **`SecurityScopes`** – Used by FastAPI to extract required scopes per endpoint

---

## 🔐 1. Define Available Scopes

```python
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={
        "me": "Read information about the current user.",
        "items": "Read items."
    },
)
```

This tells FastAPI:

* The app uses the **OAuth2 password flow**.
* The available **scopes** for access control are:

  * `"me"`: For reading personal user data.
  * `"items"`: For accessing user items.

🔸 These scopes will be advertised in your **OpenAPI docs (Swagger UI)**.
🔸 Clients can request a token with a subset of these scopes.

---

## 🔑 2. JWT Token with Scopes

In `/token`, a user logs in and receives a JWT token:

```python
access_token = create_access_token(
    data={"sub": user.username, "scopes": form_data.scopes},
    expires_delta=access_token_expires,
)
```

The token payload contains:

* `"sub"` – the subject (username)
* `"scopes"` – a list of scopes the user is authorized for

The resulting JWT (before encryption) might look like:

```json
{
  "sub": "johndoe",
  "scopes": ["me", "items"],
  "exp": 1722523922
}
```

✅ The scopes requested and granted are encoded **in the token itself**.

---

## 🔍 3. Extract and Validate the Token + Scopes

This function handles token decoding and scope validation:

```python
async def get_current_user(security_scopes: SecurityScopes, token: Annotated[str, Depends(oauth2_scheme)]):
```

### Step-by-Step What Happens:

1. **SecurityScopes**:

   * This is auto-filled by FastAPI based on the `Security(..., scopes=["xyz"])` in the endpoint.
   * It tells us **what scopes are required for this route**.

2. **Token Parsing and Validation**:

   ```python
   payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
   username = payload.get("sub")
   token_scopes = payload.get("scopes", [])
   ```

3. **Permission Check**:

   ```python
   for scope in security_scopes.scopes:
       if scope not in token_scopes:
           raise HTTPException(status_code=401, detail="Not enough permissions")
   ```

✅ This ensures the token includes **all** required scopes for the current endpoint.

---

## 🧪 4. Use Security to Enforce Scopes in Endpoints

```python
@router.get("/users/me/items/", response_model=List[Item])
async def read_own_items(
    current_user: Annotated[User, Security(get_current_user, scopes=["items"])]
):
```

* The `Security` dependency:

  * Calls `get_current_user`
  * Passes the list `scopes=["items"]` to `SecurityScopes`

* `get_current_user` checks:

  * Is the user authenticated?
  * Does the token include the `"items"` scope?

🔐 **If not**, the user gets a `401 Unauthorized`.

---

## ⚖️ 5. Summary Table

| Component                                  | Purpose                                                  |
| ------------------------------------------ | -------------------------------------------------------- |
| `OAuth2PasswordBearer(scopes=)`            | Declares available scopes for documentation & clients    |
| JWT `"scopes"` claim                       | Grants specific permissions to the token                 |
| `Security(get_current_user, scopes=[...])` | Tells FastAPI which scopes are required for a route      |
| `SecurityScopes`                           | Internally holds required scopes, passed to dependencies |
| Scope check in `get_current_user`          | Compares required vs. granted scopes                     |

---

## 🛡️ Example Use Case

Let’s say we have:

* A token for user `alice` with `["me"]`
* She accesses `/users/me/items/`, which requires `"items"`

➡️ FastAPI will:

1. Decode her token
2. See `"scopes": ["me"]` in it
3. Realize `"items"` is missing
4. Return:

```json
{
  "detail": "Not enough permissions"
}
```

---

## 📘 Why Use Scopes?

Scopes provide **fine-grained access control**:

* 🔐 `"read:profile"` – Only allow profile reading
* 🛠️ `"write:settings"` – Allow changing settings
* 👨‍👩‍👧‍👦 `"admin:users"` – Only for admin users

You can assign different scopes based on user roles or token types, and enforce them in endpoints.

---

## ✅ Best Practices

1. **Use Scopes in All Protected Endpoints**

   * Clearly document the scope requirements using `Security`.

2. **Keep Scope Naming Consistent**

   * e.g., `read:items`, `write:items`, `delete:items`.

3. **Validate Token Scopes Before Doing Work**

   * Don’t just decode the token—validate its scopes!

4. **Consider Role-to-Scope Mapping**

   * Assign scopes based on user role during token issuance.

---


