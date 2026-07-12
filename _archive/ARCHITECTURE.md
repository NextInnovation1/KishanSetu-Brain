# KisanSetu — Technical Architecture

## Stack (fixed by founder decision)

| Layer | Tech |
|---|---|
| Backend API | Node.js 20 + Express (REST, JSON) |
| Database | PostgreSQL 16 |
| Website | Vanilla JS static site (no build step) |
| Mobile apps | Android (Kotlin/Compose) + iOS (Swift 5/SwiftUI) — dual-role, screen-for-screen identical (**golden rule**) |
| Buyer web portal | later — reuses the same API |
| Auth | Phone OTP → JWT (dev OTP stub now; MSG91/Twilio later) |
| Payments | Razorpay/UPI payouts (stubbed now) |
| Messaging | WhatsApp Business API + FCM (later) |

## System diagram

```
 Android app (both roles)─┐
 iOS app (both roles)─────┤        ┌──────────────┐      ┌────────────┐
 Buyer/Ops web ───────┼──REST──┤ Node/Express ├──SQL─┤ PostgreSQL │
 Website lead form ───┘        │  (KisanSetu-Backend/)  │      └────────────┘
                               └──────┬───────┘
                    later: Razorpay ──┤── WhatsApp Business ── FCM
```

One backend, one database, four thin clients. No microservices until the monolith hurts.

## Domain model (migrations/001_init.sql)

- **users** (phone, role: farmer/buyer/ops, language) + **farmer_profiles** (FPO, village, land) + **buyer_profiles** (business type, GSTIN)
- **fpos** — partner Farmer Producer Organisations (the aggregation hubs)
- **produce** — catalog with EN/HI/GU names
- **price_feed** — per produce per day: mandi reference price + platform price for Grade A/B. Set daily by ops; this drives order pricing
- **listings** — farmer posts qty + harvest date → hub grades it (A/B, actual weighed qty) → allocated to orders → closed
- **orders / order_items** — buyer orders by produce+grade; totals computed server-side from price_feed
- **order_allocations** — which farmer listing filled which order item = **farm-to-fork traceability**, the product differentiator
- **payments** — farmer payouts (instant on pickup, UPI) and buyer invoices
- **leads** — website form intake

## Lifecycles

Listing: `posted → graded (A/B) → allocated → closed`
Order: `placed → confirmed → out_for_delivery → delivered` (or `cancelled`)

## Key flows

1. **Evening before harvest:** farmer posts listing (app or via FPO staff).
2. **Ops sets tomorrow's prices** from mandi reference (SOP in T6.6).
3. **Buyer orders by grade** for next-day delivery; server prices from price_feed.
4. **Morning at hub:** intake, weigh, grade, tag crate (QR w/ listing id + harvest date), allocate listings → order items.
5. **Dispatch via 3PL cold logistics**; status updates through ops dashboard.
6. **On pickup, farmer payout fires** (stub now → Razorpay later). Buyer gets one invoice.

## Conventions

- REST, snake_case JSON, parameterized SQL only, consistent errors `{error: string}`.
- JWT bearer; roles enforced in middleware (`requireRole`).
- Migrations are plain SQL, run in order, tracked in `schema_migrations`.
- Dev OTP is `123456` (env `DEV_OTP`); never ship to prod.
- Local dev: Postgres in Docker on **port 5433**; API on **:4000**; Android emulator reaches it at `10.0.2.2:4000`.

## Deliberate non-choices (for now)

- No microservices, no Kubernetes, no GraphQL, no ORM, no Redis — a pilot with 25 buyers does not need them.
- No owned logistics software (route planning etc.) — 3PL handles routes; we track statuses.
- Deploy only when the pilot demands it: single VM or managed Node host + managed Postgres + daily backups; static hosting for website.

## Freshness SLA instrumentation (T2.12 — the moat metric)

Every allocation carries `harvest_ts → hub_in_ts → dispatch_ts → delivered_ts`. Median harvest→door hours is reported weekly and eventually published live on the website.
