# 06 — PRD: Backend API & Database

**Doc owner:** Founder (Alpesh) · **Status:** Draft for founder approval · **Last updated:** 2026-07-12
**Siblings:** `00-GOLDEN-RULES.md` (rules this doc must obey) · `05-PRODUCT-OVERVIEW.md` (what the clients do) · `07-PRD-MOBILE-APPS.md`, `08-PRD-WEBSITE.md`, `09-PRD-OPS-DASHBOARD.md`, `19-PRD-CMS-ANALYTICS.md` (the consumers of this API — the ops dashboard and the CMS/analytics module are the two modules of the single KisanSetu Console) · `11-ARCHITECTURE.md` (system-level rationale) · `14-OPS-PLAYBOOK.md` (the human procedures these endpoints digitize).

> Golden rule #3: PLAN BEFORE CODE. This PRD supersedes the scaffold at `~/Desktop/KisanSetu-Backend` (including `migrations/001_init.sql`, which uses `mandi_price_per_kg` and has no region layer — both are corrected here). When development resumes, the scaffold is reviewed against THIS document, not the other way around.

---

## 1. Purpose

One backend serves everything: farmer app, buyer app (Android + iOS), marketing website lead form, and the internal KisanSetu Console (ops + CMS/analytics modules, one login). It is the single source of truth for prices, listings, orders, traceability, and money. It must directly serve the two metrics (Golden rule #5):

1. **Freshness** — every allocation carries the timestamp chain `harvest_ts → hub_in_ts → dispatch_ts → delivered_ts`, so median harvest→door hours is a SQL query, not an estimate.
2. **Farmer share** — the daily price feed stores both what the buyer pays (`platform_price_a/b`) and what the farmer receives (`farmer_price_a/b`), so farmer-share % is computed from records, not claimed.

Global-first, Surat-first (Golden rule #2): every table that touches money, language, time, or geography is region-scoped from day 1. Launching country #2 must be a data insert, not a migration (see `11-ARCHITECTURE.md` §7).

## 2. Users & jobs-to-be-done

| Consumer | Role claim in JWT | Jobs |
|---|---|---|
| Farmer app | `farmer` | See today's prices, post a listing, track grading/allocation status, see payout history |
| Buyer app | `buyer` | Browse graded catalog with availability, place order, track status, view traceability + invoice |
| Console — Ops module | `ops` | Set daily prices, grade listings, allocate listings→order items, update delivery status, trigger payouts, work leads, read SLA/farmer-share reports |
| Console — CMS/Analytics module | `ops` | Who sold what, who bought what, downloads & active users, trend graphs, CSV export, market targeting (where KisanSetu is live) (R7–R14 + G1–G3/M7–M10; screens C-1…C-6 in `19-PRD-CMS-ANALYTICS.md`). Same login, same JWT: any `ops` user sees both modules at MVP — per-screen permissions are a v2 non-goal |
| Website | none (public) | `POST /leads`, read nothing |
| Founder/CA | `ops` | Export reports for the weekly metrics ritual (`14-OPS-PLAYBOOK.md`) — audited CSV via R14 |

## 3. Scope (in) / Non-goals (out)

### In (MVP)
Auth (phone OTP → JWT), users + role profiles (buyers start `pending`, ops-activated), region/FPO/hub hierarchy, region settings + client bootstrap config (`GET /config`), produce catalog with pluggable translations, daily price feed, listings + grading (with one evidence photo via a storage interface) + splits, orders + server-side pricing + cart quote (`POST /orders/quote`) + two-phase allocation (evening supply-check confirm / morning binding allocation) + traceability, buyer credit enforcement, buyer quality disputes + credit notes, payments (farmer payout + buyer invoice; manual UTR entry at MVP, Razorpay UPI behind a provider interface), leads, ops report endpoints (incl. receivables aging), CMS/analytics report endpoints (R7–R14 + nightly `metrics_daily` rollup + manual `store_metrics` entry — consumed by `19-PRD-CMS-ANALYTICS.md`), market targeting (geo reference reads G1–G3 + CMS-managed serviceable cities M7–M10 + the `city_not_serviceable` order guard, §6.13 — screen C-6 in `19-PRD-CMS-ANALYTICS.md`), SLA timestamp instrumentation, batch app instrumentation (`POST /events` — **stored** to `app_events`, §6.11 S4), rate limiting, audit events, seed + migration tooling.

### NON-GOALS (explicit, with reasons)

| We will NOT build | Why not (at MVP) |
|---|---|
| **GraphQL** | Four first-party clients we control, all needing the same ~8 aggregates. REST with purpose-built endpoints is simpler to cache, rate-limit, log, and debug. GraphQL adds a resolver layer, N+1 discipline, and tooling for zero product benefit at 25 buyers. |
| **Microservices / Kubernetes** | One pilot city, one dev (founder). A modular monolith deploys in minutes and keeps every transaction in one Postgres. Split only when a module has independent scaling or team boundaries (`11-ARCHITECTURE.md` §8). |
| **ORM (Prisma/Sequelize/TypeORM)** | Schema is ~23 tables (incl. the 3 geo reference tables); the hard queries (allocation matching, SLA medians, farmer share) are exactly where ORMs get in the way. Plain SQL via `pg` with parameterized queries; migrations are numbered `.sql` files. |
| **Redis / any second datastore** | Sessions are stateless JWTs; rate limiting is in-process (single instance); queues don't exist yet. Postgres does caching-adjacent jobs (e.g. `otp_codes` table) fine at this scale. Add Redis only when we run >1 API instance AND need shared rate-limit/queue state. |
| **Event bus / message queue** | Order volume at pilot is tens/day. Side effects (WhatsApp, push) run inline with retry, or from a 1-minute cron sweep. |
| **Search engine (Elastic etc.)** | Catalog is <100 produce items; `ILIKE` is enough. |
| **File/object storage service** | Invoice PDFs are Phase 2 (post-pilot); the S3-compatible bucket is Phase 2. When added: bucket behind a `storage.js` interface; never DB blobs. **Exception (revised):** ONE grading-evidence photo path ships at MVP — the same `storage.js` interface with a **local-disk** instance at pilot (see §6.6 L6), because that photo is the evidence base for disputes (§6.12). No hosted object storage until Phase 2; only the local-disk driver. |
| **Websockets / server push** | Ops dashboard polls every 30 s; apps poll on foreground + FCM/APNs nudges later. |
| **Multi-currency conversion** | Currency is a *field* everywhere (global-first), but each region operates in exactly one currency. No FX, ever, until a region demands it. |
| **Self-service admin/permissions UI** | Roles are fixed (`farmer`,`buyer`,`ops`); ops users are inserted by migration/seed at pilot. |
| **Play Console / App Store Connect API integration** | Download counts are entered manually once a week (§6.11 S5 → `store_metrics`). Store APIs need service accounts, key management, and quota handling — for two numbers a week at pilot scale. v2 per `19-PRD-CMS-ANALYTICS.md`. |
| **Third-party analytics SaaS (Mixpanel/GA etc.)** | Thin stored rows in `app_events` (§5) + the nightly `metrics_daily` rollup answer every C-1…C-4 question. One Postgres, no data leaves our infra (DPDP posture, `18-LEGAL-COMPLIANCE.md`). |

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
  city_id     integer REFERENCES cities(id),   -- geo reference (scaffold migrations 002–004; market-targeting block below). Captured at signup (A3) — NEVER a signup blocker (§6.13)
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
  city_id       integer REFERENCES cities(id),   -- captured when the form gives one; feeds C-6 demand-from-non-serviceable-cities (§6.13)
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

-- ============ analytics (CMS module — read via R7–R14; screens in 19-PRD-CMS-ANALYTICS.md) ============
CREATE TABLE app_events (                       -- thin stored rows behind POST /events (§6.11 S4)
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     uuid REFERENCES users(id),        -- from JWT; nullable leaves room for pre-auth events later
  event_name  text NOT NULL,                    -- 'catalog_viewed', 'order_placed', ... (unknown names dropped at ingest)
  platform    text NOT NULL CHECK (platform IN ('android','ios')),
  app_version text,                             -- semver, batch-level field from the client
  region_id   uuid NOT NULL REFERENCES regions(id),
  ts          timestamptz NOT NULL,             -- client event time; server receipt time goes to logs
  props       jsonb
);
-- Retention: 180 days. The 00:30 nightly rollup (§6.6) purges older rows AFTER their aggregates
-- land in metrics_daily — raw events are ephemeral, aggregates are kept forever.

CREATE TABLE store_metrics (                    -- store-reported installs; MANUAL weekly entry at MVP (§6.11 S5)
  region_id       uuid NOT NULL REFERENCES regions(id),
  date            date NOT NULL,                -- business date in region tz
  platform        text NOT NULL CHECK (platform IN ('android','ios')),
  downloads_total int NOT NULL,                 -- lifetime installs as shown in Play Console / App Store Connect
  downloads_new   int NOT NULL DEFAULT 0,       -- new installs since the previous entry
  source          text NOT NULL DEFAULT 'manual' CHECK (source IN ('manual','api')),  -- 'api' reserved for the v2 store-API sync
  entered_by      uuid REFERENCES users(id),
  updated_at      timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (region_id, date, platform)
);
-- Downloads ≠ signups: installs live here (store-reported); signups live in users. R13/C-4 report both, never conflated.

CREATE TABLE metrics_daily (                    -- nightly rollup at 00:30 region tz (§6.6); graphs read THIS, never raw tables
  region_id          uuid NOT NULL REFERENCES regions(id),
  date               date NOT NULL,             -- business date in region tz
  currency           char(3) NOT NULL,          -- §4 money convention
  gmv                numeric(12,2) NOT NULL DEFAULT 0,  -- Σ orders.total delivered that date
  orders             int NOT NULL DEFAULT 0,            -- delivered count
  kg_moved           numeric(12,2) NOT NULL DEFAULT 0,  -- Σ order_allocations.qty on delivered orders
  dau                int NOT NULL DEFAULT 0,             -- per the canonical definitions (§6.10)
  new_farmers        int NOT NULL DEFAULT 0,
  new_buyers         int NOT NULL DEFAULT 0,
  farmer_share_pct   numeric(5,2),              -- null when nothing delivered that day (never fabricated)
  median_freshness_h numeric(6,1),              -- null when no complete timestamp chains (R2 rule: excluded, never guessed)
  computed_at        timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (region_id, date)
);

-- ============ geo reference (founder-provided dataset shape; scaffold migrations 002–004) ============
-- Column-for-column match with the countries-states-cities dataset so its dump imports as-is.
-- BY DEFAULT identity PKs accept the dataset's explicit ids; after a bulk import run
--   SELECT setval(pg_get_serial_sequence('<table>','id'), (SELECT max(id) FROM <table>));
-- region_id/subregion_id are the DATASET's continent ids — not KisanSetu's operating `regions` (§6.13).

CREATE TABLE countries (
  id              integer GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name            text NOT NULL,
  iso3            char(3),
  numeric_code    char(3),
  iso2            char(2),
  phonecode       text,
  capital         text,
  currency        text,                          -- ISO code, e.g. 'GHS'
  currency_name   text,                          -- e.g. 'Ghanaian cedi'
  currency_symbol text,                          -- e.g. 'GH₵'
  tld             text,
  native          text,
  region          text,                          -- continent name, e.g. 'Africa'
  region_id       integer,
  subregion       text,                          -- e.g. 'Western Africa'
  subregion_id    integer,
  nationality     text,
  timezones       jsonb,                         -- [{"zoneName":"Africa/Accra","gmtOffset":0,...}]
  translations    jsonb,                         -- {"kr":"가나","pt":"Gana",...}
  latitude        numeric(10,8),
  longitude       numeric(11,8),
  emoji           text,
  "emojiU"        text,
  created_at      timestamptz NOT NULL DEFAULT now(),
  updated_at      timestamptz NOT NULL DEFAULT now(),
  flag            smallint NOT NULL DEFAULT 1,
  "wikiDataId"    text,
  length          integer                        -- present in the founder's reference table
);
CREATE UNIQUE INDEX idx_countries_iso2 ON countries (iso2);
CREATE INDEX idx_countries_iso3   ON countries (iso3);
CREATE INDEX idx_countries_region ON countries (region_id);

CREATE TABLE states (
  id           integer GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name         text NOT NULL,
  country_id   integer NOT NULL REFERENCES countries(id),
  country_code char(2) NOT NULL,
  fips_code    text,
  iso2         text,
  type         text,                             -- 'province' / 'state' / NULL in parts of the dataset
  latitude     numeric(10,8),
  longitude    numeric(11,8),
  created_at   timestamptz NOT NULL DEFAULT now(),
  updated_at   timestamptz NOT NULL DEFAULT now(),
  flag         smallint NOT NULL DEFAULT 1,
  "wikiDataId" text
);
CREATE INDEX idx_states_country ON states (country_id);
CREATE INDEX idx_states_name    ON states (name);

CREATE TABLE cities (
  id           integer GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name         text NOT NULL,
  state_id     integer NOT NULL REFERENCES states(id),
  state_code   text NOT NULL,
  country_id   integer NOT NULL REFERENCES countries(id),
  country_code char(2) NOT NULL,
  latitude     numeric(10,8) NOT NULL,
  longitude    numeric(11,8) NOT NULL,
  created_at   timestamptz NOT NULL DEFAULT now(),
  updated_at   timestamptz NOT NULL DEFAULT now(),
  flag         smallint NOT NULL DEFAULT 1,
  "wikiDataId" text
);
CREATE INDEX idx_cities_state   ON cities (state_id);
CREATE INDEX idx_cities_country ON cities (country_id);
CREATE INDEX idx_cities_name    ON cities (name);

-- ============ market targeting (CMS module — screen C-6 in 19-PRD-CMS-ANALYTICS.md; endpoints §6.13) ============
-- The two tables + function below are scaffold migration 005_market_targeting.sql — like 002–004,
-- the scaffold is reviewed against THIS document when development resumes (Golden Rule 3).

CREATE TABLE market_settings (                  -- singleton: HOW serviceability is decided
  id             smallint PRIMARY KEY DEFAULT 1 CHECK (id = 1),
  targeting_mode text NOT NULL DEFAULT 'targeted' CHECK (targeting_mode IN ('targeted','auto')),
    -- 'targeted': only active target_cities rows are serviceable (pilot default; Surat seeded)
    -- 'auto':     EVERY city in the geo tables is serviceable, worldwide (founder-level switch, C-6)
  updated_by     uuid REFERENCES users(id),
  updated_at     timestamptz NOT NULL DEFAULT now()
);
-- Seeded by the migration: one row, targeting_mode='targeted'. Surat's target_cities row comes
-- from seed tooling — the pilot default is targeted + Surat active, with zero code involved.

CREATE TABLE target_cities (                    -- WHERE we are live when targeting_mode='targeted'
  id             integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  city_id        integer NOT NULL UNIQUE REFERENCES cities(id),
  active         boolean NOT NULL DEFAULT true,
  activated_at   timestamptz NOT NULL DEFAULT now(),
  deactivated_at timestamptz,                   -- set on soft-deactivate (M10); rows are never deleted
  activated_by   uuid REFERENCES users(id),
  notes          text
);
CREATE INDEX idx_target_cities_active ON target_cities (city_id) WHERE active;

-- DB truth function: ALL enforcement reads this one function — the §6.7 order guard and the
-- GET /config `serviceable` flag (§6.11). Flipping the mode changes its answer instantly; no restart.
CREATE FUNCTION city_is_serviceable(p_city_id integer) RETURNS boolean
LANGUAGE sql STABLE AS $$
  SELECT CASE
    WHEN (SELECT targeting_mode FROM market_settings WHERE id = 1) = 'auto'
      THEN EXISTS (SELECT 1 FROM cities WHERE id = p_city_id)
    ELSE EXISTS (SELECT 1 FROM target_cities WHERE city_id = p_city_id AND active)
  END;
$$;
-- Demand capture: `users.city_id` and `leads.city_id` (above) record WHERE demand comes from.
-- Signups and leads are ALWAYS accepted regardless of serviceability — only ORDERS are guarded
-- (§6.7 `city_not_serviceable`) — so C-6 can show "demand from non-serviceable cities" (§6.13).

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
CREATE INDEX idx_app_events_ts         ON app_events (ts);                 -- rollup + retention purge scans
CREATE INDEX idx_app_events_user_ts    ON app_events (user_id, ts);        -- DAU/WAU/MAU distinct-user counts (R13)
CREATE INDEX idx_app_events_name_ts    ON app_events (event_name, ts);     -- per-event trend queries
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

- **Error codes** (HTTP → `code`): `400 validation_failed` · `401 unauthorized` / `otp_invalid` / `otp_expired` / `token_expired` · `403 forbidden_role` / `user_blocked` / `account_pending` / `credit_hold` / `city_not_serviceable` (buyer's city not live — §6.13) · `404 not_found` · `409 conflict_state` (illegal lifecycle transition) / `insufficient_stock` / `duplicate` / `past_cutoff` / `claim_window_closed` · `422 price_unavailable` / `below_min_order` · `429 rate_limited` (with `Retry-After` header) · `500 internal` (message is generic; details go to logs only).
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

**Nightly metrics rollup (analytics sweep):** a second nightly cron runs at **00:30 region tz** — after the 00:15 expiry sweep, so expiries are settled before yesterday is measured — and upserts yesterday's `metrics_daily` row per region (§5): `gmv`, `orders`, `kg_moved`, `dau`, `new_farmers`, `new_buyers`, `farmer_share_pct`, `median_freshness_h`, each computed per the canonical definitions in §6.10. Idempotent: keyed on `(region_id, date)`, a re-run recomputes the row rather than duplicating it. The same job then purges `app_events` rows older than **180 days** — aggregates outlive raw events. Every CMS trend graph (`19-PRD-CMS-ANALYTICS.md` C-1…C-4) reads `metrics_daily` via R8; nothing queries the raw `app_events`/`orders` tables for time series.

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
- **City not serviceable** — `city_is_serviceable(users.city_id)` for the ordering buyer returns false (§6.13: targeted mode with no active `target_cities` row for the buyer's city) → `403 { "error": { "code": "city_not_serviceable" } }`. Orders are the ONLY thing market targeting blocks — signups and leads are always accepted, and `POST /orders/quote` still prices (§6.13). In the normal path the buyer never reaches this guard: `GET /config?city_id=` returns `serviceable:false` (§6.11) and the buyer catalog renders the "हम जल्द आ रहे हैं / Coming soon to your city" state instead of a checkout. A buyer row with NULL `city_id` (pre-capture) is treated as non-serviceable — visible, never silent — until ops backfills the city via U7.

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

### 6.10 Ops reports, SLA instrumentation & CMS analytics

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

**Canonical metric definitions (stated once, here; every report in this section, the nightly rollup (§6.6), and `19-PRD-CMS-ANALYTICS.md` reuse these — never redefine):**

- **DAU** — distinct `user_id` with ≥1 `app_events` row on that calendar day, day boundary in **region tz**. **WAU / MAU** — 7-day / 30-day rolling distinct `user_id`.
- **Active farmer** (business sense) — ≥1 **graded** listing in the period. **Active buyer** — ≥1 **delivered** order in the period. App-activity (DAU) and business-activity are different numbers; R7/C-1 shows both, labelled.
- **Repeat-rate** — buyers with ≥2 delivered orders in the window ÷ active buyers (matches `09-PRD-OPS-DASHBOARD.md` §5.9).
- **Downloads** — store-reported installs from `store_metrics` (§5). **Signups** — rows in `users`. Two distinct series; R13 (and screen C-4) report both and never conflate them.
- **GMV** — Σ `orders.total` of orders delivered that day. **kg moved** — Σ `order_allocations.qty` on delivered orders. **farmer_share_pct** — per R3 (Σ farmer payouts ÷ Σ buyer produce subtotal). **median_freshness_h** — per R2 (incomplete chains excluded, never interpolated).

**CMS / analytics reports (Console CMS module — screens C-1…C-5 in `19-PRD-CMS-ANALYTICS.md`; the module's sixth screen, C-6 Markets, rides the §6.13 geo/markets surface, not the R-series).** The R-series continues (R1–R6 above are unchanged; no renumbering needed). All require the `ops` role; list endpoints use the §6.1 envelope and error shape. Privacy: these endpoints expose per-farmer/per-buyer operational data to internal staff only — covered by the DPDP service purpose (`18-LEGAL-COMPLIANCE.md` §10); the only path for data to leave the console is R14's audited CSV.

| # | Method & path | Role | Purpose |
|---|---|---|---|
| R7 | `GET /reports/overview` | ops | C-1 snapshot: today's GMV, orders, kg moved, active farmers/buyers, DAU, new signups, open disputes, pending payouts |
| R8 | `GET /reports/metrics/daily?metrics=gmv,orders,dau,farmer_share&from=&to=` | ops | Time series for the trend graphs — reads `metrics_daily` only, never raw tables |
| R9 | `GET /reports/farmers?sort=&period=&limit=&offset=` | ops | C-2 sortable farmer list (name, village, hub, listings, kg A/B, payouts, farmer-share %, last active); paginated |
| R10 | `GET /reports/farmers/:id` | ops | C-2 drill-down: full sell history, monthly kg/earnings trend, data-quality flags |
| R11 | `GET /reports/buyers?sort=&period=&limit=&offset=` | ops | C-3 sortable buyer list (business, type, orders, GMV, AOV, repeat-rate, credit status, disputes); paginated |
| R12 | `GET /reports/buyers/:id` | ops | C-3 drill-down: full purchase history, monthly GMV trend, credit/receivables panel, dispute history |
| R13 | `GET /reports/app-usage?from=&to=` | ops | C-4: DAU/WAU/MAU, app-version adoption vs `min_supported_version`, platform split, downloads vs signups (from `app_events` + `store_metrics`) |
| R14 | `GET /reports/export?entity=farmers\|buyers\|metrics&format=csv&from=&to=` | ops | Streams CSV of the R9/R11 lists or the R8 time series; **every call writes `audit_events` action `report_exported`** (Phase 2 / CMS v1.1) |

```json
// R7: GET /reports/overview → 200 — today in region tz; "active" per the canonical definitions above
{ "date": "2026-11-12", "region": "IN-GJ-SURAT", "currency": "INR",
  "gmv": 41200.00, "orders": 18, "kg_moved": 1240.5,
  "active_farmers_today": 22, "active_buyers_today": 14, "dau": 61, "new_signups": 3,
  "open_disputes": 2, "pending_payouts": { "count": 5, "amount": 9350.00 } }
```
R7 is the only CMS read allowed to touch raw tables (today's rollup doesn't exist yet); everything historical goes through `metrics_daily`.

```json
// R8: GET /reports/metrics/daily?metrics=gmv,dau,farmer_share&from=2026-11-05&to=2026-11-07 → 200
{ "from": "2026-11-05", "to": "2026-11-07", "region": "IN-GJ-SURAT", "currency": "INR",
  "series": [ { "date": "2026-11-05", "gmv": 32100.00, "dau": 48, "farmer_share": 67.4 },
              { "date": "2026-11-06", "gmv": 35400.00, "dau": 52, "farmer_share": 68.1 },
              { "date": "2026-11-07", "gmv": 41250.00, "dau": 57, "farmer_share": 66.9 } ] }
```
R8 rules: `metrics` ⊆ {`gmv`,`orders`,`kg_moved`,`dau`,`new_farmers`,`new_buyers`,`farmer_share`,`median_freshness_h`} — unknown name → `400 validation_failed`; range capped at 366 days; a date with no rollup row returns `null` values (never a live recompute). This endpoint feeds the hand-rolled SVG chart helper shared by C-1…C-4 — no chart library, no CDN (`19-PRD-CMS-ANALYTICS.md`).

```json
// R10: GET /reports/farmers/:id → 200 — C-2 drill-down (internal console; §6.7's buyer-facing trace PII rule does not apply here)
{ "farmer": { "id": "…", "name": "Ramesh Patel", "village": "Kathor", "hub": "Kamrej Hub",
    "last_active_at": "2026-11-11T06:40:12Z" },
  "totals": { "listings": 34, "kg_graded_a": 1890, "kg_graded_b": 410,
    "payouts_total": 46530.00, "currency": "INR", "farmer_share_pct": 68.2 },
  "sell_history": [ { "listing_id": "…", "date": "2026-11-07", "produce_code": "tomato",
      "posted_qty": 100, "graded_qty": 98, "grade_split": { "A": 70, "B": 28 },
      "payout": { "amount": 1918.00, "status": "paid", "utr": "AXISN0231…" } } ],
  "monthly_trend": [ { "month": "2026-10", "kg": 620, "earnings": 12400.00 } ],
  "flags": { "no_shows": 1, "disputes_traced": 0 } }
```
`no_shows` counts listings swept `expired_no_show` (§6.6); `disputes_traced` counts disputes whose allocation traces to this farmer's listings (§6.12). R12 mirrors this shape for buyers: `purchase_history` rows (date, items by crop+grade, value, status, `invoice_no`), monthly GMV trend, the R5 receivables panel scoped to the buyer, and dispute history.

```json
// R13: GET /reports/app-usage?from=2026-11-01&to=2026-11-07 → 200
// Usage computed from app_events; installs from store_metrics — downloads ≠ signups, both reported.
{ "from": "2026-11-01", "to": "2026-11-07", "region": "IN-GJ-SURAT",
  "dau": [ { "date": "2026-11-06", "count": 52 }, { "date": "2026-11-07", "count": 57 } ],
  "wau": 118, "mau": 203,
  "platform_split": { "android": 0.83, "ios": 0.17 },
  "version_adoption": [ { "platform": "android", "app_version": "1.2.0", "users": 96, "below_min_supported": false },
                        { "platform": "android", "app_version": "1.0.3", "users": 7,  "below_min_supported": true } ],
  "downloads": { "android": { "downloads_total": 410, "new_in_range": 37 },
                 "ios":     { "downloads_total": 88,  "new_in_range": 9 },
                 "source": "manual", "last_entered_on": "2026-11-04" },
  "signups_by_role": [ { "date": "2026-11-07", "farmer": 2, "buyer": 1 } ] }
```
`below_min_supported` compares against `region_settings.min_supported_version_*` (§5) — the same floor S3 serves to clients. W1/W4 retention cohorts are a v1.1 addition to this endpoint (`19-PRD-CMS-ANALYTICS.md` phasing).

R14 streams `text/csv` (`Content-Disposition: attachment`), `format=csv` only at MVP; the `audit_events` row carries `action='report_exported'` and `detail = { entity, from, to, row_count }`, so every export is attributable (DPDP posture).

### 6.11 System, config & instrumentation

| # | Method & path | Auth | Purpose |
|---|---|---|---|
| S1 | `GET /health` | public | `{"status":"ok","db":"ok"}`, 503 if DB ping fails |
| S2 | `GET /version` | public | `{"version":"1.4.2","migration":"007"}` |
| S3 | `GET /config` | public (region-scoped) | Client bootstrap: commerce constants + force-update floors |
| S4 | `POST /events` | any authed | Batch app-instrumentation transport — **stored** to `app_events` (amended, see below) |
| S5 | `PUT /store-metrics/:date` | ops | Manual weekly entry of Play Console / App Store Connect install counts → `store_metrics` |

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

**Market-targeting extension (§6.13):** S3 additionally accepts `?city_id=` (the client's captured city) and the response then carries `"serviceable": <bool>`, straight from `city_is_serviceable()` — this is the **single public serviceability check**; there is deliberately no separate `GET /markets/check` endpoint (one bootstrap call stays one bootstrap call). Buyer app behavior on `serviceable:false`: the catalog shows the "हम जल्द आ रहे हैं / Coming soon to your city" state — browse, signup, and profile still work; only ordering is off (§6.7 guard is the server-side backstop). Without `?city_id=` the key is omitted.

```json
// GET /config?region=IN-GJ-SURAT&city_id=57996 → 200 — same body as above, plus:
{ "…": "…", "serviceable": true }
```

**S4 `POST /events`** — fire-and-forget batch transport for app analytics/instrumentation. **AMENDED for the CMS module (supersedes the earlier logs-only behavior): accepted events are now STORED**, one thin row each in `app_events` (§5), *and* still emitted as structured `pino` lines (`event:` field, §11). The server tags `user_id`/`region_id` from the JWT, takes batch-level `platform` + `app_version` from the body, and skips-and-counts unknown `name`s. Still best-effort: never blocks UX, returns `202` even if some rows are dropped. Stored events are what make DAU/WAU/MAU and screen C-4 possible (R13); retention is **180 days**, enforced by the 00:30 nightly rollup (§6.6), which folds aggregates into `metrics_daily` before purging.

```json
// POST /events
{ "platform": "android", "app_version": "1.2.0",
  "events": [ { "name": "catalog_viewed", "ts": "2026-11-07T09:12:01Z", "props": { "produce_count": 24 } },
              { "name": "cart_quote_requested", "ts": "2026-11-07T09:13:40Z", "props": { "subtotal": 990 } } ] }
// 202 { "accepted": 2, "dropped": 0 }
```
Rate-limited per §8 (rolled into the authed global bucket, with a higher batch allowance). The earlier "events are logs, not rows" rule is **withdrawn**: `app_events` is a table now — but it is still plain Postgres, so the §3 non-goals stand (no event bus, no second datastore, no analytics SaaS).

**S5 `PUT /store-metrics/:date`** — manual weekly entry of install counts read off Play Console / App Store Connect (store APIs are a v2 integration — §3 non-goals). Upserts `store_metrics` (§5) for the ops user's region with `source='manual'`, `entered_by` from the JWT; writes `audit_events` action `store_metrics_entered`. This is the C-4 downloads data source at MVP.

```json
// S5: PUT /store-metrics/2026-11-04   (ops — weekly ritual, 14-OPS-PLAYBOOK.md)
{ "rows": [ { "platform": "android", "downloads_total": 410, "downloads_new": 37 },
            { "platform": "ios",     "downloads_total": 88,  "downloads_new": 9 } ] }
// 200 { "upserted": 2, "source": "manual" }
```

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

### 6.13 Geo reference & market targeting

Launch geography is **CMS-managed configuration, not code** (Golden rule #2 — the codebase never hardcodes a city). The `market_settings` singleton (§5) holds the mode: **`targeted`** (default) — only cities explicitly activated in `target_cities` are serviceable; **`auto`** — every city in the geo tables is serviceable, worldwide. One DB truth function, `city_is_serviceable(city_id)` (§5), answers "are we live here?" and is the only thing enforcement reads — flipping the mode changes every answer instantly, no restart, no deploy. Pilot default: **targeted with Surat active (seeded)**; retargeting to, say, Ahmedabad is a founder action on screen C-6 (`19-PRD-CMS-ANALYTICS.md`), not a release. Auto/worldwide is a founder-level, post-pilot action.

**Markets vs regions (the two concepts never blur):** `target_cities` answers *"are we live here?"* — a binary serviceability fact. `regions` / `region_settings` (§5) remain the **operating unit** — prices, cutoff, delivery window, currency, tax scheme. Activating a city does NOT create a region: provisioning a region (+ hub, FPO, price feed) for a newly activated city stays a **manual ops step at MVP** (`14-OPS-PLAYBOOK.md`); a city can be serviceable-but-unprovisioned, and C-6 flags that state. v2: auto-provision a region from country defaults when a city is activated in auto mode.

**Signup/lead capture:** `POST /auth/signup` (A3) now carries `city_id` (the apps' Country → State → City picker, fed by G1–G3 — the A2 `signup_token` is accepted on the geo reads so the picker works pre-account); `POST /leads` (D1) accepts an optional `city_id`. Neither is EVER rejected for a non-serviceable city — that demand is exactly what the C-6 demand-signals strip surfaces. Only orders are blocked (§6.7, `403 city_not_serviceable`); the public check is the `GET /config` `serviceable` flag (§6.11).

| # | Method & path | Role | Purpose |
|---|---|---|---|
| G1 | `GET /geo/countries` | any authed (A2 `signup_token` accepted) | Country pick-list for the cascade pickers (C-6 + signup city capture) |
| G2 | `GET /geo/states?country_id=` | any authed (A2 `signup_token` accepted) | States of a country |
| G3 | `GET /geo/cities?state_id=&search=&limit=&offset=` | any authed (A2 `signup_token` accepted) | Cities of a state, search-as-you-type (`search` = name prefix, `ILIKE`) |
| M7 | `GET /markets` | ops | Everything C-6 renders: mode + active cities (incl. `open_orders`) + demand signals (v1.1) |
| M8 | `PUT /markets/mode` | ops | Switch `targeting_mode` (`targeted`\|`auto`); writes `audit_events` `targeting_mode_changed` |
| M9 | `POST /markets/cities` | ops | Activate a city `{city_id, notes?}`; re-activates a deactivated row; `409 duplicate` if already active; writes `city_activated` |
| M10 | `DELETE /markets/cities/:city_id` | ops | **Soft** deactivate: `active=false` + `deactivated_at`; writes `city_deactivated`; NEVER touches existing orders |

The M-series continues from §6.8 (M1–M6 are payments — no renumbering, same precedent as the R-series continuing R7–R14). G1–G3 use the §6.1 list envelope; the geo reads raise the `limit` max to 250 so a country picker fills in one call. G3 requires `state_id`; without `search` it pages the whole state alphabetically.

```json
// G3: GET /geo/cities?state_id=1064&search=sur&limit=20&offset=0 → 200
{ "data": [ { "id": 57996, "name": "Surat", "state_id": 1064, "state_code": "GJ",
              "country_id": 101, "country_code": "IN",
              "latitude": 21.19594000, "longitude": 72.83023000 },
            { "id": 58009, "name": "Surendranagar", "state_id": 1064, "state_code": "GJ",
              "country_id": 101, "country_code": "IN",
              "latitude": 22.70000000, "longitude": 71.68333000 } ],
  "total": 2, "limit": 20, "offset": 0 }
```

```json
// M7: GET /markets → 200 — mode, the active list, and where blocked demand is coming from
{ "targeting_mode": "targeted",
  "active_cities": [ { "city_id": 57996, "name": "Surat", "state": "Gujarat", "country": "India",
                       "activated_at": "2026-07-12T09:00:00Z", "activated_by": "…",
                       "notes": "pilot seed", "open_orders": 14, "region_provisioned": true } ],
  "demand_signals": [ { "city_id": 57588, "name": "Ahmedabad", "state": "Gujarat", "country": "India",
                        "leads_30d": 6, "signups_30d": 3 },
                      { "city_id": 58164, "name": "Vadodara", "state": "Gujarat", "country": "India",
                        "leads_30d": 2, "signups_30d": 1 } ] }
```

M7 rules: `open_orders` = orders not yet `delivered`/`cancelled` whose buyer's `city_id` is this city — C-6 uses it for the deactivation warning; `region_provisioned` = whether an active region covers the city (the manual ops step above); `demand_signals` (**v1.1**) = leads + signups of the last 30 days grouped by captured `city_id` where `city_is_serviceable()` is false, top 10 by combined count — read-only, so the founder sees where to expand next. In `auto` mode `active_cities` still returns the (now non-binding) target list and C-6 renders it read-only. M8 body is `{ "targeting_mode": "auto" }`; the typed-"AUTO" confirmation is a **C-6 client guard** (the API accepts any authorized M8 call and audits it). All M-series writes are audited: `targeting_mode_changed` / `city_activated` / `city_deactivated` — a C-6 action with no audit row is a test failure (§9).

**Scaffold note:** the geo reference tables ship as scaffold migrations `002_countries.sql`–`004_cities.sql` (column-compatible with the countries-states-cities dataset); `005_market_targeting.sql` (`market_settings` + `target_cities` + `city_is_serviceable()`) sits beside them — and, like everything in the scaffold, is reviewed against THIS document when development resumes (Golden Rule 3).

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
- **CMS analytics**: an event accepted by S4 appears in R13's counts for its `ts` date **within the same day** (today is served from raw `app_events`; `metrics_daily` catches up at 00:30); the nightly rollup produces **exactly one `metrics_daily` row per active region per day**, and a re-run upserts rather than duplicates; every R14 CSV export writes an `audit_events` row (`action='report_exported'`, detail = entity + date range + row count) — an export with no audit row is a test failure; R9 and R11 return the §6.1 list envelope and honor `limit`/`offset` (default 20, max 100).
- **Market targeting**: `POST /orders` from a buyer whose city is not serviceable returns `403 city_not_serviceable` (and `GET /config?city_id=` returns the same verdict the guard enforces — one function, §6.13); flipping `market_settings.targeting_mode` to `auto` via M8 makes ANY geo-table city serviceable immediately — no restart, no deploy (the guard calls `city_is_serviceable()` per request); deactivating a city (M10) is soft — existing orders are untouched and proceed to delivery while new orders fail; every C-6 action writes exactly one `audit_events` row (`targeting_mode_changed` / `city_activated` / `city_deactivated`) — an action with no audit row is a test failure; a fresh seed comes up targeted with Surat active.
- **Global-first**: inserting a second region row (e.g. `KE-NAIROBI`, KES, `Africa/Nairobi`, `tax_scheme='none'`, `languages={en,sw}`) and re-running the full API test suite scoped to it passes with **zero migrations** — this is a CI test, not a promise.
- **Non-functional**: p95 < 300 ms for all GETs at pilot data volumes (10k listings, 5k orders) on a 2-vCPU VM; smoke suite (supertest) covers the 5 core flows end-to-end and runs in CI on every push.

## 10. Phased rollout

| Phase | Contents |
|---|---|
| **MVP (pilot)** | Everything in §6 with `provider='manual'` payouts, dev/MSG91 OTP, **one local-disk grading/dispute evidence photo via `storage.js`**, no invoice PDFs. WhatsApp/push nudges optional (manual broadcast acceptable — `14-OPS-PLAYBOOK.md`). **CMS module MVP**: stored `app_events` (S4), 00:30 `metrics_daily` rollup, R7–R12, R13 basics (DAU graph, platform split), manual weekly `store_metrics` entry (S5). **Market targeting MVP** (§6.13): geo reads G1–G3, markets M7–M10, `GET /config` `serviceable` flag, `city_not_serviceable` order guard, seeded targeted+Surat — launch geography depends on it (C-6 core in `19-PRD-CMS-ANALYTICS.md`). |
| **Phase 2 (live pilot hardening)** | Razorpay UPI payouts (M5/M6), WhatsApp Business templates (order confirmed, out for delivery, payout done, daily price broadcast), FCM/APNs push, invoice PDF (`GET /orders/:id/invoice.pdf`), **swap `storage.js` from local-disk to the S3-compatible bucket** (grading/dispute photos migrate transparently) + buyer-facing listing photo gallery. **CMS v1.1**: R14 audited CSV export (moved up from Phase 3), WAU/MAU graph polish, W1/W4 retention cohorts on R13, C-6 demand-signals strip (`demand_signals` on M7, §6.13). |
| **Phase 3 (scale-up)** | Buyer subscription flags + priority allocation, standing orders (repeat weekly), FPO-SaaS endpoints (hub-scoped ops accounts), managed Postgres + PITR. **CMS v2**: Play/App Store API auto-sync into `store_metrics` (retires S5 manual entry, `source='api'`), per-screen console permissions, alerting, auto-provision a region from country defaults when a city is activated in auto mode (§6.13). |

CMS phase mapping: the phases named **MVP / v1.1 / v2** in `19-PRD-CMS-ANALYTICS.md` land in this table's **MVP / Phase 2 / Phase 3** rows respectively.

## 11. Metrics & instrumentation

Emitted as structured `pino` log events (`event:` field) and derivable from tables: `otp_requested/verified`, `buyer_activated`, `listing_posted/graded/split/expired`, `order_placed/confirmed/dispatched/delivered/cancelled`, `allocation_created/deleted`, `payout_created/paid/failed`, `invoice_created`, `credit_note_issued`, `dispute_created/resolved`, `lead_created`, `price_published`. App-side events arrive via `POST /events` (§6.11), are **stored in `app_events`** (§5), and are logged with the same `event:` shape; the 00:30 nightly rollup (§6.6) folds them into `metrics_daily`, which the Console CMS module reads through R7–R14 (§6.10). Weekly KPI SQL pack (fulfilment %, median freshness h, farmer share %, buyer repeat rate, GMV, take rate realized, dispute rate = M7, receivables aging) versioned in-repo at `reports/weekly.sql`. Request logs carry `request_id`, `user_id`, `role`, latency ms.

## 12. Open questions (owner + deadline; decisions taken provisionally so build is unblocked)

| # | Question | Provisional decision in this PRD | Owner | Deadline |
|---|---|---|---|---|
| Q1 | Farmer payout timing: on grading (before produce sells) vs on pickup/dispatch? | Payout row created at grading; ops triggers payment at pickup per `14-OPS-PLAYBOOK.md` SOP | Founder | Phase 0 exit (FPO MoU) |
| Q2 | Does the buyer pay COD/credit terms/prepaid? | Invoice on delivery, net-7 collection recorded via M4 `buyer_invoice` mark-paid; no payment gateway on buyer side at MVP | Founder | Phase 0 buyer interviews |
| Q3 | India tax module: is delivery/handling fee GST-liable when bundled with exempt fresh produce? | v1 computes `tax_amount = 0`; module boundary exists so changing it is one file | Founder + CA | Before first real invoice |
| Q4 | Single grade pair (A/B) globally, or per-region grade sets? | A/B fixed enum at MVP; revisit only if a region demands it (schema change contained to CHECK constraints) | Founder | Country #2 onboarding |
| Q5 | OTP SMS provider for non-WhatsApp farmers vs missed-call auth | SMS OTP via MSG91; missed-call auth rejected for MVP (extra vendor, marginal gain) | Founder | Phase 2 |
