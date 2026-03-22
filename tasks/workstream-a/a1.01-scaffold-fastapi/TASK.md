# Task a1.01 — Scaffold FastAPI

**Owner:** Achyut
**Estimated time:** 2-3 hours
**Goal:** Working FastAPI app with health endpoint, Supabase client, Redis client, structured logging, and all middleware configured. No business logic yet — just the foundation.

---

## What to build

### 1. `src/main.py` — FastAPI app entry

```python
from fastapi import FastAPI
from slowapi import Limiter
from slowapi.util import get_remote_address
from src.routers import health, quote, orders
from src.utils.logger import setup_logging

app = FastAPI(title="Forge API", version="1.0.0")

# Rate limiting: 100 req/min per API key
limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

# Routers
app.include_router(health.router)
app.include_router(quote.router, prefix="/api")
app.include_router(orders.router, prefix="/api")

@app.on_event("startup")
async def startup():
    setup_logging()
```

### 2. `src/config.py` — Settings from env vars

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    supabase_url: str
    supabase_service_role_key: str
    anthropic_api_key: str
    openai_api_key: str
    upstash_redis_url: str
    upstash_redis_token: str
    api_key: str  # for frontend → backend auth
    stripe_webhook_secret: str
    slack_webhook_alerts: str
    slack_webhook_orders: str
    slack_webhook_wins: str

    class Config:
        env_file = ".env.local"

settings = Settings()
```

### 3. `src/routers/health.py`

```python
@router.get("/api/health")
async def health():
    return {"status": "ok", "version": "1.0.0"}
```

### 4. `src/utils/supabase.py` — Typed async Supabase client

```python
from supabase import create_client, Client
from src.config import settings

def get_supabase() -> Client:
    return create_client(settings.supabase_url, settings.supabase_service_role_key)
```

### 5. `src/utils/redis.py` — Upstash Redis client

```python
from upstash_redis import Redis
from src.config import settings

def get_redis() -> Redis:
    return Redis(url=settings.upstash_redis_url, token=settings.upstash_redis_token)
```

### 6. `src/utils/logger.py` — Structured logging

```python
import logging
import json

class JSONFormatter(logging.Formatter):
    def format(self, record):
        return json.dumps({
            "level": record.levelname,
            "message": record.getMessage(),
            "module": record.module,
        })

def setup_logging():
    handler = logging.StreamHandler()
    handler.setFormatter(JSONFormatter())
    logging.root.addHandler(handler)
    logging.root.setLevel(logging.INFO)
```

### 7. `src/utils/slack.py` — Slack webhook helpers

```python
import httpx
from src.config import settings

async def post_to_slack(webhook_url: str, message: str):
    async with httpx.AsyncClient() as client:
        await client.post(webhook_url, json={"text": message})

async def alert(message: str):
    await post_to_slack(settings.slack_webhook_alerts, f"🚨 {message}")

async def notify_order(message: str):
    await post_to_slack(settings.slack_webhook_orders, f"📦 {message}")

async def notify_win(message: str):
    await post_to_slack(settings.slack_webhook_wins, f"🎉 {message}")
```

### 8. Auth middleware

Every route except `/api/health` must validate the `x-api-key` header against `settings.api_key`.

```python
# src/middleware/auth.py
from fastapi import Request, HTTPException
from src.config import settings

async def verify_api_key(request: Request, call_next):
    if request.url.path == "/api/health":
        return await call_next(request)
    api_key = request.headers.get("x-api-key")
    if api_key != settings.api_key:
        raise HTTPException(status_code=401, detail="Invalid API key")
    return await call_next(request)
```

---

## Files to create

- `src/main.py`
- `src/config.py`
- `src/routers/health.py`
- `src/routers/__init__.py`
- `src/utils/supabase.py`
- `src/utils/redis.py`
- `src/utils/logger.py`
- `src/utils/slack.py`
- `src/middleware/auth.py`
- `tests/test_health.py`
