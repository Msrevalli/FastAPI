## 🐞 FastAPI Debugging Guide

### ✅ Run `uvicorn` directly in code

This lets you **start your FastAPI server and attach a debugger** (like in VS Code or PyCharm).

```python
# main.py
import uvicorn
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    a = "a"
    b = "b" + a
    return {"hello world": b}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

### 💡 Why `if __name__ == "__main__"`?

This block ensures `uvicorn.run(...)`:
- Runs **only** when the file is executed directly:  
  ```bash
  python main.py
  ```
- Does **not** run when the file is imported:  
  ```python
  from main import app
  ```

It’s a standard Python pattern.

---

### 🧠 Debugger Setup in VS Code

1. Open the **Debug** panel (Ctrl+Shift+D)
2. Click **“Add Configuration…”**
3. Choose **“Python”**
4. Pick: **Python: Current File (Integrated Terminal)**
5. Add breakpoints and click ▶️ to start debugging!

Your `launch.json` config might look like:

```json
{
  "name": "FastAPI Debug",
  "type": "python",
  "request": "launch",
  "program": "${file}",
  "console": "integratedTerminal"
}
```

---

### 🧠 Debugger Setup in PyCharm

1. Open **Run > Debug...**
2. Choose the script file (e.g. `main.py`)
3. Set breakpoints and hit **Debug**

That’s it! Your FastAPI server will run, and you’ll hit breakpoints like any other Python script.

---

### 🧪 Why Debug Like This?

Because `uvicorn` is run **inside your script**, the debugger sees it like any normal Python code — no need to attach to subprocesses.

This makes it **easier to step through request handling, catch exceptions, or inspect variables** inside your route logic.

---

