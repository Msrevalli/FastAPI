Great summary of **OpenAPI Webhooks** support in FastAPI! This is a relatively new feature (OpenAPI 3.1+ and FastAPI 0.99+), designed to document events your app **can send to others**—essentially the inverse of callbacks, but focusing on *push* notifications your app initiates to URLs your users register.

---

### Quick Recap on Webhooks in FastAPI

* Webhooks are outbound event notifications your API sends to users' endpoints.
* Users register their URLs somewhere (e.g., dashboard).
* Your app sends HTTP requests (e.g. POST) to those URLs when events happen.
* You document those webhook event payloads and methods with FastAPI using the `.webhooks` router.
* The string identifier you pass (e.g. `"new-subscription"`) is the event name, **not a path**.
* The actual URL users listen on is not part of your API, it’s external and configured outside of FastAPI.

---

### Minimal example from your snippet:

```python
from datetime import datetime
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Subscription(BaseModel):
    username: str
    monthly_fee: float
    start_date: datetime

@app.webhooks.post("new-subscription")
def new_subscription(body: Subscription):
    """
    Webhook event: when a new subscription occurs, your API sends this POST
    request with this body to the user-registered URL.
    """

@app.get("/users/")
def read_users():
    return ["Rick", "Morty"]
```

---

### What happens here?

* The `/users/` endpoint is a normal API path.
* The `new-subscription` webhook is documented as a webhook event your API *sends*.
* Swagger UI (`/docs`) will show this webhook event in a dedicated **Webhooks** section.
* The event schema and HTTP method are clearly documented for your API consumers.
* Your API clients will know what JSON to expect in webhook calls, even if they host those webhook endpoints elsewhere.

---

### How this helps API users

* They can **auto-generate code** for webhook receivers based on your documented payloads.
* They understand what events your system will notify them about and how to implement their side.
* It's official, discoverable, and validated in the OpenAPI spec.

---

### A few extra points:

* **The `.webhooks` attribute is actually an `APIRouter` instance**, just scoped differently to mark these as webhook events.
* You can use other HTTP methods too: `@app.webhooks.put()`, `@app.webhooks.delete()`, etc.
* Since webhooks don’t have a fixed path, the URL itself is **not part of the API spec**, only the event name and payload are.
* Users specify the URL in your app dashboard or other registration system (out of FastAPI’s scope).

---

### Would you like help with?

* Implementing webhook sending logic from your app? (e.g., sending POST to user URLs)
* Designing a registration system for user webhook URLs?
* Working with webhook security (e.g., signing requests, validating payloads)?
* Generating client code based on webhook docs?
* Examples combining callbacks and webhooks in the same app?

---


