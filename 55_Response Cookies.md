# **FastAPI Response Cookies **

FastAPI provides flexible ways to set cookies in responses. Here's a comprehensive explanation of both approaches:

## **1. Using Response Parameter (Recommended)**
The cleanest way to set cookies while maintaining FastAPI's automatic features:

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.post("/login/")
def login(response: Response):
    # Set cookie
    response.set_cookie(
        key="session_token",
        value="abc123xyz",
        httponly=True,
        max_age=3600,  # 1 hour
        secure=True
    )
    return {"message": "Logged in successfully"}
```

**Key Benefits:**
- Maintains automatic response model validation
- Works with dependency injection
- Clean separation of concerns

## **2. Returning Response Directly**
For full control when you need to customize the entire response:

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.post("/auth/")
def auth():
    content = {"status": "authenticated"}
    response = JSONResponse(content=content)
    response.set_cookie(
        key="auth_token",
        value="xyz789abc",
        max_age=86400,  # 24 hours
        domain=".example.com",
        path="/",
        samesite="lax"
    )
    return response
```

## **Cookie Parameters Explained**
| Parameter    | Description | Example |
|-------------|------------|---------|
| `key` | Cookie name | "session_id" |
| `value` | Cookie value | "abc123" |
| `max_age` | Lifetime in seconds | `3600` |
| `expires` | Expiration datetime | `datetime(2023, 12, 31)` |
| `path` | URL path where cookie is valid | `"/api"` |
| `domain` | Domain scope | `".example.com"` |
| `secure` | HTTPS only | `True` |
| `httponly` | Inaccessible to JavaScript | `True` |
| `samesite` | CSRF protection | `"strict"`, `"lax"`, `"none"` |

## **Best Practices**
1. **Security Considerations**
   ```python
   response.set_cookie(
       key="secure_cookie",
       value="sensitive_data",
       httponly=True,
       secure=True,
       samesite="strict"
   )
   ```

2. **Clearing Cookies**
   ```python
   response.delete_cookie(key="session_token")
   ```

3. **Multiple Cookies**
   ```python
   response.set_cookie(key="prefs", value="dark_mode")
   response.set_cookie(key="tracking", value="allowed")
   ```

## **When to Use Each Approach**

| Approach | Best For | Maintains Auto Features |
|----------|----------|-------------------------|
| **Response Parameter** | Most cases - when using response models | ✅ Yes |
| **Direct Response** | Full control - custom media types, headers | ❌ No |

## **Complete Example with Dependencies**
```python
from fastapi import Depends, FastAPI, Response

app = FastAPI()

def set_auth_cookie(response: Response):
    response.set_cookie(key="auth", value="temp_token", max_age=300)
    return response

@app.post("/temp-auth/", dependencies=[Depends(set_auth_cookie)])
def temp_auth():
    return {"status": "temporary_auth_granted"}
```

This guide covers all aspects of working with cookies in FastAPI responses while maintaining security and flexibility.