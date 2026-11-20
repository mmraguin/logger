# logger — Claude Reference

> Read MEMORY.md before starting any session. It holds the decisions log, workbook architecture, money rules and SOPs. Move resolved entries to MEMORY-ARCHIVE.md when a feature is stable and unlikely to change. Never load MEMORY-ARCHIVE.md unless asked.

---

## What this is

Mobile-first transaction logger companion to `mmraguin.github.io/tab/`. Handles all writes to the Google Sheet. Files: `index.html` (all HTML + JS), `finance-logger.css` (all styles, the only stylesheet), `Images/` (`favicon.png`, `logo.png`), `apps-script/` (Apps Script mirror). Visual system matches the tab app (Inter, dark neutral surfaces, lavender primary, tokens in `:root` — old token names kept as aliases because the JS uses `var(--muted)` inline). The tab app writes the `tab_v1` cache this app reads; keep that shape. GitHub Pages — push to main.

```
WEBHOOK     = 'https://script.google.com/macros/s/AKfycbwfl8-4sVKehGHxjGJirXVOYoTOZyinPFijxhQxJVVZQbWFaeBv53hr5HHuoPzYCg5z-Q/exec'  (write)
TAB_WEBHOOK = 'https://script.google.com/macros/s/AKfycbxy2zzSSdVP4zoLO8em6hUkAnNran5lq2_U1QIbMgVRutHuVeE_KBRzkRiEbc9X7GeMQA/exec'  (read — a different deployment from the tab app's WEBHOOK, on purpose; never unify them)
```

**Role:** Finance-sheet engineer and thinking partner for Miles. Maintain and extend his personal budget Google Sheet (V1) and its Apps Script, keep the logic intact, and give honest feedback over agreeable feedback.

---

## Files and scripts (what each one does)

| File | What it does |
|------|--------------|
| `index.html` | The logger page: all HTML and JS. Log, reconcile and manual screens; posts to `WEBHOOK`, reads balances from `TAB_WEBHOOK`. |
| `finance-logger.css` | The only stylesheet. |
| `Images/favicon.png` | Favicon, apple-touch-icon and the back-to-tab button icon. |
| `Images/logo.png` | Wordmark in the header. |
| `.nojekyll` | Empty file that tells GitHub Pages to skip Jekyll and serve the files as is. Keep it. |
| `.gitignore` | Ignores `.DS_Store`. |
| `ROADMAP.md` | Open work only. |
| `MEMORY.md` / `MEMORY-ARCHIVE.md` | Active decisions and workbook notes / resolved history. |

**`apps-script/`** is a local-only mirror of the Google Sheet's Apps Script project. It is git-ignored on purpose: `Code.gs` hardcodes the Sheet ID and this repo is public. It runs only inside Google, never on GitHub Pages. Keep it identical to the live project. Never commit it.

| Script | What it does |
|--------|--------------|
| `Code.gs` | The two web endpoints. `doPost` takes rows from the logger into the Log tab. `doGet` serves the sheet data to tab and the logger. Redeploy only when these change. |
| `Activity.gs` | `syncActivity` (runs every minute) copies Log into Activity and computes every column. |
| `Budget.gs` | Budget helpers: months, goals, Left to assign, Needed, the Moves log, and Open next month. |
| `Panel.gs` | The assign-and-move panel (Auto-fill by priority, Top up savings, Cover overspent). |
| `History.gs` | `refreshLive` (hourly) rebuilds LiveHistory. `closeMonth` archives a finished month. |
| `Core.gs` | The sheet menu and small helpers the other files use. Needed by all. |
| `Backup.gs` | `makeBackup`. Run before any big change. |
| `Readme.gs` | Rebuilds the README tab inside the sheet. Optional, safe to skip. |

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

**Working rules (Sheet and Apps Script)**
- Run makeBackup before any change to the sheet.
- Use only the Apps Script opened from the YNAB sheet through Extensions. The backup copies carry their own scripts.
- Editing code changes nothing live for web apps. Redeploy only when doPost or doGet change.
- Replies follow Miles's style: short active sentences, no em dashes, semicolons, asterisks or quotation marks, no wrap-up phrases.

---

## Resources

| File | Read when... |
|---|---|
| `MEMORY.md` | Start of every session. Holds decisions, workbook architecture, money rules, SOPs |
| `apps-script/` | Changing or debugging any script. Mirror of the live Apps Script project, one file per script |
| `apps-script/Readme.gs` | Rebuilding the README tab inside the sheet |
| `Personal Budget Workbook/inputs/web-pages/` | Changing the dashboard or the logger page. Snapshots only, the live pages are on GitHub |
| `Personal Budget Workbook/outputs/retired-scripts/` | Looking up how a past one-time step worked. Never paste these back into the project |
| `ROADMAP.md` | Planning next features |

**File routing**

| File type | Destination |
|---|---|
| Source materials (page snapshots, exports, backups) | `Personal Budget Workbook/inputs/` |
| Outputs (live scripts, retired scripts) | `Personal Budget Workbook/outputs/` |

**After each session**
1. Ask: "Should I log any key decisions from this session to MEMORY.md?" Wait for confirmation before writing.
2. Keep `outputs/apps-script/` identical to the Apps Script project. When a script changes, copy the new text there the same day.
3. Never delete files in `inputs/` without explicit confirmation.

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
