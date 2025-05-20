# **FastAPI Custom Responses: HTML, Stream, File, and More **

FastAPI provides powerful tools to customize responses beyond standard JSON. Here's a comprehensive explanation of all response types and customization options.

## **1. Response Classes Overview**
FastAPI offers specialized response classes for different content types:

| Response Type | Content-Type | Best For | Performance Notes |
|--------------|-------------|----------|-------------------|
| `JSONResponse` | application/json | Default JSON | Good |
| `ORJSONResponse` | application/json | High-performance JSON | **Fastest** (requires `orjson`) |
| `UJSONResponse` | application/json | Alternative JSON | Fast but less strict |
| `HTMLResponse` | text/html | HTML content | - |
| `PlainTextResponse` | text/plain | Plain text | - |
| `RedirectResponse` | - | URL redirects | - |
| `StreamingResponse` | varies | Large files/streams | Memory efficient |
| `FileResponse` | varies | File downloads | Auto handles headers |

## **2. Setting Default Response Class**
You can change the default response class for all routes:

```python
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse

app = FastAPI(default_response_class=ORJSONResponse)

@app.get("/items/")
async def read_items():
    return [{"item_id": "Foo"}]  # Will use ORJSONResponse
```

## **3. HTML Responses**
### **Method 1: Using `response_class`**
```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.get("/", response_class=HTMLResponse)
async def home():
    return "<h1>Welcome</h1>"
```

### **Method 2: Direct `HTMLResponse`**
```python
@app.get("/custom")
async def custom_html():
    content = """
    <html>
        <body style='background: black; color: white'>
            <h1>Custom Styled Page</h1>
        </body>
    </html>
    """
    return HTMLResponse(content=content, status_code=200)
```

## **4. High-Performance JSON Responses**
### **ORJSONResponse (Fastest)**
```python
from fastapi.responses import ORJSONResponse

@app.get("/fast-json", response_class=ORJSONResponse)
async def fast_json():
    return {"message": "This uses orjson", "speed": "blazing fast"}
```
*Requires:* `pip install orjson`

### **UJSONResponse (Alternative)**
```python
from fastapi.responses import UJSONResponse

@app.get("/alt-json", response_class=UJSONResponse)
async def alt_json():
    return {"note": "Slightly faster than standard JSON"}
```
*Requires:* `pip install ujson`

## **5. File and Stream Responses**
### **File Downloads**
```python
from fastapi.responses import FileResponse

@app.get("/download")
async def download_file():
    return FileResponse(
        "big_file.zip",
        filename="download.zip",
        media_type="application/zip"
    )
```

### **Streaming Large Files**
```python
from fastapi.responses import StreamingResponse

@app.get("/stream-video")
async def stream_video():
    def iterfile():
        with open("big_video.mp4", "rb") as f:
            yield from f
            
    return StreamingResponse(iterfile(), media_type="video/mp4")
```

## **6. Redirects**
```python
from fastapi.responses import RedirectResponse

@app.get("/old-url")
async def redirect():
    return RedirectResponse("/new-url", status_code=301)

@app.get("/fastapi", response_class=RedirectResponse)
async def redirect_fastapi():
    return "https://fastapi.tiangolo.com"  # Defaults to 307
```

## **7. Custom Response Classes**
Create your own response types by subclassing `Response`:

```python
from fastapi import Response
import orjson

class PrettyJSONResponse(Response):
    media_type = "application/json"
    
    def render(self, content: Any) -> bytes:
        return orjson.dumps(content, option=orjson.OPT_INDENT_2)

@app.get("/pretty-json", response_class=PrettyJSONResponse)
async def pretty_json():
    return {"message": "This will be nicely formatted"}
```

## **8. Advanced Usage Patterns**
### **Combining Documentation with Custom Responses**
```python
@app.get(
    "/hybrid",
    response_class=HTMLResponse,
    responses={
        200: {
            "content": {"text/html": {}},
            "description": "An HTML response",
        }
    }
)
async def hybrid():
    return HTMLResponse("<h1>Works with docs!</h1>")
```

### **Dynamic Content-Type Switching**
```python
from fastapi import Request

@app.get("/smart-response")
async def smart_response(request: Request):
    accept = request.headers.get("accept", "")
    
    if "text/html" in accept:
        return HTMLResponse("<h1>HTML</h1>")
    else:
        return {"message": "JSON fallback"}
```

## **Performance Considerations**
1. **For JSON APIs**: Use `ORJSONResponse` for maximum speed
2. **Large files**: Always use `StreamingResponse` or `FileResponse`
3. **Avoid** returning large Python objects directly - let FastAPI handle serialization
4. **Custom responses** bypass automatic OpenAPI documentation

## **When to Use Each Approach**
| Use Case | Recommended Approach |
|----------|----------------------|
| Standard API | Default JSONResponse |
| High-traffic API | ORJSONResponse |
| HTML pages | HTMLResponse |
| File downloads | FileResponse |
| Large media | StreamingResponse |
| Special formats | Custom Response class |

This comprehensive approach gives you complete control over FastAPI responses while maintaining performance and documentation capabilities.