**receiving both form fields and files in the same request** — like when a user fills out a form and uploads a file at the same time. Let's break it all down clearly so you can use it effectively. ⚡

---

## 📥 Receiving Form Data + Files in One Go

### ✅ You *can* mix:
- `Form()` — for text input fields
- `File()` / `UploadFile` — for file uploads

### ❌ You *cannot* mix:
- `Form()` / `File()` **with** `Body()` (JSON data)
> Because `multipart/form-data` is used for forms and files, **not JSON**.

---

## 🧪 Example — Receiving Form Field + Two Files

```python
from typing import Annotated
from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(
    file: Annotated[bytes, File()],
    fileb: Annotated[UploadFile, File()],
    token: Annotated[str, Form()],
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }
```

### 🔍 What this endpoint expects:
- `file`: A small file (will be fully read into memory as `bytes`)
- `fileb`: A larger file (as `UploadFile`, more efficient)
- `token`: A regular form text field

---

## 🧾 Example HTML Form

If you want to test this manually in a browser:

```html
<form action="/files/" enctype="multipart/form-data" method="post">
  <input type="file" name="file">
  <input type="file" name="fileb">
  <input type="text" name="token" value="mytoken123">
  <input type="submit">
</form>
```

---

## ✅ Recap

| Feature                     | ✅ Supported                     | ❌ Not Supported              |
|-----------------------------|----------------------------------|-------------------------------|
| Multiple `File` and `Form`  | Yes                              |                               |
| Mixing `Form`/`File` with `Body` |                              | JSON in same request body      |
| Different file types        | `bytes`, `UploadFile`, or both   |                               |
| Metadata from UploadFile    | `.filename`, `.content_type`     | Not available with `bytes`    |

---

