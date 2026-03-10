# Supabase — Production Readiness

*Created: 2026-03-09*
*Last updated: 2026-03-09 — Phase 1 audit complete*
*Status: Phase 1 done, Phase 2 (manual upgrade) pending*

## Context

With a real client (Nano Fibre) being onboarded, the current Supabase Free tier needs evaluation and upgrade for production use. The portal uses Supabase (managed PostgreSQL) as its primary database, connected via `pgx` driver with a repository pattern.

**Code repo:** [tumai-programmes/limitless-portal](https://github.com/tumai-programmes/limitless-portal)
**SQL operations:** [tumai-hq/sql-hub/limitless-portal](https://github.com/tumai-hq/sql-hub) — monitoring, diagnostics, reports
**Live URL:** https://portal.limitlessmodus.com/

---

## Free Tier Limitations (March 2026)

| Resource | Free Tier Limit | Actual Usage (audited 2026-03-09) | Production Estimate (1 engagement, ~30 users) | Risk |
|----------|----------------|-----------------------------------|-----------------------------------------------|------|
| Database size | 500 MB | **12 MB** (11 tables, 36 rows, dev/seed data) | ~50–200 MB (responses, drafts, logs, uploads metadata) | Low for 1 client; will hit limit at 3–5 concurrent engagements |
| Database egress | 5 GB / month | Minimal (portal idle, no app traffic in last 24h) | ~2–10 GB (wizard auto-save, dashboard reads, API queries) | Medium — auto-save on every field change generates steady egress |
| Direct connections | 60 | ~1 (internal services only) | 5–15 (Go backend pool + admin queries) | Low |
| Pooler connections | 200 | Not used (Go uses pgxpool) | Not applicable | N/A |
| Daily backups | 7-day retention | Active | Adequate for dev; insufficient for production SLA | High — no point-in-time recovery, no manual backup/restore API |
| Branching | Not available | — | Useful for staging | Low |
| Read replicas | Not available | — | Not needed now | Low |
| SLA | None | — | No uptime guarantee | **Critical** — production with real client needs uptime commitment |
| Support | Community only | — | No priority support | High — if DB goes down during engagement, no escalation path |
| Pause after inactivity | 7 days | Active | Would pause between engagement sessions | **Critical** — auto-pause would cause downtime for active client |

## Recommendation: Upgrade to Supabase Pro

For production readiness with a real client, Supabase Pro ($25/month) is the minimum viable tier.

| Feature | Free | Pro ($25/mo) | Why It Matters |
|---------|------|-------------|----------------|
| Database size | 500 MB | 8 GB | Room for 10+ concurrent engagements |
| Egress | 5 GB/mo | 250 GB/mo | Auto-save + multiple users won't hit limit |
| Backups | 7-day daily | 14-day daily + point-in-time recovery | Can restore to any point if data corruption occurs |
| No auto-pause | No | Yes | Portal stays available between engagement sessions |
| SLA | None | 99.9% uptime | Contractual commitment for client |
| Support | Community | Email support | Escalation path for incidents |
| Log retention | 1 day | 7 days | Debug production issues |
| Branching | No | Yes | Safe schema changes via staging branch |

---

## Action Plan

### Phase 1 — Audit Current State (MCP — read-only) -- COMPLETE

*Executed: 2026-03-09 via Supabase MCP. All checks read-only, no changes made.*

| # | Action | MCP Tool | Purpose | Status |
|---|--------|----------|---------|--------|
| 1.1 | Query current database size | `execute_sql` | Measure actual MB used vs 500 MB limit | **Done** — 12 MB |
| 1.2 | List all tables with row counts | `execute_sql` + `list_tables` | Understand data distribution | **Done** — 11 tables, 36 total rows |
| 1.3 | List applied migrations | `list_migrations` | Verify migration state matches `limitless-portal/migrations/` | **Done** — 9 migrations |
| 1.4 | Check installed extensions | `list_extensions` | Document what extensions are in use | **Done** — 6 extensions |
| 1.5 | Run security advisors | `get_advisors` (security) | Identify missing RLS policies or exposed tables | **Done** — 4 ERRORS (RLS gaps) |
| 1.6 | Run performance advisors | `get_advisors` (performance) | Identify index gaps or slow query patterns | **Done** — 4 missing FK indexes, 7 unused indexes |
| 1.7 | Get project URL | `get_project_url` | Verify region from the URL | **Done** — EU (IPv6 confirms eu-west) |
| 1.8 | Check recent Postgres logs | `get_logs` (postgres) | Look for errors, connection issues, or warnings | **Done** — clean, all LOG level |
| 1.9 | Check recent API logs | `get_logs` (api) | Spot any unexpected traffic patterns | **Done** — only health checks |
| 1.10 | List storage buckets | `list_storage_buckets` | Audit what storage is configured | **Done** — empty (using AWS S3) |
| 1.11 | Get storage config | `get_storage_config` | Check file size limits, S3 protocol, image transforms | **Done** — defaults, S3 protocol on |
| 1.12 | Get publishable keys | `get_publishable_keys` | Audit which API keys are active/disabled | **Done** — 1 legacy anon key |

#### Phase 1 Audit Results

**Database:**
- Total size: 12 MB (2.4% of 500 MB free limit)
- 11 public tables, 36 live rows across all tables
- All table data sizes at 8 KB or less; largest total (data + indexes) is `users` and `uploads` at 96 KB each
- High dead row counts on `users` (49), `submissions` (38), `section_drafts` (28) from dev iteration — autovacuum has not yet run

**Tables & Row Counts:**

| Table | Rows | RLS | Notes |
|-------|-----:|-----|-------|
| participants | 13 | OFF | **Security gap** |
| submissions | 8 | ON | |
| audit_log | 5 | ON | |
| users | 5 | ON | |
| engagements | 3 | ON | |
| section_drafts | 2 | OFF | **Security gap** |
| findings | 0 | ON | |
| notification_log | 0 | OFF | **Security gap** |
| interviews | 0 | ON | |
| uploads | 0 | OFF | **Security gap** |
| responses | 0 | ON | |

**Migrations (9 applied):**

| Version | Name |
|---------|------|
| 20260304134750 | `001_initial_schema_core` |
| 20260304134806 | `002_initial_schema_uploads_interviews_findings` |
| 20260304144504 | `003_enable_rls` |
| 20260304234251 | `wizard_data_model` |
| 20260306143203 | `004_multi_tenant` |
| 20260308183417 | `006_notification_log` |
| 20260308183421 | `007_engagement_logo` |
| 20260308183441 | `008_uploads_table` |
| 20260309003708 | `add_logo_dark_url_to_engagements` |

**Extensions (6 installed):** plpgsql (1.0), supabase_vault (0.3.1), uuid-ossp (1.1), pgcrypto (1.3), pg_stat_statements (1.11), pg_graphql (1.5.11)

**Security — 4 ERRORS:**
- Tables `participants`, `section_drafts`, `notification_log`, `uploads` have **RLS disabled**
- These were added after the `003_enable_rls` migration and never got RLS enabled
- **Action needed:** new migration to enable RLS on these 4 tables before production

**Performance — 11 INFOs:**
- 4 unindexed foreign keys: `findings.created_by`, `notification_log.sent_by`, `submissions.participant_id`, `submissions.user_id`
- 7 unused indexes (expected — most tables empty or dev-only volume; keep for production)
- **Action needed:** add covering indexes for the 4 FK columns

**Project URL:** `https://caoasgslcsanrnylalhf.supabase.co` — IPv6 addresses in logs (`2a05:d018:...`) confirm AWS eu-west-1 (Ireland). Verify exact region label in dashboard.

**Logs:** Clean — all LOG level, no errors. Only internal services (postgres_exporter, mgmt-api, dashboard). No application traffic in last 24h.

**Storage:** No Supabase Storage buckets (portal uses AWS S3 directly). Default config: 50 MB file limit, S3 protocol enabled.

**API Keys:** 1 legacy anon key active. Not used by the Go backend (connects via `pgx` connection string).

#### Pre-Production Fixes Required

| # | Fix | Priority | Where | Status |
|---|-----|----------|-------|--------|
| F1 | Enable RLS on `participants`, `section_drafts`, `notification_log`, `uploads` | **Critical** | `011_fix_rls_gaps` migration | **Done** — applied 2026-03-09, security advisor now 0 errors |
| F2 | Add indexes on `findings.created_by`, `notification_log.sent_by`, `submissions.participant_id`, `submissions.user_id` | Medium | `012_add_fk_indexes` migration | **Done** — applied 2026-03-09, no more unindexed FK warnings |
| F3 | Consider disabling unused anon API key | Low | Supabase Dashboard | Pending |

### Phase 2 — Manual Steps (Supabase Dashboard)

These require the Supabase web dashboard — cannot be done via MCP.

| # | Action | Where | Notes | Status |
|---|--------|-------|-------|--------|
| 2.1 | **Upgrade to Pro tier** ($25/mo) | Dashboard > Billing | Unlocks no auto-pause, SLA, 8 GB disk, 250 GB egress | **Done** 2026-03-09 |
| 2.2 | **Verify EU region** | Dashboard > Project Settings | Confirmed eu-west-1 (Ireland) via MCP IPv6 check | **Done** 2026-03-09 |
| 2.3 | **Enable Point-in-Time Recovery (PITR)** | Dashboard > Database > Backups | Requires Small compute ($60/mo) + PITR add-on ($100+/mo). **Deferred** — daily backups (7-day) sufficient for pilot | Deferred |
| 2.4 | **Auto-pause disabled** | Dashboard > Project Settings | Pro tier: no auto-pause. Confirmed — only manual "Pause project" button visible | **Done** 2026-03-09 |
| 2.5 | **Review connection pooling settings** | Dashboard > Database > Connection Pooling | Go backend uses direct `pgx` connection (port 5432), not pooler | Pending |
| 2.6 | **Upgrade to Micro compute** | Dashboard > Compute and Disk | Free with Pro. 1 GB RAM, 2-core ARM (up from Nano 0.5 GB shared) | Pending |
| 2.7 | **Upgrade Postgres patch** | Dashboard > Infrastructure | 17.0.1.052 → 17.0.1.063 (minor security/bugfix patch) | Pending |

### Phase 3 — Post-Upgrade Verification (MCP)

After upgrade, verify everything is working correctly.

| # | Action | MCP Tool | Purpose | Status |
|---|--------|----------|---------|--------|
| 3.1 | Re-run security advisors | `get_advisors` | Confirm no new issues after tier change | Pending |
| 3.2 | Check logs for connection issues | `get_logs` | Verify no disruption from upgrade | Pending |
| 3.3 | List branches (if branching enabled) | `list_branches` | Confirm branching is available on Pro | Pending |
| 3.4 | Create a staging branch | `create_branch` | Test safe schema changes before production | Pending |
| 3.5 | Run a test SQL query | `execute_sql` | Verify connectivity post-upgrade | Pending |

### Phase 4 — Documentation

| # | Action | Where | Status |
|---|--------|-------|--------|
| 4.1 | Document backup/restore procedure | This file or separate `backup-restore.md` | Pending |
| 4.2 | Update `portal-spec.md` with tier info | `architecture/portal-spec.md` | Pending |
| 4.3 | Add Supabase operational runbook | `sql-hub/limitless-portal/supabase/` | Pending |

---

## MCP Capabilities Summary

23 Supabase MCP tools available, grouped by category:

| Category | Tools | Notes |
|----------|-------|-------|
| **Database** | `execute_sql`, `list_tables`, `list_migrations`, `list_extensions` | Full read/write SQL access |
| **Ops** | `get_advisors`, `get_logs` | Security + performance advisors, service logs (last 24h) |
| **Project** | `get_project_url`, `get_publishable_keys` | URL, API key audit |
| **Storage** | `get_storage_config`, `list_storage_buckets`, `update_storage_config` | Supabase Storage (not AWS S3) |
| **Branching** | `create_branch`, `list_branches`, `merge_branch`, `rebase_branch`, `reset_branch`, `delete_branch` | Pro-only staging environments |
| **Edge Functions** | `deploy_edge_function`, `get_edge_function`, `list_edge_functions` | Deno edge functions |
| **Schema** | `apply_migration`, `generate_typescript_types` | DDL migrations, TS type gen |
| **Docs** | `search_docs` | Search Supabase documentation |

### What MCP Cannot Do

- Billing/plan changes (upgrade, payment method)
- Region migration (set at project creation)
- Enable PITR (dashboard toggle)
- Pause/resume control (dashboard setting)
- Usage metrics (bandwidth, compute hours)

---

*Part of [Portal Architecture](portal-spec.md) — [Limitless Portal Design](../README.md)*
