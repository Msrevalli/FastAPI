## 🗂️ FastAPI Static Files Cheat Sheet

### ✅ Basic Example

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# Serve files in the "static" directory at the "/static" path
app.mount("/static", StaticFiles(directory="static"), name="static")
```

📂 **Project structure:**
```
.
├── main.py
└── static/
    ├── style.css
    └── script.js
```

Now if you start your server, you can access:

- `http://localhost:8000/static/style.css`
- `http://localhost:8000/static/script.js`

---

### ⚙️ Parameters Breakdown

| Parameter      | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `"/static"`    | URL prefix for the mounted static path.                                     |
| `directory=""` | Filesystem path to the folder with static files.                            |
| `name="..."`   | Internal name (used by FastAPI internally, not required unless you need it).|

---

### 🧠 What is *Mounting*?

"Mounting" a `StaticFiles` instance means:
- You're attaching a **self-contained mini app** that handles all requests starting with a specific path (e.g. `/static`).
- It's not included in FastAPI's OpenAPI docs or `/docs` UI.
- Think of it like: “delegate anything from this path to that app.”

---

### 🧪 Custom Example

```python
app.mount("/assets", StaticFiles(directory="public_assets"), name="assets")
```

- `http://localhost:8000/assets/image.png` → served from `public_assets/image.png`.

---

### 💡 Pro Tips

- You can serve frontend apps (like React or Vue builds) this way.
- Combine this with `Jinja2Templates` or `HTMLResponse` if you're doing server-side rendering.
- Use `StaticFiles(html=True)` to automatically serve `index.html` on root requests (great for SPAs).

---

