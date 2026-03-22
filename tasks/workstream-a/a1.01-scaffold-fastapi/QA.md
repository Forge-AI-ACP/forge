# QA: a1.01 — Scaffold FastAPI

## Automated checks (ALL must pass)

### 1. Server starts
- [ ] `poetry run uvicorn src.main:app --reload` starts without errors

### 2. Health endpoint
- [ ] `curl http://localhost:8000/api/health` returns `{"status": "ok"}`

### 3. Auth middleware
- [ ] `curl http://localhost:8000/api/quote` without header returns 401
- [ ] Same request with `x-api-key: <correct key>` proceeds normally

### 4. Connections
- [ ] Supabase client initialises without error on startup
- [ ] Redis client initialises without error on startup

### 5. Tests
- [ ] `poetry run pytest` exits 0
- [ ] `tests/test_health.py` passes

## Manual checks
- [ ] No hardcoded secrets — all read from env vars via config.py
- [ ] Structured JSON logs appear in terminal on startup
