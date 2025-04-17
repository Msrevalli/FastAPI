SQLModel is super clean and tightly integrated with FastAPI. Let's recap and walk through the next steps from where your snippet left off—after creating the `Hero` model and the engine setup.

---

### ✅ Full Minimal Example with SQLModel + FastAPI + SQLite

Here’s the full example you can copy and run:

```python
# main.py

from typing import Annotated
from fastapi import FastAPI, HTTPException, Depends
from sqlmodel import Field, SQLModel, Session, create_engine, select

# Step 1: Define the model
class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    age: int | None = Field(default=None, index=True)
    secret_name: str

# Step 2: Set up the database engine
sqlite_file_name = "database.db"
sqlite_url = f"sqlite:///{sqlite_file_name}"
connect_args = {"check_same_thread": False}

engine = create_engine(sqlite_url, echo=True, connect_args=connect_args)

# Step 3: Create the database and table
def create_db_and_tables():
    SQLModel.metadata.create_all(engine)

# Step 4: Dependency to get a session
def get_session():
    with Session(engine) as session:
        yield session

# Step 5: Create the FastAPI app
app = FastAPI()

@app.on_event("startup")
def on_startup():
    create_db_and_tables()

# Step 6: Endpoint to add a new hero
@app.post("/heroes/", response_model=Hero)
def create_hero(hero: Hero, session: Annotated[Session, Depends(get_session)]):
    session.add(hero)
    session.commit()
    session.refresh(hero)
    return hero

# Step 7: Endpoint to read heroes
@app.get("/heroes/", response_model=list[Hero])
def read_heroes(session: Annotated[Session, Depends(get_session)]):
    heroes = session.exec(select(Hero)).all()
    return heroes
```

---

### 🧪 How to Run It

1. Create a virtual environment:
   ```bash
   python -m venv env
   source env/bin/activate  # or .\env\Scripts\activate on Windows
   ```

2. Install dependencies:
   ```bash
   pip install fastapi uvicorn sqlmodel
   ```

3. Run the server:
   ```bash
   uvicorn main:app --reload
   ```

Then go to `http://127.0.0.1:8000/docs` to play with your API in Swagger UI 🎉

---

### 🛠 What's Next?

- Add validation (e.g., age ≥ 0)
- Split models into `HeroCreate`, `HeroRead` for better control
- Add update/delete endpoints
- Switch to PostgreSQL for production

Awesome! Let’s level up your FastAPI + SQLModel app.

---

## 💡 Step-by-Step Improvements

Here’s what we’ll do next:
1. **Split models** for input/output (like `HeroCreate`, `HeroRead`)
2. **Add validation** (e.g., age must be positive)
3. **Add update & delete endpoints**

---

### ✅ 1. Split Models for Create/Read

You don’t want users to send `id` manually. So you define separate models:

```python
# models.py or top of main.py

from sqlmodel import SQLModel, Field
from typing import Optional

class HeroBase(SQLModel):
    name: str
    age: Optional[int] = None
    secret_name: str

class Hero(HeroBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)

class HeroCreate(HeroBase):
    pass

class HeroRead(HeroBase):
    id: int
```

---

### ✅ 2. Add Validation (age >= 0)

You can use Pydantic validators like this:

```python
from pydantic import field_validator

class HeroBase(SQLModel):
    name: str
    age: Optional[int] = None
    secret_name: str

    @field_validator("age")
    @classmethod
    def age_must_be_positive(cls, v):
        if v is not None and v < 0:
            raise ValueError("Age must be a positive number")
        return v
```

---

### ✅ 3. Update and Delete Endpoints

Add to your FastAPI routes:

```python
from fastapi import Path

@app.get("/heroes/{hero_id}", response_model=HeroRead)
def read_hero(hero_id: int, session: Annotated[Session, Depends(get_session)]):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    return hero

@app.patch("/heroes/{hero_id}", response_model=HeroRead)
def update_hero(hero_id: int, hero_data: HeroCreate, session: Annotated[Session, Depends(get_session)]):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    for key, value in hero_data.dict(exclude_unset=True).items():
        setattr(hero, key, value)
    session.add(hero)
    session.commit()
    session.refresh(hero)
    return hero

@app.delete("/heroes/{hero_id}")
def delete_hero(hero_id: int, session: Annotated[Session, Depends(get_session)]):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    session.delete(hero)
    session.commit()
    return {"ok": True}
```

---
