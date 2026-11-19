# RAASTA – streetwear store

Next.js 14 (App Router) · TypeScript · PostgreSQL · Prisma. Session cookies, bcrypt passwords, Zod validation, server-side pricing.

## Run it

1. `docker run -d --name raasta-db -e POSTGRES_USER=raasta -e POSTGRES_PASSWORD=raasta -e POSTGRES_DB=raasta -p 5432:5432 postgres:16` (or any PostgreSQL)
2. `cp .env.example .env`, then set `ADMIN_EMAIL` and a strong `ADMIN_PASSWORD` (leave `S3_*` empty locally)
3. `npm install && npx prisma db push && npm run db:seed && npm run dev`
4. Open http://localhost:3000. The admin signs in at `/login` and lands on `/admin`.

Without `SMTP_HOST`, verification and reset links are printed in the server console.

## How the pieces work

- **Accounts:** register, sign in, sign out, saved address, order history, email verification, password reset (1-hour link, signs out every device).
- **Admin login:** the same `/login` page. Only users with role ADMIN and a verified email reach `/admin` or `/api/admin/*`. Every admin page and API route checks this itself.
- **Admin tools:** overview with revenue chart, best sellers and low stock; create, edit, hide and delete products; upload photos; edit variants and stock; orders with search, filter and status changes (cancelling or refunding restocks); settings for announcement, delivery fees and discount codes.
- **Server-side validation:** `lib/pricing.ts` is the only place totals are calculated. The browser sends variant ids and quantities, never prices. Stock is decremented inside a transaction with a conditional update, so two buyers cannot take the last unit.
- **Payments:** only cash on delivery is live. bKash, Nagad and SSLCommerz are shown as disabled. Adding one means creating a payment session in `app/api/orders/route.ts` and confirming it with the gateway's webhook before marking the order paid. That needs your merchant credentials.

## Deploy on Netlify

1. Create a free PostgreSQL database on Neon or Supabase. Copy the pooled address into `DATABASE_URL` and the direct address into `DIRECT_URL`.
2. Create a Cloudflare R2 bucket for photos. Turn on its public address, create an API token, and note the endpoint, bucket name, keys and public URL.
3. On your own computer, put the same values in `.env`, then run `npm install`, `npx prisma db push` and `npm run db:seed`. This creates the tables and the admin account. Netlify does not run these for you.
4. Upload the project to a GitHub repository.
5. In Netlify choose Add new site, Import from Git, pick the repository, and add every variable from `.env.example` under Site configuration, Environment variables. Set `APP_URL` to your Netlify address.
6. Deploy. Sign in at `/login` with the admin email and password.

Netlify functions have time limits (10 seconds on the free plan). Pick the Netlify functions region closest to your database.

## Before going live

- Use HTTPS (Netlify provides it) and set `APP_URL` to your real address.
- Set SMTP credentials so emails send.
- Rate limits are stored in the database, so they work across serverless instances.
- Change the seeded admin password and create a second admin account as a backup.
- Run `npm audit` and keep Next.js up to date.
