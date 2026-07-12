# 06 — PRD: Backend API & Database

**Doc owner:** Founder (Alpesh) · **Status:** Draft for founder approval · **Last updated:** 2026-07-12
**Siblings:** `00-GOLDEN-RULES.md` (rules this doc must obey) · `05-PRODUCT-OVERVIEW.md` (what the clients do) · `07-PRD-MOBILE-APPS.md`, `08-PRD-WEBSITE.md`, `09-PRD-OPS-DASHBOARD.md` (the four consumers of this API) · `11-ARCHITECTURE.md` (system-level rationale) · `14-OPS-PLAYBOOK.md` (the human procedures these endpoints digitize).

> Golden rule #3: PLAN BEFORE CODE. This PRD supersedes the scaffold at `~/Desktop/KisanSetu-Backend` (including `migrations/001_init.sql`, which uses `mandi_price_per_kg` and has no region layer — both are corrected here). When development resumes, the scaffold is reviewed against THIS document, not the other way around.

---

## 1. Purpose

One backend serves everything: farmer app, buyer app (Android + iOS), marketing website lead form, and the internal ops dashboard. It is the single source of truth for prices, listings, orders, traceability, and money. It must directly serve the two metrics (Golden rule #5):

1. **Freshness** — every allocation carries the timestamp chain `harvest_ts → hub_in_ts → dispatch_ts → delivered_ts`, so median harvest→door hours is a SQL query, not an estimate.
2. **Farmer share** — the daily price feed stores both what the buyer pays (`platform_price_a/b`) and what the farmer receives (`farmer_price_a/b`), so farmer-share % is computed from records, not claimed.

Global-first, Surat-first (Golden rule #2): every table that touches money, language, time, or geography is region-scoped from day 1. Launching country #2 must be a data insert, not a migration (see `11-ARCHITECTURE.md` §7).

## 2. Users & jobs-to-be-done

| Consumer | Role claim in JWT | Jobs |
|---|---|---|
| Farmer app | `farmer` | See today's prices, post a listing, track grading/allocation status, see payout history |
| Buyer app | `buyer` | Browse graded catalog with availability, place order, track status, view traceability + invoice |
| Ops dashboard | `ops` | Set daily prices, grade listings, allocate listings→order items, update delivery status, trigger payouts, work leads, read SLA/farmer-share reports |
| Website | none (public) | `POST /leads`, read nothing |
| Founder/CA | `ops` | Export reports for the weekly metrics ritual (`14-OPS-PLAYBOOK.md`) |

## 3. Scope (in) / Non-goals (out)

### In (MVP)
Auth (phone OTP → JWT), users + role profiles (buyers start `pending`, ops-activated), region/FPO/hub hierarchy, region settings + client bootstrap config (`GET /config`), produce catalog with pluggable translations, daily price feed, listings + grading (with one evidence photo via a storage interface) + splits, orders + server-side pricing + cart quote (`POST /orders/quote`) + two-phase allocation (evening supply-check confirm / morning binding allocation) + traceability, buyer credit enforcement, buyer quality disputes + credit notes, payments (farmer payout + buyer invoice; manual UTR entry at MVP, Razorpay UPI behind a provider interface), leads, ops report endpoints (incl. receivables aging), SLA timestamp instrumentation, batch app instrumentation (`POST /events`), rate limiting, audit events, seed + migration tooling.

### NON-GOALS (explicit, with reasons)

| We will NOT build | Why not (at MVP) |
|---|---|
| **GraphQL** | Four first-party clients we control, all needing the same ~8 aggregates. REST with purpose-built endpoints is simpler to cache, rate-limit, log, and debug. GraphQL adds a resolver layer, N+1 discipline, and tooling for zero product benefit at 25 buyers. |
| **Microservices / Kubernetes** | One pilot city, one dev (founder). A modular monolith deploys in minutes and keeps every transaction in one Postgres. Split only when a module has independent scaling or team boundaries (`11-ARCHITECTURE.md` §8). |
| **ORM (Prisma/Sequelize/TypeORM)** | Schema is ~15 tables; the hard queries (allocation matching, SLA medians, farmer share) are exactly where ORMs get in the way. Plain SQL via `pg` with parameterized queries; migrations are numbered `.sql` files. |
| **Redis / any second datastore** | Sessions are stateless JWTs; rate limiting is in-process (single instance); queues don't exist yet. Postgres does caching-adjacent jobs (e.g. `otp_codes` table) fine at this scale. Add Redis only when we run >1 API instance AND need shared rate-limit/queue state. |
| **Event bus / message queue** | Order volume at pilot is tens/day. Side effects (WhatsApp, push) run inline with retry, or from a 1-minute cron sweep. |
| **Search engine (Elastic etc.)** | Catalog is <100 produce items; `ILIKE` is enough. |
| **File/object storage service** | Invoice PDFs are Phase 2 (post-pilot); the S3-compatible bucket is Phase 2. When added: bucket behind a `storage.js` interface; never DB blobs. **Exception (revised):** ONE grading-evidence photo path ships at MVP — the same `storage.js` interface with a **local-disk** instance at pilot (see §6.6 L6), because that photo is the evidence base for disputes (§6.12). No hosted object storage until Phase 2; only the local-disk driver. |
| **Websockets / server push** | Ops dashboard polls every 30 s; apps poll on foreground + FCM/APNs nudges later. |
| **Multi-currency conversion** | Currency is a *field* everywhere (global-first), but each region operates in exactly one currency. No FX, ever, until a region demands it. |
| **Self-service admin/permissions UI** | Roles are fixed (`farmer`,`buyer`,`ops`); ops users are inserted by migration/seed at pilot. |

## 4. Global-first conventions (binding)

- **Phones**: stored E.164 (`+919876543210`); validated `^\+[1-9]\d{6,14}$`. Rejected otherwise — no local formats in the DB, ever.
- **Timestamps**: `timestamptz`, stored UTC. "Today" for prices/delivery is computed in the **region's IANA timezone** (`regions.timezone`, e.g. `Asia/Kolkata`).
- **Money**: `numeric(12,2)` + `currency char(3)` (ISO 4217) on every money-bearing table. INR at launch; the column is what makes country #2 an insert.
- **Naming**: `reference_market_price`, never `mandi_price`. "Mandi" may appear only in UI copy for India (see `10-DESIGN-SYSTEM.md` voice rules).
- **Geography**: `regions → hubs → farms(farmer_profiles)` hierarchy. Every price, order, and listing resolves to a region.
- **Tax**: `tax_scheme` per region; computation is a per-region module (`tax/in_gst.js` first instance). No GST columns in generic tables — tax lines live on the order.
- **i18n**: produce names in a `produce_names` translation table keyed by BCP-47 lang code — adding a language is inserts, not `ALTER TABLE`. App UI strings live in the clients (`07-PRD-MOBILE-APPS.md`), not the API.
- **API style**: REST, JSON, `snake_case`, plural nouns, `Authorization: Bearer <jwt>`. Dev API on `:4000`, dev Postgres in Docker on `:5433`, dev OTP `123456` (env `DEV_OTP`, hard-disabled when `NODE_ENV=production`).

## 5. Database schema (DDL — authoritative)

Numbered migrations (`migrations/00X_*.sql`), tracked in `schema_migrations`. `pgcrypto` for `gen_random_uuid()`. All PKs `uuid` (supersedes the scaffold's `serial` produce id).

```sql
-- ============ geography & partners ============
CREATE TABLE regions (
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  code          text UNIQUE NOT NULL,          -- 'IN-GJ-SURAT'
  name          text NOT NULL,                 -- 'Surat, Gujarat'
  country_code  char(2) NOT NULL,              -- ISO 3166-1: 'IN'
  currency      char(3) NOT NULL,              -- ISO 4217: 'INR'
  timezone      text NOT NULL,                 -- IANA: 'Asia/Kolkata'
  default_lang  text NOT NULL DEFAULT 'en',    -- BCP-47
  languages     text[] NOT NULL DEFAULT '{en}',-- offered in UI: {en,hi,gu}
  tax_scheme    text NOT NULL DEFAULT 'none',  -- 'in_gst' | 'none' | future schemes
  reference_market_label text NOT NULL DEFAULT 'market', -- UI word: 'mandi' for India
  status        text NOT NULL DEFAULT 'active' CHECK (status IN ('active','paused')),
  created_at    timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE region_settings (                  -- one row per region; source of truth for GET /config (§6.11)
  region_id                     uuid PRIMARY KEY REFERENCES regions(id),
  currency                      char(3) NOT NULL,                        -- mirrors regions.currency for client convenience
  delivery_fee                  numeric(12,2) NOT NULL DEFAULT 50.00,    -- flat per order; ₹50 at pilot
  delivery_fee_waiver_threshold numeric(12,2) NOT NULL DEFAULT 4000.00,  -- subtotal ≥ this ⇒ delivery_fee waived
  min_order_value               numeric(12,2) NOT NULL DEFAULT 1000.00,  -- reject orders whose subtotal is below this
  order_cutoff_local            time NOT NULL DEFAULT '18:00',           -- D-1 order cutoff, interpreted in region tz
  delivery_window               text NOT NULL DEFAULT '06:00-10:00',     -- D0 delivery window, region tz
  publish_public_stats          boolean NOT NULL DEFAULT false,          -- gates R4 /reports/sla/public + website counter
  min_supported_version_android text NOT NULL DEFAULT '1.0.0',           -- force-update floor (semver)
  min_supported_version_ios     text NOT NULL DEFAULT '1.0.0',
  support_phone                 text,                                    -- shown in app help/contact
  whatsapp_number               text,                                    -- shown in app help/contact
  updated_at                    timestamptz NOT NULL DEFAULT now(),
  updated_by                    uuid REFERENCES users(id)
);

CREATE TABLE fpos (                             -- partner producer organisations
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  region_id     uuid NOT NULL REFERENCES regions(id),
  name          text NOT NULL,
  district      text,
  contact_name  text,
  contact_phone text CHECK (contact_phone ~ '^\+[1-9][0-9]{6,14}$'),
  mou_signed_on date,
  status        text NOT NULL DEFAULT 'active' CHECK (status IN ('prospect','active','paused')),
  created_at    timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE hubs (                             -- physical collection/grading points
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  region_id     uuid NOT NULL REFERENCES regions(id),
  fpo_id        uuid NOT NULL REFERENCES fpos(id),
  name          text NOT NULL,                 -- 'Kamrej Hub'
  address       text,
  geo_lat       numeric(9,6),
  geo_lng       numeric(9,6),
  status        text NOT NULL DEFAULT 'active' CHECK (status IN ('active','paused')),
  created_at    timestamptz NOT NULL DEFAULT now()
);

-- ============ people ============
CREATE TABLE users (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  phone       text UNIQUE NOT NULL CHECK (phone ~ '^\+[1-9][0-9]{6,14}$'),  -- E.164
  name        text,
  role        text NOT NULL CHECK (role IN ('farmer','buyer','ops')),
  language    text NOT NULL DEFAULT 'en',      -- BCP-47; UI language
  region_id   uuid NOT NULL REFERENCES regions(id),
  status      text NOT NULL DEFAULT 'active'    -- farmers signup 'active'; buyers signup 'pending' (§6.2/§6.3), ops flips to 'active'
                CHECK (status IN ('pending','active','blocked')),
  created_at  timestamptz NOT NULL DEFAULT now(),
  last_login_at timestamptz
);

CREATE TABLE farmer_profiles (
  user_id       uuid PRIMARY KEY REFERENCES users(id),
  hub_id        uuid NOT NULL REFERENCES hubs(id),   -- default collection hub
  village       text,
  land_area     numeric(8,2),                  -- in land_unit
  land_unit     text NOT NULL DEFAULT 'acre' CHECK (land_unit IN ('acre','hectare','bigha')),
  upi_vpa       text,                          -- payout destination (India instance)
  payout_method text NOT NULL DEFAULT 'upi' CHECK (payout_method IN ('upi','bank','cash_recorded')),
  created_at    timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE buyer_profiles (
  user_id        uuid PRIMARY KEY REFERENCES users(id),
  business_name  text NOT NULL,
  business_type  text NOT NULL CHECK (business_type IN
                   ('restaurant','hotel','caterer','cloud_kitchen','retail','processor','other')),
  tax_id         text,                         -- GSTIN in India; generic column name
  address        text NOT NULL,                -- default delivery address
  geo_lat        numeric(9,6),
  geo_lng        numeric(9,6),
  credit_terms   int NOT NULL DEFAULT 7,        -- net-N days once past the pay-on-delivery intro (buyer's first 4 orders)
  exposure_cap   numeric(12,2) NOT NULL DEFAULT 25000.00,  -- max outstanding receivable before order-placement credit_hold
  created_at     timestamptz NOT NULL DEFAULT now()
);

-- ============ catalog & prices ============
CREATE TABLE produce (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  code        text UNIQUE NOT NULL,            -- 'tomato', stable machine key
  name_en     text NOT NULL,                   -- canonical fallback
  category    text NOT NULL DEFAULT 'vegetable'
                CHECK (category IN ('vegetable','fruit','leafy','herb','other')),
  unit        text NOT NULL DEFAULT 'kg' CHECK (unit IN ('kg','bunch','piece')),
  image_key   text,                            -- client asset key for the picture grid
  active      boolean NOT NULL DEFAULT true,
  sort_order  int NOT NULL DEFAULT 100
);

CREATE TABLE produce_names (                    -- adding a language = INSERTs, no ALTER
  produce_id  uuid NOT NULL REFERENCES produce(id) ON DELETE CASCADE,
  lang        text NOT NULL,                   -- 'hi', 'gu', 'th', 'sw', ...
  name        text NOT NULL,
  PRIMARY KEY (produce_id, lang)
);

CREATE TABLE price_feed (
  id                     uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  region_id              uuid NOT NULL REFERENCES regions(id),
  produce_id             uuid NOT NULL REFERENCES produce(id),
  price_date             date NOT NULL,        -- business date in region tz
  currency               char(3) NOT NULL,
  reference_market_price numeric(12,2) NOT NULL, -- per unit; 'mandi price' in Indian UI only
  platform_price_a       numeric(12,2) NOT NULL, -- buyer pays, grade A
  platform_price_b       numeric(12,2) NOT NULL, -- buyer pays, grade B
  farmer_price_a         numeric(12,2) NOT NULL, -- farmer receives, grade A
  farmer_price_b         numeric(12,2) NOT NULL, -- farmer receives, grade B
  set_by                 uuid NOT NULL REFERENCES users(id),
  published_at           timestamptz,          -- null = draft, invisible to farmer/buyer
  created_at             timestamptz NOT NULL DEFAULT now(),
  UNIQUE (region_id, produce_id, price_date),
  CHECK (farmer_price_a <= platform_price_a AND farmer_price_b <= platform_price_b)
);

-- ============ supply: listings ============
CREATE TABLE listings (
  id                uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  farmer_id         uuid NOT NULL REFERENCES users(id),
  hub_id            uuid NOT NULL REFERENCES hubs(id),
  produce_id        uuid NOT NULL REFERENCES produce(id),
  qty               numeric(10,2) NOT NULL CHECK (qty > 0),   -- in produce.unit
  harvest_date      date NOT NULL,             -- farmer's declared harvest day
  status            text NOT NULL DEFAULT 'posted'
                      CHECK (status IN ('posted','graded','allocated','closed','cancelled')),
  grade             text CHECK (grade IN ('A','B')),           -- set at grading
  graded_qty        numeric(10,2),             -- weighed at hub
  graded_by         uuid REFERENCES users(id),
  graded_at         timestamptz,
  harvest_ts        timestamptz,               -- set at grading; COPIED onto each order_allocation at allocation (§6.3/§6.7)
  hub_in_ts         timestamptz,               -- set at grading (weigh-in); COPIED onto order_allocation at allocation
  photo_ref         text,                      -- storage.js key of the grading-evidence photo (one per farmer-crop-day; §6.6 L6)
  parent_listing_id uuid REFERENCES listings(id),  -- set on the B-part of a graded split
  created_at        timestamptz NOT NULL DEFAULT now()
);
-- Split rule: one listing carries ONE grade. If a hub grades 100 kg as 70 A + 30 B,
-- the original row becomes grade A / graded_qty 70 and a child row (parent_listing_id set)
-- is created with grade B / graded_qty 30. Payouts and traceability follow rows, so
-- both parts stay attributable to the same farmer.

-- ============ demand: orders ============
CREATE TABLE orders (
  id               uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  buyer_id         uuid NOT NULL REFERENCES users(id),
  region_id        uuid NOT NULL REFERENCES regions(id),
  status           text NOT NULL DEFAULT 'placed'
                     CHECK (status IN ('placed','confirmed','out_for_delivery','delivered','cancelled')),
  delivery_date    date NOT NULL,
  delivery_address text NOT NULL,              -- snapshot at order time
  currency         char(3) NOT NULL,
  subtotal         numeric(12,2) NOT NULL,     -- server-computed, never client-supplied
  delivery_fee     numeric(12,2) NOT NULL DEFAULT 0,
  tax_amount       numeric(12,2) NOT NULL DEFAULT 0,  -- via region tax module
  total            numeric(12,2) NOT NULL,
  invoice_no       text UNIQUE,                -- assigned at delivery: KS-INSRT-202611-000123
  cancel_reason    text,
  placed_at        timestamptz NOT NULL DEFAULT now(),
  confirmed_at     timestamptz,
  dispatched_at    timestamptz,
  delivered_at     timestamptz,
  cancelled_at     timestamptz
);

CREATE TABLE order_items (
  id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id   uuid NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  produce_id uuid NOT NULL REFERENCES produce(id),
  grade      text NOT NULL CHECK (grade IN ('A','B')),
  qty        numeric(10,2) NOT NULL CHECK (qty > 0),
  unit_price numeric(12,2) NOT NULL,           -- copied from price_feed at placement
  line_total numeric(12,2) NOT NULL
);

-- Traceability + freshness SLA chain (the moat — Golden rule #5)
CREATE TABLE order_allocations (
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  order_item_id uuid NOT NULL REFERENCES order_items(id) ON DELETE CASCADE,
  listing_id    uuid NOT NULL REFERENCES listings(id),
  qty           numeric(10,2) NOT NULL CHECK (qty > 0),
  harvest_ts    timestamptz,   -- COPIED from listings.harvest_ts at allocation creation (morning binding phase, §6.7)
  hub_in_ts     timestamptz,   -- COPIED from listings.hub_in_ts at allocation creation
  dispatch_ts   timestamptz,   -- crate left hub on 3PL vehicle (stamped on allocation)
  delivered_ts  timestamptz,   -- buyer door (stamped on allocation)
  created_at    timestamptz NOT NULL DEFAULT now(),
  UNIQUE (order_item_id, listing_id)
);

-- ============ disputes (buyer quality claims) ============
-- Backs 14-OPS-PLAYBOOK §8 (2-hour claim window) and metric M7. A claim is filed against the
-- specific allocation (so it resolves to one farmer's produce), photographed, then resolved to
-- a credit note / replacement / rejection. Defined before payments so credit_note can FK here.
CREATE TABLE disputes (
  id                  uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  order_allocation_id uuid NOT NULL REFERENCES order_allocations(id),
  buyer_id            uuid NOT NULL REFERENCES users(id),   -- denormalized for aging/lookup
  reason              text NOT NULL,            -- 'spoiled','short_weight','wrong_grade','wrong_item','other'
  photo_ref           text,                     -- storage.js key of buyer-supplied evidence photo
  claimed_at          timestamptz NOT NULL DEFAULT now(),   -- must be within 2 h of delivered_ts (ops-enforced)
  resolution          text CHECK (resolution IN ('credit','replacement','rejected')),  -- null = open
  amount              numeric(12,2) CHECK (amount IS NULL OR amount >= 0),  -- credit magnitude when resolution='credit'
  resolved_by         uuid REFERENCES users(id),
  resolved_at         timestamptz,
  created_at          timestamptz NOT NULL DEFAULT now()
);

-- ============ money ============
CREATE TABLE payments (
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  type          text NOT NULL CHECK (type IN ('farmer_payout','buyer_invoice','credit_note')),
  payee_user_id uuid NOT NULL REFERENCES users(id),
  listing_id    uuid REFERENCES listings(id),  -- set for farmer_payout
  order_id      uuid REFERENCES orders(id),    -- set for buyer_invoice AND credit_note (the invoice being adjusted)
  dispute_id    uuid REFERENCES disputes(id),  -- set for a credit_note raised from a dispute (§6.12)
  currency      char(3) NOT NULL,
  amount        numeric(12,2) NOT NULL CHECK (amount > 0),
  status        text NOT NULL DEFAULT 'pending'
                  CHECK (status IN ('pending','processing','paid','failed')),
  provider      text NOT NULL DEFAULT 'manual' CHECK (provider IN ('manual','razorpay_upi')),
  provider_ref  text,                          -- payout id from provider
  utr           text,                          -- bank UTR shown to farmer
  failure_reason text,
  due_date      date,                          -- buyer_invoice: delivery_date + buyer credit_terms; null for payouts. Drives credit_hold + R5 aging
  created_at    timestamptz NOT NULL DEFAULT now(),
  paid_at       timestamptz
);
-- credit_note rows carry amount > 0 (the credit magnitude), reduce the buyer's outstanding on order_id,
-- and are created 'paid' (an accounting adjustment, not a collection). See §6.12.

-- ============ auth & intake ============
CREATE TABLE otp_codes (
  id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  phone      text NOT NULL,
  code_hash  text NOT NULL,                    -- sha256(code + OTP_PEPPER)
  purpose    text NOT NULL DEFAULT 'login',
  attempts   int NOT NULL DEFAULT 0,           -- max 5, then dead
  expires_at timestamptz NOT NULL,             -- now() + 5 min
  used_at    timestamptz,
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE leads (
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name          text NOT NULL,
  phone         text NOT NULL,                 -- E.164 normalized server-side; default cc from region
  business_type text,
  message       text,
  source        text NOT NULL DEFAULT 'website', -- 'website','meta_ads','referral','event'
  region_id     uuid REFERENCES regions(id),
  status        text NOT NULL DEFAULT 'new'
                  CHECK (status IN ('new','contacted','sampled','converted','dropped')),
  assigned_to   uuid REFERENCES users(id),
  notes         text,
  created_at    timestamptz NOT NULL DEFAULT now()
);

-- ============ audit ============
CREATE TABLE audit_events (                     -- append-only; every state mutation by ops
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  actor_id    uuid REFERENCES users(id),
  entity_type text NOT NULL,                   -- 'order','listing','payment','price_feed',...
  entity_id   uuid NOT NULL,
  action      text NOT NULL,                   -- 'status:placed→confirmed','grade','payout:paid',...
  detail      jsonb,
  created_at  timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE invoice_sequences (                -- per region+month invoice numbering
  region_id uuid NOT NULL REFERENCES regions(id),
  yyyymm    char(6) NOT NULL,
  next_seq  int NOT NULL DEFAULT 1,
  PRIMARY KEY (region_id, yyyymm)
);

-- ============ indexes ============
CREATE INDEX idx_listings_hub_status   ON listings (hub_id, status);
CREATE INDEX idx_listings_farmer       ON listings (farmer_id, created_at DESC);
CREATE INDEX idx_price_feed_lookup     ON price_feed (region_id, price_date, produce_id);
CREATE INDEX idx_orders_buyer          ON orders (buyer_id, placed_at DESC);
CREATE INDEX idx_orders_region_date    ON orders (region_id, delivery_date, status);
CREATE INDEX idx_alloc_item            ON order_allocations (order_item_id);
CREATE INDEX idx_alloc_listing         ON order_allocations (listing_id);
CREATE INDEX idx_payments_payee        ON payments (payee_user_id, created_at DESC);
CREATE INDEX idx_payments_status       ON payments (status) WHERE status IN ('pending','processing');
CREATE INDEX idx_leads_status          ON leads (status, created_at DESC);
CREATE INDEX idx_audit_entity          ON audit_events (entity_type, entity_id, created_at DESC);
CREATE INDEX idx_otp_phone             ON otp_codes (phone, created_at DESC);
CREATE INDEX idx_disputes_alloc        ON disputes (order_allocation_id);
CREATE INDEX idx_disputes_open         ON disputes (resolution, claimed_at DESC);   -- open-claim worklist (resolution IS NULL)
CREATE INDEX idx_payments_invoice_due  ON payments (due_date) WHERE type = 'buyer_invoice' AND status <> 'paid';  -- R5 aging + credit_hold
```

## 6. API specification

Base URL dev: `http://localhost:4000/v1`. All responses JSON `snake_case`. Auth = `Authorization: Bearer <jwt>` unless marked **public**.

### 6.1 Conventions

- **List envelope**: `{"data": [...], "total": 137, "limit": 20, "offset": 0}` — `limit` default 20, max 100.
- **Error shape** (uniform; supersedes scaffold's `{error: string}`):

```json
{ "error": { "code": "validation_failed", "message": "qty must be > 0",
             "field_errors": { "qty": "must be > 0" } } }
```

- **Error codes** (HTTP → `code`): `400 validation_failed` · `401 unauthorized` / `otp_invalid` / `otp_expired` / `token_expired` · `403 forbidden_role` / `user_blocked` / `account_pending` / `credit_hold` · `404 not_found` · `409 conflict_state` (illegal lifecycle transition) / `insufficient_stock` / `duplicate` / `past_cutoff` / `claim_window_closed` · `422 price_unavailable` / `below_min_order` · `429 rate_limited` (with `Retry-After` header) · `500 internal` (message is generic; details go to logs only).
- **Idempotency**: `POST /orders` and `POST /payments/...` accept an optional `Idempotency-Key` header; a repeat with the same key returns the original response (keys stored 24 h in an `idempotency_keys` table).
- **JWT claims**: `{ sub: user_id, role, region_id, iat, exp }`. HS256, secret from env `JWT_SECRET` (≥32 bytes). Expiry 30 days; renewal = new OTP login (no refresh-token machinery at MVP).

### 6.2 Auth & OTP

| # | Method & path | Auth | Role | Purpose |
|---|---|---|---|---|
| A1 | `POST /auth/otp/request` | public | — | Send OTP SMS |
| A2 | `POST /auth/otp/verify` | public | — | Verify OTP → token or signup ticket |
| A3 | `POST /auth/signup` | public (signup_token) | — | Create account with role choice |

**A1** request/response:
```json
// POST /auth/otp/request
{ "phone": "+919876543210" }
// 200
{ "request_id": "7f0c...", "expires_in_s": 300, "resend_after_s": 30 }
```
Behavior: 6-digit code, 5-min TTL, sha256+pepper stored. Dev mode (`NODE_ENV!=production` and `DEV_OTP` set): code is `123456`, no SMS sent. Prod SMS provider behind `sms.js` interface (MSG91 first instance; Twilio for non-India regions). Errors: `429 rate_limited` (see §8), `400 validation_failed` (non-E.164).

**A2**:
```json
// POST /auth/otp/verify
{ "phone": "+919876543210", "code": "123456", "request_id": "7f0c..." }
// 200 — existing user
{ "token": "eyJhbGci...", "user": { "id": "…", "phone": "+919876543210", "name": "Ramesh",
  "role": "farmer", "status": "active",
  "region": { "code": "IN-GJ-SURAT", "currency": "INR" }, "language": "hi" } }
// 200 — new phone (client then calls POST /auth/signup with this signup_token)
{ "signup_required": true, "signup_token": "eyJ...15min..." }
```
Errors: `401 otp_invalid` (attempts++; 5 wrong = code dead), `401 otp_expired`.

**A3**:
```json
// POST /auth/signup
{ "signup_token": "eyJ...", "name": "Ramesh Patel", "role": "farmer",
  "language": "hi", "region_code": "IN-GJ-SURAT" }
// 201 → same shape as A2 success
```
`role` ∈ {`farmer`,`buyer`} only — `ops` users are created by seed/admin insert, never self-signup. **Status at creation:** `farmer` → `active`; `buyer` → `pending` (a pending buyer can sign in and browse but CANNOT place orders until ops activates — see §6.3 U10 and §6.7 `account_pending`). The A3 `201` body is the same shape as A2 success and carries `user.status`, so the client can route a new buyer straight to a "your account is under review" state. Farmer must then complete `PATCH /farmer-profiles/me` before posting listings (enforced with `409 conflict_state`, code `profile_incomplete`).

### 6.3 Users & profiles

| # | Method & path | Role | Purpose |
|---|---|---|---|
| U1 | `GET /me` | any | Current user (incl. `status`) + profile |
| U2 | `PATCH /users/me` | any | Update `name`, `language` (**renamed** from `PATCH /me`) |
| U3 | `PATCH /farmer-profiles/me` | farmer | Upsert `{hub_id, village, land_area, land_unit, upi_vpa}` (**renamed** from `PUT /me/farmer_profile`) |
| U4 | `PATCH /buyer-profiles/me` | buyer | Upsert `{business_name, business_type, tax_id, address}` (**renamed** from `PUT /me/buyer_profile`) |
| U5 | `GET /users?role=&status=&q=&limit=&offset=` | ops | Search users (q matches name/phone; `status=pending` = activation queue) |
| U6 | `GET /users/:id` | ops | Full user + profile + last activity |
| U7 | `PATCH /users/:id` | ops | Set `status` (`active`/`blocked`), fix profile fields |
| U8 | `GET /hubs`, `GET /fpos` | any authed | Pick-lists (region-scoped from JWT) |
| U9 | `POST /hubs`, `PATCH /hubs/:id`, `POST /fpos`, `PATCH /fpos/:id` | ops | Partner management |
| U10 | `PATCH /users/:id/activate` | ops | Activate a `pending` buyer → `status='active'`; writes `audit_events` action `buyer_activated`; `409 conflict_state` if not currently `pending` |

**Endpoint renames (canon):** the self-service profile/identity routes are now `PATCH /users/me`, `PATCH /farmer-profiles/me`, `PATCH /buyer-profiles/me` (plural-noun collections with a `/me` selector), replacing the earlier `/me/*` forms. `GET /me` is retained as the identity/bootstrap read. The daily price feed read is `GET /prices/today` (§6.5). These names match what `07-PRD-MOBILE-APPS.md` consumes.

`GET /me` response example:
```json
{ "id": "…", "phone": "+919876543210", "name": "Ramesh Patel", "role": "farmer",
  "status": "active", "language": "hi", "region": { "code": "IN-GJ-SURAT", "currency": "INR",
  "timezone": "Asia/Kolkata", "reference_market_label": "mandi" },
  "farmer_profile": { "hub": { "id": "…", "name": "Kamrej Hub" }, "village": "Kathor",
    "land_area": 1.5, "land_unit": "acre", "upi_vpa": "ramesh@upi" } }
// A buyer under review returns "role":"buyer","status":"pending" and a buyer_profile block incl. credit_terms/exposure_cap.
```

### 6.4 Produce catalog

| # | Method & path | Role | Purpose |
|---|---|---|---|
| P1 | `GET /produce?lang=hi` | any authed | Active produce, name resolved to `lang` with `name_en` fallback |
| P2 | `POST /produce` / `PATCH /produce/:id` | ops | Manage catalog + translations (`names: {"hi": "टमाटर", "gu": "ટામેટા"}`) |

```json
// GET /produce?lang=hi → 200
{ "data": [ { "id": "…", "code": "tomato", "name": "टमाटर", "name_en": "Tomato",
  "category": "vegetable", "unit": "kg", "image_key": "produce_tomato", "sort_order": 10 } ], "total": 24 }
```

### 6.5 Daily price feed

| # | Method & path | Role | Purpose |
|---|---|---|---|
| F1 | `GET /prices/today?date=2026-11-05` | any authed | Batch today's price cards for region (default: today in region tz; `?date=` overrides). **Renamed** from `GET /prices` |
| F2 | `PUT /prices/:date` | ops | Bulk upsert draft rows for the date |
| F3 | `POST /prices/:date/publish` | ops | Publish (atomic; sets `published_at`) |
| F4 | `GET /prices/history?produce_id=&days=30` | ops | Trend for the price-setting screen |

```json
// F2: PUT /prices/2026-11-06   (ops)
{ "rows": [ { "produce_id": "…", "reference_market_price": 32.00,
    "platform_price_a": 30.00, "platform_price_b": 24.00,
    "farmer_price_a": 21.00, "farmer_price_b": 16.00 } ] }
// 200 { "upserted": 24, "published": false }

// F1: GET /prices/today → 200
{ "date": "2026-11-06", "currency": "INR", "published_at": "2026-11-05T14:32:11Z",
  "data": [ { "produce": { "id": "…", "code": "tomato", "name": "टमाटर", "unit": "kg" },
    "reference_market_price": 32.00, "platform_price_a": 30.00, "platform_price_b": 24.00,
    "farmer_price_a": 21.00, "farmer_price_b": 16.00 } ] }
```
Rules: farmer/buyer see only published rows; farmer app shows `farmer_price_*` vs `reference_market_price`; buyer app shows `platform_price_*` vs `reference_market_price` (struck through). Editing after publish is allowed until the first order of the day references the row → then `409 conflict_state` (correction = ops cancels affected orders first; this keeps invoices truthful). Server validates farmer share on publish: warn (not block) if `farmer_price_a / platform_price_a < 0.60` — the ≥60% target is Golden rule #5.

### 6.6 Listings & grading

| # | Method & path | Role | Purpose |
|---|---|---|---|
| L1 | `POST /listings` | farmer | Post produce for pickup |
| L2 | `GET /listings?mine=true&status=` | farmer | My listings |
| L3 | `GET /listings?hub_id=&status=&harvest_date=` | ops | Grading queue etc. |
| L4 | `GET /listings/:id` | owner or ops | Detail incl. payout estimate/actual |
| L5 | `POST /listings/:id/cancel` | farmer (while `posted`), ops | Cancel |
| L6 | `POST /listings/:id/grade` | ops | Weigh + grade (+ optional split) + evidence photo (multipart) |
| L7 | `POST /listings/:id/close` | ops | Terminal close (leftover/spoilage) |
| L8 | `GET /listings/estimate?produce_id=&qty=&grade_mix=` | farmer | Estimated farmer payout for a prospective listing — **no row created** |

```json
// L8: GET /listings/estimate?produce_id=…&qty=100&grade_mix=A:70,B:30 → 200
// grade_mix optional; omitted ⇒ estimate assumes all grade A. Uses latest published farmer_price_* for region.
{ "produce_id": "…", "qty": 100, "currency": "INR",
  "price_basis_date": "2026-11-06", "note_key": "estimate_from_today_prices",
  "breakdown": [ { "grade": "A", "qty": 70, "farmer_price": 21.00, "amount": 1470.00 },
                 { "grade": "B", "qty": 30, "farmer_price": 16.00, "amount": 480.00 } ],
  "estimated_payout": 1950.00 }
```
This is the read behind the farmer app's "what will I earn?" slider; it shares the pricing helper that stamps `estimated_payout` on `POST /listings` (L1). If no published price exists for the region it returns `422 price_unavailable`.

```json
// L1: POST /listings   (farmer)
{ "produce_id": "…", "qty": 100, "harvest_date": "2026-11-07" }
// 201
{ "id": "…", "status": "posted", "produce": { "code": "tomato", "name": "टमाटर", "unit": "kg" },
  "qty": 100, "harvest_date": "2026-11-07", "hub": { "name": "Kamrej Hub" },
  "estimated_payout": { "currency": "INR", "grade_a": 2100.00, "grade_b": 1600.00,
    "note_key": "estimate_from_today_prices" } }
```
Validation: `harvest_date` ≥ today and ≤ today+7 (region tz); default in app = tomorrow. Farmer must have a complete `farmer_profile`. Max 10 open listings per farmer (`429`-style guard as `409 conflict_state`).

```json
// L6: POST /listings/:id/grade   (ops)  — Content-Type: multipart/form-data
//   field "data" (JSON): { "grade":"A","graded_qty":70,"hub_in_ts":"2026-11-07T02:30:00Z",
//                          "split": { "grade":"B","graded_qty":30 } }
//   field "photo" (optional file): one grading-evidence image (jpeg/png, ≤5 MB)
// 200
{ "listing": { "id": "…", "status": "graded", "grade": "A", "graded_qty": 70,
               "harvest_ts": "2026-11-07T00:30:00Z", "hub_in_ts": "2026-11-07T02:30:00Z",
               "photo_ref": "grade/2026-11-07/<farmer_id>/tomato.jpg" },
  "split_listing": { "id": "…", "status": "graded", "grade": "B", "graded_qty": 30,
                     "parent_listing_id": "…" } }
```
Side effects: `harvest_ts` **and** `hub_in_ts` are **stored on the listing at grading** (`harvest_ts` = harvest_date at 06:00 region tz unless ops overrides; `hub_in_ts` = supplied weigh-in time); farmer notified ("आपकी फ़सल ग्रेड हो गई — A: 70 kg"). These two timestamps are later **copied onto each `order_allocation`** when the morning allocation is created (§6.7) — grading is where the freshness chain begins. Grading a listing whose farmer has no published farmer price today → `422 price_unavailable`.

**Grading-evidence photo (MVP, revised non-goal in §3):** the optional `photo` part is written through a `storage.js` interface — `put(key, bytes) → ref` / `url(ref)` — whose **only** MVP driver is **local disk** (pilot runs single-instance). Key is deterministic: `grade/{harvest_date}/{farmer_id}/{produce_code}.jpg` → **one photo per farmer-per-crop-per-day** (a re-grade overwrites). `listings.photo_ref` stores the returned key. This photo is the evidence base for disputes (§6.12); Phase 2 swaps the driver for the S3-compatible bucket with no call-site changes. Photo is optional so grading never blocks on a missing camera.

**Listing expiry (lifecycle sweep):** a listing still `posted` at the **end of `harvest_date + 1 day` (region tz)** is a no-show. A daily cron sweep (runs 00:15 region tz) sets such rows to `status='closed'` with audit action `listing_closed` / `detail.reason='expired_no_show'`; no payout row is created. To stop stale rows from inflating supply, `available_qty` (§6.7 O1) **excludes any `posted` listing once its `harvest_date` 12:00 (region tz) has passed** — i.e. exclusion begins the same day, before the overnight sweep formally closes it. Expiring/expired listings surface on the ops **Today** screen (`09-PRD-OPS-DASHBOARD.md`) so a hub can chase or reopen them. This keeps `available_qty` truthful and frees the farmer's 10-open-listings budget (L1 cap).

### 6.7 Buyer catalog, orders & allocation

| # | Method & path | Role | Purpose |
|---|---|---|---|
| O1 | `GET /catalog?delivery_date=` | buyer | Buyer catalog: per produce+grade price, `available_qty`, `stock_flag` (`in_stock`/`low`/`sold_out`), freshness |
| O1q | `POST /orders/quote` | buyer | Server-price a cart (subtotal / delivery_fee / total) **without creating an order** |
| O2 | `POST /orders` | buyer | Place order (server-priced) |
| O3 | `GET /orders?mine=true&status=` | buyer | My orders |
| O4 | `GET /orders?status=&delivery_date=` | ops | Order board |
| O5 | `GET /orders/:id` | owner or ops | Detail + items + traceability |
| O6 | `POST /orders/:id/cancel` | buyer (while `placed`), ops (before `out_for_delivery`) | Cancel with reason |
| O7 | `POST /orders/:id/status` | ops | Advance lifecycle |
| O8 | `POST /order_items/:id/allocations` | ops | Allocate listing→item |
| O9 | `DELETE /allocations/:id` | ops | Undo allocation (before dispatch) |
| O10 | `PATCH /allocations/:id` | ops | Stamp SLA timestamps |
| O11 | `GET /allocation_board?delivery_date=` | ops | Items vs graded supply, with suggestions |

```json
// O1: GET /catalog?delivery_date=2026-11-08 → 200
{ "delivery_date": "2026-11-08", "currency": "INR", "data": [
  { "produce": { "id": "…", "code": "tomato", "name_en": "Tomato", "name_local": "टमाटर", "unit": "kg" },
    "reference_market_price": 32.00,
    "grades": [
      { "grade": "A", "unit_price": 30.00, "available_qty": 240, "stock_flag": "in_stock", "freshness_hint_h": 24 },
      { "grade": "B", "unit_price": 24.00, "available_qty": 12,  "stock_flag": "low",      "freshness_hint_h": 24 },
      { "grade": "B2","unit_price": 0.00,  "available_qty": 0,   "stock_flag": "sold_out", "freshness_hint_h": 24 } ] } ] }
```
`available_qty` = Σ graded, unallocated listing qty at active hubs + Σ posted qty for the target harvest window (flagged `includes_posted: true` so ops over-promise risk is visible), **excluding expired/expiring listings per the §6.6 expiry rule**. `stock_flag`: `sold_out` when `available_qty = 0`, `low` when `0 < available_qty ≤ region low-stock threshold` (default 20 units — a `region_settings`-adjacent constant), else `in_stock`. Zero-availability rows still render (buyers see the price range) but are non-orderable client-side.

```json
// O1q: POST /orders/quote   (buyer) — same body shape as POST /orders, minus delivery_address
{ "delivery_date": "2026-11-08",
  "items": [ { "produce_id": "…", "grade": "A", "qty": 25 },
             { "produce_id": "…", "grade": "B", "qty": 10 } ] }
// 200 — priced with the SAME server-side engine as POST /orders; no row written, no stock held
{ "delivery_date": "2026-11-08", "currency": "INR",
  "items": [ { "produce_id": "…", "grade": "A", "qty": 25, "unit_price": 30.00, "line_total": 750.00 },
             { "produce_id": "…", "grade": "B", "qty": 10, "unit_price": 24.00, "line_total": 240.00 } ],
  "subtotal": 990.00, "delivery_fee": 50.00, "tax_amount": 0.00, "total": 1040.00,
  "min_order_value": 1000.00, "meets_minimum": false,
  "delivery_fee_waived": false, "delivery_fee_waiver_threshold": 4000.00 }
```
`POST /orders/quote` is the cart-screen preview: it runs the identical pricing path as `POST /orders` (server prices, delivery-fee waiver, tax module) but is side-effect-free — no order row, no idempotency key, no stock reservation, and it does **not** enforce the account-pending/credit guards (those fire only on real placement). `meets_minimum=false` lets the client disable checkout below `min_order_value` without a failed POST.

```json
// O2: POST /orders   (buyer, Idempotency-Key recommended)
{ "delivery_date": "2026-11-08", "delivery_address": "Hotel Gateway, Ring Road, Surat",
  "items": [ { "produce_id": "…", "grade": "A", "qty": 25 },
             { "produce_id": "…", "grade": "B", "qty": 10 } ] }
// 201
{ "id": "…", "status": "placed", "delivery_date": "2026-11-08", "currency": "INR",
  "items": [ { "id": "…", "produce_code": "tomato", "grade": "A", "qty": 25,
               "unit_price": 30.00, "line_total": 750.00 } ],
  "subtotal": 990.00, "delivery_fee": 50.00, "tax_amount": 0.00, "total": 1040.00 }
```
Pricing is 100% server-side from the published price row for `delivery_date` (fallback: latest published date ≤ delivery_date; none → `422 price_unavailable`). All monetary rules read `region_settings` (§5):

- **Delivery date & cutoff:** `delivery_date` ≥ tomorrow (region tz), ≤ today+7. An order for next-day delivery must be placed **before `order_cutoff_local` (18:00 D-1, region tz)**; past cutoff the client offers the following day. A same-day-for-tomorrow request after cutoff → `409 conflict_state` code `past_cutoff`. Fulfilled window is `delivery_window` (06:00–10:00 D0).
- **Minimum order:** `subtotal ≥ min_order_value` (₹1,000). Below → `422 below_min_order` (the quote's `meets_minimum` mirrors this).
- **Delivery fee:** flat `delivery_fee` (₹50), **waived when `subtotal ≥ delivery_fee_waiver_threshold` (₹4,000)**. Revisit with 3PL quotes per `14-OPS-PLAYBOOK.md`.
- **Tax:** via region tax module — India instance v1 returns 0 (fresh produce GST-exempt; see Open questions Q3).

**Order-placement guards (real placement only; `POST /orders/quote` skips these):**
- **Pending buyer** (`users.status='pending'`) → `403 { "error": { "code": "account_pending" } }`. The account must be ops-activated (§6.3 U10) first.
- **Credit hold** — computed against the buyer's outstanding `buyer_invoice` payments (unpaid `amount` minus any `credit_note`): if **outstanding exposure > `buyer_profiles.exposure_cap`** (₹25,000) **OR any buyer_invoice is ≥ 10 days past `due_date`** → `403 { "error": { "code": "credit_hold" } }`. A buyer's **first 4 orders are pay-on-delivery** (invoice due on the delivery date, `due_date = delivery_date`); from the 5th order onward, `due_date = delivery_date + credit_terms` days (net-7). "First 4" counts the buyer's non-cancelled orders.

```json
// O5 (delivered order) — traceability block, the premium differentiator
{ "id": "…", "status": "delivered", "invoice_no": "KS-INSRT-202611-000123",
  "items": [ { "id": "…", "produce_code": "tomato", "grade": "A", "qty": 25,
    "trace": [ { "farmer_name": "Ramesh Patel", "village": "Kathor", "hub": "Kamrej Hub",
      "qty": 25, "harvest_ts": "2026-11-07T00:30:00Z", "hub_in_ts": "2026-11-07T02:30:00Z",
      "dispatch_ts": "2026-11-07T04:10:00Z", "delivered_ts": "2026-11-07T08:05:00Z",
      "harvest_to_door_h": 7.6 } ] } ] }
```
Farmer PII rule: trace exposes farmer first name + village only — no phone, no payout amounts.

**Two-phase allocation (canonical).** Confirmation and binding allocation are deliberately split across two clock times, because at cutoff we know *demand* but the graded *supply* only exists the next morning:

- **Phase 1 — EVENING supply check (18:00 D-1).** After the order cutoff, ops runs a region-level supply check: for each ordered produce+grade, does projected graded+posted supply cover demand? If yes (or ops accepts the gap), the order moves `placed → confirmed` with its **provisional quantities** — the buyer's ordered lines are the plan. **No `order_allocations` rows are created to confirm.** `confirmed` means "we intend to fulfil this," not "this crate is assigned." `confirmed_at` is stamped; buyer notified.
- **Phase 2 — MORNING binding allocation (06:30–07:15 D0).** After the night's harvest is weighed and **graded**, ops creates the **binding `order_allocations`** (O8) that map each `order_item` to specific graded `listings`. This is the first point real crates are committed. Only once every item is fully allocated can the order advance to `out_for_delivery`.

Because allocations are created in the morning against already-graded listings, **the freshness timestamps live on the listing first and are copied onto the allocation at creation:** `harvest_ts` and `hub_in_ts` are written to the `listings` row at grading (§6.6 L6) and **copied into each `order_allocation` when O8 creates it** (the allocation is a point-in-time snapshot; later edits to the listing do not retro-change delivered allocations). `dispatch_ts` and `delivered_ts` are stamped later on the allocation as the crate moves.

**Lifecycle enforcement** (`409 conflict_state` otherwise): orders `placed → confirmed → out_for_delivery → delivered`; `cancelled` reachable from `placed`/`confirmed`. `placed → confirmed` requires only the Phase-1 supply check — **not** allocations. `POST /orders/:id/status {"status": "out_for_delivery"}` requires every item fully allocated (Phase 2) and stamps `dispatch_ts` on allocations that lack it; `delivered` stamps `delivered_ts`, assigns `invoice_no` (from `invoice_sequences`, format `KS-{region_short}-{YYYYMM}-{seq:06}`), and creates the `buyer_invoice` payment row **with `due_date` set per the buyer's credit terms** (pay-on-delivery for the first 4 orders ⇒ `due_date = delivery_date`; thereafter `delivery_date + credit_terms`). Allocation (O8) validates grade+produce match and remaining qty both sides, and **copies `harvest_ts`/`hub_in_ts` from the listing onto the new allocation**; listing flips to `allocated` when fully consumed. Cancelling a confirmed order releases its allocations back to `graded`.

### 6.8 Payments & payouts

| # | Method & path | Role | Purpose |
|---|---|---|---|
| M1 | `GET /payments?mine=true&type=farmer_payout` | farmer | Payout history (amount, date, UTR) |
| M2 | `GET /payments?status=pending&type=` | ops | Payout/invoice worklists |
| M3 | `POST /payouts` | ops | Create payout for a graded listing |
| M4 | `POST /payments/:id/mark_paid` | ops | Manual settle `{utr}` (MVP path) |
| M5 | `POST /payments/:id/execute` | ops | Execute via provider (Razorpay UPI, Phase 2) |
| M6 | `POST /webhooks/razorpay` | public + signature | Provider status callbacks |

```json
// M3: POST /payouts   (ops)
{ "listing_id": "…" }
// 201 — amount computed server-side: graded_qty × farmer_price_{grade} for harvest_date
{ "id": "…", "type": "farmer_payout", "payee_user_id": "…", "listing_id": "…",
  "currency": "INR", "amount": 1470.00, "status": "pending", "provider": "manual" }
```
Rules: one payout per listing (`409 duplicate`); listing must be `graded`/`allocated`/`closed` with `graded_qty`. M4 sets `paid` + `utr` + `paid_at` and notifies the farmer ("पैसा भेज दिया गया — ₹1,470, UTR …"). M5/M6 (Phase 2) go through `payments/provider.js` interface: `createPayout(payment) → {provider_ref}`, webhook maps provider states → `processing/paid/failed`; `failed` returns to the ops worklist with `failure_reason`. Webhook auth: Razorpay signature header verified; unverified → 401 and alert log.

**Buyer-side payment rows.** A `buyer_invoice` row is created automatically at `delivered` (§6.7) with `due_date` per the buyer's credit terms; ops records collection with M4 `mark_paid {utr}` (Q2's net-7 path). A `credit_note` row (§6.12) is a negative-adjustment against a specific `order_id` (and its `dispute_id`): it reduces that buyer's outstanding balance used by the credit-hold / receivables-aging logic. Both types are visible to ops in M2 worklists (`type=buyer_invoice` / `type=credit_note`); the buyer sees the net of invoice − credit notes on the order detail.

### 6.9 Leads

| # | Method & path | Auth | Purpose |
|---|---|---|---|
| D1 | `POST /leads` | public (rate-limited) | Website form intake |
| D2 | `GET /leads?status=&limit=&offset=` | ops | Lead inbox |
| D3 | `PATCH /leads/:id` | ops | Status, assignment, notes |

```json
// D1: POST /leads
{ "name": "Kiran Shah", "phone": "9876501234", "business_type": "restaurant",
  "message": "Need daily tomatoes ~30kg", "source": "website" }
// 201 { "id": "…", "received": true }
```
Phone normalized to E.164 server-side (default country code from the site's region, `+91` at launch). Honeypot field `website_url` — non-empty → silently accept and discard (bot trap).

### 6.10 Ops reports & SLA instrumentation

| # | Method & path | Role | Purpose |
|---|---|---|---|
| R1 | `GET /reports/summary?date=` | ops | Daily digest: orders by status, kg graded, fulfilment %, pending payouts |
| R2 | `GET /reports/sla?from=&to=` | ops | Freshness: median + p90 harvest→door hours, per produce and overall; count of allocations with incomplete chains |
| R3 | `GET /reports/farmer_share?from=&to=` | ops | Σ farmer payouts ÷ Σ buyer produce subtotal (delivered orders), overall + per produce |
| R4 | `GET /reports/sla/public` | public | Cached (1 h) `{median_harvest_to_door_h, farmer_share_pct, updated_at}` for the website's live counter (`08-PRD-WEBSITE.md`) — published only when ops flips `region_settings.publish_public_stats=true` |
| R5 | `GET /reports/receivables_aging?as_of=` | ops | Outstanding buyer invoices (net of credit notes) bucketed `0-7 / 8-14 / 15-30 / 30+` days past `due_date`, per buyer with `exposure` vs `exposure_cap` and a `credit_hold` flag |
| R6 | `GET /reports/disputes?from=&to=` | ops | Dispute count + rate (disputes ÷ delivered orders), resolution mix (credit/replacement/rejected), Σ credit-note value — the source of metric **M7** in `14-OPS-PLAYBOOK.md` |

```json
// R5 → 200
{ "as_of": "2026-11-12", "currency": "INR",
  "totals": { "outstanding": 84200.00, "bucket_0_7": 61000.00, "bucket_8_14": 15200.00,
              "bucket_15_30": 8000.00, "bucket_30_plus": 0.00 },
  "by_buyer": [ { "buyer_id": "…", "business_name": "Hotel Gateway", "outstanding": 27300.00,
                  "exposure_cap": 25000.00, "oldest_days_overdue": 11, "credit_hold": true } ] }
```
Aging is computed as `Σ(unpaid buyer_invoice.amount) − Σ(credit_note.amount)` per buyer, bucketed by `now() − due_date`. `credit_hold=true` mirrors the §6.7 order guard (exposure > cap OR any invoice ≥10 days overdue), so ops sees the same verdict the API enforces.

```json
// R2 → 200
{ "from": "2026-11-01", "to": "2026-11-07",
  "overall": { "median_h": 31.4, "p90_h": 44.0, "allocations": 182, "incomplete_chains": 6 },
  "by_produce": [ { "code": "tomato", "median_h": 29.9, "p90_h": 41.2 } ],
  "breaches_over_48h": [ { "order_id": "…", "hours": 51.5 } ] }
```
Instrumentation rules: an allocation with any missing timestamp is counted in `incomplete_chains` and excluded from medians (never guessed). Report R2 is the input to the weekly metrics ritual in `14-OPS-PLAYBOOK.md`.

### 6.11 System, config & instrumentation

| # | Method & path | Auth | Purpose |
|---|---|---|---|
| S1 | `GET /health` | public | `{"status":"ok","db":"ok"}`, 503 if DB ping fails |
| S2 | `GET /version` | public | `{"version":"1.4.2","migration":"007"}` |
| S3 | `GET /config` | public (region-scoped) | Client bootstrap: commerce constants + force-update floors |
| S4 | `POST /events` | any authed | Batch app-instrumentation transport |

**S3 `GET /config`** — the single call every client makes on launch (before or after login). Region resolves from the JWT when present, else from `?region=IN-GJ-SURAT` (default = the launch region). Reads `region_settings` (§5); values are region-scoped so country #2 is a data insert.

```json
// GET /config  (or GET /config?region=IN-GJ-SURAT) → 200
{ "region": "IN-GJ-SURAT", "currency": "INR", "timezone": "Asia/Kolkata",
  "order_cutoff": "18:00", "delivery_window": "06:00-10:00",
  "min_order_value": 1000.00, "delivery_fee": 50.00, "delivery_fee_waiver_threshold": 4000.00,
  "support_phone": "+912612XXXXXX", "whatsapp_number": "+9190000XXXXX",
  "publish_public_stats": true,
  "min_supported_version": { "android": "1.0.0", "ios": "1.0.0" } }
```
`min_supported_version` drives the client force-update gate; because S3 is public and pre-auth, an out-of-date app can learn it must upgrade before the user even logs in.

**S4 `POST /events`** — fire-and-forget batch transport for app analytics/instrumentation. Accepts an array of events; the server timestamps receipt, tags `user_id`/`role`/`region_id` from the JWT, and emits each as a structured `pino` line (`event:` field, §11). Best-effort: it never blocks UX and returns `202` even if some rows are dropped (bad `name`s are skipped and counted).

```json
// POST /events
{ "events": [ { "name": "catalog_viewed", "ts": "2026-11-07T09:12:01Z", "props": { "produce_count": 24 } },
              { "name": "cart_quote_requested", "ts": "2026-11-07T09:13:40Z", "props": { "subtotal": 990 } } ] }
// 202 { "accepted": 2, "dropped": 0 }
```
Rate-limited per §8 (rolled into the authed global bucket, with a higher batch allowance). No new table at MVP — events are logs, not rows (consistent with the "no event bus / no second datastore" non-goals).

### 6.12 Disputes & credit notes

Buyer quality claims against a delivered order. Digitizes `14-OPS-PLAYBOOK.md` §8 (the **2-hour claim window**) and feeds metric **M7**. A claim targets a specific `order_allocation` so it resolves to exactly one farmer's produce (traceability cuts both ways), is optionally photographed, and is resolved by ops to a credit note, a replacement, or a rejection.

| # | Method & path | Role | Purpose |
|---|---|---|---|
| X1 | `POST /orders/:id/disputes` | buyer (own order) | File a claim (multipart: `data` JSON + optional `photo`) |
| X2 | `GET /disputes?status=open&limit=&offset=` | ops | Open-claim worklist (`status` ∈ `open`/`resolved`) |
| X3 | `GET /disputes/:id` | owner or ops | Detail incl. allocation trace + photo url |
| X4 | `POST /disputes/:id/resolve` | ops | Resolve → `credit` / `replacement` / `rejected` |

```json
// X1: POST /orders/:id/disputes   (buyer) — Content-Type: multipart/form-data
//   field "data": { "order_allocation_id": "…", "reason": "spoiled" }
//   field "photo": optional evidence image (stored via storage.js, §6.6)
// 201
{ "id": "…", "order_allocation_id": "…", "reason": "spoiled", "status": "open",
  "claimed_at": "2026-11-07T09:40:00Z", "photo_ref": "dispute/…/…jpg" }
```
Guard: `claimed_at` must be **within 2 h of the allocation's `delivered_ts`** — later → `409 conflict_state` code `claim_window_closed`. One open dispute per allocation (`409 duplicate`).

```json
// X4: POST /disputes/:id/resolve   (ops)
{ "resolution": "credit", "amount": 250.00 }
// 200 — for resolution='credit', a credit_note payment row is created against the order's buyer_invoice
{ "id": "…", "resolution": "credit", "amount": 250.00, "resolved_at": "2026-11-07T11:05:00Z",
  "credit_note_payment_id": "…" }
```
Resolution effects: `credit` → creates a `credit_note` payment (`type='credit_note'`, `order_id`, `dispute_id`, `amount`, status `paid`) that reduces the buyer's outstanding balance (feeds §6.7 credit-hold and R5 aging); `replacement` → logged for ops fulfilment, no money row; `rejected` → closed with reason. All four endpoints write `audit_events` (`dispute_created` / `dispute_resolved`).

## 7. Edge cases & error states (canonical decisions)

| Case | Decision |
|---|---|
| Farmer posts listing, no price published for harvest_date yet | Allowed; `estimated_payout` uses latest published prices with `note_key: estimate_from_today_prices`. Grading blocks (`422`) until price published. |
| Graded qty ≠ posted qty | Normal — graded_qty is truth. Farmer sees both ("Posted 100 kg → weighed 98 kg"). |
| Whole listing rejected at hub (spoilage) | L7 close with `graded_qty=0`, reason in audit detail; no payout row; farmer notified with reason key. |
| Order placed but supply undershoots | Ops partially allocates, then either edits the order down (ops-only `PATCH /order_items/:id {qty}` before confirm, re-priced + audited) or cancels the item; buyer notified. Fulfilment % in R1 tracks this. |
| Buyer cancels after confirm | `403 forbidden_role` — buyer path exists only in `placed`. Buyer calls ops; ops cancels with reason (releases allocations). |
| Two ops grade the same listing | Second gets `409 conflict_state` (status guard is `WHERE status='posted'` compare-and-set). |
| Same-day price re-publish after orders exist | Blocked `409`; correction procedure documented in `14-OPS-PLAYBOOK.md`. |
| OTP brute force | 5 wrong attempts kills the code; request limits in §8; **per-phone lockout 1 h after 5 failed attempts** (canonical). |
| Clock skew from hub tablets | All SLA timestamps are server-set (`now()`) unless ops explicitly backfills via O10, which is audited. |
| JWT of blocked user | Middleware re-checks `users.status` on every request (single indexed PK lookup); `403 user_blocked`. |
| Duplicate order double-tap | `Idempotency-Key`; without it, identical order from same buyer within 60 s → `409 duplicate`. |
| Pending buyer tries to order | `403 account_pending`; buyer can browse/quote but not place. Cleared by ops activation (U10). |
| Buyer over credit limit / overdue | `403 credit_hold` at placement (exposure > cap OR invoice ≥10 days overdue). Quote still works so the buyer sees the cart; ops clears by collecting (M4) or raising `exposure_cap`. |
| Confirm at evening without graded supply | Allowed — Phase-1 confirm is a supply *check*, not an allocation; provisional qtys stand. If morning grading undershoots, ops edits the item down or cancels it (existing under-supply row). No `order_allocations` exist until morning. |
| Buyer files claim after 2 h | `409 claim_window_closed` — the claim window is delivered_ts + 2 h (`14-OPS-PLAYBOOK.md` §8). |
| Listing never shows up at hub | Daily sweep auto-closes `posted` rows past `harvest_date + 1 day` as `expired_no_show`; excluded from `available_qty` from `harvest_date` 12:00; surfaced on ops Today screen. |
| Order for tomorrow placed after 18:00 cutoff | `409 past_cutoff`; client re-offers the next delivery date. |
| Grading tablet has no camera / photo fails | Grade proceeds without a photo (`photo_ref` null); dispute handling still works, just without hub-side evidence. |

## 8. Rate limiting, security, backups

**Rate limits** (in-process token buckets; keyed per phone/IP/user; single instance assumption per Non-goals):

| Scope | Limit |
|---|---|
| `POST /auth/otp/request` | 5/phone/hour, 20/IP/hour |
| `POST /auth/otp/verify` | 10/phone/hour; **1 h per-phone lockout after 5 failed attempts** (canonical) |
| `POST /leads` | 10/IP/hour, 50/IP/day |
| `POST /events` | 60 batches/min/user (rolled into the authed bucket) |
| `GET /config` | 60/IP/min (public, cacheable) |
| Authed endpoints (global) | 300 req/min/user; ops 600 |
| Response on breach | `429` + `Retry-After` |

**Security**: TLS terminated at reverse proxy (Caddy) — API never listens publicly on plain HTTP in prod. `helmet` headers; CORS allowlist = website + ops dashboard origins only (mobile apps don't need CORS). Parameterized SQL exclusively (repo lint greps for template-literal interpolation into query strings). JWT HS256 with ≥32-byte secret; secrets only via env (never in repo); `.env.example` documents every var. OTP codes hashed (sha256 + `OTP_PEPPER`). PII minimization: trace endpoints expose farmer first name + village only; logs redact `phone`, `upi_vpa`, `tax_id`, `utr`. All ops mutations write `audit_events`. Payout mutation endpoints (M3–M5) additionally log full request bodies to audit. Dependency policy: `npm audit` in CI, Express + `pg` + `jsonwebtoken` + `helmet` + `pino` and little else. Backups encrypted at rest.

**Backup policy**: nightly `pg_dump` at 02:00 region tz → compressed, encrypted, uploaded off-host (object storage), retained 30 days; weekly full kept 12 weeks. Monthly restore drill into a scratch DB with row-count sanity checks (drill logged in `14-OPS-PLAYBOOK.md`). Targets at pilot: **RPO 24 h, RTO 4 h**; at scale-up move to managed Postgres with PITR → RPO ≤ 5 min (see `11-ARCHITECTURE.md` §9).

## 9. Acceptance criteria (per module)

- **Auth**: new E.164 phone completes OTP→signup→JWT in ≤3 calls; wrong OTP ×5 dead-ends; `DEV_OTP` provably inert when `NODE_ENV=production` (startup assertion refuses to boot if both set); blocked user rejected on next request.
- **Prices**: ops drafts + publishes 24 produce rows in ≤2 calls; farmer/buyer never see drafts; publish warns below 60% farmer share; re-publish after first referencing order returns 409.
- **Listings**: farmer posts in one call and sees estimated payout; 70/30 split grade produces two traceable rows summing to weighed qty; grading without published price returns 422.
- **Orders**: totals recomputed server-side — a tampered client price changes nothing; order with full allocations can reach `delivered`, which assigns a unique sequential invoice_no and creates the invoice payment row; every illegal lifecycle jump returns `409 conflict_state`.
- **Traceability/SLA**: delivered order detail shows farmer name+village+harvest_ts per item; R2 median over seeded fixture data matches hand-computed value; incomplete chains counted, not interpolated.
- **Payouts**: payout amount equals `graded_qty × farmer_price_{grade}` for the harvest date's published row; double payout for one listing returns 409; farmer sees UTR in M1 after M4.
- **Two-phase allocation**: an order reaches `confirmed` at the evening supply check with **zero** `order_allocations`; it cannot advance to `out_for_delivery` until every item is fully allocated the next morning; each allocation's `harvest_ts`/`hub_in_ts` equal the values stored on its source listing at grading.
- **Buyers & credit**: a new buyer is `pending` and `POST /orders` returns `403 account_pending` until U10 activates them; a buyer over `exposure_cap` or with an invoice ≥10 days overdue is blocked with `403 credit_hold` while `POST /orders/quote` still returns a full price; the first 4 orders are due on delivery and the 5th onward is net-`credit_terms`.
- **Quote/config**: `POST /orders/quote` returns the same subtotal/fee/total as the eventual `POST /orders` but writes no row; `GET /config` returns the region's cutoff/window/min-order/fee/waiver + `min_supported_version` with zero auth.
- **Disputes**: a claim filed >2 h after `delivered_ts` returns `409 claim_window_closed`; a `credit` resolution creates a `credit_note` that lowers the buyer's outstanding in R5 by exactly `amount`.
- **Listing expiry**: a `posted` listing past `harvest_date + 1 day` (region tz) is swept to `closed` with `reason=expired_no_show`, drops out of `available_qty` (from `harvest_date` 12:00), and frees the farmer's open-listing budget.
- **Global-first**: inserting a second region row (e.g. `KE-NAIROBI`, KES, `Africa/Nairobi`, `tax_scheme='none'`, `languages={en,sw}`) and re-running the full API test suite scoped to it passes with **zero migrations** — this is a CI test, not a promise.
- **Non-functional**: p95 < 300 ms for all GETs at pilot data volumes (10k listings, 5k orders) on a 2-vCPU VM; smoke suite (supertest) covers the 5 core flows end-to-end and runs in CI on every push.

## 10. Phased rollout

| Phase | Contents |
|---|---|
| **MVP (pilot)** | Everything in §6 with `provider='manual'` payouts, dev/MSG91 OTP, **one local-disk grading/dispute evidence photo via `storage.js`**, no invoice PDFs. WhatsApp/push nudges optional (manual broadcast acceptable — `14-OPS-PLAYBOOK.md`). |
| **Phase 2 (live pilot hardening)** | Razorpay UPI payouts (M5/M6), WhatsApp Business templates (order confirmed, out for delivery, payout done, daily price broadcast), FCM/APNs push, invoice PDF (`GET /orders/:id/invoice.pdf`), **swap `storage.js` from local-disk to the S3-compatible bucket** (grading/dispute photos migrate transparently) + buyer-facing listing photo gallery. |
| **Phase 3 (scale-up)** | Buyer subscription flags + priority allocation, standing orders (repeat weekly), FPO-SaaS endpoints (hub-scoped ops accounts), CSV exports, managed Postgres + PITR. |

## 11. Metrics & instrumentation

Emitted as structured `pino` log events (`event:` field) and derivable from tables: `otp_requested/verified`, `buyer_activated`, `listing_posted/graded/split/expired`, `order_placed/confirmed/dispatched/delivered/cancelled`, `allocation_created/deleted`, `payout_created/paid/failed`, `invoice_created`, `credit_note_issued`, `dispute_created/resolved`, `lead_created`, `price_published`. App-side events arrive via `POST /events` (§6.11) and are logged with the same `event:` shape. Weekly KPI SQL pack (fulfilment %, median freshness h, farmer share %, buyer repeat rate, GMV, take rate realized, dispute rate = M7, receivables aging) versioned in-repo at `reports/weekly.sql`. Request logs carry `request_id`, `user_id`, `role`, latency ms.

## 12. Open questions (owner + deadline; decisions taken provisionally so build is unblocked)

| # | Question | Provisional decision in this PRD | Owner | Deadline |
|---|---|---|---|---|
| Q1 | Farmer payout timing: on grading (before produce sells) vs on pickup/dispatch? | Payout row created at grading; ops triggers payment at pickup per `14-OPS-PLAYBOOK.md` SOP | Founder | Phase 0 exit (FPO MoU) |
| Q2 | Does the buyer pay COD/credit terms/prepaid? | Invoice on delivery, net-7 collection recorded via M4 `buyer_invoice` mark-paid; no payment gateway on buyer side at MVP | Founder | Phase 0 buyer interviews |
| Q3 | India tax module: is delivery/handling fee GST-liable when bundled with exempt fresh produce? | v1 computes `tax_amount = 0`; module boundary exists so changing it is one file | Founder + CA | Before first real invoice |
| Q4 | Single grade pair (A/B) globally, or per-region grade sets? | A/B fixed enum at MVP; revisit only if a region demands it (schema change contained to CHECK constraints) | Founder | Country #2 onboarding |
| Q5 | OTP SMS provider for non-WhatsApp farmers vs missed-call auth | SMS OTP via MSG91; missed-call auth rejected for MVP (extra vendor, marginal gain) | Founder | Phase 2 |
