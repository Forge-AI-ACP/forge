# Task 0.02 — Supabase Schema

**Owner:** CK
**Estimated time:** 2-3 hours
**Goal:** Apply the full database schema to Supabase, set up RLS, and generate TypeScript types into `contracts/`.

---

## What to build

### 1. Apply the schema

The full SQL schema is in `context.md` lines 143–481. Copy it into `supabase/migrations/20260322000000_init.sql` and apply it.

Tables to create:
- `orders`
- `deliveries`
- `feedback`
- `pipeline_improvements`
- `execution_logs`
- `pipeline_metrics`
- `task_patterns`
- `pipeline_versions`
- `referrals`

Enums to create:
- `task_type` — automation, content, agent, integration, data, other
- `payment_status` — pending, paid, refunded, failed
- `execution_status` — queued, classifying, running, qa_review, delivered, failed, revision, cancelled
- `deliverable_type` — file, url, code_repo, api_endpoint, report, other
- `improvement_type` — prompt_update, flow_change, new_step, parameter_tune

Extensions needed:
- `uuid-ossp`
- `vector` (for future embeddings on task_patterns)

### 2. RLS policies

Apply as defined in `context.md`:
- `orders`, `deliveries`, `feedback`: customers see only their own rows
- All other tables: service role only

### 3. Indexes

Apply all indexes from `context.md` (idx_orders_customer, idx_orders_status, etc.)

### 4. updated_at trigger

Apply the `update_updated_at()` trigger function on `orders` and `task_patterns`.

### 5. Generate TypeScript types

After applying schema to local Supabase:

```bash
supabase gen types typescript --local > contracts/types.ts
```

Also manually create `contracts/enums.ts`:

```typescript
export type TaskType = 'automation' | 'content' | 'agent' | 'integration' | 'data' | 'other'
export type PaymentStatus = 'pending' | 'paid' | 'refunded' | 'failed'
export type ExecutionStatus = 'queued' | 'classifying' | 'running' | 'qa_review' | 'delivered' | 'failed' | 'revision' | 'cancelled'
export type DeliverableType = 'file' | 'url' | 'code_repo' | 'api_endpoint' | 'report' | 'other'
export type ImprovementType = 'prompt_update' | 'flow_change' | 'new_step' | 'parameter_tune'
```

And copy the SQL into `contracts/database.sql` as the source of truth.

### 6. Seed data

Create `supabase/seed.sql` with 2-3 test orders in various states so local dev has data to work with.

---

## Files to create/edit

- `supabase/migrations/20260322000000_init.sql`
- `supabase/seed.sql`
- `contracts/database.sql` (copy of migration SQL)
- `contracts/types.ts` (generated)
- `contracts/enums.ts` (manual)

---

## Commands

```bash
supabase start                  # Start local Supabase
supabase db reset               # Apply migrations + seed
supabase gen types typescript --local > contracts/types.ts
```
