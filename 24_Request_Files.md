One of the coolest and most practical features of FastAPI — **file uploads** using `File` and `UploadFile`. Here’s a quick and structured guide with real-world insights, so you can confidently handle file uploads like a pro. 🚀

---

## 📦 File Upload Basics

### 🔧 Prerequisite
Make sure you install:
```bash
pip install python-multipart
```
Because file uploads (via forms) use the `multipart/form-data` encoding.

---

## 🆚 `bytes` vs `UploadFile`

| Feature            | `bytes`                             | `UploadFile`                             |
|--------------------|--------------------------------------|-------------------------------------------|
| Reads whole file   | ✅ Entirely in memory                | ❌ Not until you call `.read()`           |
| Good for small files | ✅                                  | ✅ Better for large files                 |
| Metadata (filename, type) | ❌                            | ✅ Access to `.filename`, `.content_type` |
| Async support      | ❌                                  | ✅ Supports `await .read()`, `.write()`   |

---

## 🧪 Basic Examples

### 1. Get file contents as `bytes`
```python
@app.post("/files/")
async def create_file(file: Annotated[bytes, File()]):
    return {"file_size": len(file)}
```

### 2. Use `UploadFile` to handle larger files
```python
@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename}
```

---

## ✨ Optional File Uploads

```python
@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile | None = None):
    if not file:
        return {"message": "No file sent"}
    return {"filename": file.filename}
```

---

## 📂 Multiple File Uploads

### 1. As `bytes` (all in memory)
```python
@app.post("/files/")
async def create_files(files: Annotated[list[bytes], File()]):
    return {"file_sizes": [len(file) for file in files]}
```

### 2. As `UploadFile` (memory-efficient)
```python
@app.post("/uploadfiles/")
async def create_upload_files(files: list[UploadFile]):
    return {"filenames": [file.filename for file in files]}
```

### Example HTML Form
```python
@app.get("/")
async def main():
    return HTMLResponse("""
    <body>
    <form action="/uploadfiles/" enctype="multipart/form-data" method="post">
    <input name="files" type="file" multiple>
    <input type="submit">
    </form>
    </body>
    """)
```

---

## 🛠 UploadFile Methods

All async:
```python
contents = await file.read()
await file.seek(0)
await file.write(b"new data")
await file.close()
```

Or sync:
```python
contents = file.file.read()
```

---

## 🧾 File Metadata with `File(...)`

```python
@app.post("/uploadfile/")
async def create_upload_file(
    file: Annotated[UploadFile, File(description="Upload your file here")]
):
    return {"filename": file.filename}
```

---

## ⚠️ JSON + File = ❌

You **cannot** mix JSON body fields with `File()` or `Form()` in the same request body. This is a **protocol-level** limitation (not FastAPI’s fault). Multipart form data and JSON are two different content types.

---

## ✅ Summary

- Use `File()` for raw bytes or file metadata.
- Prefer `UploadFile` for **large or multiple** files.
- Optional uploads? Just default to `None`.
- Need both form fields and files? Combine `Form()` and `File()` params — but **not JSON**.

---

