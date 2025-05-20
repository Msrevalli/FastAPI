Here's a concise summary and practical guide based on what you've posted about using **templates in FastAPI with Jinja2**:

---

## ✅ FastAPI with Jinja2 Templates

### 🔧 Setup

Install Jinja2:

```bash
pip install jinja2
```

Create the following project structure:

```
project/
│
├── main.py
├── templates/
│   └── item.html
└── static/
    └── styles.css
```

---

### 📦 `main.py`

```python
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()

# Mount static files
app.mount("/static", StaticFiles(directory="static"), name="static")

# Jinja2 templates directory
templates = Jinja2Templates(directory="templates")


@app.get("/items/{id}", response_class=HTMLResponse)
async def read_item(request: Request, id: str):
    return templates.TemplateResponse(
        name="item.html",  # Template file name
        request=request,   # Required by Jinja2Templates
        context={"id": id} # Data for template
    )
```

---

### 🖼 `templates/item.html`

```html
<html>
<head>
    <title>Item Details</title>
    <link href="{{ url_for('static', path='/styles.css') }}" rel="stylesheet">
</head>
<body>
    <h1><a href="{{ url_for('read_item', id=id) }}">Item ID: {{ id }}</a></h1>
</body>
</html>
```

---

### 🎨 `static/styles.css`

```css
h1 {
    color: green;
}
```

---

### 🔍 Key Concepts

* `Jinja2Templates(directory="templates")`: Creates the template environment.
* `request`: Must be passed to the template for `url_for()` to work.
* `url_for('static', path='/...')`: Generates correct static file URLs.
* `url_for('read_item', id=id)`: Generates dynamic internal links.
* `context={"id": id}`: Variables you use inside the template (like `{{ id }}`).

---

### 🚀 Run

```bash
uvicorn main:app --reload
```

Access in browser:
[http://127.0.0.1:8000/items/42](http://127.0.0.1:8000/items/42)

---


