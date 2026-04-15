---
name: jira-tool
description: Manage and continue development of the Jira Issue Monitor web application for Clark Hsu. Use this skill when the user asks to update, fix, or add features to the Jira monitor dashboard, or when starting a new conversation about this project. Contains GitHub repo info, Jira credentials, project architecture, and deployment workflow.
---

# Jira Tool — Project Skill

This skill provides everything needed to continue developing the **Jira Issue Monitor** web app in a new conversation.

## Quick Start

The Manus webdev project name is **`jira-monitor`** (path: `/home/ubuntu/jira-monitor`).

**Continuing in the SAME conversation** (sandbox already has the project directory):
- Dev server should already be running. Verify with `webdev_check_status`, then start coding.

**Starting a NEW conversation** (fresh sandbox, no project directory):

> **IMPORTANT constraint**: `webdev_init_project` requires the project directory to already exist at `/home/ubuntu/jira-monitor`. You MUST `git clone` first, THEN call `webdev_init_project`.

Follow this exact order:
```bash
# Step 1: Clone from GitHub to create the local directory
git clone https://github.com/clarkman-meta/jira-tool.git /home/ubuntu/jira-monitor
cd /home/ubuntu/jira-monitor && pnpm install
```
Then call `webdev_init_project` with name `jira-monitor` — this re-binds the Manus S3 `origin` remote so that `webdev_save_checkpoint` can deploy correctly.

Do NOT skip `webdev_init_project` and just use `webdev_restart_server` — the `origin` remote will point to GitHub instead of Manus S3, and checkpoints will fail to deploy.

## GitHub Repository

- **URL**: https://github.com/clarkman-meta/jira-tool
- **Branch**: `main`
- **Write Token**: `<GITHUB_PAT>`

To push changes after a checkpoint:
```bash
cd /home/ubuntu/jira-monitor
# Add remote if not already present:
git remote add github https://clarkman-meta:<GITHUB_PAT>@github.com/clarkman-meta/jira-tool.git
git push github main
```

## Jira Credentials

- **Base URL**: `https://metarl.atlassian.net`
- **Email**: `clarkman@meta.com`
- **API Token**: `<JIRA_API_TOKEN>`
- **My Account ID**: `712020:f04ded31-3e91-47eb-bad9-d5e624e2b95f`
- **My Display Name**: Clark Hsu (username: `clarkman`)

These are already stored as server-side env vars in Manus project secrets: `JIRA_EMAIL`, `JIRA_API_TOKEN`, `JIRA_BASE_URL`, `JIRA_MY_ACCOUNT_ID`.

## Project Architecture

**Stack**: React 19 + Tailwind 4 + Express 4 + tRPC 11 + MySQL (Drizzle ORM)

**Key files**:

| File | Purpose |
|------|---------|
| `server/jira.ts` | Jira API proxy — cursor-based pagination, `fetchOpenIssues()`, `fetchSingleIssue()` |
| `server/routers.ts` | tRPC routes: `jira.issues`, `jira.issue`, `projects.*`, `watchlist.*`, `hidden.*` |
| `server/db.ts` | DB helpers + `seedDefaultProjects()` (runs on startup) |
| `drizzle/schema.ts` | Tables: `users`, `jiraProjects`, `watchedIssues`, `hiddenIssues` |
| `client/src/pages/Dashboard.tsx` | Main dashboard — filter bar, sortable table, My Issues toggle, time filter |
| `client/src/pages/AdminProjects.tsx` | Admin UI to add/edit/delete projects and their custom JQL |

## Three Monitored Projects

| Project | Key | Codename | Custom JQL |
|---------|-----|----------|------------|
| Dragon | DGTK | diamond | `parent = DGTK-234 AND statusCategory != Done AND status != Closed` |
| SSG | TPZ | topaz | `project = TPZ AND summary ~ "[P2]" AND statusCategory != Done AND status != Closed` |
| Hypernova2 | KITE | kitefin | `project = KITE AND "Build[Dropdown]" IN (P0, P1) AND status NOT IN (Closed, Done)` |

Custom JQL is stored in the `jiraProjects` DB table and editable via the Admin page (no code changes needed to update filters).

## Dashboard Features

- **My Issues toggle** — ON by default; server-side JQL involvement query (assignee OR reporter OR watcher OR comment ~ "clarkman"); amber highlight
- **Time filter** — Preset chips (7d/14d/30d/90d) + free-input days field; default 30 days
- **Priority filter chips** — P0 (red) / P1 (orange) / P2 (yellow); multi-select; KITE uses build field as effective priority
- **Stage filter chips** — All / SMT / FATP / EVT / DVT / PVT / NPI
- **Keyword search** — Matches title and latest comment
- **Sortable columns** — Priority, Updated, Status, Assignee, Reporter, Issue key (default: Priority asc → Updated desc)
- **Pin Issue** — Add any issue key to always show at top (fetched live from Jira, even if closed)
- **Hide (×) button** — Per-row button to suppress issues; stored in DB
- **Hidden Issues panel** — Filter bar button with count badge; dropdown lists hidden issues per project with "Restore" button
- **Auto-refresh toggle** — Every 5 minutes
- **Closed/Done issues excluded** — JQL filters out Done/Closed statuses

## DB Schema Notes

- `jiraProjects`: `id, key, name, codename, color, jiraUrl, order, enabled, titleFilter, issueTypeFilter, customJql`
- `watchedIssues`: `id, issueKey, projectKey, createdAt`
- `hiddenIssues`: `id, issueKey, projectKey, createdAt`

When adding new DB columns: `pnpm drizzle-kit generate` then `pnpm drizzle-kit migrate`.

## Development Workflow

1. Read `todo.md` at project root for pending items
2. Make changes then `pnpm check` (TypeScript) then `pnpm test` (Vitest, 10 tests)
3. `webdev_save_checkpoint` before delivering to user
4. Push to GitHub: `git push github main`

## Deployment Architecture — IMPORTANT

This project uses **two separate git remotes** with different roles:

| Remote | URL | Role |
|--------|-----|------|
| `origin` | `s3://vida-prod-gitrepo/webdev-git/...` (Manus-managed S3) | **Live deployment source** — `webdev_save_checkpoint` pushes here and triggers rebuild/redeploy |
| `github` | github.com/clarkman-meta/jira-tool | Backup only — push manually after each checkpoint |

**Key insight**: Manus deploys from `origin` (S3), NOT from GitHub. `git push github main` only updates the backup and does NOT redeploy the live site.

**Within the same conversation** (sandbox already has `/home/ubuntu/jira-monitor` with correct remotes):
```
Make changes → pnpm check → pnpm test → webdev_save_checkpoint → git push github main
```

**Starting a NEW conversation** (fresh sandbox, project directory missing):

Follow this exact 5-step order:
1. `git clone` from GitHub (creates local directory — required before `webdev_init_project`)
2. `webdev_init_project` with name `jira-monitor` — re-binds Manus S3 `origin`
3. `webdev_check_status` — verify dev server is running
4. Make code changes, then `pnpm check` + `pnpm test`
5. `webdev_save_checkpoint` — deploys to live site
6. `git push github main` — sync backup

**Common errors in new conversations**:
- `"Project not found"` when calling `webdev_init_project` → you forgot to `git clone` first (directory must exist)
- `webdev_save_checkpoint` fails or site not updated → `origin` points to GitHub instead of S3; re-run `git clone` then `webdev_init_project`
- GitHub push succeeds but live site unchanged → normal; GitHub is backup only, `webdev_save_checkpoint` is the only deploy trigger
- `webdev_save_checkpoint` called before `webdev_init_project` → will commit to wrong remote; start over with correct sequence
