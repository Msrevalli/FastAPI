Here's a **clear breakdown** of how to test **WebSockets** in FastAPI using `TestClient`.

---

## ✅ What This Example Shows

You're creating a FastAPI app with:

1. A **normal HTTP endpoint** (`GET /`)
2. A **WebSocket endpoint** (`/ws`)
3. Two tests:

   * One for the HTTP route
   * One for the WebSocket route

---

## 🔎 Code Explanation

### 1. App definition

```python
from fastapi import FastAPI, WebSocket
from fastapi.testclient import TestClient

app = FastAPI()

@app.get("/")
async def read_main():
    return {"msg": "Hello World"}

@app.websocket("/ws")
async def websocket(websocket: WebSocket):
    await websocket.accept()
    await websocket.send_json({"msg": "Hello WebSocket"})
    await websocket.close()
```

### 2. Test for normal HTTP route

```python
def test_read_main():
    client = TestClient(app)
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

✅ This checks that your HTTP route is responding correctly.

### 3. Test for WebSocket route

```python
def test_websocket():
    client = TestClient(app)
    with client.websocket_connect("/ws") as websocket:
        data = websocket.receive_json()
        assert data == {"msg": "Hello WebSocket"}
```

✅ This does the following:

* Opens a WebSocket connection using `websocket_connect()`.
* Waits for and receives JSON data.
* Asserts that the data is correct.
* Closes the WebSocket automatically at the end of the `with` block.

---

## 🧪 Summary: WebSocket Testing with FastAPI

| Step                  | Function                                 |
| --------------------- | ---------------------------------------- |
| `websocket_connect()` | Opens the WebSocket connection           |
| `receive_json()`      | Reads a message from the server          |
| `with` block          | Ensures the connection is closed cleanly |
| `assert`              | Verifies the message content             |

---

## ⚠️ Notes

* `TestClient` uses **`requests` under the hood for HTTP** and **`starlette.testclient` for WebSocket**.
* This test client **runs the app in-process** — no need to run a real server.
* It's best for **unit tests and integration tests**.

---

