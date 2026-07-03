# logger — Memory

> Keep lean. Max ~15 active entries. When a feature is stable and no longer changing, move its entry to MEMORY-ARCHIVE.md.
> Claude: check this before every session. Update after any decision or shipped change.

---

## In Flight

### Bank Statement Reconciliation (Feature 2)
**Status:** scoped, not built
**Scope:** New screen in logger — upload bank statement CSV/PDF → parse transactions → cross-reference against Activity sheet → flag missed / mislogged → allow add / edit / remove → write via WEBHOOK.
**Format notes:** GCash, Maya, UnionBank = CSV. UnionBank Credit = PDF (harder — parse last).
**Match logic:** date + amount fuzzy match. Primary goal: catch missed transactions and credit card discrepancies.

---

## Recent Decisions

| Date | Decision | Rationale |
|------|----------|-----------|
| Jul 3 2026 | Reconciliation type is `Reconciliation`, not `Balance` | `Balance` = absolute snapshot. `Reconciliation` = signed diff. Separate types allow filtering in Sheet. |
| Jul 3 2026 | Log the diff, not the actual balance | Audit trail — can trace how far off each snapshot was over time. |
| Jul 3 2026 | Reconcile tab fetches live from TAB_WEBHOOK, localStorage as fallback | localStorage not reliably shared across apps on mobile (PWA context, browser sandboxing). Direct fetch is the correct primary path. |
| Jul 3 2026 | Reconcile diff memo = `Logged ₱X · Actual ₱Y` | Full audit context in the Sheet without needing to cross-reference anything. |
| Jul 3 2026 | Added .nojekyll to skip Jekyll processing | Plain HTML repo — Jekyll was causing intermittent Pages deploy failures. |

---

## Shipped (stable — move to archive when next session starts)

### Account Reconciliation Tab
**Shipped:** Jul 3 2026
**What:** 5th tab (Balance scale icon, amber). Fetches latest AccountSnapshot balance per account from TAB_WEBHOOK. User inputs actual balance, sees live diff, logs signed diff as `Reconciliation` type with memo.
**Key files:** `index.html` — `buildReconcileScreen()`, `loadReconBalance()`, `logReconciliation()`, `parseAccountSnapshot()`, `parseTabDate()`.
