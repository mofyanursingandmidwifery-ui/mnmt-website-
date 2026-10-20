# MNMT website — Mofya Nursing and Midwifery Training

Public website, student accounts, student dashboard and admin dashboard for MNMT.
Built with **Next.js 16** (App Router) and **Supabase** (database, authentication, private file storage).

## What lives where

| Area | Location |
|---|---|
| Homepage (design ported from the original Claude Design canvas) | `src/components/HomePage.tsx`, `src/app/globals.css` |
| Sign in / password reset / email-link handling | `src/app/login`, `src/app/forgot-password`, `src/app/reset-password`, `src/app/auth/callback` |
| Student dashboard & order payment page | `src/app/dashboard` |
| Admin dashboard | `src/app/admin` |
| Business settings (WhatsApp, contact, payment instructions) | `src/lib/site.ts` |
| Session refresh + route protection | `src/proxy.ts` |
| Database schema, security rules, storage, seed data | `supabase/migrations/*.sql` (applied in filename order) |
| Automated tests | `scripts/security_test.py`, `scripts/e2e_test.mjs`, `scripts/admin_e2e_test.mjs` |

## Security model (summary)

- Every table has **row-level security**. Students can only read their own profile, registrations, bookings, orders, payments and questions.
- **Admins** are the accounts listed in `public.admins`. Only admins can approve, confirm, verify, publish or edit content.
- **Prices and order totals** are calculated in the database (`place_order`), never taken from the browser.
- **Payments**: students submit a manual payment claim (`submit_manual_payment`) that stays `pending`; only an admin can verify it (`verify_payment`). Nothing in the website marks a payment as paid automatically.
- **Study files** live in the private `study-materials` bucket. A student can download a file only when they have a paid order containing it; links expire after 5 minutes.
- **Payment proofs** live in the private `payment-proofs` bucket (owner + admins only).
- The browser only ever receives the Supabase URL and the **publishable** key. No secret key is used anywhere in this code.

## Environment variables

See `.env.example`. Required: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`, `NEXT_PUBLIC_SITE_URL`.

## Running locally

```bash
npm install
cp .env.example .env.local   # then fill in the values
npm run dev                  # http://localhost:3000
```

## Database migrations

The migrations in `supabase/migrations/` have already been applied to the MNMT Supabase project.
To set up a new Supabase project (e.g. a test copy), run them in filename order in the Supabase SQL editor,
or with the Supabase CLI: `supabase link --project-ref <ref>` then `supabase db push`.

## Deploying (Vercel)

1. Import this GitHub repository in Vercel (framework: Next.js; default build settings).
2. Add the three environment variables above (Production and Preview). Set `NEXT_PUBLIC_SITE_URL` to the live address.
3. Deploy. Every push to `main` redeploys automatically.
4. In Supabase → Authentication → URL Configuration: set **Site URL** to the live address and add
   `https://<your-domain>/auth/callback` to **Redirect URLs** (otherwise email links will not work).

## Payment providers (future)

The `payments` table already has `provider`, `provider_reference` and methods for `airtel_money`, `mtn_momo`, `zamtel_kwacha` and `card`.
An automated provider must be added **server-side only** (a Supabase Edge Function or a Next.js route handler that verifies the
provider's webhook signature with a secret stored in server environment variables), which then records the payment as verified.
Never call a payment provider's secret API from browser code.
