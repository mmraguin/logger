# logger — Roadmap

Only open work lives here. Finished items are logged in MEMORY-ARCHIVE.md.

## Next

- **Bank statement reconciliation (Feature 2)** — upload a statement, match on date + amount against Activity, flag missed or mislogged entries, allow add / edit / remove, write via `WEBHOOK`. CSV first (GCash, Maya, UnionBank), PDF last (UnionBank Credit).

## Workbook (Apps Script / Sheet)

- Move the Credit, Accounts and IOUs tabs off formulas (next speed gain).
- Retire the old Plan tab now that Goals is final.
- Rename the Q2_2026_Groups table.
- Confirm the hourly `refreshLive` trigger exists.
- Test the year-change routine before January.
- Decide whether to keep or drop TrueIncome (no reader in MonthSnapshot).
