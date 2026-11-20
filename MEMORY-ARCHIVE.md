# logger — Memory Archive

> Resolved entries rotated from MEMORY.md. Reference only — do not re-open unless the feature is actively changing again.

---

## Shipped (stable — move to archive when next session starts)

### Account Reconciliation Tab (logger)
**Shipped:** Jul 3 2026
**What:** 5th tab (Balance scale icon, amber). Fetches latest AccountSnapshot balance per account from TAB_WEBHOOK. User inputs actual balance, sees live diff, logs signed diff as `Reconciliation` type with memo.
**Key files:** `index.html` — `buildReconcileScreen()`, `loadReconBalance()`, `logReconciliation()`, `parseAccountSnapshot()`, `parseTabDate()`.

## Archived tab-side decisions (old tab design)

| Date | Decision | Rationale |
|------|----------|-----------|
| Jul 3 2026 | No percentage on group total header (tab) | Bar + `₱X / ₱Y` already encodes it. A third encoding adds no information. |
| Jul 3 2026 | Group total as header (label row), not a category card at the bottom (tab) | Bottom placement looked like another category. Header makes ownership clear. |
| Jul 3 2026 | `.goal-group` wrapper + `32px` gap between groups (tab) | Gestalt proximity: tight internal spacing, relaxed between-group spacing. |
| Jul 6 2026 | Pull to refresh built on `#pw`: `.ptr` indicator (rotates while dragging, spins during fetch), `overscroll-behavior: contain` on `.page` (tab) | Matches native feel and stops the browser's own bounce from fighting it. Threshold 64px, drag capped at 96px, 0.5x resistance. |
| Jul 7 2026 | Replaced tab `index.html`/`style.css` with the `tabv2` redesign (Instrument Sans / Inconsolata / Fraunces) | Prototyped in sibling `tabv2`, then promoted wholesale as the new visual system. |

Plan tab progress-bar item (obsolete): the Plan tab was replaced by Budget in the Nov 20 2026 tab redesign.


## Completed roadmap items

- Logger restyled to match the tab redesign (Nov 20 2026)
- Back-to-tab button, brand logo and favicon (Nov 20 2026)
- Account reconciliation tab (Jul 3 2026)
