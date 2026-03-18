# LedgerRx — Phase 1 Scaffold

A full-stack QuickBooks Online diagnostic and cleanup copilot application built with React, Express, tRPC, and Drizzle ORM. This is **Phase 1 of the project**, which includes the complete foundational architecture, OAuth scaffold, and UI — stopping cleanly at the credential requirement boundary.

## Overview

**LedgerRx** helps accounting teams maintain healthy QuickBooks data by:

1. **Connecting** to QuickBooks Online via OAuth 2.0
2. **Running** comprehensive diagnostic checks across transactions, vendors, customers, and accounts
3. **Surfacing** findings by severity (critical, warning, info)
4. **Suggesting** AI-powered cleanup actions via the Copilot (Phase 3)

## Phase 1 Deliverables

### ✅ Complete

- **Database schema** — `qboConnections`, `healthcheckRuns`, `findings`, `copilotSuggestions` tables
- **Backend architecture** — Modular structure with auth, qbo, diagnostics, findings, copilot, sync modules
- **tRPC API** — Full router setup for all modules with type-safe client/server communication
- **QuickBooks OAuth 2.0** — Connect/callback routes, credential gate, token exchange scaffold
- **Frontend UI** — Dark professional theme, DashboardLayout with sidebar navigation, all page scaffolds
- **Dashboard** — Connection status, stats, quick actions, last run summary
- **Healthcheck UI** — Check catalog, run history, execution scaffold
- **Findings UI** — List, filter, resolve/dismiss findings
- **Copilot UI** — Suggestion list, accept/reject actions (LLM integration in Phase 3)
- **Settings page** — QBO connection management, credential status, Intuit portal link

### 🚧 Scaffolded (Deferred to Phase 2+)

- **Diagnostic check execution** — Check runners call QBO API and evaluate rules (Phase 2)
- **Data sync** — Incremental sync from QuickBooks using Change Data Capture (Phase 2)
- **LLM integration** — AI suggestion generation for findings (Phase 3)
- **Auto-fix actions** — Apply cleanup suggestions to QuickBooks (Phase 3+)

## Architecture

```
ledgerrx/
├── client/                           # React 19 frontend
│   ├── src/
│   │   ├── pages/
│   │   │   ├── Home.tsx             # Landing page (redirects to dashboard if authenticated)
│   │   │   ├── Dashboard.tsx        # Main dashboard with QBO connection state
│   │   │   ├── Healthcheck.tsx      # Run diagnostics UI scaffold
│   │   │   ├── Findings.tsx         # View and manage findings
│   │   │   ├── Copilot.tsx          # AI suggestions (Phase 3)
│   │   │   └── Settings.tsx         # QBO connection settings
│   │   ├── components/
│   │   │   ├── DashboardLayout.tsx  # Sidebar navigation wrapper
│   │   │   └── ui/                  # shadcn/ui components
│   │   ├── lib/trpc.ts              # tRPC client
│   │   ├── index.css                # Dark professional theme (OKLCH palette)
│   │   └── App.tsx                  # Routes and layout
│   └── index.html
│
├── server/                           # Express + tRPC backend
│   ├── modules/
│   │   ├── auth/                    # Session helpers
│   │   ├── qbo/                     # QuickBooks OAuth module
│   │   │   ├── index.ts             # Connection DB helpers
│   │   │   └── oauth.ts             # OAuth URL builder, token exchange
│   │   ├── diagnostics/             # Check execution scaffold
│   │   │   ├── index.ts             # Check registry and runner
│   │   │   └── runs.ts              # Healthcheck run DB helpers
│   │   ├── findings/                # Finding query helpers
│   │   ├── copilot/                 # Copilot suggestion scaffold
│   │   └── sync/                    # QBO data sync scaffold
│   ├── routers/
│   │   ├── qbo.ts                   # tRPC: connect, status, disconnect
│   │   ├── findings.ts              # tRPC: list, get, updateStatus
│   │   ├── diagnostics.ts           # tRPC: listRuns, getRun, startRun
│   │   ├── copilot.ts               # tRPC: listSuggestions, updateStatus
│   │   └── sync.ts                  # tRPC: trigger (scaffold)
│   ├── routers.ts                   # Main appRouter with all feature routers
│   ├── db.ts                        # Database connection and user helpers
│   ├── qboRoutes.ts                 # Express route: GET /api/qbo/callback
│   └── _core/
│       ├── index.ts                 # Server entry point (registers OAuth + QBO routes)
│       ├── context.ts               # tRPC context with user auth
│       ├── trpc.ts                  # tRPC instance and procedures
│       ├── oauth.ts                 # Manus OAuth callback handler
│       ├── sdk.ts                   # Session and auth helpers
│       └── ... (other core utilities)
│
├── drizzle/
│   ├── schema.ts                    # Database schema (users, qboConnections, findings, etc.)
│   └── 0001_*.sql                   # Generated migration
│
├── .env.example                     # Environment variable template
├── README.md                        # This file
└── todo.md                          # Feature checklist
```

## Getting Started

### Prerequisites

- Node.js 22+ and pnpm
- MySQL 8.0+ or TiDB (database provided in Manus)
- Intuit Developer account (free at https://developer.intuit.com)

### 1. Local Development Setup

```bash
cd ledgerrx
pnpm install
```

The database schema is already applied. The dev server starts automatically and watches for changes.

### 2. QuickBooks Credentials (Required for OAuth)

To enable the "Connect QuickBooks" flow, you need three credentials from Intuit:

#### Step 1: Get Your Credentials from Intuit

1. Go to https://developer.intuit.com and sign in (create a free account if needed)
2. Click **My Apps** and create a new app (or open an existing one)
3. Navigate to **Keys & OAuth**
4. Copy the following values:
   - **Client ID** (from Sandbox or Production keys)
   - **Client Secret** (from Sandbox or Production keys)

#### Step 2: Publish LedgerRx to Get Your Redirect URI

Before you can register the redirect URI in Intuit, you need to know your hosted domain:

1. Click the **Publish** button in the Manus Management UI (top-right)
2. Once published, you'll get a URL like: `https://ledgerrx-abc123.manus.space`
3. Your redirect URI is: `https://ledgerrx-abc123.manus.space/api/qbo/callback`

#### Step 3: Register the Redirect URI in Intuit

1. In your Intuit app, go to **Keys & OAuth** → **Redirect URIs**
2. Click **Add URI** and paste: `https://<your-published-domain>/api/qbo/callback`
3. Save

#### Step 4: Add Credentials to Your Environment

After publishing, add these environment variables to your Manus project:

| Variable | Value | Where to Find |
|----------|-------|---------------|
| `QBO_CLIENT_ID` | Your Client ID | Intuit → Keys & OAuth → Client ID |
| `QBO_CLIENT_SECRET` | Your Client Secret | Intuit → Keys & OAuth → Client Secret |
| `QBO_REDIRECT_URI` | `https://<your-domain>/api/qbo/callback` | Registered above |
| `QBO_ENVIRONMENT` | `sandbox` (for testing) or `production` | Your choice |

**Important:** Never commit these credentials to version control. Use the Manus secrets management UI to store them securely.

### 3. Test the OAuth Flow

1. Visit your published LedgerRx URL
2. Sign in (Manus OAuth)
3. On the Dashboard, click **Connect QuickBooks**
4. You'll be redirected to Intuit to authorize
5. After authorization, you'll return to the Dashboard with your connection active

## What Works in Phase 1

### ✅ Frontend

- **Landing page** — Redirects authenticated users to dashboard
- **Dashboard** — Shows QBO connection status, stats, quick actions
- **Healthcheck UI** — Lists available checks, shows run history
- **Findings UI** — Displays findings with severity badges, resolve/dismiss actions
- **Copilot UI** — Shows suggestion list (no LLM generation yet)
- **Settings** — Manage QBO connection, view credential status
- **Sidebar navigation** — Full navigation with all modules
- **Dark theme** — Professional OKLCH-based palette

### ✅ Backend

- **OAuth scaffold** — Complete URL builder, token exchange, credential gate
- **OAuth callback route** — `GET /api/qbo/callback` handles Intuit redirect
- **Connection management** — Upsert, deactivate, retrieve QBO connections
- **tRPC routers** — All modules have type-safe API procedures
- **Database** — Schema applied, all tables ready

### 🚧 What's Stubbed

- **Diagnostic checks** — Catalog defined, execution engine deferred to Phase 2
- **Healthcheck execution** — `startRun` creates a pending record, no actual checks run
- **Data sync** — Scaffold ready, implementation deferred to Phase 2
- **Copilot suggestions** — Data model ready, LLM integration deferred to Phase 3
- **Auto-fix actions** — Payload structure defined, execution deferred to Phase 3+

## Development Workflow

### Adding a New Feature

1. **Update todo.md** with the feature as a new `[ ]` item
2. **Update database schema** if needed:
   ```bash
   # Edit drizzle/schema.ts
   pnpm drizzle-kit generate
   # Review the generated SQL, then apply via webdev_execute_sql
   ```
3. **Add tRPC procedure** in `server/routers/<feature>.ts`
4. **Build UI** in `client/src/pages/<Feature>.tsx`
5. **Write tests** in `server/<feature>.test.ts` using Vitest
6. **Mark complete** in todo.md as `[x]`

### Running Tests

```bash
pnpm test
```

### Building for Production

```bash
pnpm build
pnpm start
```

## Environment Variables

See `.env.example` for the full template. Key variables:

| Variable | Required | Description |
|----------|----------|-------------|
| `QBO_CLIENT_ID` | ✅ | Intuit OAuth Client ID |
| `QBO_CLIENT_SECRET` | ✅ | Intuit OAuth Client Secret |
| `QBO_REDIRECT_URI` | ✅ | OAuth redirect URI (must match Intuit registration) |
| `QBO_ENVIRONMENT` | ✅ | `sandbox` or `production` |
| `DATABASE_URL` | ✅ | MySQL connection string (auto-injected in Manus) |
| `JWT_SECRET` | ✅ | Session signing secret (auto-injected in Manus) |

All other variables are auto-injected by the Manus platform.

## Next Steps After Phase 1

### Phase 2: Diagnostic Execution & Data Sync

- Implement check runners that call the QuickBooks API
- Build the 7 diagnostic checks (uncategorized transactions, duplicate vendors, etc.)
- Implement incremental data sync using QBO's Change Data Capture API
- Wire the execution engine to `startRun` mutation

### Phase 3: AI Copilot & Auto-Fix

- Integrate the built-in LLM to generate cleanup suggestions
- Implement suggestion generation workflow
- Build auto-fix payload builders for common issues
- Add "Apply Fix" actions that write back to QuickBooks

### Phase 4+: Advanced Features

- Batch operations and scheduled runs
- Audit trail and change history
- Custom check rules and thresholds
- Integration with other accounting tools

## Troubleshooting

### "QuickBooks credentials not configured"

**Cause:** `QBO_CLIENT_ID`, `QBO_CLIENT_SECRET`, or `QBO_REDIRECT_URI` are missing from environment.

**Solution:** Add the three credentials to your Manus project secrets (see "Getting Started" above).

### "OAuth redirect failed"

**Cause:** The redirect URI in your code doesn't match what's registered in Intuit.

**Solution:** Verify the exact URI in your Intuit app matches your published domain: `https://<your-domain>/api/qbo/callback`

### "Invalid session cookie"

**Cause:** User is not authenticated with Manus OAuth.

**Solution:** Click "Sign in" on the home page to complete Manus authentication first.

## File Structure Quick Reference

| Path | Purpose |
|------|---------|
| `client/src/pages/` | Page components (Dashboard, Healthcheck, etc.) |
| `server/modules/` | Business logic modules (auth, qbo, findings, etc.) |
| `server/routers/` | tRPC API procedures |
| `drizzle/schema.ts` | Database schema definition |
| `.env.example` | Environment variable template |
| `todo.md` | Feature checklist |

## License

MIT

## Support

For issues or questions, refer to the README sections above or check the inline code comments in the modules.

---

**LedgerRx Phase 1** — Built with React, Express, tRPC, Drizzle, and Manus OAuth.
