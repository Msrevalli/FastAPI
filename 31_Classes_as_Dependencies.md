**Dependency Injection**, especially using **classes as dependencies**. Let's break this down clearly and thoroughly to help you really understand how and why this works—and how it helps in real projects.

---

## 🚀 What is Dependency Injection?

**Dependency Injection (DI)** is a way to tell FastAPI:  
> "Hey, before calling this function (usually a route), please get this other thing ready first, and give it to the function as a parameter."

That "thing" could be:
- Query parameters
- A database session
- A validated user object
- Some shared configuration
- Or **anything else your route needs**

FastAPI handles the "preparation" and **injects** the result into your route function.

---

## 🧠 What is a Dependency?

A **dependency** is usually a function—or anything *callable*—that returns a value. FastAPI will:

1. Look at the dependency’s parameters
2. Get the values from the request (query, headers, etc.)
3. Call the function/class with those values
4. Return the result to your route

---

## ✅ Example: Function-Based Dependency

```python
async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}
```

Usage:

```python
@app.get("/items/")
async def read_items(commons: Annotated[dict, Depends(common_parameters)]):
    return commons
```

FastAPI extracts `q`, `skip`, and `limit` from the query string, calls `common_parameters`, and passes the result as `commons`.

---

## 🧱 Using Classes as Dependencies

Since Python classes are callable (you call `MyClass()` to create an object), FastAPI treats them the same way.

### 🧪 Rewriting the above with a Class

```python
class CommonQueryParams:
    def __init__(self, q: str | None = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit
```

This behaves exactly like the function above. Now you use it like:

```python
@app.get("/items/")
async def read_items(commons: Annotated[CommonQueryParams, Depends(CommonQueryParams)]):
    return {
        "q": commons.q,
        "skip": commons.skip,
        "limit": commons.limit,
    }
```

Now `commons` is an **instance of `CommonQueryParams`** — with nice autocompletion in your editor like `.q`, `.skip`, `.limit`. 💡

---

## 🧠 Why Use Classes?

### ✅ Cleaner Code
Encapsulate related logic or values (especially helpful for bigger logic).

### ✅ Reusability
Just like functions, but easier to scale when state or methods are involved.

### ✅ Editor Support
You get autocomplete, type checking, and error hints.

---

## 📌 Type Annotation Note

```python
commons: Annotated[CommonQueryParams, Depends(CommonQueryParams)]
```

- First `CommonQueryParams` is for **type hinting** (helps your editor, doesn't affect FastAPI).
- `Depends(CommonQueryParams)` is what **FastAPI uses** to resolve the dependency.

You *could* write:

```python
commons: Annotated[Any, Depends(CommonQueryParams)]
```

…but then your editor won’t know what `commons` is, so you lose autocomplete. Always better to be explicit!

---

## 🔁 Summary

| Concept | Function | Class |
|--------|----------|--------|
| Is it callable? | ✅ Yes | ✅ Yes |
| Used as dependency? | ✅ Yes | ✅ Yes |
| Best for... | Simple values | Structured/grouped data |
| Editor support | ❌ Limited (dict) | ✅ Full (attributes/methods) |

---

### ✅ Example with real logic

```python
class Pagination:
    def __init__(self, skip: int = 0, limit: int = 10):
        self.skip = skip
        self.limit = limit

@app.get("/books/")
def list_books(pagination: Annotated[Pagination, Depends()]):
    return {
        "skip": pagination.skip,
        "limit": pagination.limit
    }
```

You could later add logging, config, or user filtering to `Pagination` without changing all the routes that use it.

---

