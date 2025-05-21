**Extra Data Types** supported by FastAPI/Pydantic — 

---

### Extra Data Types Supported

| Type                   | Python Type          | JSON / OpenAPI Representation                                                            |
| ---------------------- | -------------------- | ---------------------------------------------------------------------------------------- |
| **UUID**               | `uuid.UUID`          | String UUID (e.g., `"3fa85f64-5717-4562-b3fc-2c963f66afa6"`)                             |
| **datetime.datetime**  | `datetime.datetime`  | ISO 8601 datetime string (e.g., `"2008-09-15T15:53:00+05:00"`)                           |
| **datetime.date**      | `datetime.date`      | ISO 8601 date string (e.g., `"2008-09-15"`)                                              |
| **datetime.time**      | `datetime.time`      | ISO 8601 time string (e.g., `"14:23:55.003"`)                                            |
| **datetime.timedelta** | `datetime.timedelta` | Float representing total seconds (e.g., `3600.0`) or ISO 8601 duration string (advanced) |
| **frozenset**          | `frozenset`          | List of unique items (duplicates removed)                                                |
| **bytes**              | `bytes`              | Base64-encoded string, treated as `str` with binary format                               |
| **Decimal**            | `decimal.Decimal`    | Treated as `float` in JSON                                                               |

---

### Why use these?

* **Validation:** Incoming data is automatically validated and converted.
* **Editor support:** Autocompletion and type hints.
* **OpenAPI docs:** Proper schema and example generation.
* **Automatic serialization:** Returned data is converted correctly.

---
```python
from datetime import datetime, time, timedelta
from typing import Annotated
from uuid import UUID

from fastapi import Body, FastAPI

app = FastAPI()


@app.put("/items/{item_id}")
async def read_items(
    item_id: UUID,
    start_datetime: Annotated[datetime, Body()],
    end_datetime: Annotated[datetime, Body()],
    process_after: Annotated[timedelta, Body()],
    repeat_at: Annotated[time | None, Body()] = None,
):
    start_process = start_datetime + process_after
    duration = end_datetime - start_process
    return {
        "item_id": item_id,
        "start_datetime": start_datetime,
        "end_datetime": end_datetime,
        "process_after": process_after,
        "repeat_at": repeat_at,
        "start_process": start_process,
        "duration": duration,
    }
```

---

### 🌐 Example Request URL

```
PUT /items/3fa85f64-5717-4562-b3fc-2c963f66afa6
```

---

### 📦 Example JSON Request Body

```json
{
  "start_datetime": "2025-05-21T10:00:00",
  "end_datetime": "2025-05-21T12:00:00",
  "process_after": 600.0,
  "repeat_at": "14:00:00"
}
```

* `start_datetime` and `end_datetime`: ISO 8601 datetime strings.
* `process_after`: a `timedelta` in **seconds** (600 seconds = 10 minutes).
* `repeat_at`: a `time` value in `HH:MM:SS` format.

---

### 🧠 Explanation of Extra Types

| Type        | Python Type          | JSON Format                              |
| ----------- | -------------------- | ---------------------------------------- |
| `UUID`      | `uuid.UUID`          | `"3fa85f64-5717-4562-b3fc-2c963f66afa6"` |
| `datetime`  | `datetime.datetime`  | `"2025-05-21T10:00:00"`                  |
| `date`      | `datetime.date`      | `"2025-05-21"`                           |
| `time`      | `datetime.time`      | `"14:00:00"`                             |
| `timedelta` | `datetime.timedelta` | `600.0` (seconds as float)               |
| `frozenset` | `frozenset`          | List, converted to set                   |
| `bytes`     | `bytes`              | Base64 string / binary format            |
| `Decimal`   | `decimal.Decimal`    | Treated like `float`                     |

---


