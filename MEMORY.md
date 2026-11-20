# logger — Memory

> Keep lean. Max ~15 active entries. When a feature is stable and no longer changing, move its entry to MEMORY-ARCHIVE.md.
> Claude: check this before every session. Update after any decision or shipped change.
> Covers logger, the tab dashboard, and the Personal Budget Workbook (Sheet and Apps Script). Last updated: 2026-11-20.

---

## In Flight

### Bank Statement Reconciliation (Feature 2)
**Status:** scoped, not built
**Scope:** New screen in logger. Upload bank statement CSV/PDF, parse transactions, cross-reference against Activity sheet, flag missed or mislogged, allow add / edit / remove, write via WEBHOOK.
**Format notes:** GCash, Maya, UnionBank = CSV. UnionBank Credit = PDF (harder, parse last).
**Match logic:** date + amount fuzzy match. Primary goal: catch missed transactions and credit card discrepancies.
**Lives in:** logger (has write access), not tab.

---


## Webhooks (intentionally different)
- Logger `WEBHOOK` (write, POST) and `TAB_WEBHOOK` (read, GET) are separate Apps Script deployments, and the tab app has its own read `WEBHOOK`. They are meant to differ. Do not unify them. URLs are in CLAUDE.md.

## Recent Decisions

### Logger and tab

| Date | Decision | Rationale |
|------|----------|-----------|
| Jul 3 2026 | Reconciliation type is `Reconciliation`, not `Balance` | `Balance` = absolute snapshot. `Reconciliation` = signed diff. Separate types allow filtering in Sheet. |
| Jul 3 2026 | Log the diff, not the actual balance | Audit trail. Can trace how far off each snapshot was over time. |
| Jul 3 2026 | Reconcile tab fetches live from TAB_WEBHOOK, localStorage as fallback | localStorage not reliably shared across apps on mobile (PWA context, browser sandboxing). Direct fetch is the correct primary path. |
| Jul 3 2026 | Reconcile diff memo = `Logged ₱X · Actual ₱Y` | Full audit context in the Sheet without cross-referencing. |
| Jul 3 2026 | Added .nojekyll to skip Jekyll processing | Plain HTML repo. Jekyll was causing intermittent Pages deploy failures. |
| Nov 20 2026 | One brand icon: `Images/favicon.png` (favicon, apple-touch-icon, back-to-tab button) and `Images/logo.png` (wordmark), copied from tab. Old `icon-192`, `apple-touch-icon`, `logo`, `tab-mark` PNGs and the orphan `style.css` removed. | Single source for brand images across both repos. |
| Jul 3 2026 | Webhook over direct Sheets API | API key + Sheet ID would both be client-side in a public repo. Personal finance data, not worth the privacy tradeoff. |
| Jul 3 2026 | No Apps Script warm-up trigger | Probabilistic, burns quota, doesn't fix slow reads. |
| Jul 3 2026 | Accept Apps Script cold start; fix perceived perf with loading states | Cache-first already covers the blank screen. Cold start only matters on explicit refresh. |

### Budget workbook (2026-11-19)

- Kept V1 and its formatting. V2 rebuild is shelved. Chose scripts over formulas for speed.
- Trimmed spare rows from the big tabs. Hid dead tabs: Sheet40, YNAB, Transactions, Form Responses 1.
- Built the Goals tab. It sets priority and goal type per category.
- Retired Budget column K (FutureBudgeted) and the CurrentYearFormatting formulas. LiveHistory and History hold values.
- Built the assign and move panel with Auto-fill by priority, Top up savings and Cover overspent. Replaced the Moves-driven tools. The Moves tab stays as a log.
- Fixed the tab. sheet month headers so each month block reads its own month. Overspent last month reads negative Available from the previous month.
- Cut over Activity from formulas to script values. GroupSort reads the live Rollups table (H to M), so the dashboard sorts in true group order.
- Moved the logger to a new deployment in the YNAB project. Code.gs writes phone entries to Log. The old logger deployment lived in a project that could not be found. Consolidated the project to eight files.
- Added a README tab, built by Readme.gs.
- ExternalInflow is all money coming in: Income plus Reimbursement, in every row.

---

## Workbook Reference

### Goal and scope
- The workbook is the single source of all financial movement.
- Current year: tracks transactions, sets the budget, assigns money, tracks IOUs and credit.
- Previous years: a historical archive for year on year comparison.

### Architecture
- Phone logger (GitHub Pages, repo mmraguin/logger) posts ten columns to the logger web app. doPost writes them to the Log tab.
- syncActivity runs every minute. It computes every Activity column from Log and writes rows 108 and up. Log row k lands in Activity row k plus 106. Rows 2 to 107 are typed values from January and February.
- Activity feeds Budget, MonthSnapshot, AccountSnapshot and the tab. sheet. MonthSnapshot and the tab. sheet use pivots of Activity.
- refreshLive rebuilds LiveHistory hourly from History plus the current year.
- Dashboard (GitHub Pages, repo mmraguin/tab) reads the dashboard web app through doGet.
- Activity columns A to Y: BudgetRowID, TxnID, GroupKey, GroupSort, CategoryID, AccountID, TransactionDate, BillingCycle, Type, Amount, Merchant, Payment Method, Category, Group, Memo, Review, CCExpense, CCPayment, TrueBalance, TrueExpense, TrueIncome, AdjustedIncome, ExternalInflow, NetCashImpact, NetReimbursement.
- Budget columns: A BudgetRowID, B GroupKey, C GroupID, D CategoryID, E GroupSort, F MonthStart, G Group, H Category, I Yearly Goal, J Monthly Goal, K retired, L ManualAssigned, M AssignedThisMonth (L plus O), N Activity, O Rollover, P Available, Q Needed, R Status. A to J are values.

### Money rules
- Ready to Assign = CashStart + AdjustedIncome − TotalAssigned (sum of Budget M).
- Payday cutoff is the 27th. UnionBank Credit cutoff is the 14th. Some transfers shift to the earlier month.
- AdjustedIncome excludes income within two days of the 27th.
- Reimbursement carries an expense category. It nets against the expense and restores that category.
- Rollover includes negatives. Every category rolls over.
- Cash End on MonthSnapshot reads AccountSnapshot, not ExternalInflow.
- Needed: monthly spending categories use monthly goal minus assigned and rollover. Saving categories (priority 3 or lower with a target) use the monthly amount capped at the gap to the target. ByDate spreads the remaining target over the months left.
- Priority: 1 essentials (electricity, groceries, Lulay, phone, medicine, doctor, pet food, litter, transpo). 2 all other groups except sinking, emergency and goals. 3 sinking and emergency. 4 goals.
- Money assigned to a future month reduces the current month Ready to Assign.
- Assigning writes only to Budget column L and logs to Moves. Miles assigns at payday or when he updates income and sees money left.

### SOPs
- Before any change: run makeBackup. The copy carries its own script. Never edit it.
- Payday: log the income, open the panel, Auto-fill by priority, Top up savings, Cover overspent, Apply.
- Each month: Open next month. After the month ends, Close a month.
- Each year: close every month. Set CurrentYearFormatting K1 to the new year. Not yet tested.
- Redeploy only when doPost or doGet change: Deploy, Manage deployments, New version. Roll back by choosing the previous version.
- Triggers that must exist: syncActivity every minute, refreshLive every hour.

### Open questions and next work
- Credit, Accounts and IOUs tabs still run on formulas. Next speed gain.
- Retire the old Plan tab now that Goals is final.
- Rename the Q2_2026_Groups table.
- Confirm the hourly refreshLive trigger exists.
- Test the year change routine before January.
- TrueIncome has no reader in MonthSnapshot. Keep or drop.
