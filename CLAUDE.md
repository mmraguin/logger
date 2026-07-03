# logger — Claude Reference

> Read MEMORY.md before starting any session. Move resolved entries to MEMORY-ARCHIVE.md when a feature is stable and unlikely to change.

---

## What this is

Mobile-first transaction logger companion to `mmraguin.github.io/tab/`. Handles all writes to the Google Sheet. Two files: `index.html` (all HTML + CSS + JS), `finance-logger.css` (all styles). GitHub Pages — push to main.

```
WEBHOOK     = 'https://script.google.com/macros/s/AKfycbwfl8-4sVKehGHxjGJirXVOYoTOZyinPFijxhQxJVVZQbWFaeBv53hr5HHuoPzYCg5z-Q/exec'  (write)
TAB_WEBHOOK = 'https://script.google.com/macros/s/AKfycbxy2zzSSdVP4zoLO8em6hUkAnNran5lq2_U1QIbMgVRutHuVeE_KBRzkRiEbc9X7GeMQA/exec'  (read — same as tab app)
```

---

## Active Projects

See MEMORY.md for full context. Current in-flight:

- **Bank statement reconciliation (Feature 2)** — not built. Scoped: upload bank statement CSV (GCash, Maya, UnionBank export CSV; UnionBank Credit exports PDF — harder), cross-reference against Activity sheet transactions, flag missed/mislogged entries, allow add/edit/remove before writing via WEBHOOK.

---

## Rules — follow these exactly

**Code style**
- Modern JS is fine here (`const`, `let`, arrow functions, template literals) — unlike tab which uses `var` only.
- All styles go in `finance-logger.css`, never in `<style>` blocks.

**Write path**
- All writes go via `appendRows(rows, d)`. Pass `rows` as array of arrays, `d` as descriptor for success screen + dupe detection.
- Row format: `[timestamp, type, amount, sheetDate, merchant, paymentMethod, category, group, memo, review]`
- Amount sign: negative for Expense/Transfer-out, positive for Income/Reimbursement/Transfer-in. Reconciliation uses signed diff (positive = you have more than logged, negative = you have less).
- `toSheetDate(iso)` converts YYYY-MM-DD → M/D/YYYY for Sheets.

**Transaction types**
- Valid types: `Expense`, `Income`, `Transfer`, `Balance`, `Reimbursement`, `Reconciliation`
- `Balance` = account snapshot (amount = actual balance, not a diff)
- `Reconciliation` = account diff (amount = actual − logged). Logged with memo `Logged ₱X · Actual ₱Y` for audit trail.

**Reconcile tab**
- Reads live from `TAB_WEBHOOK` (GET) on open. Falls back to `tab_v1` localStorage if fetch fails.
- Parses `data.accountSnapshot` — finds most recent entry per account via `parseTabDate()` (UTC ISO → local Date).
- Logs signed diff as `Reconciliation` type — not `Balance`.
- `parseTabDate()` mirrors tab's `parseDate()`: extract UTC components, construct local Date.

**Adding a new mode**
- Add to `VALID_MODES` array in `selectMode()`.
- Add `else if (mode === 'newmode')` in both `selectMode()` and `cancelConfirm()`.
- Add a tab in `.tabbar` HTML with matching `data-mode`.
- Add `.tab.active[data-mode="newmode"]` color in `finance-logger.css`.
- Add `<div class="screen" id="screen-newmode">` in content area.

---

## Architecture

```
User action (screenshot / manual / reconcile)
    │
    ├── screenshot → Gemini API (parse) → buildReceiptCard() → appendRows()
    ├── manual     → buildManualForm()  → submitManual()    → appendRows()
    ├── transfer   → buildManualForm()  → submitManual()    → appendRows() ×2
    └── reconcile  → TAB_WEBHOOK (GET) → logReconciliation() → appendRows()
                          │
                    appendRows() → POST WEBHOOK → Google Sheet (Log tab)
```

**Modes:** `expense` (default), `income`, `transfer`, `manual`, `reconcile`

**Merchant rules:** `MERCHANT_RULES` array — keyword → `{ paymentMethod, category }`. Applied after Gemini parse and on manual merchant input. Auto-fills Via and Category fields.

---

## Reference

### Constants

```javascript
PAYMENT_METHODS = ['GCash','Maribank','BDO','BPI','Maya','UnionBank','UnionBank Credit','Grab Wallet','Shopee Pay','Cash','BanKo','CIMB']
INCOME_CATS     = ['Salary','Freelance','Gift','IOU']
EXPENSE_CATS    = all keys in CATEGORY_MAP except INCOME_CATS
```

### Row format (what gets written to Sheet)

| Index | Field | Notes |
|-------|-------|-------|
| 0 | Timestamp | `new Date().toISOString()` |
| 1 | Type | Expense / Income / Transfer / Balance / Reimbursement / Reconciliation |
| 2 | Amount | Signed (negative = expense/outflow) |
| 3 | Date | M/D/YYYY — use `toSheetDate()` |
| 4 | Merchant | Payee / label |
| 5 | Payment Method | Must match tab's ACCOUNTS list |
| 6 | Category | From CATEGORY_MAP keys |
| 7 | Group | From CATEGORY_MAP values |
| 8 | Memo | Optional notes |
| 9 | Review | Myself / Others / Both (blank for Reimbursement / Balance / Reconciliation) |

### Settings (localStorage)

| Key | Value |
|-----|-------|
| `gemini` | Gemini API key (for screenshot parsing) |
| `sheetId` | Google Sheet ID |
| `googleApi` | Google API key (Sheets API — not currently used for reads) |
| `lastMode` | Last active tab mode |
| `lastPaymentMethod` | Pre-fills Via field on next manual entry |
