# CLAUDE.md — KisanSetu Brain

This repo is the **planning corpus** (source of truth) for KisanSetu — docs only, no code. Code lives in sibling repos (`KisanSetu-Backend`, `-Website`, `-Android`, `-iOS`), which are **frozen** until the founder approves the PRDs (Golden Rule #3).

## Read first, in this order
1. [README.md](README.md) — index, status, reading order.
2. [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) — the constitution; every edit is subordinate to it.
3. [21-AI-EXECUTION-PLAYBOOK.md](21-AI-EXECUTION-PLAYBOOK.md) — how AI-assisted work runs here: skills, MCP, hooks, agent boundaries, testing, build order.

## Fixed decisions (do not relitigate; anchor docs in parentheses)
- **Payment gateway = Razorpay, fixed (founder, 14 Jul 2026).** All payment work — backend payouts/webhooks/collections, Android, iOS — starts from [21-AI-EXECUTION-PLAYBOOK.md](21-AI-EXECUTION-PLAYBOOK.md) §10 and [06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6.8. Provider interface stays (Rule 2); the India instance is Razorpay, no other PSP evaluated.
- **Backend stack = TypeScript + Express + Drizzle ORM `v1.0.0-rc.4` + Postgres 16** ([20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §1).
- **Mobile = Kotlin/Compose + Swift/SwiftUI**, Fancall MVVM below the View; **website & Console = vanilla JS** (intentional).
- **One mobile app, two user types; Android ↔ iOS screen-for-screen parity** (Rule 1).

## Editing rules for this repo
- Cross-reference by exact filename (`see 06-PRD-BACKEND.md §6.8`); verify the target section exists.
- Numbers are sourced or labeled (founder estimate / illustrative → flagged for Phase 0 validation).
- No TBD without an owner and a deadline.
- Region-agnostic terms in anything schema/spec-shaped (`tax_id`, `reference_market_price`, `postal_code`); India-specific words only in India UI copy (Rule 2).
- A change that touches a golden rule needs founder sign-off + changelog entry in [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) + same-sitting sweep of affected docs.
- Keep each doc's **Last updated** header line and changelog current when editing it.
- `_archive/` is historical input — never edit it, never cite it as authority.

## What Claude must NOT do here (founder-only)
Approve PRDs, change golden rules or the two metrics, make pricing/go-no-go decisions, or mark anything "APPROVED". Full boundary list: [21-AI-EXECUTION-PLAYBOOK.md](21-AI-EXECUTION-PLAYBOOK.md) §7.3.
