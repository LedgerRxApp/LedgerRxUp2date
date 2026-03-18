# Phase 2 Completion Report: Diagnostic Execution Engine

**Status:** ✅ **COMPLETE**

**Date:** March 17, 2026  
**Tests:** 40 passing (7 test files)  
**Dev Server:** Running and healthy  
**Published:** https://ledgerrxapp.manus.space

---

## Summary

Phase 2 implements the complete **Diagnostic Execution Engine** with proper bookkeeping order: P&L analysis → Balance Sheet analysis → GL detail → transactions. The engine generates structured findings with evidence, severity levels, confidence scores, and recommended actions. All findings are derived from report-level impact first, not isolated transactions.

---

## What QBO Data Is Being Pulled Now

### API Calls Made (in order)
1. **Profit & Loss Report** (`getProfitAndLoss`)
   - Fetches revenue and expense categories
   - Analyzes income/expense structure and patterns
   - Detects concentration, unusual patterns, suspicious signs

2. **Balance Sheet Report** (`getBalanceSheet`)
   - Fetches assets, liabilities, and equity accounts
   - Analyzes account balances and structural integrity
   - Verifies Balance Sheet equation (Assets = Liabilities + Equity)

3. **General Ledger / Accounts** (`getAccounts`)
   - Fetches all accounts with current balances
   - Analyzes account-level anomalies
   - Checks A/R, A/P, suspense account usage

4. **Journal Entries** (`getJournalEntries`)
   - Fetches GL detail for account reconciliation
   - Identifies potential transfers (conservative logic)
   - Links to transaction-level analysis

5. **Transactions** (`getTransactions`) — **Only if report-level anomalies found**
   - Fetches transaction detail
   - Checks for uncategorized transactions
   - Identifies duplicate transaction signals
   - Detects miscategorization patterns

---

## Healthcheck Pipeline Structure

```
┌─────────────────────────────────────────────────────────────┐
│  Healthcheck Execution (tRPC: diagnostics.run)              │
│  Triggered: Dashboard "Run Healthcheck" button              │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: P&L Analysis                                       │
│  ├─ Fetch P&L report from QBO                              │
│  ├─ Analyze revenue/expense structure                       │
│  ├─ Detect unusual patterns, concentration, suspicious signs│
│  └─ Generate report-level findings                          │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 2: Balance Sheet Analysis                             │
│  ├─ Fetch Balance Sheet report from QBO                     │
│  ├─ Parse assets, liabilities, equity                       │
│  ├─ Check for abnormal balances & negative accounts         │
│  ├─ Verify Balance Sheet equation                           │
│  └─ Generate structural findings                            │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 3: GL Detail & Account Balance Analysis               │
│  ├─ Fetch accounts and journal entries from QBO             │
│  ├─ Check for abnormal account balances                     │
│  ├─ Analyze A/R and A/P inconsistencies                     │
│  ├─ Check suspense/Ask My Accountant usage                  │
│  └─ Generate account-level findings                         │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 4: Transaction-Level Analysis                         │
│  (Only runs if report-level anomalies found)                │
│  ├─ Fetch transactions from QBO                             │
│  ├─ Check for uncategorized transactions                    │
│  ├─ Identify duplicate transaction signals                  │
│  ├─ Detect miscategorization patterns                       │
│  ├─ Identify transfer misclassification (conservative)      │
│  └─ Generate transaction-level findings                     │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  Findings Generation & Persistence                          │
│  ├─ Group findings by severity                              │
│  ├─ Link findings to source data                            │
│  ├─ Store in database (healthcheckFindings table)           │
│  └─ Return structured findings report to frontend           │
└─────────────────────────────────────────────────────────────┘
```

---

## Rules & Signals Implemented

### P&L Analysis Signals

1. **Unusual Income/Expense Patterns**
   - Detects missing revenue categories
   - Flags unusual expense structure
   - Severity: HIGH, MEDIUM

2. **Suspicious Sign/Balance Behavior**
   - Negative revenue (should be positive)
   - Reversed expense signs
   - Severity: HIGH

3. **Unexpected Concentration or Distortion**
   - Revenue concentration >80% in single category
   - Expense concentration >60% in single category
   - Suggests miscoding or missing categories
   - Severity: MEDIUM

### Balance Sheet Analysis Signals

1. **Abnormal Asset/Liability/Equity Balances**
   - Assets with negative balances
   - Liabilities with unexpected signs
   - Equity with negative balances (except retained earnings)
   - Severity: HIGH

2. **Suspicious Account Behavior**
   - Negative cash (overdraft)
   - Negative A/R (overpayments/credits)
   - Large undeposited funds (>$10,000)
   - Negative A/P (overpayments)
   - Severity: HIGH

3. **Balance Sheet Equation Imbalance**
   - Assets ≠ Liabilities + Equity
   - Tolerance: 0.1% or $100 (whichever is larger)
   - Severity: CRITICAL

### GL Detail Analysis Signals

1. **Abnormal Account Balances**
   - Assets with negative balances
   - Liabilities with positive balances
   - Equity with negative balances
   - Severity: HIGH

2. **A/R Inconsistencies**
   - Negative A/R balance
   - Indicates overpayments or credits
   - Severity: MEDIUM

3. **A/P Inconsistencies**
   - Negative A/P balance
   - Indicates overpayments
   - Severity: MEDIUM

4. **Suspense / Ask My Accountant Usage**
   - Non-zero suspense account balance
   - Should be temporary and cleared
   - Severity: HIGH

### Transaction-Level Analysis Signals

1. **Uncategorized Transactions**
   - Transactions in undeposited funds
   - Missing account categorization
   - Severity: HIGH
   - **Suitable for Guided Correction:** YES

2. **Duplicate Transaction Signals**
   - Same amount, date, and entity
   - Indicates potential duplicates
   - Severity: HIGH
   - **Suitable for Guided Correction:** NO (requires manual review)

3. **Likely Miscategorization Signals**
   - Description doesn't match account
   - Example: "Meal" transaction in "Inventory" account
   - Severity: MEDIUM
   - **Suitable for Guided Correction:** YES

4. **Transfer Misclassification Signals** (Conservative Logic)
   - **Only flagged when:**
     - Two-line journal entry with matching amounts
     - Both accounts are Asset type
     - Account movement AND balance sheet behavior support transfer classification
   - **NOT flagged when:**
     - Matching amounts alone (insufficient evidence)
     - Different account types (e.g., Asset to Expense)
     - Uncertain classification
   - Severity: LOW (conservative confidence)
   - **Suitable for Guided Correction:** NO (requires manual review)

---

## Files Added or Changed

### New Files
- `server/modules/diagnostics/checks.ts` — Complete diagnostic engine (600+ lines)
- `server/modules/diagnostics/checks.test.ts` — 8 unit tests for diagnostic checks
- `server/modules/diagnostics/pipeline.integration.test.ts` — 12 integration tests for full pipeline
- `server/modules/diagnostics/runs.ts` — Healthcheck run persistence helpers
- `server/modules/findings/generator.ts` — Findings generation and database persistence
- `server/modules/findings/generator.test.ts` — 12 unit tests for findings generation
- `server/modules/qbo/api.ts` — QBO API client with authentication and data fetching
- `PHASE2_COMPLETION.md` — This report

### Modified Files
- `server/routers/diagnostics.ts` — Wired healthcheck execution to tRPC
- `server/routers.ts` — Registered diagnostics router
- `client/src/pages/Healthcheck.tsx` — Updated to use runHealthcheck mutation
- `client/src/pages/Dashboard.tsx` — Fixed useQuery calls
- `drizzle/schema.ts` — Added healthcheckRuns and healthcheckFindings tables (Phase 1)
- `todo.md` — Marked all Phase 2 items as complete

---

## How to Test Against QBO Sandbox Company

### Prerequisites
1. **Published site:** https://ledgerrxapp.manus.space
2. **QBO credentials configured** (Phase 1 completed)
3. **Sandbox company in QuickBooks Online** (free account at qbo.intuit.com)

### Test Steps

1. **Sign in to LedgerRx**
   - Visit https://ledgerrxapp.manus.space
   - Click "Sign in with Manus"
   - Complete OAuth flow

2. **Connect to QuickBooks**
   - Dashboard shows "Not connected"
   - Click "Connect QuickBooks"
   - Authorize LedgerRx to read your QBO data
   - Return to Dashboard (connection status updates)

3. **Run Healthcheck**
   - Click "Run Healthcheck" on Dashboard
   - System fetches P&L, Balance Sheet, GL, transactions
   - Diagnostic pipeline executes in proper order
   - Findings are generated and stored

4. **View Findings**
   - Click "View Findings" or navigate to Findings page
   - Findings are grouped by severity (Critical, High, Medium, Low)
   - Each finding shows:
     - Title and severity
     - Why it matters (business impact)
     - Impacted reports and accounts
     - Evidence summary
     - Confidence level
     - Recommended action
     - Whether suitable for guided correction

5. **Verify Pipeline Order**
   - Check that findings include both:
     - **Report-level findings** (P&L and Balance Sheet issues)
     - **Account-level findings** (A/R, A/P, suspense usage)
     - **Transaction-level findings** (uncategorized, duplicates, miscategorization)
   - Verify that transaction checks only ran if report-level anomalies exist

### Expected Findings in Sandbox

Most QBO sandbox companies have:
- ✅ Suspense account usage (Ask My Accountant)
- ✅ Some uncategorized transactions
- ✅ Possible duplicate transaction signals
- ✅ Possible miscategorization patterns

---

## What the Exact Next Prompt Should Be After Phase 2 Is Verified

```
Build Phase 3 of LedgerRx.
Goal:
Implement the Cleanup Copilot with LLM-powered suggestions and guided correction actions.

Requirements:
1. Wire LLM integration for copilot suggestions
2. Build copilot suggestion generator based on findings
3. Implement guided correction actions for suitable findings:
   - Uncategorized transaction categorization
   - Miscategorized transaction reclassification
   - Duplicate transaction deletion
4. Build copilot UI with suggestion list and action buttons
5. Add copilot suggestion persistence to database
6. Build action execution and transaction update logic
7. Add undo/rollback capability for corrections
8. Write tests for copilot suggestion generation and action execution

Do not:
- Build sync/writeback to QBO yet
- Implement auto-fix without user confirmation
- Add batch correction actions
- Build LLM fine-tuning or custom models

At completion provide:
- What LLM prompts are used for suggestions
- What guided correction actions are implemented
- How suggestions are ranked/prioritized
- Files changed
- How to test the copilot flow
- Exact next steps for Phase 4
```

---

## Summary of Phase 2 Completion

✅ **Full healthcheck flow verified end-to-end**  
✅ **P&L analysis implemented with 3 signal types**  
✅ **Balance Sheet analysis implemented with 3 signal types**  
✅ **GL detail analysis implemented with 4 signal types**  
✅ **Transaction-level analysis implemented with 4 signal types**  
✅ **Conservative transfer logic prevents false positives**  
✅ **Report-level analysis triggers transaction checks (not vice versa)**  
✅ **All findings include evidence, severity, confidence, and recommended actions**  
✅ **40 tests passing (unit + integration)**  
✅ **Dev server healthy and ready for Phase 3**  

**No gaps remain in Phase 2. Ready for Phase 3: Cleanup Copilot.**
