### ✅ **What is `BackgroundTasks` in FastAPI?**

`BackgroundTasks` lets you **run tasks *after* sending a response** to the client.  
Perfect for actions like:
- Sending emails
- Logging
- File processing
- Webhooks

The client doesn’t wait for the task to complete — they get an immediate response.

---

### 🚀 **Basic Example**

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()

def write_notification(email: str, message=""):
    with open("log.txt", "w") as f:
        f.write(f"notification for {email}: {message}")

@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}
```

---

### 🔁 **`.add_task()` arguments**

- Positional args: `write_notification, email`
- Keyword args: `message="some notification"`

FastAPI queues this task *after* the response.

---

### 🔌 **Dependency Injection + BackgroundTasks**

You can pass `background_tasks` into dependencies too.

```python
from fastapi import BackgroundTasks, Depends, FastAPI
from typing import Annotated

app = FastAPI()

def write_log(message: str):
    with open("log.txt", "a") as log:
        log.write(message)

def get_query(background_tasks: BackgroundTasks, q: str | None = None):
    if q:
        background_tasks.add_task(write_log, f"found query: {q}\n")
    return q

@app.post("/send-notification/{email}")
async def send_notification(
    email: str,
    background_tasks: BackgroundTasks,
    q: Annotated[str, Depends(get_query)]
):
    background_tasks.add_task(write_log, f"message to {email}\n")
    return {"message": "Message sent"}
```

> ✅ FastAPI merges all background tasks from dependencies into one queue.

---

### ⚠️ When *not* to use it

For **heavy processing** or **cross-server background work**, use tools like:
- [Celery](https://docs.celeryq.dev/en/stable/index.html)
- [RQ](https://python-rq.org/)
- [Dramatiq](https://dramatiq.io/)

These need a message broker like Redis or RabbitMQ.

---

