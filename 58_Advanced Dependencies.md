# **FastAPI Parameterized Dependencies **

Parameterized dependencies allow you to create reusable dependency logic that can be configured with different parameters. Here's a comprehensive explanation:

## **1. Basic Parameterized Dependency**

```python
from fastapi import Depends, FastAPI
from typing import Annotated

app = FastAPI()

class ContentChecker:
    def __init__(self, required_content: str):
        self.required_content = required_content

    def __call__(self, q: str = ""):
        return self.required_content in q

# Create configured instances
bar_checker = ContentChecker("bar")
foo_checker = ContentChecker("foo")

@app.get("/check-bar/")
async def check_bar(has_bar: Annotated[bool, Depends(bar_checker)]):
    return {"contains_bar": has_bar}

@app.get("/check-foo/")
async def check_foo(has_foo: Annotated[bool, Depends(foo_checker)]):
    return {"contains_foo": has_foo}
```

## **2. Key Concepts**

### **The `__call__` Method**
- Makes instances callable like functions
- FastAPI treats these instances as dependencies
- Receives all the same parameter types as regular dependencies

### **Configuration via `__init__**
- Lets you parameterize the dependency behavior
- Configuration happens when creating the instance
- The instance becomes a reusable, configured dependency

## **3. Practical Use Cases**

### **Role-Based Authorization**
```python
class RoleChecker:
    def __init__(self, required_role: str):
        self.required_role = required_role

    def __call__(self, user_role: str = Depends(get_current_user_role)):
        return user_role == self.required_role

admin_check = RoleChecker("admin")
editor_check = RoleChecker("editor")

@app.get("/admin-only", dependencies=[Depends(admin_check)])
async def admin_area():
    return {"message": "Admin dashboard"}
```

### **Content Validation**
```python
class ContentValidator:
    def __init__(self, min_length: int):
        self.min_length = min_length

    def __call__(self, content: str):
        if len(content) < self.min_length:
            raise HTTPException(400, f"Content too short (min {self.min_length} chars)")
        return content

validate_long = ContentValidator(10)
validate_short = ContentValidator(3)

@app.post("/long-text/")
async def post_long(text: Annotated[str, Depends(validate_long)]):
    return {"processed_text": text}
```

## **4. Advanced Patterns**

### **Dependency with Sub-dependencies**
```python
class RateLimiter:
    def __init__(self, calls_per_minute: int):
        self.calls_per_minute = calls_per_minute

    def __call__(self, 
                 user_id: str = Depends(get_current_user_id),
                 cache: Redis = Depends(get_redis)):
        key = f"ratelimit:{user_id}"
        current = cache.incr(key)
        if current > self.calls_per_minute:
            raise HTTPException(429, "Rate limit exceeded")
        return True

standard_limit = RateLimiter(100)
premium_limit = RateLimiter(1000)
```

### **Combining with Other Dependencies**
```python
@app.get("/complex-endpoint/")
async def complex_endpoint(
    is_admin: Annotated[bool, Depends(RoleChecker("admin"))],
    is_valid: Annotated[bool, Depends(ContentChecker("important"))],
    rate_limited: Annotated[bool, Depends(RateLimiter(10))]
):
    return {"admin": is_admin, "valid": is_valid}
```

## **5. Best Practices**

1. **Clear Naming**: Use descriptive names for dependency classes
2. **Single Responsibility**: Each dependency should do one thing well
3. **Reusable Instances**: Create configured instances at module level
4. **Type Annotations**: Always annotate dependency parameters
5. **Error Handling**: Raise appropriate HTTPExceptions

## **Comparison with Alternatives**

| Approach | Pros | Cons |
|----------|------|------|
| **Parameterized Dependencies** | Reusable, configurable | Slightly more complex |
| **Multiple Similar Dependencies** | Simple | Code duplication |
| **Dependency with Config Params** | Flexible | Less explicit configuration |

This pattern is especially powerful for:
- Authorization checks
- Rate limiting
- Content validation
- Feature flags
- Any reusable business logic with configuration

The security utilities in FastAPI (OAuth2, JWT, etc.) use this same pattern internally.