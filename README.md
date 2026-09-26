# LL Test Portal

A learner-licence practice portal built with Next.js, TypeScript, Tailwind CSS, and Supabase. The included questions are development examples only and are not an official government question bank.

## Requirements

- Node.js 20.9 or newer
- npm
- A Supabase project (optional for the public landing page; required for accounts and persisted data)

## Local development

```bash
npm install
Copy-Item .env.example .env.local
npm run dev
```

Open <http://localhost:3000>. Add the values from your Supabase project to `.env.local` to enable authentication and database-backed features. The site shows a setup state rather than pretending a disabled payment provider has accepted money.

## Environment variables

- `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` connect the app to Supabase. The anon key is protected by database row-level security.
- `SUPABASE_SERVICE_ROLE_KEY` is server-only and is used to resolve mobile sign-ins to the account email. Never add a `NEXT_PUBLIC_` prefix to this key.
- `NEXT_PUBLIC_APP_URL` is the origin used to build authentication callback links.
- `NEXT_PUBLIC_MERCHANT_QR_IMAGE_URL` is the public HTTPS URL or root-relative path of your actual merchant QR image. It is blank until you configure it.
- `NEXT_PUBLIC_MERCHANT_WHATSAPP_NUMBER` is the WhatsApp number in international digits; the example is India country code `91` plus the number provided.
- `PAYMENT_PROVIDER` stays `disabled` until a provider integration is implemented. The Razorpay variables are reserved for that future integration; entering credentials alone does not enable payments.

## Supabase setup

1. Create a Supabase project and copy its project URL and anon key into `.env.local`.
2. In the Supabase SQL editor, run `supabase/migrations/0001_initial_schema.sql`, then `supabase/migrations/0002_manual_qr_wallet_topups.sql`.
3. Run `supabase/seed.sql` to add clearly labelled development questions and default settings.
4. In Authentication settings, enable email/password sign-in, set the site URL, and allow callback URLs `http://localhost:3000/auth/callback` and `https://<your-domain>/auth/callback`.
5. Create the first administrator by promoting its authenticated user UUID in the SQL editor: `update public.profiles set role = 'admin' where id = '<auth-user-uuid>';` Keep the service-role key server-side only.

## Payments

Wallet top-ups use manual merchant-QR review. Configure the real QR image and WhatsApp number, then users can submit the amount and UPI transaction reference. They send the screenshot to the configured WhatsApp number; an admin must verify both the reference and the merchant account before approving. Approval credits the wallet once and creates a ledger transaction; rejection creates no credit. The screenshot is shared through WhatsApp, not uploaded to this app.

The merchant QR is stored at `public/merchant-qr.jpg` and the example URL is `/merchant-qr.jpg`. For Vercel, make sure the actual QR asset is included in the GitHub repository. Do not replace it with a placeholder.

Razorpay is a separate future integration. Keep `PAYMENT_PROVIDER=disabled` until server-side order creation and signature-verified webhook processing are implemented. Never credit a wallet from a browser callback.

The seeded development practice test is free so the test engine can be exercised without adding funds. Configure a price in Admin > Test settings only after you are ready to collect manual QR payments and verify each request.

## Deployment

### GitHub

```bash
git init
git add .
git commit -m "Build LL Test Portal"
git branch -M main
git remote add origin https://github.com/<account>/<repository>.git
git push -u origin main
```

### Vercel

Import the repository in Vercel, select the Next.js framework preset, and add the variables from `.env.example` in Project Settings. Set `NEXT_PUBLIC_APP_URL` to the production URL and add that URL to Supabase Authentication's allowed redirect URLs. Deploy only after the Supabase migration has been applied. Do not expose `SUPABASE_SERVICE_ROLE_KEY` or payment secrets as `NEXT_PUBLIC_` variables.

## Scripts

- `npm run dev` starts the development server.
- `npm run typecheck` checks TypeScript.
- `npm run lint` runs ESLint.
- `npm run build` creates the production build.