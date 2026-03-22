# Task 0.01 — Monorepo Scaffold

**Owner:** Achyut
**Estimated time:** 2-3 hours
**Goal:** Set up the Turborepo monorepo so all three workstreams can install dependencies and run their apps locally.

---

## What to build

### 1. Root `package.json`

```json
{
  "name": "forge",
  "private": true,
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "typecheck": "turbo run typecheck",
    "test": "turbo run test"
  },
  "devDependencies": {
    "turbo": "latest"
  }
}
```

### 2. `turbo.json`

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": { "dependsOn": ["^build"], "outputs": [".next/**", "dist/**"] },
    "dev": { "cache": false, "persistent": true },
    "lint": {},
    "typecheck": {},
    "test": { "cache": false }
  }
}
```

### 3. `apps/web/package.json`

Scaffold a Next.js 14 App Router project with:
- `next@14`, `react@18`, `react-dom@18`
- `typescript`, `@types/react`, `@types/node`
- `tailwindcss`, `postcss`, `autoprefixer`
- `@supabase/supabase-js`, `@supabase/ssr`

Scripts: `dev`, `build`, `lint`, `typecheck`

### 4. `apps/api/pyproject.toml`

Poetry project with:
- `python = "^3.12"`
- `fastapi`, `uvicorn[standard]`, `pydantic[email]` (v2)
- `supabase`, `anthropic`, `openai`, `httpx`
- `stripe`, `resend`, `upstash-redis`
- `slowapi`, `python-dotenv`
- Dev deps: `pytest`, `pytest-asyncio`, `httpx` (for testing)

### 5. `apps/api/Dockerfile`

```dockerfile
FROM python:3.12-slim
WORKDIR /app
RUN pip install poetry
COPY pyproject.toml poetry.lock* ./
RUN poetry install --no-root --no-dev
COPY src/ ./src/
CMD ["poetry", "run", "uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 6. `packages/shared/package.json`

```json
{
  "name": "@forge/shared",
  "version": "0.0.1",
  "main": "./src/index.ts",
  "dependencies": {
    "@supabase/supabase-js": "*",
    "zod": "^3"
  }
}
```

### 7. `.github/workflows/ci.yml`

```yaml
name: CI
on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]

jobs:
  ci:
    name: ci
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm install
      - run: npm run typecheck
      - run: npm run lint
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install poetry && cd apps/api && poetry install
      - run: cd apps/api && poetry run pytest
```

### 8. `supabase/config.toml`

Run `supabase init` inside the `supabase/` folder to generate this.

---

## Files to create/edit

- `package.json` (root)
- `turbo.json`
- `apps/web/package.json`
- `apps/web/next.config.js`
- `apps/web/tsconfig.json` (strict mode)
- `apps/web/tailwind.config.ts`
- `apps/api/pyproject.toml`
- `apps/api/Dockerfile`
- `apps/api/src/main.py` (minimal FastAPI app with `/api/health` endpoint)
- `packages/shared/package.json`
- `packages/shared/tsconfig.json`
- `.github/workflows/ci.yml`
- `supabase/config.toml`
