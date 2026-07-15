# 21 — AI Execution Playbook (how the build actually runs)

**Doc owner:** Founder (Alpesh) · **Status:** Draft for founder approval · **Last updated:** 2026-07-14
**Siblings:** [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) (build order & QA — this doc is its AI-tooling companion) · [20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) (canonical code patterns) · [11-ARCHITECTURE.md](11-ARCHITECTURE.md) (system shape) · [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md)

**What this is:** the operating manual for AI-assisted development. KisanSetu is built by a solo founder working with Claude Code; this doc fixes *in advance* what will and won't be built, in what order, which bugs we expect and how we catch them, and exactly which AI tools — skills, MCP servers, hooks, agents — do which job and which jobs they are forbidden from doing. Like [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md), it activates only after PRD approval (Golden Rule #3 — development is currently **PAUSED**).

---

## 1. What WILL be built / what will NOT

### 1.1 Will be built (the whole surface, per the PRDs)

| Surface | Repo | Stack (fixed) | PRD |
|---|---|---|---|
| Backend API + Console (ops + CMS modules) | `KisanSetu-Backend` | **TypeScript + Express + Drizzle ORM `v1.0.0-rc.4` + Postgres 16**; Console = static vanilla-JS served at `/ops` | [06-PRD-BACKEND.md](06-PRD-BACKEND.md), [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md), [19-PRD-CMS-ANALYTICS.md](19-PRD-CMS-ANALYTICS.md) |
| Marketing website | `KisanSetu-Website` | Vanilla JS, no build step (intentional — not a stack inconsistency) | [08-PRD-WEBSITE.md](08-PRD-WEBSITE.md) |
| Android app (dual-role) | `KisanSetu-Android` | Kotlin + Jetpack Compose + Material3, Fancall MVVM layers below the View | [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md), [20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §2 |
| iOS app (dual-role) | `KisanSetu-iOS` | Swift 5 + SwiftUI + XcodeGen, Fancall MVVM layers below the View | [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md), [20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §3 |
| Integrations (Phase 2 of [06-PRD-BACKEND.md](06-PRD-BACKEND.md) §10) | backend | **Razorpay (fixed PSP — §10 of this doc)**, MSG91 SMS OTP, WhatsApp Business API, FCM/APNs | [06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6.8, [11-ARCHITECTURE.md](11-ARCHITECTURE.md) §1 |

### 1.2 Will NOT be built (binding non-goals, consolidated)

Product non-goals (each PRD's Scope/NON-GOALS section wins on detail):
- **No microservices, no Kubernetes, no Redis, no message queue** at MVP — modular monolith until [11-ARCHITECTURE.md](11-ARCHITECTURE.md) §2 exit triggers fire.
- **No React/framework** for website or Console (vanilla JS decided — [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) Q1); **no webviews** for core mobile flows.
- **No in-app buyer payment collection at MVP** — buyers pay on invoice ([07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) non-goals). When it arrives, it is Razorpay (§10) — but it does not arrive before the trade is validated.
- **No B2C features** — no consumer catalog, referral loops, gamification, engagement pushes (Golden Rule #6).
- **No second city/region features** — region #2 is configuration, not code (Golden Rule #2); no FX, no cross-region anything.
- **No public store launch at MVP** — internal/closed tracks + TestFlight only ([12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) §8).
- **No ML/AI product features at MVP** — no price prediction, no photo-grading models, no chatbots. Grading is human + evidence photo.

AI-execution non-goals (what Claude/agents will NOT do — hard boundaries, restated with owners in §7.3):
- No PRD approvals, golden-rule changes, pricing decisions, or go/no-go calls — founder only.
- No production deploys, prod secrets handling, or live payment-key operations by agents.
- No outward communication (WhatsApp broadcasts, buyer/farmer messages, store submissions) initiated by agents.
- No code merged to `main` without the founder-reviewed PR ritual of [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) §5 — AI review supplements, never replaces, the founder's own review on money paths.

## 2. Build order — what happens one-by-one

The dependency-driven order from [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) §2/§9 is the schedule; this table adds the *why-this-before-that* and the AI-execution note per step:

| # | Weeks | What | Depends on | AI-execution note |
|---|---|---|---|---|
| 1 | W1 | Scaffold gap-review vs PRDs; repos + CI up; **all third-party applications filed** (Play, Apple, **Razorpay KYC**, WABA, SMS DLT) | PRD approval | `Explore` agents map each scaffold vs its PRD → keep/rewrite verdict logged in [15-TASKS-BACKLOG.md](15-TASKS-BACKLOG.md) |
| 2 | W1 | Website v1 live (leads → stub) | nothing (that's the point) | frontend-design skill + Playwright checks; founder sells from week 1 |
| 3 | W1–3 | Backend: auth/OTP → catalog+price_feed → listings → orders+allocations → payments stub → leads → `app_events` | migrations green | TDD skill per module; smoke tests 1–5 built alongside, not after |
| 4 | W3 | **API contract frozen** | smoke suite green | Breaking changes now require PRD amendment first (Brain is source of truth) |
| 5 | W3–4 | Console ops module (prices, grading, allocation, delivery, payouts) | frozen API | Gate A: business can run on ops + WhatsApp alone |
| 6 | W4–5 | Console CMS module (C-1..C-6) | ops module, `app_events` stored | dataviz skill for graphs; same vanilla-JS app |
| 7 | W3–8 | Mobile, in cross-platform units (shells → OTP login → farmer Home/New Listing/My Listings/Payments → buyer Catalog/Cart/Orders/Detail+trace) | frozen API, design tokens | One unit = Android + iOS + strings + events, never platform-sequential (Golden Rule #1); parity checklist per PR |
| 8 | W8 | Code freeze for integrations | buyer flow complete | — |
| 9 | W8–9 | Integrations: **Razorpay payouts (M5/M6)**, real SMS OTP, FCM/APNs, WhatsApp templates | third-party approvals from W1 | test keys → live keys only after KYC clears; webhook signature tests mandatory |
| 10 | W9–10 | Hardening, field test #2, pilot dry run (5 real orders) | everything | ultracode multi-agent review sweep over each repo before release-candidate tags (§8.4) |

If W8 slips >1 week: **cut scope, not parity** — WhatsApp broadcast and invoice PDFs drop first ([12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) §9).

## 3. Expected bugs — क्या bug होगा (predicted failure modes, and the net that catches each)

Honest prediction beats surprise. These are the bug classes we *expect*, by layer — each with its catch-net. Every one of these becomes a standing test or hook, not a hope.

### 3.1 Backend
| Predicted bug | Why it will happen | Catch-net |
|---|---|---|
| Money arithmetic drift (float vs `numeric`, rounding on splits/credit notes) | JS number semantics; 70/30 grade splits | Drizzle `numeric` columns everywhere money flows; smoke test 3 hand-checks totals; payout = `graded_qty × price` asserted in tests |
| Business-date/timezone bugs (price_feed date, listing expiry sweep at region tz) | UTC storage + region-tz "business day" is the classic trap | Unit tests pinned to `Asia/Kolkata` AND a second tz (the `KE-NAIROBI` CI region test, [11-ARCHITECTURE.md](11-ARCHITECTURE.md) §6) |
| Allocation race / double-allocation | Two ops actions on the same listing | Compare-and-set SQL transitions, 409 on violation ([11-ARCHITECTURE.md](11-ARCHITECTURE.md) §10.4); concurrency test in smoke suite |
| Idempotency misses (double order, double payout) | Retries on flaky hub connectivity | `Idempotency-Key` on `POST /orders`/payments ([06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6.1); "one payout per listing → 409" test |
| Webhook forgery / replay (Razorpay) | Public endpoint | Signature verification + unverified→401 test + idempotent event handling (§10.2) |
| Drizzle RC breakage on upgrade | Pinned `1.0.0-rc.4` → GA bump | Pin exact version; bump in a dedicated PR with full smoke suite; re-pin at GA ([20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §1.1) |
| Migration drift between environments | Solo dev, append-only SQL | `drizzle-kit` migrations run in CI on fresh Postgres every PR; never edit a merged migration (hook, §6) |
| AES envelope mismatch (key/IV/padding) between backend and apps | Three implementations of one crypto scheme | Byte-compatibility fixture test: one ciphertext committed as a fixture, all three platforms must decrypt it ([20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §5) |

### 3.2 Mobile (both platforms — parity means bugs must be fixed twice or prevented once)
| Predicted bug | Why | Catch-net |
|---|---|---|
| Parity drift (one platform ahead) | Time pressure | Cross-platform unit workflow + checklist ([12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) §3); no tracker state for "half-shipped" |
| Offline queue conflicts / duplicate listing submits | Airplane-mode mid-submit is a *tested scenario* | v1.1 queue rule: server timestamps win; client-side dedupe token; manual script step 6 |
| Low-end Android jank/OOM (2 GB devices) | Compose on the actual farmer device | Device matrix includes a 2 GB device from W5, not at the end; baseline profiles if needed |
| fontScale 2.0 money truncation | Big tabular numerals + Hindi strings | Parity checklist line; screenshot at fontScale 2.0 / `.accessibility3` per PR |
| OTP autofill differences (Android SMS Retriever vs iOS suggestion) | Platform-different mechanisms, same UX required | Manual release script step 2 on both platforms |
| Push token rotation / notification-permission divergence | FCM vs APNs semantics | Integration test on staging in W9; token refresh handled in repository layer |
| Hindi/Gujarati text overflow in fixed-height components | Devanagari/Gujarati line-height + long compound words | Design tokens sized against HI strings ([10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)); screenshot grid includes HI |

### 3.3 Website + Console
| Predicted bug | Why | Catch-net |
|---|---|---|
| i18n key misses (EN present, HI/GU missing) | Three languages, hand-maintained | CI key-completeness check ([12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) §5) mirrored as a local hook (§6) |
| Lead-form spam | Public POST | Rate limit + dedupe already specced ([06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6.9); Playwright test hits the limit |
| Console table perf on grading queue | Vanilla JS, growing rows | Pagination from day 1 (list envelope, [06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6.1); revisit-if->3k-lines rule |
| Role-guard misses on Console screens | Two modules, one app | Every Console fetch goes through one API helper that hard-fails on 401/403; smoke test 1 covers guards |

### 3.4 Integrations
| Predicted bug | Why | Catch-net |
|---|---|---|
| Razorpay KYC delay blocks live payouts | Third-party clock | Filed W1; manual UPI fallback is a *designed* mode (`provider='manual'`), not an emergency ([11-ARCHITECTURE.md](11-ARCHITECTURE.md) §9) |
| Test/live key mix-up | Two key sets, solo dev | Env-only keys, named `RAZORPAY_TEST_*` / `RAZORPAY_LIVE_*`; server refuses to boot with live keys when `NODE_ENV!=production` (mirror of the `DEV_OTP` guard) |
| Duplicate webhook deliveries | Razorpay retries on non-200 | Idempotent handler keyed on provider event id (§10.2) |
| WhatsApp template rejections | Meta review is picky | Templates drafted + submitted early in W8; manual broadcast fallback per [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) |
| SMS DLT registration delay | 2–4 week clock | Filed W1; dev OTP acceptable on staging until it clears |

## 4. Skills — which Claude Code skill runs when

Process skills come first (they set the approach), implementation skills second. The standing map:

| Moment | Skill | Non-negotiable? |
|---|---|---|
| Before ANY new feature/component work | `superpowers:brainstorming` → then `superpowers:writing-plans` for multi-step work | Yes |
| Writing any feature/bugfix code | `superpowers:test-driven-development` — test first, then code | Yes (backend); mobile UI: acceptance-criteria-first |
| Any bug, test failure, unexpected behavior | `superpowers:systematic-debugging` before proposing fixes | Yes |
| Before claiming anything "done/fixed/passing" | `superpowers:verification-before-completion` + `verify` (drive the real flow, not just tests) | Yes |
| Before merging / ending a branch | `superpowers:requesting-code-review` → `code-review`; `superpowers:finishing-a-development-branch` | Yes |
| Writing/refactoring any code | `karpathy-guidelines` (surgical changes, no overcomplication) | Standing |
| Website & Console UI work | `frontend-design` (intentional visual design per [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)) | When UI |
| CMS graphs (C-1..C-4) | `dataviz` before the first line of chart code | When charts |
| Design-system sync with the design board | `figma:figma-generate-library` / `figma:figma-design-to-code`; `figma2swift` for iOS screens | When designing |
| Repo memory upkeep | `claude-md-management:revise-claude-md` after pattern-setting sessions | Weekly-ish |
| Post-merge cleanup | `simplify` / `code-simplifier` on the unit's diff | Per unit |

## 5. MCP servers — which are used, for what (and which are NOT)

### 5.1 Used during the build
| MCP | Job | Phase |
|---|---|---|
| **context7** | Current docs for Drizzle `1.0.0-rc.4`, Express, zod, Compose/Material3, SwiftUI — *always check before writing integration code; training data is stale for an RC-pinned ORM* | All |
| **Figma** | Design board → design tokens/components; parity screenshot references ([10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)) | W1–8 |
| **chrome-devtools / playwright** | Website + Console e2e: lead form, price-setting → grading → allocation run-through, lighthouse/perf on low-bandwidth profile | W1, W3–5, releases |
| **firebase** | FCM setup for Android push (server key, `google-services.json` wiring) | W9 |
| **sentry** | Crash reporting if Sentry wins the W5 decision ([12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) Q2) | W5+ |
| **postman** (optional) | API collection mirroring the frozen contract; collection runs as a second smoke layer | W3+ |

### 5.2 Explicitly NOT used for the build
- **Mixpanel MCP** — analytics is **first-party** ([19-PRD-CMS-ANALYTICS.md](19-PRD-CMS-ANALYTICS.md)): `app_events` → `metrics_daily` → Console. No third-party analytics vendor at MVP.
- **Higgsfield / Canva / Meta Ads / Vibe / Windsor / Upwork / Gmail / Calendar / ClickUp** — GTM- and founder-ops-side tools ([04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md)); they never touch product repos.
- **Pinecone** — no vector search in the product at MVP. Zero embeddings, zero indexes.
- Anything requiring OAuth the founder hasn't connected — agents flag, never work around.

## 6. Hooks — the local guardrails (mirror of CI, but instant)

Hooks are the pre-CI tripwires configured in Claude Code settings (created/managed via `hookify`). CI remains the authority; hooks make violations fail *before commit*:

| Hook | Fires on | Blocks/warns |
|---|---|---|
| **Region-agnostic grep** | Edit/Write in any product repo | Block: `gst_number`, `mandi_price`, `pincode` in schema/code; hardcoded `₹`, `+91`, city names outside region config & UI strings (Golden Rule #2) |
| **Secrets guard** | Edit/Write + pre-commit | Block: `.env` values, AES keys/IVs, `RAZORPAY_*` values, JWT secrets in diffs ([20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §5.4 — keys are never in source) |
| **Migration immutability** | Edit on `db/migrations/*` | Block edits to already-merged migration files (append-only rule) |
| **Parity reminder** | Stop, after edits under `KisanSetu-Android` or `KisanSetu-iOS` | Warn if the session touched one platform's screens without the sibling unit branch existing (Golden Rule #1) |
| **i18n completeness** | Edit on string tables | Warn when an EN key lands without HI/GU keys (GU may be machine-draft flagged) |
| **Money-path review flag** | Edit under `payments/`, `orders` pricing, payout services | Warn: founder review required on this PR; agents cannot self-approve (§7.3) |
| **Raw-SQL interpolation** | Edit/Write backend | Block string-concatenated SQL; Drizzle-bound or `sql`-tagged only ([06-PRD-BACKEND.md](06-PRD-BACKEND.md) §8) |

## 7. Agents — कौन agent क्या करेगा, क्या नहीं करेगा

### 7.1 The working set

| Agent | Does | Does NOT |
|---|---|---|
| **Explore** (read-only) | Scaffold gap-reviews (W1), "where is X handled", pre-change impact sweeps | Never edits; never reviews for correctness (that's the reviewers') |
| **Plan / feature-dev:code-architect** | Implementation blueprints per cross-platform unit; file-level build sequence | Never decides product scope — PRDs decide |
| **general-purpose** | Multi-step research + mechanical multi-file tasks under an approved plan | Money-path logic without a review pass |
| **feature-dev:code-explorer** | Traces execution paths before refactors (Fancall-pattern conformance checks) | — |
| **feature-dev:code-reviewer / pr-review-toolkit:code-reviewer** | Per-PR review: conventions, bugs, [20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) conformance | Cannot approve its own author-session's work — different session/agent reviews |
| **pr-review-toolkit:silent-failure-hunter** | Every PR touching error handling, payments, webhooks, offline queue | — |
| **pr-review-toolkit:pr-test-analyzer** | Coverage check on smoke-suite additions per unit | — |
| **pr-review-toolkit:type-design-analyzer** | New TS types/Drizzle schemas, Kotlin/Swift models | — |
| **code-simplifier** | Post-green cleanup on the unit's diff | Never "simplifies" away a golden-rule seam (provider interfaces, region abstraction) |
| **claude-code-guide** | Tooling questions (hooks/skills/SDK/API) | — |
| **Workflows (ultracode)** | Release-gate sweeps: multi-agent audit of a repo before RC tags; corpus-wide doc audits (like the one that produced this doc's sibling edits) | Not for trivial single-file changes |

### 7.2 Standing agent rituals
- **Per unit:** architect blueprint → TDD implementation → reviewer + silent-failure-hunter + test-analyzer sweep → simplifier → founder PR review.
- **Per release tag (W5+ weekly train):** ultracode workflow — parallel finders (correctness / security / parity / region-agnostic) with adversarial verification; findings triaged per [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) §7.4 (P0 same-day).
- **Weekly:** Brain-vs-code drift check — any behavior divergence gets a same-week PRD update (DoD #7).

### 7.3 Hard boundaries (agents/AI never do these — founder-only)
1. Approve PRDs, change golden rules, alter the two metrics, or make go/no-go calls.
2. Deploy to production, touch prod secrets, rotate live keys, or run live-money operations (payout execution is an *ops human* action through the Console — M5).
3. Communicate outward: no messages to farmers/buyers/FPO, no store submissions, no WhatsApp sends.
4. Merge money-path code (payments, pricing, payouts, credit) without explicit founder review of that PR.
5. Ship one mobile platform ahead of the other — an agent asked to do so must refuse and cite Golden Rule #1.
6. Invent product scope: anything not in a PRD goes to the backlog as a proposal, not into code.

## 8. Testing — कैसे होगी

Layered, from [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) §7, with the AI additions made explicit:

1. **Automated smoke suite** (backend, CI, <60 s): the 5 core flows — auth, listing lifecycle, server-side order pricing, allocation+traceability, payout+leads. Written TDD-style *with* the modules (W1–3), not after.
2. **CI per repo:** backend lint+migrations+smoke on fresh Postgres; Android `assembleDebug`+unit tests; iOS `xcodegen && xcodebuild`; website HTML validation + i18n completeness + JS-size gate + region grep. The `KE-NAIROBI` region test keeps Rule 2 honest forever.
3. **Manual release scripts:** 12-step farmer script (HI UI) + 10-step buyer script (EN UI) on the device matrix (2 GB Android + Pixel 6a; iPhone SE + current) before every tag.
4. **AI review gates:** per-unit agent sweep + per-release ultracode workflow (§7.2). Findings are triaged P0/P1/P2 — the workflow *reports*, humans *decide*.
5. **Field testing (the real QA):** 3 farmers at the hub (W7, W9 — <90 s unassisted listing) and 2 pilot kitchens (W8 — <3 min order). Field findings labeled `field` are auto-P1.
6. **Crash reporting** from first internal build (W5 tool decision); bar ≥99.5% crash-free before pilot.
7. **Crypto fixture test** (all three repos): committed ciphertext fixture must encrypt/decrypt identically on backend, Android, iOS — catches AES drift the day it happens (§3.1).

## 9. Per-surface execution recipes — कैसे बनेगा

### 9.1 Backend (और Console) — `KisanSetu-Backend`
- **Stack & layout:** exactly [20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §1 (TS + Express + Drizzle, MVC spine, response envelope, AES + API-key middleware). Schema lives in `db/schema/*` — *the* source of truth; migrations generated, never hand-edited after merge.
- **Order within the repo:** follow §2 row 3; each module = zod validators + service + repository + supertest coverage in one PR.
- **Console:** static vanilla JS served at `/ops` by the same Express instance; ops module first (W3–4), CMS module second (W4–5).
- **Tooling:** context7 for Drizzle-RC APIs; TDD skill; silent-failure-hunter on every payments/webhook PR; Postman collection optional after the W3 contract freeze.
- **Payments:** §10 — provider interface with the Razorpay adapter, `provider='manual'` until KYC + Phase 2.

### 9.2 Website — `KisanSetu-Website`
- Vanilla JS, no build step; static hosting with PR preview deploys; `POST /leads` is its only backend dependency (stub OK for 2 weeks).
- Built W1 with `frontend-design` + [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md) tokens; verified with Playwright (form, i18n, mobile viewport) + Lighthouse budget (<100 KB JS gate).
- Impact stats (freshness, farmer share) wired to the cached public endpoint only when ops enables publication ([08-PRD-WEBSITE.md](08-PRD-WEBSITE.md)).

### 9.3 Android — `KisanSetu-Android`
- Kotlin + Compose + Material3; Fancall MVVM below the View ([20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §2): Retrofit/OkHttp with `AESCryptoInterceptor`, repositories, StateFlow VMs (pending §7 ratification), EncryptedSharedPreferences session.
- Flavors `dev/staging/prod`; Play internal track W5; the 2 GB test device is in the loop from W5.
- Every screen lands only as part of a cross-platform unit (§2 row 7) with the parity checklist.

### 9.4 iOS — `KisanSetu-iOS`
- Swift + SwiftUI; XcodeGen (`project.yml` committed, `.xcodeproj` ignored); Fancall MVVM with the thin service-protocol delta ([20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §3), Codable + `.convertFromSnakeCase`, Keychain session.
- xcconfig per configuration for env; TestFlight internal W5, external (Beta App Review, 3 days' slack) W8.
- `figma2swift` assists screen implementation from the design board; parity screenshots vs Android in every unit PR.

## 10. Payment gateway — Razorpay (FIXED decision)

> **Founder decision (14 Jul 2026): Razorpay is THE payment gateway/PSP for KisanSetu India. It is not a default, a preference, or a candidate — it is fixed.** No other PSP will be evaluated or integrated. Any payment work on backend, Android, or iOS starts from this section and the anchors it points to. The provider *interface* stays (Golden Rule #2 — Razorpay is the India instance of an abstract payment provider), but the India instance does not change.

### 10.1 What Razorpay covers, per surface

| Surface | Razorpay product | When | Anchor |
|---|---|---|---|
| **Backend — farmer payouts** | Razorpay UPI payouts (RazorpayX) via `payments/provider.ts` adapter: `createPayout(payment) → {provider_ref}` | Phase 2 (M5/M6); MVP runs `provider='manual'` | [06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6.8 |
| **Backend — webhooks** | `POST /webhooks/razorpay`, signature-verified (unverified → 401 + alert), idempotent on provider event id | Phase 2 | [06-PRD-BACKEND.md](06-PRD-BACKEND.md) M6 |
| **Backend — buyer collection** | Razorpay Payment Links / Checkout for invoice collection, *when* buyer-side online payment starts (post-MVP; buyers pay on invoice at MVP) | Post-pilot decision | [06-PRD-BACKEND.md](06-PRD-BACKEND.md) Q2 |
| **Android** | Razorpay Android Checkout SDK (Kotlin) — added ONLY when in-app buyer collection is de-non-goaled; until then the app contains **no payment SDK** | Post-MVP | [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) non-goals |
| **iOS** | Razorpay iOS SDK (Swift/CocoaPods) — same trigger, same release as Android (parity) | Post-MVP | [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) non-goals |

### 10.2 Engineering rules (binding)
1. `require`/`import` of anything Razorpay **only inside `payments/`** on the backend ([11-ARCHITECTURE.md](11-ARCHITECTURE.md) §10.7); mobile SDKs, when they arrive, wrap behind one `PaymentProvider` interface per platform — same seam, both platforms, same release.
2. Keys are env-only: `RAZORPAY_KEY_ID` / `RAZORPAY_KEY_SECRET` / `RAZORPAY_WEBHOOK_SECRET` (+ `_TEST` variants). Never in source, never in the apps ([20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §5.4). Server refuses to boot with live keys outside `NODE_ENV=production`.
3. Webhook handler is idempotent (provider event id unique-keyed) — Razorpay redelivers on non-200.
4. Manual fallback is permanent: `provider='manual'` + ops-recorded UTR remains a first-class path even after Razorpay is live ([11-ARCHITECTURE.md](11-ARCHITECTURE.md) §9 — software never blocks a delivery, or a payout).
5. Razorpay KYC application is a **W1 action** ([12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) §9); staging uses test keys the whole way.
6. Country #2 gets its own provider adapter behind the same interface (PIX/M-Pesa/etc. per [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) Part B) — that future fact never dilutes the India decision above.

## 11. Open questions (owner + deadline)

| # | Question | Provisional decision | Owner | Deadline |
|---|---|---|---|---|
| Q1 | RazorpayX payout rail vs standard Razorpay payouts product — exact product SKU + pricing at integration time | Decide from Razorpay's then-current payout products at W8; interface unaffected | Founder | W8 |
| Q2 | Buyer-side online collection (payment links on invoices) — post-pilot or during pilot hardening? | Post-pilot; invoice+UTR flow must prove itself first | Founder | Pilot exit review |
| Q3 | Hook set of §6 — configure at W1 or incrementally? | All seven at W1 (they're cheap); tune noise weekly | Founder | W1 |
| Q4 | Sentry vs Crashlytics (carried from [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) Q2) | Leaning Sentry (one tool: apps + backend) | Founder | W5 |

**Changelog**
- 2026-07-14 — v1.0. Created: consolidated scope (will/won't), build order with AI notes, predicted-bug register, skills/MCP/hooks/agents mapping, testing layers, per-surface recipes, and the **fixed Razorpay decision** (§10). Companion edits landed the Razorpay anchors in 06/07/11/20/03/README.
