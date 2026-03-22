# QA: p1.01 — Scaffold Next.js App

## Automated checks (ALL must pass)

### 1. Build
- [ ] `cd apps/web && npm run build` exits 0
- [ ] `cd apps/web && npm run typecheck` exits 0
- [ ] `cd apps/web && npm run lint` exits 0

### 2. Dev server
- [ ] `npm run dev` starts, localhost:3000 loads without console errors

### 3. Supabase connection
- [ ] Server client imports from `@supabase/ssr` correctly
- [ ] `Database` type is imported from `contracts/types.ts` — not redefined locally

### 4. Middleware
- [ ] Visiting `/dashboard` without auth redirects to `/`
- [ ] Visiting `/` loads normally

### 5. shadcn/ui
- [ ] `components.json` exists
- [ ] At least one shadcn component (Button) renders without error on the placeholder page

## Manual checks

- [ ] localhost:3000 loads, no errors in browser console
- [ ] Tailwind styles are applying (check by inspecting placeholder page)
- [ ] No hardcoded Supabase keys — reads from env vars
