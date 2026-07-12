# 00 — The Golden Rules

These six rules are the constitution of KisanSetu. Every document in this Brain, every design, every line of future code, every hiring decision and every sales promise is subordinate to them. They exist because each one encodes a lesson that killed a competitor, a trap the founder consciously rejected, or a discipline that a solo-founder company cannot afford to lose.

**How to use this document:** when you are unsure whether a decision is right, test it against all six rules. If it violates any one of them, the decision is wrong — escalate to the founder rather than "just this once" it. Exceptions can only be granted by the founder, in writing, in the relevant Brain document.

---

## Rule 1 — Android ↔ iOS PARITY

**The rule:** The Android and iOS apps are screen-for-screen, feature-for-feature, design-identical. One shared PRD ([07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md)) and one design system ([10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)) govern both. **A change to one platform is incomplete until mirrored on the other.**

**Why it exists:** A solo founder cannot afford two divergent products. Divergence compounds: a "quick Android-only fix" becomes a screen that behaves differently, then a support burden, then a full rewrite. Parity also protects the brand promise — a buyer demoing the app to a peer on the other platform must see the same product. Platform-idiomatic *implementation* (Compose vs SwiftUI, Material3 type scale vs SF Pro Dynamic Type) is expected; *product* divergence is not.

**In practice:**
- There is exactly one mobile PRD. There will never be a "07a-PRD-ANDROID.md".
- Every ticket that touches a mobile screen is defined once and carries two checkboxes: `[ ] Android` `[ ] iOS`. The ticket is not "done" until both are checked. See [15-TASKS-BACKLOG.md](15-TASKS-BACKLOG.md).
- Screen inventory, navigation graph, empty states, error states, and copy strings are identical. String tables are shared assets (same keys on both platforms).
- Release trains ship together. If iOS review is delayed, Android waits (feature-flag if a critical fix must go out).
- Design review compares screenshots side by side, both platforms, both languages (EN + HI), before sign-off.

**Violations (real shapes this takes):**
- "The kg-stepper is fiddly in SwiftUI, let's ship Android first and catch iOS up next sprint." → No. Same release.
- "Android got a Payments filter because a farmer asked for it." → Feature was added to one PRD-less platform; revert or mirror immediately.
- "iOS uses a bottom sheet here, Android uses a full screen, because that's more idiomatic." → Idiomatic *components* are fine only if the flow, information and actions are identical. If a user on either platform would describe the flow differently, it's a violation.
- A bug fixed on one platform and left open on the other for more than one release cycle.

---

## Rule 2 — GLOBAL-FIRST, SURAT-FIRST

**The rule:** The product is designed as a **global platform** — any geography with smallholder farmers and food businesses — and launched **hyper-locally in Surat**. Build region-agnostic, launch local. Neither half of the rule may be dropped.

**Why it exists:** The smallholder→HoReCa gap is not an Indian quirk; it exists across Southeast Asia, Africa and LATAM (see [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md), Global section). Retrofitting i18n, multi-currency and tax abstraction into an India-hardcoded codebase is a rewrite. Conversely, "global" without a beachhead is how startups die of vagueness — Surat gives us density, founder proximity, and a winnable first market.

**In practice (the engineering checklist — every PRD must comply):**
- **i18n from day 1:** all user-facing text in string tables; EN/HI/GU at launch; languages pluggable per region.
- **Currency abstracted:** INR at launch, but every money value carries a currency field in the data model. No `₹` hardcoded outside India UI copy.
- **Phones:** E.164 everywhere (`+919876543210`), never 10-digit assumptions.
- **Time:** timezone-aware timestamps (UTC storage, local display).
- **Generic domain terms in the schema:** `reference_market_price`, not `mandi_price`. "Mandi" may appear only in UI copy for the India region. Same for tax: a per-region tax abstraction of which GST is the Indian instance.
- **Geography as data:** region → hub → farm hierarchy in the data model, so a new city or country is configuration, not code.
- **Launch geography is configuration, not code:** the CMS Markets screen ([19-PRD-CMS-ANALYTICS.md](19-PRD-CMS-ANALYTICS.md) C-6) decides where we are live; the codebase never hardcodes a city.
- **Payment/messaging providers behind interfaces:** Razorpay-UPI and WhatsApp Business API are India instances of abstract payout/messaging providers.

**And the Surat-first half:**
- All Phase 0 validation, first crops, first FPO, first buyers: Surat only. See [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md).
- No second city until Surat pilot passes its go/no-go gates. Expansion path: Surat → Gujarat → tier-2 India → Indian metros → international.

**Violations:**
- A column named `gst_number`, `mandi_price`, or `pincode` in the schema (use `tax_id`, `reference_market_price`, `postal_code`).
- Hindi strings embedded in Kotlin/Swift source instead of string tables.
- "Let's also sign up a couple of buyers in Ahmedabad, it's only 3 hours away." → Not until the Surat gate passes.
- Designing a feature that only makes sense in India (e.g. hard dependence on UPI semantics) with no abstraction seam.
- The opposite failure: spending a week building a currency-conversion service before we have one paying buyer in Surat. Global-first means *not blocking* globalization, not *building* it early.

---

## Rule 3 — PLAN BEFORE CODE

**The rule:** No development until the PRDs in this Brain are approved by the founder. The Brain is the source of truth; code serves the plan, never the reverse. **Development is currently PAUSED.**

**Why it exists:** The app is ~10% of this business; logistics, trust and supply are the rest. Code written before validation is a liability that anchors thinking ("we already built it this way"). Scaffolds already exist — and the temptation to "just extend" them is exactly what this rule blocks. Phase 0 (10 buyer interviews, 1 FPO MoU, mandi shadow, trademark check, crops frozen, real unit economics) can invalidate whole product areas; the kill/pivot criterion (<3 of 10 buyers willing to switch → FPO-SaaS-first pivot) could reshape everything.

**In practice:**
- Every PRD contains: Purpose, Users & JTBD, Scope/NON-GOALS, functional spec, edge cases, acceptance criteria, phased rollout, metrics, open questions. A PRD with unanswered open questions on the critical path cannot be approved.
- Approval is explicit: the founder marks each PRD "APPROVED + date" in the document header. No verbal approvals.
- The existing scaffolds (`KisanSetu-Backend`, `-Website`, `-Android`, `-iOS`) are frozen. After approval they get a review against the PRDs — keep, refactor, or discard per component. They are input, not a head start to defend.
- Spikes/prototypes to answer a PRD open question are allowed, must be throwaway, and must be labeled as such.

**Violations:**
- "While we wait on the buyer interviews I'll build the catalog API, we'll need it anyway." → You don't know that yet.
- Extending a scaffold because the code is already there.
- Writing a PRD *around* what the scaffold already does (code-driven planning — the exact inversion this rule forbids).
- "The founder said it sounded good on a call." → Not approval.

---

## Rule 4 — ASSET-LIGHT

**The rule:** KisanSetu owns **no farms, no trucks, no warehouses**. Supply aggregation, weighing and grading happen at **partner FPO** collection hubs; transport is **3rd-party cold logistics (3PL)**. We bring demand, software, standards and data.

**Why it exists:** This is the most empirically validated rule in the Brain. Every asset-heavy B2B2C fresh player died in 2024–25 — Otipy (shut down May 2025), Fraazo, ReshaMandi, Greenikk, Deep Rooted — and WayCool, the only agritech unicorn, is distressed under the weight of its own infrastructure. Asset-heavy fresh means capital burn scales linearly with GMV; asset-light means software margins on a physical flow. See [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md) for the full graveyard.

**In practice:**
- Capacity problems are solved by **adding partners**, not buying assets. Hub at capacity → second FPO. Delivery unreliable → second 3PL, SLA penalties in contract.
- Our capex is laptops, phones, and grading equipment we may lend to FPO hubs (small, moveable, recoverable — acceptable).
- The FPO relationship is deepened with **software** (the FPO procurement SaaS in [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)) — that's the moat, not owning their shed.
- Contracts and the ops playbook ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md)) are written assuming partner execution: quality standards, weighing protocols, SLA timestamps enforced through process + software, not ownership.

**Violations:**
- "A used Tata Ace is only ₹4 lakh and would solve our morning delivery problem." → It also makes us a logistics company. Find a second 3PL.
- Leasing a warehouse "temporarily" for overflow.
- Hiring drivers or grading staff onto our payroll (the ops lead coordinates; FPO staff grade).
- Taking inventory risk — buying produce we haven't matched to buyer demand. (Allocation-before-pickup is the model; see [06-PRD-BACKEND.md](06-PRD-BACKEND.md).)
- Slow-motion version: capex creeping up quarter over quarter in "small" exceptions. Review asset spend monthly against this rule.

---

## Rule 5 — THE TWO METRICS

**The rule:** Two numbers define KisanSetu and both are published:
1. **Freshness:** median hours harvest→buyer's door. Target **<36h**, promise **<48h**.
2. **Farmer share:** farmer's % of the final buyer price. Target **≥60%** (vs ~30–40% status quo), and we publish it.

**Every feature must serve at least one of these metrics.** Every roadmap debate ends with "which metric does this move?"

**Why it exists:** The core math of the business is one lever: a shorter chain simultaneously roughly doubles the farmer's share AND delivers produce days fresher. Freshness is why buyers switch; farmer share is why supply stays. Publishing both keeps us honest and is itself marketing (see [04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md)). A startup this small cannot afford features that serve neither.

**In practice:**
- Instrumentation is non-optional: every allocation carries `harvest_ts → hub_in_ts → dispatch_ts → delivered_ts` ([06-PRD-BACKEND.md](06-PRD-BACKEND.md)). If a crate moves without timestamps, that's an ops incident.
- Farmer share is computed per order from actual payout vs actual buyer invoice — not a marketing average. The ops dashboard ([09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md)) reports both metrics weekly.
- Feature intake template includes the line: "Metric served: freshness / farmer share / (both)". A blank answer means the feature waits.
- Trade-offs resolve toward the metrics: e.g. a cheaper 3PL that adds 12 hours fails the freshness test even if it improves margin.
- The metrics appear publicly: website impact stats, buyer traceability view, farmer payout slips showing struck-through reference market price.

**Violations:**
- Building a farmer-side chat/community feature ("engagement!") — serves neither metric. B2C-brain contamination; see rule 6.
- Quietly widening the freshness promise to 72h to make a difficult week look green — the metric is changed only by founder decision, publicly.
- Reporting farmer share from list prices instead of settled payouts.
- A dashboard full of vanity metrics (downloads, DAU) crowding out the two that matter.

---

## Rule 6 — B2B, SALES-LED

**The rule:** KisanSetu is a B2B business. Revenue comes from **tens of business customers** (HoReCa first, later modern retail and processors) acquired by **direct, founder-led sales**. Ads and content support sales; they never replace it. We do not chase consumer downloads, virality, or "users".

**Why it exists:** B2C fresh delivery is a graveyard (see the dead-players table in [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md) — every one of Otipy/Fraazo/Deep Rooted had a consumer leg). B2C needs millions of users and discount-fueled retention; B2B needs ~25 restaurants ordering ~daily at ~₹2,000 AOV to prove the model (validate this figure in Phase 0 — [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)). A solo technical founder wins B2B by showing up with a sample crate, not by outspending Zepto.

**In practice:**
- The sales motion is the founder with sample crates, 5 visits/day during pilot; WhatsApp Business catalog + daily price broadcast; Google Business Profile; geo-fenced Meta lead-gen at ₹15–20k/month pilot budget. See [04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md).
- Success counts are buyer logos, weekly repeat rate (target ≥70%), fulfilment rate (≥95%) — not installs.
- The farmer app exists to secure **supply**, not as a growth surface. Farmer acquisition rides FPO trust, village demo days, and the payout slip as the advertisement.
- Product priorities favor buyer retention (catalog reliability, traceability, one clean invoice) and ops throughput over any growth-hack feature.
- Pricing conversations happen in person, and price changes are sales decisions, not A/B tests.

**Violations:**
- "Let's open the buyer catalog to households, extra volume is extra volume." → That's B2C fresh delivery. It killed better-funded companies.
- Spending on brand-awareness ads before the sales pipeline is saturated.
- Measuring the week by app downloads.
- Building referral loops, gamification, streaks, or push-notification "re-engagement" campaigns for farmers.
- Hiring a "growth marketer" before hiring the ops lead and field-sales person ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) sets the hiring order).

---

## Precedence and change control

- Where any Brain document appears to conflict with these rules, **the rules win** and the document must be fixed.
- Changing a golden rule requires: founder decision + written rationale in this file's changelog below + a sweep of affected Brain docs in the same sitting.

**Changelog**
- 2026-07-12 — v1.0. Initial six rules codified from founder decisions and verified market evidence (July 2026).
