# 20 — CODE ARCHITECTURE (canonical patterns from Fancall)

**Doc owner:** Founder (Alpesh) · **Status:** Draft for founder approval · **Last updated:** 2026-07-14
**Siblings:** [06-PRD-BACKEND.md](06-PRD-BACKEND.md) (what the backend does) · [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) (what the apps do) · [11-ARCHITECTURE.md](11-ARCHITECTURE.md) (system shape) · [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md)

---

## 0. Why this doc exists (founder decision, 12 Jul 2026)

KisanSetu's code structure is **copied from the founder's existing production apps** — the same patterns Fancall proved in production:

- **Backend → MVC**, modelled on `~/Documents/Fancall-Backend` (its **v3** slice only — v1/v2 are legacy, do not copy).
- **Android → MVVM**, modelled on `~/Documents/New_Android_Fancall`.
- **iOS → MVVM**, modelled on `~/Documents/iOS-Fancall`.
- **API-key gate + request/response AES encode-decode**: same mechanism on all three, copied as-is (then re-keyed — see §5).

This doc is the canonical spec of those patterns. Where it refines an earlier PRD decision, **this doc wins** and the PRD has been updated to match (noted inline). Golden rules still apply — most importantly **Android ↔ iOS parity** ([00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) #1): the two apps use the identical MVVM layering, the identical API contract, and the identical crypto so a feature is written once conceptually and mirrored exactly.

> Fable's flag for founder ratification: two Fancall choices are **downgraded** for KisanSetu, not copied verbatim — (a) secrets are **not** hardcoded in source (Fancall ships the AES key/IV and `x-api-key` inside the app binary and `ApiKeys.swift`/`EncryptionUtils.kt`); KisanSetu sources them from build config / xcconfig / server env. (b) The mobile session token moves to **EncryptedSharedPreferences (Android) / Keychain (iOS)** instead of plain SharedPreferences/UserDefaults. Same mechanism, safer storage. See §5.4.

---

## 1. Backend — MVC in **TypeScript** with **Drizzle ORM** (Fancall v3 layering, modern stack)

**Founder decision (12 Jul 2026):** the backend is written in **TypeScript** (not plain JS), and the data layer is **Drizzle ORM `v1.0.0-rc.4`** — NOT Knex+Objection. We keep Fancall's proven MVC *layering, request lifecycle, response envelope, and API-key/AES crypto*, but the language is TS and the persistence layer is Drizzle. This is the outcome of the 2026 ORM review (Objection.js is frozen — no release since Sep 2024, its author moved on; Drizzle is TS-first, actively maintained, and best-fit for our analytics-heavy SQL). Everything below reflects this.

### 1.1 Stack (canonical)
**TypeScript 5.x (ESM, `tsx` for dev, `tsc`/`tsup` build)** · Node.js 20 · Express · PostgreSQL 16 · **Drizzle ORM `v1.0.0-rc.4`** (`drizzle-orm`) + **`drizzle-kit`** (schema → SQL migrations) + **`pg`** (node-postgres driver) · `zod` (validation — pairs natively with Drizzle via `drizzle-zod`) · `jsonwebtoken` (JWT) · `bcryptjs` (hashing) · a small AES helper (`node:crypto`, Fancall's `cryptlib` scheme — §5) · `pino` (logging; `pino-http` for access logs) · `i18n` · `swagger-jsdoc`/`swagger-ui-express` (API docs).

> Version note: as of mid-2026 Drizzle's stable `latest` tag is `0.45.x` and **v1 is in release-candidate (`1.0.0-rc.4`, published 2026-06-27)**. The founder has chosen to build on **rc.4** and track v1 to GA. Pin the exact RC in `package.json` (`"drizzle-orm": "1.0.0-rc.4"`, `"drizzle-kit": "1.0.0-rc.4"`) and bump deliberately; re-pin to the stable `1.0.0` the moment it ships. `zod` is used instead of `node-input-validator` because Drizzle + `drizzle-zod` generate request/row schemas from the table definitions (one source of truth, full type inference).

### 1.2 Folder structure (TypeScript + Drizzle)
```
src/
  app.ts                – Express bootstrap: middleware chain, mounts routes
  server.ts             – http listen / graceful shutdown
  config/               – env.ts (typed env via zod), constants.ts (codes, roles, prefixes)
  db/
    index.ts            – drizzle(pg pool) client, exported typed `db`
    schema/             – Drizzle table definitions (users.ts, listings.ts, orders.ts, …) — THE schema source of truth
    migrations/         – drizzle-kit-generated SQL migrations (+ meta/)
    seed.ts             – seed script (targeted + Surat market row, produce, price feed)
  routes/               – versioned route group; index.ts top router
  controllers/          – THIN HTTP layer: parse req → call service → set res locals → next()
  services/             – business logic; returns { data, statusCode, status, message }
  repositories/         – data-access: typed Drizzle queries per aggregate (BaseRepository generic)
  middleware/           – auth guards, responseMiddleware, errorHandler, rateLimit, apiKey, decryptBody
  validators/           – zod schemas (many derived from db/schema via drizzle-zod)
  helpers/              – crypto.ts (encryptData/decryptedData/encryptOtp), logger.ts
  errors/               – AppError class (extends Error; static factories per HTTP code)
  types/                – shared TS types / augmentations (Express Request, JWT payload)
drizzle.config.ts       – drizzle-kit config (schema path, out dir, dialect: 'postgresql', dbCredentials)
tsconfig.json · package.json
```
- **No `models/` folder.** Drizzle has no active-record model classes — the `db/schema/*` table definitions ARE the schema, and typed queries live in `repositories/`. Soft-delete + timestamps are plain columns (`deleted_at timestamptz`, `created_at`/`updated_at` with DB defaults + an `updated_at` touch in the repository base), not ORM lifecycle hooks.
- **Migrations = `drizzle-kit generate` (schema diff → SQL) then `drizzle-kit migrate`.** The geo + market-targeting tables from the scaffold's raw `002–005.sql` are re-expressed as Drizzle schema; `drizzle-kit` emits the equivalent SQL. The raw `.sql` scaffold files remain the reference for column shape when authoring the schema.

### 1.3 Request lifecycle (the MVC spine — unchanged from Fancall except the DB layer)
```
Request
 → global middleware (pino-http, cors, i18n, body parsers incl. text for ciphertext)
 → API-KEY middleware (§5.1)  → BODY-DECRYPT middleware (§5.2)  → rate limit
 → route file:  [auth guard?] → errorHandler(controllerFn) → responseMiddleware
 → controller (thin) → service (business logic) → repository → Drizzle query → Postgres
 ← controller sets res.locals.data / statusCode / status / msg, calls next()
 ← responseMiddleware builds { message, status, data } envelope, ENCRYPTS it (§5.3), res.json()
```

### 1.4 The contracts to copy exactly
- **Controllers never call `res.json`** and never manage transactions. They set the response locals and `next()`, or `throw new AppError(code, message)`.
- **`errorHandler(handler)`** wraps every controller: runs the handler inside a **Drizzle transaction** (`db.transaction(async (tx) => …)`), commits on success, **rolls back on throw**, maps `err.statusCode ?? 500`, delegates to `responseMiddleware`. (Same wrapper role as Fancall's Knex-transaction version — the mechanism is identical, the driver is Drizzle.)
- **Uniform response envelope** (same shape for success and error, only the code differs): `{ message, status, data }` → AES-encrypted on the wire (§5.3).
- **`AppError` class** (`errors/`): static factories `notFound`(404), `validation`(422), `unauthorized`(401), `forbidden`(403), `conflict`(409), `badRequest`(400), `server`(500) — each logs on construction. Typed.
- **`BaseRepository` generic**: wraps a Drizzle table, provides `findById`, soft-delete-aware `list`, `create`, `update` (auto-stamps `updated_at`), and `softDelete` (sets `deleted_at`). Feature repositories extend it with typed Drizzle queries. This replaces Fancall's Objection `AppModel`.
- **Naming:** files kebab or camel (`auth.controller.ts`, `auth.service.ts`, `user.repository.ts`, `users.ts` schema), routes dotted (`auth.route.ts`). Types PascalCase.

### 1.5 KisanSetu deltas from Fancall (deliberate)
- **TypeScript, not JavaScript**; **Drizzle, not Knex+Objection** (this section).
- Drop the `/api/api/v3` double-prefix accident → clean single version prefix — for KisanSetu that is `/v1` ([06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6, dev base `http://localhost:4000/v1`), not Fancall's `v3` numbering.
- Roles are the KisanSetu set (`farmer`/`buyer`/`ops`, [06-PRD-BACKEND.md](06-PRD-BACKEND.md)), not Fancall's numeric fan/celebrity/brand.
- One monolith, no socket.io/RabbitMQ/Agora at MVP ([11-ARCHITECTURE.md](11-ARCHITECTURE.md) non-goals stand).
- The hard analytics/allocation SQL (SLA `percentile_cont` medians, farmer-share %, greedy listing→order-item allocation) is written with Drizzle's SQL builder or `sql`-tagged raw for full control — Drizzle keeps you at SQL level, which suits these queries.
- **Payments: Razorpay is the fixed India PSP** (founder decision, 14 Jul 2026) — one adapter behind `payments/provider.ts`; nothing Razorpay-specific outside `payments/` ([11-ARCHITECTURE.md](11-ARCHITECTURE.md) §10.7). Engineering rules + the mobile-SDK plan live in [21-AI-EXECUTION-PLAYBOOK.md](21-AI-EXECUTION-PLAYBOOK.md) §10.

---

## 2. Android — MVVM (Kotlin, New_Android_Fancall pattern)

### 2.1 Stack
Kotlin · **XML layouts + ViewBinding/DataBinding** (Fancall is not Compose) · Retrofit + OkHttp + Gson · **Dagger 2** (manual components, no Hilt) · Coroutines · Room (local) + SharedPreferences (session) · Glide/Coil.

> **Parity note vs 07-PRD:** [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) §5.0 targets **Jetpack Compose** (founder's stated stack for the new app). Fancall's reference is XML. **Decision for founder to ratify:** keep the Fancall *MVVM layering, DI, network + crypto stack* but render UI in **Jetpack Compose** (modern, and the design system in [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md) is defined for Compose/Material3). The layers below the View are copied as-is; only the View technology differs. If the founder prefers XML-exact parity with Fancall, flip §5.0 of 07 — but Compose is the recommendation.

### 2.2 Package structure (layers)
```
core/         – Session, AES.kt, AESCryptoInterceptor.kt, AppPreferences (SharedPreferences wrapper)
di/           – Dagger: ApplicationComponent → ActivityComponent → FragmentComponent; ViewModelModule (multibind)
data/
  service/    – Retrofit interfaces
  repository/ – repository INTERFACES
  datasource/ – *LiveDataSource impls (the repository impl) extending BaseDataSource (execute{} wrapper)
  pojo/       – DTOs (request/response), DataWrapper, ResponseBody
ui/
  base/       – BaseActivity, BaseFragment<T:ViewBinding>, BaseViewModel, APILiveData, ViewModelFactory
  <feature>/  – fragment/ + viewmodel/ (+ adapter/, bottomsheet/)
utils/        – EncryptionUtils, RetrofitClient, Validator
```

### 2.3 MVVM layering (unidirectional — copy exactly)
```
View (Activity/Fragment, ViewBinding)
 → ViewModel (extends BaseViewModel; exposes APILiveData<T> = MutableLiveData<DataWrapper<T>>)
   → Repository (interface)
     → *LiveDataSource (impl, BaseDataSource.execute{}) → Retrofit Service → OkHttp → API
 ← datasource returns DataWrapper<T> → VM sets APILiveData.value → View's observer updates UI
```
- ViewModels get the repository **constructor-injected** (Dagger); multibound via `@ViewModelKey` + `ViewModelFactory`.
- State is **LiveData** (`APILiveData<T>`), success/error discriminated by `DataWrapper.responseBody` vs `.throwable`. (KisanSetu may modernise to **StateFlow** with Compose — same VM→View contract, ratify with §2.1.)
- `BaseFragment.onError()` centralises error→message and **401/601 → logout**.

### 2.4 KisanSetu deltas
- Package `com.kisansetu.app` (one app, two user types — [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) §5.0), `farmer`/`buyer` feature modules import `core` only.
- Session token → **EncryptedSharedPreferences** (Fancall uses plain SharedPreferences — §5.4).
- Consider Hilt over manual Dagger (optional; manual Dagger is fine to copy).

---

## 3. iOS — MVVM (Swift, iOS-Fancall pattern)

### 3.1 Stack
Swift · **UIKit + Storyboards/XIB** (Fancall) · Moya-on-Alamofire · SwiftyJSON + custom `Mappable` · hand-rolled `Bindable<T>` (no Combine/Rx) · Realm (local) + UserDefaults (session) · CocoaPods.

> **Parity note vs 07-PRD:** [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) targets **SwiftUI**. Fancall is UIKit. **Same decision as Android (§2.1):** copy Fancall's MVVM layering + network + crypto, but render in **SwiftUI** (matches the design system and Android parity). Ratify alongside §2.1 so both platforms move together — that IS the parity rule.

### 3.2 Group structure (layers)
```
AppBase/                – AppDelegate, lifecycle
WebService/             – THE network layer:
  APIKeys/              – header names/values, AES key + IV, response keys, status codes
  EndPoints/            – endpoint enum (path builder)
  Manager/ApiManager    – Moya TargetType: makeRequest, setHeaders, task (encrypt), decrypt on response
  Helper/               – Mappable, DataResultModel<T>, String.encryptData()/decryptData()
Scenes/
  Models/               – domain models (Mappable)
  Module/<Feature>/Controllers/<Screen>/  – <Screen>VC.swift + <Screen>VM.swift (paired)
AppUtility/             – BaseViewController, UserDefault @propertyWrapper, Managers, Extensions
```

### 3.3 MVVM layering (copy exactly)
```
View (UIViewController; SwiftUI View in KisanSetu)
 → owns a ViewModel; VM exposes Bindable<Result<T, AppError>> (→ @Published/StateObject in SwiftUI)
   → ApiManager.shared.makeRequest(endPoint:, methodType:, parameter:) (service/repository role)
     → Moya provider → Alamofire → API
 ← response decrypted + mapped → VM sets .value → View's bound closure updates UI
```
- One paired `…VC` + `…VM` per screen. VM validates inputs, returns `AppError?`, publishes results.
- `AppError` enum (LocalizedError) with nested network/validation/file cases + `.custom`.
- **KisanSetu delta:** insert a thin **service protocol** between VM and the API client (Fancall's VM calls the concrete `ApiManager` singleton directly → not unit-testable). One protocol makes VMs testable without the network.

### 3.4 KisanSetu deltas
- Bundle `com.kisansetu.app`; SwiftUI + `@Published`/`@StateObject` instead of `Bindable`/UIKit.
- Session token → **Keychain** (Fancall uses UserDefaults — §5.4).
- Codable + `.convertFromSnakeCase` decoder instead of SwiftyJSON+Mappable (backend is snake_case JSON — cleaner and native).

---

## 4. The one API contract all three share

Backend defines it ([06-PRD-BACKEND.md](06-PRD-BACKEND.md)); both apps consume it identically (parity):
- **Base:** `/v1/...` — the single version prefix set by [06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6 (dev base `http://localhost:4000/v1`); snake_case JSON.
- **Every request carries:** `x-api-key` (encrypted, §5.1), `Authorization: Bearer <jwt>` when authed, `Content-Type: text/plain` when the body is an encrypted blob (else `application/json`), `accept-language`.
- **Envelope (before encryption):** request body = the feature JSON; response = `{ message, status, data }`.
- **On the wire:** both bodies are a single AES-Base64 ciphertext string (§5). Multipart uploads are the exception — the file parts are plaintext and the JSON travels as an `encryptedBody` form field.

---

## 5. API key + encode/decode (the security pattern — copy the mechanism, re-key it)

This is the piece the founder explicitly wants copied. Same on backend, Android, iOS.

### 5.1 API-key gate
- Header **`x-api-key`**. The value is a **fixed app secret that is itself AES-encrypted** before sending (`encrypt("<APP_KEY>")`). Server decrypts it and compares to the configured `API_KEY`.
- Backend gate is the **first middleware** on the protected route group; rejects missing/invalid key with 400 before anything else runs. A few routes are exempt (public website, signed webhooks).
- **Dual-key rotation (copy this):** if the primary key fails, try a secondary (`API_KEY_2`); a secondary match flags the request so the **response is encrypted with the secondary key too** — lets you rotate keys with zero downtime.

### 5.2 Request decode (server)
Body arrives as an encrypted text blob (or `{ encryptedBody }` for multipart). Middleware `decryptedData(body)` → JSON → `req.body`. Falls back to the secondary key on failure. (This is why `bodyParser.text()` is registered before `bodyParser.json()`.)

### 5.3 Response encode (server) + client decode
Response middleware builds `{ message, status, data }`, `encryptData(payload)` (primary or secondary per the request's key), sends the ciphertext string. Clients decrypt with the shared secret before parsing.

**Algorithm (identical across all three, copy exactly):** **AES-256-CBC, PKCS7 padding, Base64 output.**
- **Key** = `SHA-256(secretKeyString)` truncated to 32 bytes.
- **IV** = the raw secret-IV string (16 bytes).
- Backend: `cryptlib` (`getHashSha256(key,32)`, `encrypt/decrypt`). Android: `AES.kt` (`AES/CBC/PKCS5Padding`, SHA-256→32-byte key, IV = first 16 bytes of key string) hooked as an **OkHttp interceptor** (`AESCryptoInterceptor`). iOS: vendored `CryptLib` (CommonCrypto) via `String.encryptData()/decryptData()`, applied in `ApiManager.task`.
- **OTP blobs** use a separate native AES-256-CBC with a **random per-call IV** (`iv:ciphertext` hex) — copy this for OTP too.

### 5.4 Storage & secrets — KisanSetu HARDENS the copy (founder to ratify)
| Concern | Fancall (as-is) | KisanSetu canon |
|---|---|---|
| AES key/IV + `x-api-key` | **hardcoded in source** (`ApiKeys.swift`, `EncryptionUtils.kt`, in-binary) | injected from **build config / xcconfig / server `.env`**; never checked into the app repo in plaintext |
| Static IV reused | IV derived from key, reused every call | acceptable for the envelope (matches backend); **OTP uses random IV** as Fancall already does |
| Mobile session JWT | plain SharedPreferences / UserDefaults | **EncryptedSharedPreferences (Android) / Keychain (iOS)** |
| Key rotation | dual-key `_2` path | keep the dual-key path; document a rotation runbook in [11-ARCHITECTURE.md](11-ARCHITECTURE.md) |

The **mechanism** is copied verbatim; only key-storage and token-storage are hardened. The wire format stays byte-compatible so all three stay in lockstep.

---

## 6. What NOT to copy from Fancall (explicit)
- Backend v1/v2 trees, the monolithic route handlers embedded in `routes/index.js`, the legacy AES-OFB `get-*-private-key.js` helpers.
- The `/api/api/v3` double-prefix bug.
- Hardcoded secrets in app source; `data/temp/` scratch (81 files); the dead `AppKeyStore.kt`; the double-registered logging interceptor.
- Fancall's domain (fan/celebrity/brand/calls/payments-for-calls) — only the **structure** transfers, not the features.

---

## 7. Open questions (founder to ratify)
| # | Question | Recommendation | Owner / deadline |
|---|---|---|---|
| 1 | Backend language + data layer | **RESOLVED (founder, 12 Jul 2026): TypeScript + Drizzle ORM `v1.0.0-rc.4`** (not JS, not Knex+Objection). Copy Fancall's MVC layering only. §1 rewritten; 06/11 updated. | Closed |
| 2 | Mobile UI tech: Compose/SwiftUI (07 PRD) vs XML/UIKit (Fancall) | **Compose + SwiftUI**, copy Fancall's layers below the View | Founder, PRD approval |
| 3 | Mobile state holder: LiveData/Bindable (Fancall) vs StateFlow/@Published | **StateFlow + @Published** (modern, parity-friendly) | Founder, PRD approval |
| 4 | Secrets hardening (§5.4) — accept the hardened storage? | Yes — mechanism copied, storage hardened | Founder, PRD approval |
| 5 | DI on Android: manual Dagger (Fancall) vs Hilt | Either; Hilt is less boilerplate | Founder, before build |
