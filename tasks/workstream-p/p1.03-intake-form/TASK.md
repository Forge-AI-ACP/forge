# Task p1.03 — Intake Form + Quote

**Owner:** Prateek
**Estimated time:** 3-4 hours
**Goal:** The intake form at `/order` where customers describe their task, get a price quote from A's API, and proceed to payment.

---

## What to build

### 1. Page: `apps/web/src/app/order/page.tsx`

Multi-step form (or single long form — your call on UX):

**Step 1: Describe your task**
- `task_description` — large textarea, min 50 chars, "Tell us what you need done in detail"
- `task_type` — dropdown: Automation / Content / AI Agent / Integration / Data / Other (maps to `TaskType` enum)
- `customer_name` — text input
- `customer_email` — email input
- `customer_notes` — optional textarea "Anything else we should know?"
- File upload — up to 3 files, max 50MB each (pdf, csv, xlsx, png, jpg, zip)
  - Upload to Supabase Storage bucket `order-attachments`
  - Store URLs in `uploaded_file_urls` array

**Step 2: Review quote**
- After form fills in, call `POST /api/quote` (Achyut's API)
- Show: complexity score, estimated delivery time, price
- If quote API is not ready yet: show a placeholder price (€49) and skip the API call
- CTA: "Looks good — pay now →"

**Step 3: Redirect to Stripe**
- On confirm, write order to Supabase `orders` table with `payment_status: pending`
- Redirect to Stripe Checkout (p1.04 handles this — for now just write to DB and show a "Processing..." state)

### 2. Form validation

Use React Hook Form + Zod. Schema:

```typescript
import { z } from 'zod'

const intakeSchema = z.object({
  task_description: z.string().min(50, 'Please describe your task in more detail'),
  task_type: z.enum(['automation', 'content', 'agent', 'integration', 'data', 'other']),
  customer_name: z.string().min(2),
  customer_email: z.string().email(),
  customer_notes: z.string().optional(),
})
```

### 3. Quote API call

```typescript
// POST to Achyut's API
const quote = await fetch(`${process.env.API_BASE_URL}/api/quote`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'x-api-key': process.env.API_KEY },
  body: JSON.stringify({ task_description, task_type })
})
```

If `API_BASE_URL` is not set or call fails — fall back to showing flat price €49.

### 4. Write to Supabase

On form submit (before Stripe redirect):
```typescript
const { data, error } = await supabase.from('orders').insert({
  customer_email,
  customer_name,
  task_description,
  task_type,
  customer_notes,
  uploaded_file_urls,
  quoted_price_cents: quote.suggested_price_cents ?? 4900,
  payment_status: 'pending',
  execution_status: 'queued',
})
```

---

## Files to create/edit

- `apps/web/src/app/order/page.tsx`
- `apps/web/src/app/order/actions.ts` (server actions for Supabase writes)
- `apps/web/src/lib/quote.ts` (quote API fetch helper)

---

## Notes

- This is a `'use client'` component (it's a form)
- Import `TaskType` from `contracts/enums.ts` — do not redefine
- Stripe integration comes in p1.04 — just write to DB and show "We'll be in touch" for now
- File uploads to Supabase Storage: bucket must exist (CK creates it in 0.02)
