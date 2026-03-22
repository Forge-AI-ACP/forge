# Task p1.02 — Landing Page

**Owner:** Prateek
**Estimated time:** 3-4 hours
**Goal:** Full landing page at `/` that converts visitors into customers. Vibe code this — make it look excellent.

---

## What to build

Single page at `apps/web/src/app/page.tsx` with these sections in order:

### 1. Nav
- Logo (text: "Forge") left
- Single CTA button right: "Get it done →" → links to `/order`
- Minimal, no menu

### 2. Hero
- Headline: something like "Describe the task. We deliver it." (vibe it — make it punchy)
- Subheadline: 1-2 lines on what Forge does
- Primary CTA button: "Start your task →" → `/order`
- Secondary: "See how it works ↓" → scrolls to how-it-works section
- Visual: clean, could be a mockup, abstract shape, or just strong typography

### 3. How it works (3 steps)
1. Describe your task
2. Pay a flat fee
3. Get it delivered by AI

### 4. What we do (task types)
Show the 6 task types as cards or pills:
- Automation, Content, AI Agents, Integrations, Data Analysis, Custom

### 5. Pricing
- Simple, flat: one price tier or "from €X"
- "No subscriptions. Pay per task."

### 6. Social proof / testimonials
- 3 placeholder testimonials (real ones come later from `feedback` table)
- Can be fake/placeholder for now — mark with a comment `{/* TODO: pull from feedback table in p1.x */}`

### 7. FAQ
- 3-5 common questions (What kinds of tasks? How fast? What if I'm not happy?)

### 8. Footer
- Logo, tagline
- Links: Privacy, Terms (placeholder hrefs for now)
- © 2026 Forge

---

## Design direction

- Dark or light — your call, make it look premium
- Tailwind + shadcn/ui components only
- Mobile responsive
- No animations required yet (keep it fast to build)

---

## Files to create/edit

- `apps/web/src/app/page.tsx`
- `apps/web/src/components/nav.tsx`
- `apps/web/src/components/footer.tsx`
- Any section components you want to extract

---

## Notes

- This is a Server Component — no `'use client'` unless needed
- The CTA button always goes to `/order` — that's the only conversion goal
- Do NOT connect to Supabase or any API in this task
