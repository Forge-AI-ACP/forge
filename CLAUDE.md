# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Read this ENTIRELY before executing any task. Every section is load-bearing.

---

## PROJECT IDENTITY

**Codename:** Forge — an AI-powered service factory. Customers describe a task, pay, AI delivers it. Repeat requests become standalone products.
**Team:** Prateek (P) — Storefront, Achyut (A) — Factory, CK (C) — Brain
**Build method:** Agentic coding via Claude Code. Each person runs their workstream from this root.
**Production target:** NOT a prototype. Every line ships. Every commit must be deployable.
**Deep context:** `forge-blueprint.md` has the full build plan and task breakdowns. `three_workstream_dependency_map.html` shows the critical-path dependency graph. `context.md` has the full verbose specification including complete SQL schema.

---

## MONOREPO STRUCTURE

```
forge/
├── claude.md                          ← YOU ARE HERE
├── .env.example                       ← All env vars (never commit .env)
├── .env.local                         ← Local overrides (gitignored)
├── turbo.json                         ← Turborepo config
├── package.json                       ← Root workspace config
│
├── contracts/                         ← SINGLE SOURCE OF TRUTH
│   ├── database.sql                   ← All Supabase table definitions
│   ├── types.ts                       ← Shared TypeScript types (generated from SQL)
│   ├── enums.ts                       ← All enum values used across the system
│   ├── api-schemas.ts                 ← Zod schemas for all API request/response shapes
│   └── events.ts                      ← All event names + payloads
│
├── tasks/                             ← TASK ORCHESTRATION SYSTEM
│   ├── phase-0/                       ← Foundation (all three, sync)
│   │   ├── 0.01-monorepo/
│   │   │   ├── TASK.md                ← What to build
│   │   │   ├── QA.md                  ← How to verify it works
│   │   │   └── DONE                   ← Created when QA passes (contains QA output log)
│   │   └── 0.02-supabase-schema/
│   │       ├── TASK.md
│   │       ├── QA.md
│   │       └── data/schema.sql        ← Reference copy of contracts/database.sql
│   ├── workstream-p/
│   │   ├── p1.01-scaffold-nextjs/
│   │   │   ├── TASK.md
│   │   │   ├── QA.md
│   │   │   ├── DEPS.md               ← Lists task dependencies
│   │   │   └── DONE
│   │   └── ...
│   ├── workstream-a/
│   └── workstream-c/
│
├── apps/
│   ├── web/                           ← Next.js 14 App Router — Prateek owns
│   │   └── src/
│   │       ├── app/                   ← App Router pages (see page structure below)
│   │       ├── components/            ← React components
│   │       ├── lib/                   ← Utilities, Supabase client, Stripe helpers
│   │       ├── hooks/                 ← Custom React hooks
│   │       └── emails/                ← React Email templates
│   │
│   └── api/                           ← Python FastAPI — Achyut owns
│       └── src/
│           ├── main.py                ← FastAPI app entry, middleware, startup
│           ├── config.py              ← Settings from env vars (Pydantic BaseSettings)
│           ├── routers/               ← quote.py, orders.py, health.py, admin.py
│           ├── services/              ← classifier.py, pricing.py, delivery.py, llm_router.py
│           ├── pipelines/             ← base.py, router.py, generic.py, automation.py, content.py, agent.py, integration.py, data.py
│           ├── qa/                    ← checker.py, escalation.py
│           ├── workers/               ← order_consumer.py, scheduler.py
│           └── utils/                 ← supabase.py, slack.py, cost_tracker.py, logger.py
│
├── packages/shared/                   ← Shared TS: supabase.ts, constants.ts, validators.ts
│
├── supabase/
│   ├── migrations/                    ← SQL migration files (sequential)
│   ├── seed.sql                       ← Test data for local dev
│   └── config.toml
│
├── .github/workflows/
│   ├── ci.yml                         ← Lint + type check + test on PR
│   ├── deploy-web.yml                 ← Auto-deploy web to Vercel on merge to main
│   └── deploy-api.yml                 ← Auto-deploy api to Railway on merge to main
│
└── scripts/
    ├── setup.sh                       ← One-command local dev setup
    ├── seed-test-order.ts             ← Creates test order for integration testing
    ├── run-qa.sh                      ← Runs QA for a specific task folder
    └── check-contracts.sh             ← Validates all apps against contracts/
```

### Frontend page structure (`apps/web/src/app/`)

```
app/
├── page.tsx                     ← Landing page
├── order/
│   ├── page.tsx                 ← Intake form
│   └── [id]/
│       ├── confirmed/page.tsx   ← Post-payment confirmation
│       ├── page.tsx             ← Order status (realtime)
│       └── feedback/page.tsx    ← Feedback form
├── dashboard/
│   ├── page.tsx                 ← Customer order list
│   └── [id]/page.tsx            ← Order detail
├── reviews/page.tsx             ← Wall of Love
├── products/[slug]/page.tsx     ← Productised service pages
├── admin/                       ← Internal dashboard (auth-gated)
└── api/webhooks/stripe/route.ts ← Stripe webhook handler
```

---

## COMMANDS

```bash
# Install
npm install                    # All workspaces

# Frontend (apps/web)
npm run dev                    # Dev server
npm run build                  # Production build
npm run typecheck              # TypeScript check
npm run lint                   # ESLint

# Backend (apps/api)
poetry run uvicorn src.main:app --reload   # Dev server
poetry run pytest                          # All tests
poetry run pytest tests/path/test_file.py::test_name  # Single test

# Supabase
supabase start                 # Local instance
supabase db reset              # Reset + apply migrations + seed
supabase migration new <name>  # New migration
supabase db push               # Push migrations to remote
supabase gen types typescript --local > contracts/types.ts  # Regen types

# QA / Scripts
bash scripts/run-qa.sh tasks/workstream-p/p1.01-scaffold-nextjs
bash scripts/check-contracts.sh
npx ts-node scripts/seed-test-order.ts
```

---

## CONTRACTS — THE LAW

`contracts/` is the single source of truth. No workstream may change a contract without a PR reviewed by all three.

1. **Never redefine a type** that exists in `contracts/types.ts` — always import it.
2. **Never invent a DB column** — if it's not in `contracts/database.sql`, it doesn't exist.
3. **Never invent an API endpoint** — if it's not in the current TASK.md, don't build it.
4. **Python Pydantic models** must mirror the Zod schemas in `contracts/api-schemas.ts` exactly.
5. **To add a field/table/enum:** edit the contract first (PR, all three approve), then implement.

### Core database tables

| Table | Owner | Key columns |
|-------|-------|-------------|
| `orders` | P→A | `id`, `customer_email`, `task_description`, `task_type`, `complexity_score`, `quoted_price_cents`, `payment_status`, `execution_status`, `stripe_checkout_session_id`, `assigned_pipeline`, `metadata` |
| `deliveries` | A→P | `id`, `order_id`, `deliverable_type`, `deliverable_url`, `execution_log`, `pipeline_used`, `pipeline_version`, `qa_passed`, `qa_attempts` |
| `feedback` | P→C | `id`, `order_id`, `delivery_id`, `rating`, `comment`, `revision_requested`, `tags`, `sentiment_score`, `issue_category` |
| `pipeline_improvements` | C→A | `id`, `pipeline_name`, `improvement_type`, `description`, `before/after_success_rate`, `status` (proposed/approved/applied/rejected) |
| `execution_logs` | A | `id`, `order_id`, `step_name`, `step_index`, `duration_ms`, `tokens_used`, `cost_cents`, `provider`, `model`, `success`, `error_message` |
| `pipeline_metrics` | C | aggregated per pipeline per period: `success_count`, `avg_cost_cents`, `avg_rating` |
| `task_patterns` | C | `pattern_name`, `frequency`, `product_opportunity_score`, `embedding` (vector 1536) |
| `pipeline_versions` | A | `pipeline_name`, `version`, `config` JSONB, `prompt_templates`, `is_active` |
| `referrals` | P | `referrer_code`, `referred_order_id`, `credit_cents`, `status` |

Enums: `task_type` (automation/content/agent/integration/data/other), `payment_status` (pending/paid/refunded/failed), `execution_status` (queued/classifying/running/qa_review/delivered/failed/revision/cancelled), `deliverable_type` (file/url/code_repo/api_endpoint/report/other).

RLS: `orders`, `deliveries`, `feedback` — customers see only their own rows. All other tables: service role only.

Full SQL schema is in `context.md` (lines 143–481) and will live in `contracts/database.sql` once created.

---

## TASK SYSTEM

### Per-task folder structure

```
tasks/workstream-X/x1.01-task-name/
├── TASK.md        ← WHAT to build (prompt + specs + file paths + contracts to reference)
├── QA.md          ← HOW to verify (automated checks + manual checks)
├── DEPS.md        ← WHICH tasks must be DONE before starting
├── data/          ← Reference data, sample inputs, test fixtures
└── DONE           ← Created ONLY when QA passes. Contains QA output log.
```

### Execution protocol

```
1. Check DEPS.md → All listed tasks have a DONE file?
   - NO: skip to next task with satisfied deps. Notify blocker in #forge-general.
   - YES: proceed

2. Read TASK.md thoroughly — note files to create, contracts to reference, env vars needed

3. Execute the task — import types from contracts/, never redefine locally

4. Run ALL checks in QA.md — if any fail, fix and re-run. Never skip.

5. When ALL QA checks pass:
   - Create DONE file with QA output log
   - git add + commit: "task(x1.01): short description"
   - Push to feature branch
   - If this is an interface point: notify in Slack
```

### Task naming

```
Phase 0:   0.01, 0.02 ...        (foundation)
Prateek:   p1.01, p1.02 ...      (phase 1), p2.01 ... (phase 2)
Achyut:    a1.01, a1.02 ...
CK:        c1.01, c1.02 ...
```

---

## QA PROTOCOL

### QA.md template (every task must have this)

```markdown
## Automated checks (must ALL pass)
### 1. Build check
- [ ] npm run build / poetry run pytest exits 0
- [ ] No TypeScript errors, no linting errors

### 2. Contract compliance
- [ ] All types imported from contracts/types.ts
- [ ] All Supabase queries match contracts/database.sql column names
- [ ] All API payloads match contracts/api-schemas.ts

### 3. Functional check
- [ ] [specific test command or curl]
- [ ] [expected output]

### 4. Integration check (if interface point)
- [ ] Write test data matching the contract → verify downstream workstream reads it
- [ ] npx ts-node scripts/seed-test-order.ts → order appears

## Manual checks
- [ ] [visual/UX requirements]
```

### QA severity levels

| Level | When | Fail action |
|-------|------|-------------|
| **GATE** | After every task | Cannot create DONE file. Fix before proceeding. |
| **MILESTONE** | After sprint completion | All three run integration test together. Must pass before next sprint. |
| **SMOKE** | After every deploy to main | Automated: landing loads, API health responds, Supabase connects. |

### Sprint milestone checklists

**Sprint 1 (days 1–3) — "One order flows through"**
```
[ ] P: Landing page loads at production URL
[ ] P: Intake form submits to Supabase
[ ] P: Stripe checkout completes (test mode)
[ ] P: Order confirmation email sends
[ ] A: Classifier returns complexity score for test task
[ ] A: Generic pipeline runs and produces output
[ ] A: Delivery row created in Supabase
[ ] A: QA check runs (even if basic)
[ ] C: All tables exist with correct schema
[ ] C: OrderBot posts to #forge-orders on new paid order
[ ] C: Execution logs are being written
[ ] INTEGRATION: Submit test order → flows through all three → delivery received
```

**Sprint 2 (days 4–7) — "Feedback loop works"**
```
[ ] P: Feedback form writes to Supabase; order status page shows realtime updates
[ ] A: Automation, content, agent pipelines work; QA catches bad output; retry/error handling works
[ ] C: Feedback ingestion auto-tags sentiment; admin dashboard shows orders + revenue; AlertBot fires on failure
[ ] INTEGRATION: Submit order → delivery → give feedback → visible in admin dashboard
```

**Sprint 3 (week 2) — "Customers self-serve"**
```
[ ] P: Customer dashboard with auth; revision request flow; retargeting pixel + first ad
[ ] A: Revision handler re-runs pipeline; multi-provider LLM router; sandbox for code tasks
[ ] C: Pattern detection; cost optimiser; weekly Slack digest
[ ] INTEGRATION: Customer places order → revises → gives feedback → pattern detected
```

**Sprint 4 (week 3) — "System scales"**
```
[ ] P: Referral system; Wall of Love auto-populates from 4+ star feedback
[ ] A: Pipeline A/B testing; caching layer; horizontal scaling (2+ workers)
[ ] C: Product opportunity scorer; churn detector
[ ] INTEGRATION: 50 test orders → concurrency handled → product opportunity surfaces
```

---

## TECH STACK

| Layer | Choice |
|-------|--------|
| Frontend | Next.js 14 App Router, TypeScript strict, Tailwind CSS + shadcn/ui |
| Forms | React Hook Form + Zod (schemas from `contracts/api-schemas.ts`) |
| Backend | Python FastAPI (3.12+), async everywhere, Pydantic v2 |
| Database | Supabase (Postgres + Auth + Realtime + Storage) |
| Queue | Supabase Realtime + Redis (Upstash) |
| AI | Claude API primary, GPT-4o fallback, Ollama cost tier |
| Payments | Stripe Checkout |
| Email | Resend + React Email |
| Hosting | Vercel (web), Railway (api, Dockerfile-based) |
| Monitoring | Sentry + PostHog |
| File uploads | Supabase Storage (max 50MB: pdf, csv, xlsx, png, jpg, zip) |

---

## CODING STANDARDS

### TypeScript (apps/web, packages/shared)
- Strict mode. No `any`. No `@ts-ignore`.
- Server Components by default; client components only for forms and realtime.
- Data fetching: server components → Supabase server client; client components → SWR or Supabase Realtime.
- Auth: Supabase Auth with middleware. Magic link primary, Google OAuth secondary.
- Analytics: PostHog — track `page_view`, `order_started`, `order_paid`, `delivery_viewed`, `feedback_submitted`.
- No `console.log` in production — use `console.error()` for errors, PostHog for analytics.

### Python (apps/api)
- Type hints everywhere. Pydantic v2 for all models.
- `supabase-py` async client — no raw SQL outside of migrations.
- Every external call (Supabase, Stripe, AI APIs) must be in try/except with structured logging.
- Rate limiting: slowapi middleware, 100 req/min per API key.
- Testing: pytest + pytest-asyncio. Fixtures for mock orders.

### Pipeline architecture

Every pipeline inherits from `BasePipeline` (`apps/api/src/pipelines/base.py`):

```python
class BasePipeline(ABC):
    name: str       # e.g. "automation", "content"
    version: int    # increments on changes

    @abstractmethod
    async def execute(self, order: Order) -> PipelineResult: ...

    async def run(self, order: Order) -> Delivery:
        # log start → execute → qa_check → deliver or retry/escalate
        # QA retries up to 3 attempts before escalating to human
        # Every step logged to execution_logs table

    async def qa_check(self, order, result) -> QAResult:
        # check_output_exists + check_output_format + check_llm_self_review
```

Commit format: `task(p1.03): build landing hero section`
Commit types: `feat`, `fix`, `task`, `qa`, `refactor`, `docs`

---

## INTERFACE POINTS

| # | From → To | Trigger | Contract |
|---|-----------|---------|----------|
| 1 | P → A | Stripe webhook → `orders` row with `payment_status: paid` | `orders` table |
| 2 | A → P | `POST /api/quote` → `{ complexity_score, confidence, suggested_price_cents, suggested_pipeline }` | `QuoteRequest/Response` in api-schemas.ts |
| 3 | A → P | Pipeline complete → `deliveries` row + `orders.execution_status: delivered` → P sends email | `deliveries` table |
| 4 | P → C | Feedback form → `feedback` insert → C runs sentiment analysis, enriches tags | `feedback` table |
| 5 | C → A | Brain detects pattern/bad feedback → `pipeline_improvements` row → posts to `#forge-engine` | `pipeline_improvements` table |
| 6 | C → P | Weekly digest → customer segments, churn risks → Slack `#forge-feedback` → P adjusts marketing | Slack message |

---

## ENVIRONMENT VARIABLES

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=      # Backend only. Never expose to frontend.
SUPABASE_DB_URL=                # Direct DB for migrations

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=

# AI Providers
ANTHROPIC_API_KEY=              # Claude (primary)
OPENAI_API_KEY=                 # GPT-4o (fallback)
OLLAMA_BASE_URL=http://localhost:11434

# Email
RESEND_API_KEY=
FROM_EMAIL=

# Monitoring
NEXT_PUBLIC_SENTRY_DSN=
SENTRY_AUTH_TOKEN=
NEXT_PUBLIC_POSTHOG_KEY=
NEXT_PUBLIC_POSTHOG_HOST=https://eu.i.posthog.com

# Slack (separate webhook per channel)
SLACK_WEBHOOK_ORDERS=
SLACK_WEBHOOK_ALERTS=
SLACK_WEBHOOK_WINS=
SLACK_WEBHOOK_DEPLOYS=

# Redis (Upstash)
UPSTASH_REDIS_URL=
UPSTASH_REDIS_TOKEN=

# Cloudflare R2
R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=forge-assets

# App
NEXT_PUBLIC_APP_URL=
API_BASE_URL=
API_KEY=                        # Frontend → Backend auth
```

---

## DEPLOYMENT

| Service | Connected to | Trigger |
|---------|-------------|---------|
| Vercel | `apps/web/` | Auto-deploy on merge to `main`; preview on PR to `dev` |
| Railway | `apps/api/` (Dockerfile) | Auto-deploy on merge to `main` |
| Supabase | EU (Frankfurt, GDPR) | `supabase db push` for migrations |

---

## ACCESS / CREDENTIALS

| Account | Owner | Others |
|---------|-------|--------|
| GitHub org | A | All (admin) |
| Supabase | C | All (admin) |
| Vercel | P | A+C (viewer) |
| Railway | A | P+C (viewer) |
| Stripe | P | All (team member) |
| Sentry / PostHog | C | All (team member) |
| Upstash Redis | A | C (viewer) |
| Cloudflare / Resend | P | — |
| Slack / Linear | C | All |

Credential sharing: use 1Password or Bitwarden team vault. Never share secrets over Slack/WhatsApp.

---

## SLACK CHANNELS

| Channel | Purpose |
|---------|---------|
| `#forge-general` | Decisions, blockers, standups |
| `#forge-alerts` | Cross-workstream bugs — report here, do NOT fix another owner's code |
| `#forge-engine` | Pipeline code, AI output issues, improvement suggestions |
| `#forge-feedback` | Customer feedback, C's weekly intelligence digest |
| `#forge-orders` | Bot: new paid order notifications |
| `#forge-wins` | Bot: deliveries, 5-star ratings, milestones |
| `#forge-deploys` | Bot: GitHub deploy notifications |

---

## GIT WORKFLOW

```
main       ← Production. Auto-deploys. PRs only. Must pass CI.
dev        ← Integration. All feature branches merge here first.
p/feat-*   ← Prateek's branches
a/feat-*   ← Achyut's branches
c/feat-*   ← CK's branches
```

PR to `dev` requires 1 review. `dev → main` requires all three approvals. Every PR must pass CI (lint + typecheck + tests).

---

## ESCALATION RULES

1. **Code doesn't compile:** Fix it. Never create DONE without passing QA.
2. **Dependency not done:** Skip to next task with satisfied deps. Post in `#forge-general`.
3. **Contract needs changing:** PR against `contracts/`. Post in `#forge-general`. All three approve before merge.
4. **Integration broken:** Both owners on the interface debug together using `scripts/seed-test-order.ts`.
5. **AI pipeline bad output:** Log input/output in `#forge-engine`. Iterate prompt. Track in `pipeline_improvements`.
6. **Task taking 2x estimate:** STOP. Break into smaller tasks. Create new task folders. Update DEPS.md.

---

## ANTI-HALLUCINATION RULES

1. Always read TASK.md first — do not infer what to build from the task name.
2. Always import types from `contracts/` — if the type doesn't exist there, add it to contracts first.
3. Never invent a database column — if it's not in `contracts/database.sql`, it doesn't exist.
4. Never invent an API endpoint — if it's not in the current TASK.md, don't build it.
5. Never install a dependency not listed in the tech stack without checking first.
6. Always run QA.md checks — never mark a task DONE without running them.
7. One task at a time — do not "quickly also add" something from a future task.
8. If you find a bug in another workstream's code: don't fix it — file it in `#forge-alerts`.
9. Commit after every task that passes QA — not after every file, not after three tasks.
10. The `contracts/` folder is always right — if there's a conflict, fix the code, not the contract.
