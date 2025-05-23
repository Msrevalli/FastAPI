FastAPI script that uses `OAuth2PasswordBearer`, a fake token decoder, and dependency injection to return user details:

```python
from typing import Annotated

from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")


class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool | None = None


def fake_decode_token(token: str) -> User:
    return User(
        username=token + "fakedecoded",
        email="john@example.com",
        full_name="John Doe"
    )


async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]) -> User:
    user = fake_decode_token(token)
    return user


@app.get("/users/me")
async def read_users_me(current_user: Annotated[User, Depends(get_current_user)]):
    return current_user
```

### 💡 To test this:

1. Run the app with `uvicorn`:

   ```bash
   uvicorn your_script_name:app --reload
   ```
2. Send a request to `/users/me` with a Bearer token:

   ```bash
   curl -H "Authorization: Bearer test123" http://127.0.0.1:8000/users/me
   ```

It will return:

```json
{
  "username": "test123fakedecoded",
  "email": "john@example.com",
  "full_name": "John Doe",
  "disabled": null
}
```


