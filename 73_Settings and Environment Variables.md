Here's a concise yet thorough summary and example to understand **FastAPI settings management using Pydantic Settings and environment variables**:

---

# FastAPI Settings & Environment Variables with Pydantic Settings

---

## Why use Pydantic Settings?

* Environment variables are strings only.
* Your app needs typed & validated config (e.g., ints, emails).
* Pydantic Settings automates reading env vars, validating & converting them.
* Supports `.env` files for easier local config.

---

## Setup

```bash
pip install pydantic-settings python-dotenv
```

`python-dotenv` is needed if you want to load `.env` files.

---

## Step 1: Define Settings Model

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    app_name: str = "Awesome API"
    admin_email: str
    items_per_user: int = 50

    # Tell Pydantic where to find the .env file
    model_config = SettingsConfigDict(env_file=".env")
```

---

## Step 2: Use Settings in FastAPI

### Simple global usage:

```python
settings = Settings()

from fastapi import FastAPI

app = FastAPI()

@app.get("/info")
async def info():
    return {
        "app_name": settings.app_name,
        "admin_email": settings.admin_email,
        "items_per_user": settings.items_per_user,
    }
```

---

## Step 3: Use `.env` file

Create a `.env` file in your project root:

```
ADMIN_EMAIL="deadpool@example.com"
APP_NAME="ChimichangApp"
```

Pydantic reads these automatically thanks to `env_file=".env"` config.

---

## Step 4: Use Dependency Injection & Cache for Performance

To avoid reloading the settings on every request, use:

```python
from functools import lru_cache
from typing import Annotated
from fastapi import Depends, FastAPI

from .config import Settings  # your settings file

app = FastAPI()

@lru_cache
def get_settings() -> Settings:
    return Settings()

@app.get("/info")
async def info(settings: Annotated[Settings, Depends(get_settings)]):
    return {
        "app_name": settings.app_name,
        "admin_email": settings.admin_email,
        "items_per_user": settings.items_per_user,
    }
```

---

## Step 5: Override Settings for Testing

```python
from fastapi.testclient import TestClient
from .main import app, get_settings
from .config import Settings

client = TestClient(app)

def get_settings_override():
    return Settings(admin_email="testing_admin@example.com")

app.dependency_overrides[get_settings] = get_settings_override

def test_info():
    response = client.get("/info")
    assert response.json() == {
        "app_name": "Awesome API",
        "admin_email": "testing_admin@example.com",
        "items_per_user": 50,
    }
```

---

# Summary

* Use `pydantic_settings.BaseSettings` to define your config class.
* Pydantic reads env vars, validates, and converts types automatically.
* Use `.env` files with `model_config = SettingsConfigDict(env_file=".env")`.
* Use `@lru_cache` on your `get_settings` dependency to cache the settings.
* Override the settings dependency easily during tests.

---

