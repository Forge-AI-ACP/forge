# QA: 0.02 — Supabase Schema

## Automated checks (ALL must pass)

### 1. Local Supabase runs
- [ ] `supabase start` exits 0
- [ ] `supabase db reset` exits 0 (migrations + seed apply cleanly)

### 2. All tables exist
Run in Supabase Studio (localhost:54323) or via psql:
- [ ] `orders` table exists with all columns
- [ ] `deliveries` table exists with all columns
- [ ] `feedback` table exists with all columns
- [ ] `pipeline_improvements` table exists
- [ ] `execution_logs` table exists
- [ ] `pipeline_metrics` table exists
- [ ] `task_patterns` table exists (with `embedding vector(1536)` column)
- [ ] `pipeline_versions` table exists
- [ ] `referrals` table exists

### 3. Enums exist
- [ ] `task_type`, `payment_status`, `execution_status`, `deliverable_type`, `improvement_type` all exist

### 4. RLS enabled
- [ ] RLS is enabled on all 9 tables
- [ ] Policies exist on `orders`, `deliveries`, `feedback`

### 5. Contracts generated
- [ ] `contracts/types.ts` exists and is non-empty
- [ ] `contracts/enums.ts` exists with all 5 enum types
- [ ] `contracts/database.sql` exists

### 6. Seed data
- [ ] `supabase db reset` populates at least 2 rows in `orders`

## Manual checks

- [ ] Open Supabase Studio → Table Editor → verify `orders` columns match `context.md` exactly
- [ ] No columns are missing or misnamed
