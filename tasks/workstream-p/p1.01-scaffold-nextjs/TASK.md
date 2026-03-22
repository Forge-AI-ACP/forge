# Task p1.01 — Scaffold Next.js App

**Owner:** Prateek
**Estimated time:** 2-3 hours
**Goal:** Working Next.js 14 app with Tailwind, shadcn/ui, Supabase client, and auth middleware. No UI content yet — just the foundation everything else builds on.

---

## What to build

### 1. Next.js 14 App Router setup

Inside `apps/web/`:
- TypeScript strict mode
- App Router (not Pages Router)
- Tailwind CSS configured
- shadcn/ui initialised (`npx shadcn-ui@latest init`)
  - Style: Default
  - Base color: Neutral
  - CSS variables: yes

### 2. Supabase clients

`apps/web/src/lib/supabase/server.ts` — server component client:
```typescript
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'
// standard Supabase SSR server client setup
```

`apps/web/src/lib/supabase/client.ts` — browser client:
```typescript
import { createBrowserClient } from '@supabase/ssr'
// standard browser client
```

Import types from `contracts/types.ts` — do not redefine Database type locally.

### 3. Auth middleware

`apps/web/src/middleware.ts`:
- Refresh session on every request
- Protect `/dashboard` and `/admin` routes — redirect to `/` if not authenticated
- Public routes: `/`, `/order`, `/order/*`, `/reviews`, `/products/*`

### 4. Environment variables

`apps/web/.env.local` (gitignored, pulled from Doppler):
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
```

### 5. Basic layout

`apps/web/src/app/layout.tsx`:
- Sets up fonts (Inter or Geist)
- Wraps app in any providers needed
- Basic metadata (title: "Forge — AI Service Factory")

`apps/web/src/app/page.tsx`:
- Placeholder: `<main><h1>Forge</h1></main>` — real content comes in p1.02

### 6. shadcn/ui components to install now

```bash
npx shadcn-ui@latest add button input label textarea card badge
```

---

## Files to create/edit

- `apps/web/src/app/layout.tsx`
- `apps/web/src/app/page.tsx` (placeholder)
- `apps/web/src/app/globals.css`
- `apps/web/src/lib/supabase/server.ts`
- `apps/web/src/lib/supabase/client.ts`
- `apps/web/src/middleware.ts`
- `apps/web/next.config.js`
- `apps/web/tailwind.config.ts`
- `apps/web/tsconfig.json`
- `apps/web/components.json` (shadcn config)

---

## Notes

- Do NOT build any real UI in this task — placeholder only
- Do NOT connect to any API endpoints yet
- Import `Database` type from `contracts/types.ts` not locally defined
