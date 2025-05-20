# **FastAPI Additional Responses in OpenAPI **

This guide explains how to document multiple response types and status codes in your FastAPI OpenAPI schema.

## **1. Basic Additional Response Declaration**
You can declare extra responses beyond the default using the `responses` parameter:

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    id: str
    value: str

class ErrorMessage(BaseModel):
    message: str

@app.get(
    "/items/{item_id}",
    response_model=Item,
    responses={
        404: {"model": ErrorMessage, "description": "Item not found"},
        418: {"description": "I'm a teapot"} 
    }
)
async def read_item(item_id: str):
    if item_id == "foo":
        return {"id": "foo", "value": "bar"}
    return JSONResponse(
        status_code=404,
        content={"message": "Item not found"}
    )
```

Key points:
- The `404` response uses a Pydantic model for documentation
- The `418` response is documented without a model
- You must return the `JSONResponse` directly for non-200 responses

## **2. Multiple Response Content Types**
Document that an endpoint can return different media types:

```python
from fastapi.responses import FileResponse

@app.get(
    "/items/{item_id}",
    response_model=Item,
    responses={
        200: {
            "content": {
                "application/json": {},
                "image/png": {}
            },
            "description": "Either JSON or PNG"
        }
    }
)
async def read_item(item_id: str, img: bool = False):
    if img:
        return FileResponse("image.png", media_type="image/png")
    return {"id": "foo", "value": "bar"}
```

## **3. Combining Response Information**
Merge information from multiple sources:

```python
@app.get(
    "/items/{item_id}",
    response_model=Item,
    responses={
        200: {
            "content": {
                "application/json": {
                    "example": {"id": "bar", "value": "The bar tenders"}
                }
            },
            "description": "Item requested by ID"
        },
        404: {
            "model": ErrorMessage,
            "description": "Item not found"
        }
    }
)
```

## **4. Reusable Response Templates**
Define common responses and merge them:

```python
common_responses = {
    401: {"description": "Unauthorized"},
    403: {"description": "Forbidden"}, 
}

@app.get(
    "/admin/items/",
    response_model=list[Item],
    responses={
        **common_responses,
        200: {"description": "Admin item list"}
    }
)
```

## **5. Complete Response Documentation**
Full OpenAPI response documentation supports:

```python
responses={
    200: {
        "description": "Success",
        "content": {
            "application/json": {
                "schema": {"$ref": "#/components/schemas/Item"},
                "examples": {
                    "normal": {
                        "value": {"id": "foo", "value": "bar"}
                    },
                    "special": {
                        "summary": "Special case",
                        "value": {"id": "sp", "value": "special"}
                    }
                }
            }
        },
        "headers": {
            "X-Request-ID": {
                "description": "Request ID",
                "schema": {"type": "string"}
            }
        }
    }
}
```

## **Key Benefits**
1. **Better Documentation**: Shows all possible responses in Swagger UI
2. **Client Generation**: Helps generate better client code
3. **API Contracts**: Clearly defines error cases
4. **Flexibility**: Supports multiple response types per endpoint

## **When to Use**
- API endpoints with multiple success responses
- Endpoints that return different error formats
- APIs needing precise OpenAPI documentation
- When building client SDKs from your API

This approach gives you complete control over how your API responses are documented while maintaining FastAPI's automatic validation and serialization for the main success case.