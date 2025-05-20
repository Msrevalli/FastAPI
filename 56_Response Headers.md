# **FastAPI Response Headers **

FastAPI provides multiple ways to set custom headers in responses. Here's a comprehensive guide to working with response headers:

## **1. Using Response Parameter (Recommended Approach)**
The cleanest method that maintains FastAPI's automatic features:

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/items/")
def read_items(response: Response):
    # Set custom headers
    response.headers["X-Custom-Header"] = "CustomValue"
    response.headers["Cache-Control"] = "no-cache, no-store"
    return {"message": "Hello World"}
```

**Key Benefits:**
- Works with automatic response model validation
- Maintains all FastAPI features
- Clean separation of concerns

## **2. Returning Response Directly**
For complete control when you need to customize the entire response:

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/secure-data/")
def get_secure_data():
    content = {"data": "sensitive information"}
    headers = {
        "X-Content-Type-Options": "nosniff",
        "X-Frame-Options": "DENY",
        "Content-Security-Policy": "default-src 'self'"
    }
    return JSONResponse(content=content, headers=headers)
```

## **3. Common Header Patterns**

### **Security Headers**
```python
security_headers = {
    "Strict-Transport-Security": "max-age=63072000; includeSubDomains; preload",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "X-XSS-Protection": "1; mode=block",
    "Content-Security-Policy": "default-src 'self'",
    "Referrer-Policy": "strict-origin-when-cross-origin"
}
```

### **Caching Headers**
```python
cache_headers = {
    "Cache-Control": "public, max-age=3600",
    "ETag": "33a64df551425fcc55e4d42a148795d9f25f89d4"
}
```

### **CORS Headers**
```python
cors_headers = {
    "Access-Control-Allow-Origin": "https://example.com",
    "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
    "Access-Control-Allow-Headers": "Content-Type"
}
```

## **4. Best Practices**

1. **Standard Headers** - Use standard header names from [IANA's registry](https://www.iana.org/assignments/message-headers/message-headers.xhtml)
2. **Security First** - Always include basic security headers
3. **Performance** - Use caching headers for static resources
4. **Readability** - Use header constants for better code maintenance

```python
from fastapi.responses import Response

@app.get("/best-practice/")
def best_practice(response: Response):
    response.headers.update({
        "Cache-Control": "no-store",
        "X-API-Version": "1.0",
        "X-Request-ID": "123e4567-e89b-12d3-a456-426614174000"
    })
    return {"status": "success"}
```

## **5. Advanced Patterns**

### **Using Dependencies for Common Headers**
```python
from fastapi import Depends, FastAPI, Response

app = FastAPI()

def add_security_headers(response: Response):
    response.headers.update({
        "X-Content-Type-Options": "nosniff",
        "X-Frame-Options": "DENY"
    })
    return response

@app.get("/protected/", dependencies=[Depends(add_security_headers)])
def protected_route():
    return {"data": "sensitive"}
```

### **Dynamic Headers Based on Content**
```python
@app.get("/dynamic/")
def dynamic_headers(response: Response):
    if some_condition:
        response.headers["X-Special-Flag"] = "enabled"
    return {"feature": "dynamic"}
```

## **Comparison of Approaches**

| Approach | Best For | Maintains Auto Features | Performance |
|----------|----------|-------------------------|-------------|
| **Response Parameter** | Most cases | ✅ Yes | ⚡ Fast |
| **Direct Response** | Full control | ❌ No | ⚡ Fast |
| **Dependencies** | Cross-cutting concerns | ✅ Yes | ⚡ Fast |

## **Complete Example with All Techniques**
```python
from fastapi import Depends, FastAPI, Response
from fastapi.responses import JSONResponse

app = FastAPI()

# Shared security headers
SECURITY_HEADERS = {
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY"
}

def add_common_headers(response: Response):
    response.headers.update(SECURITY_HEADERS)
    response.headers["X-Request-ID"] = generate_request_id()
    return response

@app.get("/complete-example/", dependencies=[Depends(add_common_headers)])
def complete_example(response: Response):
    # Add route-specific headers
    response.headers["X-Custom-Logic"] = "processed"
    
    # Return with automatic model processing
    return {"status": "complete"}

@app.get("/direct-example/")
def direct_example():
    content = {"status": "direct"}
    headers = {**SECURITY_HEADERS, "X-Special": "value"}
    return JSONResponse(content=content, headers=headers)
```

This guide covers all aspects of working with response headers in FastAPI while maintaining security, performance, and clean code organization.