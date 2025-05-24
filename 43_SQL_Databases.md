**FastAPI + SQLModel script** for managing a simple `Hero` database, complete with all the logic for creating, reading, and deleting heroes using dependency injection and `yield`-based session management:

```python
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, Query
from sqlmodel import Field, Session, SQLModel, create_engine, select


# --- SQLModel ORM Model ---
class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    age: int | None = Field(default=None, index=True)
    secret_name: str


# --- Database Setup ---
sqlite_file_name = "database.db"
sqlite_url = f"sqlite:///{sqlite_file_name}"

connect_args = {"check_same_thread": False}  # Needed for SQLite
engine = create_engine(sqlite_url, connect_args=connect_args)


# --- Create DB & Tables ---
def create_db_and_tables():
    SQLModel.metadata.create_all(engine)


# --- Dependency: Session Generator ---
def get_session():
    with Session(engine) as session:
        yield session


SessionDep = Annotated[Session, Depends(get_session)]


# --- FastAPI App ---
app = FastAPI()


@app.on_event("startup")
def on_startup():
    create_db_and_tables()


# --- Routes ---

# Create a hero
@app.post("/heroes/", response_model=Hero)
def create_hero(hero: Hero, session: SessionDep) -> Hero:
    session.add(hero)
    session.commit()
    session.refresh(hero)
    return hero


# Read all heroes with pagination
@app.get("/heroes/", response_model=list[Hero])
def read_heroes(
    session: SessionDep,
    offset: int = 0,
    limit: Annotated[int, Query(le=100)] = 100,
) -> list[Hero]:
    heroes = session.exec(select(Hero).offset(offset).limit(limit)).all()
    return heroes


# Read a single hero by ID
@app.get("/heroes/{hero_id}", response_model=Hero)
def read_hero(hero_id: int, session: SessionDep) -> Hero:
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    return hero


# Delete a hero by ID
@app.delete("/heroes/{hero_id}")
def delete_hero(hero_id: int, session: SessionDep):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    session.delete(hero)
    session.commit()
    return {"ok": True}
```

---

### ✅ How to Run

1. Save this as `main.py`
2. Run it with:

```bash
uvicorn main:app --reload
```

3. Open your browser at: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

---

Your script is a complete FastAPI application that uses **SQLModel** (built on top of SQLAlchemy and Pydantic) to implement a simple **CRUD API for managing heroes** in a SQLite database.

---

### ✅ What It Does

1. **Defines a `Hero` model** (with SQLModel for ORM mapping).
2. **Creates a SQLite database** and initializes tables on startup.
3. **Implements endpoints to:**

   * Create a new hero (`POST /heroes/`)
   * Read all heroes with pagination (`GET /heroes/`)
   * Read a hero by ID (`GET /heroes/{hero_id}`)
   * Delete a hero (`DELETE /heroes/{hero_id}`)

---

### 📦 Tech Stack

* **FastAPI**: For building the web API.
* **SQLModel**: ORM for database interaction.
* **SQLite**: Simple file-based database.
* **Dependency Injection**: Using `Depends` to inject a DB session.
* **Validation & Docs**: Built-in via Pydantic (from SQLModel) and FastAPI.

---

### 🧠 Key Concepts Used

#### ✅ `Annotated[Session, Depends(get_session)]`

This injects a database session into route functions cleanly.

```python
SessionDep = Annotated[Session, Depends(get_session)]
```

#### ✅ `yield`-based Dependency

In `get_session()`, the `yield` pattern ensures the session is closed after use:

```python
def get_session():
    with Session(engine) as session:
        yield session
```

#### ✅ `@app.on_event("startup")`

Used to automatically create database tables when the app starts:

```python
@app.on_event("startup")
def on_startup():
    create_db_and_tables()
```

---

### 📋 Example Hero JSON

```json
{
  "name": "Spider-Man",
  "age": 18,
  "secret_name": "Peter Parker"
}
```

---

### 🚀 Run It

You can run the app with:

```bash
uvicorn my_script:app --reload
```

Then test it at `http://127.0.0.1:8000/docs`

---

Sure! Your script is a **FastAPI application** that uses **SQLModel** (an ORM library built on SQLAlchemy + Pydantic) to manage a simple `Hero` database. It covers full **CRUD operations** (Create, Read, Delete) with proper **dependency injection** and **session management** using `yield`.

---

## 🔧 Breakdown of Your Script

### 1. **Model Definition**

```python
class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    age: int | None = Field(default=None, index=True)
    secret_name: str
```

* This defines a SQLModel model called `Hero`.
* `table=True` makes this model correspond to a database table.
* Fields: `id`, `name`, `age`, and `secret_name`.
* `id` is optional and auto-incremented.
* `name` and `age` are indexed for faster querying.

---

### 2. **Database Configuration**

```python
sqlite_file_name = "database.db"
sqlite_url = f"sqlite:///{sqlite_file_name}"
connect_args = {"check_same_thread": False}
engine = create_engine(sqlite_url, connect_args=connect_args)
```

* Using SQLite as the database.
* `check_same_thread=False` is needed for SQLite to allow usage across threads.

---

### 3. **Creating Tables**

```python
def create_db_and_tables():
    SQLModel.metadata.create_all(engine)
```

* Creates the database tables based on models when the app starts.

---

### 4. **Session Dependency with `yield`**

```python
def get_session():
    with Session(engine) as session:
        yield session
```

* Dependency that **opens a session**, **yields it**, and **automatically closes** it afterward.
* `SessionDep = Annotated[Session, Depends(get_session)]` lets you reuse this type hint easily.

---

### 5. **FastAPI App & Startup Hook**

```python
app = FastAPI()

@app.on_event("startup")
def on_startup():
    create_db_and_tables()
```

* FastAPI app is created.
* On startup, it ensures tables exist (useful for first-run).

---

## 📦 API Routes

### ✅ POST `/heroes/` — Create a Hero

```python
@app.post("/heroes/")
def create_hero(hero: Hero, session: SessionDep) -> Hero:
    session.add(hero)
    session.commit()
    session.refresh(hero)
    return hero
```

* Takes a `Hero` object from the request body.
* Adds it to the DB, commits, and returns the created hero.

---

### 📖 GET `/heroes/` — Read All Heroes

```python
@app.get("/heroes/")
def read_heroes(session: SessionDep, offset: int = 0, limit: Annotated[int, Query(le=100)] = 100):
    heroes = session.exec(select(Hero).offset(offset).limit(limit)).all()
    return heroes
```

* Supports pagination with `offset` and `limit`.
* Limits to max 100 records per request.
* Returns a list of heroes.

---

### 🔍 GET `/heroes/{hero_id}` — Read Hero by ID

```python
@app.get("/heroes/{hero_id}")
def read_hero(hero_id: int, session: SessionDep) -> Hero:
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    return hero
```

* Fetches a single hero by `hero_id`.
* Returns 404 if not found.

---

### ❌ DELETE `/heroes/{hero_id}` — Delete Hero by ID

```python
@app.delete("/heroes/{hero_id}")
def delete_hero(hero_id: int, session: SessionDep):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    session.delete(hero)
    session.commit()
    return {"ok": True}
```

* Deletes a hero by ID.
* Returns success flag or 404 if not found.

---

## 🧠 Concepts You’re Using

| Concept                     | Purpose                                  |
| --------------------------- | ---------------------------------------- |
| `yield`-based dependency    | For opening and cleaning up DB sessions  |
| `Depends` and `Annotated`   | Clean and reusable dependency injection  |
| `@app.on_event("startup")`  | Setup DB tables at app launch            |
| `sqlmodel.select(...)`      | SQL-style querying with model classes    |
| `Query(le=100)`             | Query parameter validation               |
| `response_model` (implicit) | FastAPI uses the model for response docs |

---

## ✅ Great Practices

* 👍 Clean separation of concerns (models, dependencies, logic).
* 🔒 Proper error handling with `HTTPException`.
* 🧪 Reusable session dependency.
* 📚 Pagination support on list endpoints.

---



