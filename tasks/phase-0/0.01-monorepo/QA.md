# QA: 0.01 — Monorepo Scaffold

## Automated checks (ALL must pass)

### 1. Dependencies install
- [ ] `npm install` from repo root exits 0 with no errors
- [ ] `cd apps/api && poetry install` exits 0

### 2. Dev servers start
- [ ] `cd apps/web && npm run dev` starts without errors (Next.js running on localhost:3000)
- [ ] `cd apps/api && poetry run uvicorn src.main:app --reload` starts without errors

### 3. Health check
- [ ] `curl http://localhost:8000/api/health` returns `{ "status": "ok" }`

### 4. TypeScript
- [ ] `cd apps/web && npm run typecheck` exits 0
- [ ] `cd apps/web && npm run lint` exits 0

### 5. Python tests
- [ ] `cd apps/api && poetry run pytest` exits 0 (even with 0 tests collected)

### 6. CI workflow valid
- [ ] `.github/workflows/ci.yml` exists and is valid YAML

## Manual checks

- [ ] Folder structure matches CLAUDE.md exactly
- [ ] No hardcoded secrets anywhere — all env vars reference `.env.local`
- [ ] `tsconfig.json` has `"strict": true`
