# QA: p1.03 — Intake Form + Quote

## Automated checks (ALL must pass)

### 1. Build + types
- [ ] `npm run build` exits 0
- [ ] `npm run typecheck` exits 0
- [ ] `TaskType` imported from `contracts/enums.ts` — not redefined

## Manual checks

- [ ] `/order` page loads without errors
- [ ] Form validation works — submitting empty form shows errors
- [ ] `task_description` under 50 chars shows "Please describe your task in more detail"
- [ ] Valid form submission writes a row to `orders` table in Supabase (check Studio)
- [ ] `payment_status` is `pending` on the new row
- [ ] Quote API call fires (check network tab) — if it fails, fallback price €49 shows
- [ ] File upload works — files appear in Supabase Storage `order-attachments` bucket
- [ ] Form is mobile responsive at 375px
- [ ] No console errors
