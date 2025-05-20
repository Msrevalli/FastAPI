# **FastAPI Dynamic Status Codes **

FastAPI allows you to dynamically change response status codes while maintaining all its automatic features. Here's a comprehensive explanation:

## **1. Basic Dynamic Status Code**

```python
from fastapi import FastAPI, Response, status

app = FastAPI()

items = {}

@app.put("/items/{item_id}", status_code=200)
def update_or_create_item(item_id: str, response: Response):
    if item_id not in items:
        items[item_id] = {"value": "New item"}
        response.status_code = status.HTTP_201_CREATED
    return items[item_id]
```

**Behavior:**
- Default status code is `200 OK` (set in decorator)
- Changes to `201 Created` when new item is created
- Maintains automatic response model processing

## **2. Using Status Code Constants**

Best practice is to use FastAPI's status constants:

```python
from fastapi import status

# Instead of hardcoding:
response.status_code = 201

# Use:
response.status_code = status.HTTP_201_CREATED
```

Common status codes:
- `HTTP_200_OK`
- `HTTP_201_CREATED` 
- `HTTP_204_NO_CONTENT`
- `HTTP_400_BAD_REQUEST`
- `HTTP_404_NOT_FOUND`

## **3. With Response Models**

Works seamlessly with response models:

```python
from pydantic import BaseModel

class Item(BaseModel):
    id: str
    value: str

@app.put("/model-items/{item_id}", response_model=Item, status_code=200)
def update_model_item(item_id: str, response: Response):
    if item_id not in items:
        items[item_id] = {"id": item_id, "value": "New"}
        response.status_code = status.HTTP_201_CREATED
    return items[item_id]
```

## **4. In Dependencies**

You can set status codes in dependencies:

```python
def verify_item_exists(item_id: str, response: Response):
    if item_id not in items:
        response.status_code = status.HTTP_404_NOT_FOUND
        return None
    return items[item_id]

@app.get("/verified-items/{item_id}")
def get_item(item: dict = Depends(verify_item_exists)):
    if item is None:
        return {"error": "Not found"}
    return item
```

## **5. Complete CRUD Example**

```python
from fastapi import FastAPI, Response, status, HTTPException
from pydantic import BaseModel

app = FastAPI()

class Product(BaseModel):
    id: str
    name: str
    price: float

products = {}

@app.post("/products/", status_code=201)
def create_product(product: Product):
    if product.id in products:
        raise HTTPException(400, "Product exists")
    products[product.id] = product
    return product

@app.get("/products/{product_id}", status_code=200)
def get_product(product_id: str, response: Response):
    if product_id not in products:
        response.status_code = status.HTTP_404_NOT_FOUND
        return {"error": "Not found"}
    return products[product_id]
```

## **Key Considerations**

1. **Decorator Status Code**:
   - Sets the default status code
   - Can be overridden in the function

2. **Response Parameter**:
   - Must be named `response`
   - Changes apply before automatic model processing

3. **Error Cases**:
   - For errors, consider `HTTPException` instead
   - Works better with FastAPI's error handling

4. **Performance**:
   - No performance impact vs fixed status codes
   - Status is set before response processing

## **When to Use Dynamic Status Codes**

| Use Case | Example Status Code |
|----------|---------------------|
| Resource creation | 201 Created |
| Conditional update | 200 OK or 201 Created | 
| Empty responses | 204 No Content |
| Asynchronous processing | 202 Accepted |
| Deprecated endpoints | 410 Gone |

This approach gives you full control over HTTP semantics while maintaining FastAPI's convenience features.