# Cosmos Freight Ltd

Freight, customs and tracking site for a London freight forwarder. Next.js 14 (App Router), Tailwind CSS 3, no database required.

## Run it

```bash
npm install
cp .env.example .env.local     # then set SESSION_SECRET (openssl rand -hex 32)
npm run dev                    # http://localhost:3000
```

Production: `npm run build && npm start`. `SESSION_SECRET` (32+ characters) is required when `NODE_ENV=production`.

Demo login: `demo@cosmosfreight.example` / `Cosmos2026!` (seeded on first sign-in). New accounts start with an empty portal.

## Routes

| Route | Purpose |
|---|---|
| `/` | Home: live map, tracker, instant rate |
| `/track` | Tracking with milestones, route map, sensor readings |
| `/quote` | Five-step quote and booking wizard |
| `/services` | Ocean, air, road, warehousing, customs |
| `/customs` | UK duty and VAT estimator, checklist, guides |
| `/network` | Hubs, schedules, warehouse map |
| `/developers` | API reference with a live mock API and webhook signer |
| `/about` | Company, emissions calculator, offsets |
| `/support` | FAQ, claims, offices, chat |
| `/login` | Sign in, create account, reset password |
| `/dashboard` | Client portal (requires sign-in) |

## Structure

```
app/                 routes (each page.js is a thin wrapper) and /api/auth/*
components/          Header, Footer, Logo, LoginForm, AuthProvider, PageMount
lib/core.js          helpers, data, pricing engine, dot-matrix WorldMap
lib/db.js            demo account data and per-user portal state
lib/ui.js            shared UI pieces (modal, cards, downloads)
lib/pages/*.js       one module per page
lib/server/          session cookie, user store, rate limiter (server only)
middleware.js        protects /dashboard, bounces signed-in users off /login
public/brand/        logo SVG/PNG files; app/icon.svg is the favicon
```

## Authentication

- `POST /api/auth/login | signup | logout`, `GET /api/auth/me`.
- Passwords are hashed with scrypt and a per-user salt. The session is an HMAC-signed, HttpOnly, SameSite=Lax cookie (Secure on HTTPS), 12 hours or 30 days with "keep me signed in".
- Five failed sign-ins lock that email and IP for 30 seconds. Failure messages are generic.
- `middleware.js` checks the cookie before `/dashboard` is served.

## Things to know before going live

- **User store.** `lib/server/users.js` writes `data/users.json`. That is fine locally; serverless hosts have read-only or temporary disks, so replace it with a real database (the file exposes four functions: `findUser`, `createUser`, `verifyLogin`, `publicUser`).
- **Rate limiter** is in memory, per instance. Use a shared store when running several instances.
- **Password reset** shows the message only; no email is sent. Wire it to your mail provider.
- **Portal data is sample data.** Shipments, invoices and the API are demo content held in the browser session; bookings reset on refresh. Customs rates are illustrative, so check GOV.UK before relying on them.
- **Page modules are imperative.** The ten pages were ported from a single-file prototype as plain DOM modules mounted by `components/PageMount.js`, not rewritten as React components. The shell, navigation, auth and login form are React. Pages render on the client, so the first paint shows an empty `<main>` until JavaScript runs.
- Fonts load from Google Fonts via a `<link>` in `app/layout.js`.
