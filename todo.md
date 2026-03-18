# LedgerRx Phase 1 TODO

## Architecture & Scaffold
- [x] Database schema: qboConnections table
- [x] Database schema: findings table
- [x] Database schema: copilotSuggestions table
- [x] Database schema: healthcheckRuns table
- [x] Apply schema migrations via webdev_execute_sql

## Server Modules
- [x] server/modules/auth/index.ts — session helpers
- [x] server/modules/qbo/index.ts — QuickBooks OAuth module
- [x] server/modules/qbo/oauth.ts — OAuth URL builder and token exchange scaffold
- [x] server/modules/diagnostics/index.ts — empty scaffold
- [x] server/modules/findings/index.ts — findings query helpers
- [x] server/modules/copilot/index.ts — copilot scaffold
- [x] server/modules/sync/index.ts — sync scaffold

## tRPC Routers
- [x] server/routers/qbo.ts — connect, callback, status, disconnect
- [x] server/routers/findings.ts — list, get, create
- [x] server/routers/diagnostics.ts — scaffold
- [x] server/routers/copilot.ts — scaffold
- [x] server/routers/sync.ts — scaffold
- [x] Wire all routers into server/routers.ts

## Frontend
- [x] Global theme (dark, professional palette) in index.css
- [x] App.tsx routing with DashboardLayout
- [x] Dashboard page with Connect QuickBooks state handling
- [x] Healthcheck run screen UI scaffold
- [x] Settings page (QBO credentials status)
- [x] Sidebar navigation with all module links

## Documentation
- [x] .env.example with all required variables
- [x] README.md with setup, redirect URI, credentials, and next steps

## Tests
- [x] Vitest: qbo router status procedure
- [x] Vitest: findings router list procedure

# LedgerRx Phase 2 TODO — Diagnostic Execution Engine

## QBO API Client
- [x] Build QBO API client with token refresh and error handling
- [x] Fetch P&L statement (ProfitAndLoss query)
- [x] Fetch Balance Sheet (BalanceSheet query)
- [x] Fetch General Ledger (JournalEntry query for account detail)
- [x] Fetch transactions (Transaction query with filters)
- [x] Cache strategy for API calls (deferred to Phase 3)

## Diagnostic Checks (in order)
- [x] P&L analysis: revenue/expense structure, missing accounts (scaffold)
- [x] Balance Sheet analysis: asset/liability/equity structure (scaffold)
- [x] Account-level detail: GL balances vs. P&L/BS
- [x] Uncategorized transactions check
- [x] Abnormal account balances check
- [x] Duplicate transaction signals check
- [x] Miscategorization signals check
- [x] Transfer misclassification check (conservative logic)
- [x] A/R inconsistencies check
- [x] A/P inconsistencies check
- [x] Suspense/Ask My Accountant usage check
- [x] Report-to-detail mismatches check (scaffold)
- [x] Reconciliation-impacting anomalies check (scaffold)

## Findings Generation
- [x] Finding data model with evidence, severity, confidence
- [x] Finding grouping by severity
- [x] Evidence linking to source data
- [x] Recommended actions per finding type

## Healthcheck Execution
- [x] Wire healthcheck run to tRPC procedure
- [x] Execute diagnostic pipeline
- [x] Store findings in database
- [x] Update healthcheck run status
- [x] Return structured findings report

## Testing
- [x] Vitest: QBO API client mocking
- [x] Vitest: each diagnostic check with sample data (8 tests)
- [x] Vitest: findings generation and evidence linking (12 tests)
- [x] Integration test: full healthcheck pipeline (all 28 tests passing)


# LedgerRx Phase 2 Refinement TODO — Accuracy & False Positive Reduction

## Account Detection Refactoring
- [x] Replace string-based account detection with QBO account type/subtype
- [x] Create account type helpers (isAsset, isLiability, isEquity, isExpense, isIncome)
- [x] Update P&L analysis to use account types instead of name matching
- [x] Update Balance Sheet analysis to use account types instead of name matching
- [x] Update GL detail analysis to use account types instead of name matching

## Materiality Awareness
- [x] Define materiality thresholds (% of total or absolute amounts)
- [x] Skip findings for amounts below materiality threshold
- [x] Prioritize findings by financial statement impact
- [x] Add materiality metadata to findings

## Transaction Drilldown Improvement
- [x] Only drill into transactions when specific findings require it
- [x] Create drilldown triggers for each finding type
- [x] Skip broad transaction checks if no report-level anomalies
- [x] Add transaction filter to only fetch relevant transactions

## Findings Quality Strengthening
- [x] Ensure each finding clearly ties to report-level impact
- [x] Improve evidence summaries to show financial impact
- [x] Add financial statement impact metrics to findings
- [x] Clarify recommended actions with specific next steps

## Testing & Verification
- [x] Write tests for refined account detection logic
- [x] Write tests for materiality thresholds
- [x] Write tests for transaction drilldown triggers
- [x] Verify false positive reduction
- [x] Verify diagnostic order remains correct


# LedgerRx Phase 2 Hardening — QBO Type/Subtype Classification & Accuracy

## Account Classification Refactoring
- [ ] Create QBO account type/subtype mapping to financial statement groups
- [ ] Replace string-based suspense account detection with type/subtype logic
- [ ] Replace string-based undeposited funds detection with type/subtype logic
- [ ] Replace string-based A/R detection with type/subtype logic
- [ ] Replace string-based A/P detection with type/subtype logic
- [ ] Update P&L analysis to use account types
- [ ] Update Balance Sheet analysis to use account types
- [ ] Update GL detail analysis to use account types

## Materiality Filtering
- [ ] Add materiality thresholds ($100 minimum, 0.1% of total)
- [ ] Skip findings below materiality in P&L analysis
- [ ] Skip findings below materiality in Balance Sheet analysis
- [ ] Skip findings below materiality in GL detail analysis
- [ ] Skip findings below materiality in transaction analysis

## Transaction Drilldown Narrowing
- [ ] Only drill into transactions if P&L anomalies found
- [ ] Only drill into transactions if Balance Sheet anomalies found
- [ ] Skip broad transaction checks if no report-level anomalies
- [ ] Filter transactions to only relevant ones (e.g., suspense account only)

## Findings Quality Improvement
- [ ] Tie each finding to specific report-level impact
- [ ] Improve evidence to show financial statement distortion
- [ ] Add financial impact metrics (amount, % of total)
- [ ] Clarify recommended actions with specific next steps

## Testing & Verification
- [ ] Test account classification logic with QBO types
- [ ] Test materiality filtering
- [ ] Test transaction drilldown narrowing
- [ ] Verify diagnostic order remains correct
- [ ] Verify false positive reduction
