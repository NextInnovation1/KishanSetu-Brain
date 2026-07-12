# KisanSetu Brain

**The complete planning corpus for KisanSetu — the source of truth that precedes all code.**

> **STATUS: PLANNING — DEVELOPMENT IS PAUSED.**
> Code scaffolds exist (`~/Desktop/KisanSetu-Backend`, `-Website`, `-Android`, `-iOS`) but they are **not approved**. The documents in this directory supersede them. When the founder approves the PRDs, the existing scaffolds get a fresh review against these documents before a single new line of code is written. See rule 3 below.

KisanSetu ("setu" = bridge; working title — trademark check pending, owner: Alpesh, deadline: end of Phase 0) is a tech-led, **asset-light** B2B supply chain connecting smallholder farmers directly to food businesses (HoReCa first). Farmers list produce; partner FPO hubs aggregate, weigh and grade; 3rd-party cold logistics deliver within 24–48h of harvest; farmers are paid instantly to UPI on pickup; buyers get one invoice and farm-to-fork traceability. Founder: Alpesh, Surat.

---

## The 6 Golden Rules (memorize these first)

Full definitions with violation examples live in [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md). The short form:

1. **Android ↔ iOS PARITY** — the two mobile apps are screen-for-screen, feature-for-feature, design-identical. A change to one platform is incomplete until mirrored on the other.
2. **GLOBAL-FIRST, SURAT-FIRST** — build region-agnostic (i18n, currency abstraction, E.164 phones, generic domain terms), launch hyper-local in Surat.
3. **PLAN BEFORE CODE** — no development until the PRDs in this directory are approved by the founder.
4. **ASSET-LIGHT** — no owned farms, trucks or warehouses; partner FPOs + 3PL cold chain, always.
5. **THE TWO METRICS** — median hours harvest→door (target <36h, promise <48h) and farmer's share of final price (target ≥60%, published). Every feature must serve at least one.
6. **B2B, SALES-LED** — profitability via tens of business customers, not millions of users.

---

## What's in the Brain

| File | What it is | Primary owner-audience |
|---|---|---|
| [README.md](README.md) | This index. Status, rules, reading order. | Everyone, first |
| [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) | The 6 non-negotiables, in practice, with violation examples. | Everyone |
| [01-VISION-MISSION.md](01-VISION-MISSION.md) | Vision, mission, global ambition with Surat beachhead, what KisanSetu is/is not, 10-year view. | Founder, investors, new hires |
| [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md) | TAM/SAM/SOM with sources, post-harvest loss, farmer demographics, competitive landscape (incl. the dead players as thesis proof), the wedge, why-now, global markets. | Founder, investors |
| [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md) | Revenue lines (take rate, logistics fee, buyer subscription, FPO SaaS), unit economics, validation plan. | Founder, investors |
| [04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md) | Founder-led sales motion, channel plan, ad budgets, farmer-side acquisition via FPOs, PR angle. | Founder, first sales hire |
| [05-PRODUCT-OVERVIEW.md](05-PRODUCT-OVERVIEW.md) | The whole product surface in one view: dual-role mobile app, website, ops dashboard, how they connect. | Everyone building |
| [06-PRD-BACKEND.md](06-PRD-BACKEND.md) | Backend PRD: domain model, API surface, auth, state machines, SLA timestamps. | Backend dev |
| [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) | ONE PRD for BOTH apps (rule 1): farmer flow + buyer flow, screen by screen. | Android + iOS devs |
| [08-PRD-WEBSITE.md](08-PRD-WEBSITE.md) | Marketing site PRD: hero, broken-chain visual, how-it-works, lead capture. | Web dev |
| [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md) | OPS module of the internal KisanSetu Console (one web app, two modules — see 19): price setting, grading queue, allocation, payouts, SLA reports. | Web dev, ops lead |
| [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md) | Colors, type, components, voice & tone; governs all four surfaces. See also `design/design-board.html`. | Designer, all devs |
| [11-ARCHITECTURE.md](11-ARCHITECTURE.md) | System architecture, stack decisions (Node 20/Express, Postgres 16, Kotlin/Compose, Swift/SwiftUI, vanilla-JS web), environments, conventions. | All devs |
| [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) | Build order, milestones, dependency graph — activates only after PRD approval. | All devs |
| [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) | Phase 0 validation → pilot launch (1 FPO, 2 crops, 15–25 buyers) → go/no-go gates. | Founder, ops lead |
| [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) | Daily hub operations: price setting, grading, allocation, delivery, payout runbook. | Ops lead |
| [15-TASKS-BACKLOG.md](15-TASKS-BACKLOG.md) | The living backlog, sequenced against 12-DEVELOPMENT-PLAN.md. | Everyone |
| [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) | Risk register: supply reliability, FPO dependence, buyer churn, regulatory, funding. | Founder |
| [17-FUNDRAISE-FINANCE.md](17-FUNDRAISE-FINANCE.md) | Burn plan, raise narrative, milestones-to-money mapping. | Founder, investors |
| [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) | Incorporation, FSSAI, GST, APMC rules, FPO/buyer agreements, trademark, data protection. | Founder, CA |
| [19-PRD-CMS-ANALYTICS.md](19-PRD-CMS-ANALYTICS.md) | CMS/Analytics module of the Console: who sold what, who bought what, app downloads, active users, trend graphs, CSV export. | Founder, ops lead, web dev |

## Reading order for a new team member

**Day 1 — understand the company (90 minutes):**
1. This README (you are here).
2. [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) — the constitution. Everything else is subordinate to it.
3. [01-VISION-MISSION.md](01-VISION-MISSION.md) — why this exists and where it goes.
4. [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md) — why now, why here, why us; who died trying and why we won't.

**Day 1–2 — understand the business:**
5. [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)
6. [04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md)
7. [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) — especially Phase 0 and the kill/pivot criterion.

**Day 2–3 — understand the product:**
8. [05-PRODUCT-OVERVIEW.md](05-PRODUCT-OVERVIEW.md)
9. [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)
10. Then the PRD for your surface: [06-PRD-BACKEND.md](06-PRD-BACKEND.md) / [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) / [08-PRD-WEBSITE.md](08-PRD-WEBSITE.md) / [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md) with its sibling Console module [19-PRD-CMS-ANALYTICS.md](19-PRD-CMS-ANALYTICS.md) — plus [11-ARCHITECTURE.md](11-ARCHITECTURE.md).

**Before you write any code:**
11. [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) and [15-TASKS-BACKLOG.md](15-TASKS-BACKLOG.md) — and confirm the PRDs are founder-approved. If they are not, you plan; you do not build (rule 3).

**Ongoing reference:** [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md), [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md), [17-FUNDRAISE-FINANCE.md](17-FUNDRAISE-FINANCE.md), [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md).

## Current status & immediate next steps

| Item | Status |
|---|---|
| Planning corpus (this directory) | In progress — being written now |
| Development | **PAUSED** until PRD approval (rule 3) |
| Phase 0 validation (10 buyer interviews, 1 FPO MoU, mandi shadow, trademark check, 2 beachhead crops frozen, real unit economics) | Not started — the very next real-world action; details in [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) |
| Kill/pivot criterion | <3 of 10 Surat HoReCa buyers willing to switch → pivot to FPO-SaaS-first wedge |
| Existing code scaffolds | Frozen; will be re-reviewed against approved PRDs |
| Pitch deck | `KisanSetu-Pitch-Deck-v2.pptx` (July 2026 figures) in `~/Downloads/` |

## Conventions used across the Brain

- **Cross-references** are by exact filename (e.g. "see [06-PRD-BACKEND.md](06-PRD-BACKEND.md)").
- **Numbers are sourced or labeled.** Verified figures cite their source and date; founder estimates are marked as such. Anything illustrative (e.g. ₹2,000 AOV) is flagged for Phase 0 validation.
- **Domain terms are region-agnostic** in schemas and specs (`reference_market_price`, never `mandi_price`); India-specific words appear only in India UI copy. See rule 2.
- **No TBDs without an owner and a deadline.**
- Older draft docs at `~/Desktop/KisanSetu-Docs/` (PLAN.md, TASKS.md, ARCHITECTURE.md, DESIGN.md) are historical input. Where they conflict with this Brain (e.g. Surat-only framing), **this Brain wins.**
