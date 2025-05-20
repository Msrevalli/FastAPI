Absolutely — here’s a **clear and simple explanation** of how **WebSockets in FastAPI** work, broken down step by step:

---

## 🚀 What Are WebSockets?

* **WebSockets** let the **server and client** talk **back and forth continuously** without making a new HTTP request each time.
* Unlike traditional HTTP (which is one-way: client → server), WebSockets are **two-way**: client ⇄ server.

---

## 📦 How to Use WebSockets in FastAPI

### 1. **Install the required library**

```bash
pip install websockets
```

---

### 2. **Create a FastAPI WebSocket endpoint**

```python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()  # Accept the WebSocket connection
    while True:
        data = await websocket.receive_text()  # Receive a message from the client
        await websocket.send_text(f"Message text was: {data}")  # Send a reply
```

✅ This creates a WebSocket endpoint at: `ws://localhost:8000/ws`

---

### 3. **Add a simple HTML page to test it**

You can serve this HTML from FastAPI using `HTMLResponse`:

```python
from fastapi.responses import HTMLResponse

html = """
<!DOCTYPE html>
<html>
  <body>
    <h1>WebSocket Chat</h1>
    <form onsubmit="sendMessage(event)">
      <input type="text" id="messageText" autocomplete="off"/>
      <button>Send</button>
    </form>
    <ul id="messages"></ul>
    <script>
      var ws = new WebSocket("ws://localhost:8000/ws");
      ws.onmessage = function(event) {
        var messages = document.getElementById("messages");
        var message = document.createElement("li");
        message.textContent = event.data;
        messages.appendChild(message);
      };
      function sendMessage(event) {
        var input = document.getElementById("messageText");
        ws.send(input.value);
        input.value = '';
        event.preventDefault();
      }
    </script>
  </body>
</html>
"""

@app.get("/")
async def get():
    return HTMLResponse(html)
```

💡 When you run this and visit `http://localhost:8000`, you’ll get a web page where you can:

* Type messages
* Click "Send"
* See the messages echoed back by the server

---

### 4. **Handle Disconnections & Multiple Clients**

To manage multiple users and handle disconnections:

#### Create a connection manager:

```python
class ConnectionManager:
    def __init__(self):
        self.active_connections: list[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)
```

#### Use it in your WebSocket endpoint:

```python
from fastapi import WebSocketDisconnect

manager = ConnectionManager()

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: int):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast(f"Client #{client_id} says: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f"Client #{client_id} left the chat")
```

---

### 5. **Use FastAPI Features like Depends, Cookies, Query**

WebSockets support dependencies just like HTTP endpoints:

```python
from fastapi import Depends, Query, WebSocketException, status
from typing import Annotated

async def get_token(
    token: Annotated[str | None, Query()] = None
):
    if not token:
        raise WebSocketException(code=status.WS_1008_POLICY_VIOLATION)
    return token

@app.websocket("/secure-ws")
async def websocket_endpoint(websocket: WebSocket, token: Annotated[str, Depends(get_token)]):
    await websocket.accept()
    await websocket.send_text(f"Your token: {token}")
```

---

## ✅ Summary

| Feature                  | Description                                |
| ------------------------ | ------------------------------------------ |
| `WebSocket`              | For creating real-time two-way connections |
| `receive_text()`         | Read message from client                   |
| `send_text()`            | Send message to client                     |
| `WebSocketDisconnect`    | Exception to detect client disconnection   |
| `Depends`, `Query`, etc. | Fully supported in WebSocket endpoints     |

---

