---
name: embedded-knowledge
description: "Build and maintain the embedded-knowledge knowledge base website at github.com/clarkman-meta/embedded-knowledge. Use when user asks to: add articles or content to the knowledge base, update existing articles, organize technical notes into the website, or build a documentation site for embedded firmware / hardware topics including Qualcomm non-HLOS and sensor IC content."
---

# Embedded Knowledge

Build a React + Tailwind static knowledge base website that organizes engineering Q&A and research into a navigable, categorized documentation site.

## Project Repository

> **GitHub**: `https://github.com/clarkman-meta/embedded-knowledge`
> **Live site (GitHub Pages)**: `https://clarkman-meta.github.io/embedded-knowledge/`
> **Live site (Manus)**: `https://embedded-knowledge.manus.space` *(requires webdev session, secondary)*
> **Manus project name**: `embedded-kb`

When the user asks to add content or modify the knowledge base in **any new conversation**, read `references/github-credentials.md` first for the authenticated clone URL and push instructions.

Quick reference:
```bash
# Use the token-embedded URL (see references/github-credentials.md for full token)
git clone https://clarkman-meta:<TOKEN>@github.com/clarkman-meta/embedded-knowledge.git /home/ubuntu/embedded-kb
cd /home/ubuntu/embedded-kb && pnpm install
git config user.email "manus@embedded-knowledge" && git config user.name "Manus"
```

After making changes:
```bash
git add . && git commit -m "Add: <description>" && git push origin main
# GitHub Actions automatically builds and deploys to GitHub Pages (~1-2 min)
```

> Token expiry: ~2026-07-13. See `references/github-credentials.md` for renewal instructions.

## Deployment Mechanism (CRITICAL)

### Primary: GitHub Pages (use this in all new conversations)

> **git push to main → GitHub Actions auto-builds → GitHub Pages updates (~1-2 min)**

The canonical live site is now **GitHub Pages**:
```
https://clarkman-meta.github.io/embedded-knowledge/
```

Deployment chain:
```
Code changes → git push origin main → GitHub Actions (deploy.yml) → GitHub Pages live
```

**Standard update workflow for any new conversation:**

1. **Clone from GitHub** (see `references/github-credentials.md` for full token):
   ```bash
   git clone https://clarkman-meta:<TOKEN>@github.com/clarkman-meta/embedded-knowledge.git /home/ubuntu/embedded-kb
   cd /home/ubuntu/embedded-kb && pnpm install
   git config user.email "manus@embedded-knowledge" && git config user.name "Manus"
   ```
2. **Make content changes** in `client/src/lib/kb-data.ts`
3. **Push to deploy**:
   ```bash
   git add . && git commit -m "Add: <description>" && git push origin main
   ```
4. **Verify** at `https://clarkman-meta.github.io/embedded-knowledge/` after ~1-2 min

> **No `webdev_save_checkpoint` needed.** GitHub Actions handles build and deployment automatically.

### Routing: Hash-based (important for GitHub Pages)

The app uses `wouter` with `useHashLocation` hook. URLs look like:
- `https://clarkman-meta.github.io/embedded-knowledge/#/category/qcom`
- `https://clarkman-meta.github.io/embedded-knowledge/#/article/qcom/sensor-subsystem/sensor-layered-arch`

Do NOT switch back to history-based routing — GitHub Pages cannot serve sub-paths without a server.

### Secondary: Manus site (only if webdev session is available)

The `embedded-knowledge.manus.space` site still exists but requires `webdev_save_checkpoint` in the original Manus session. Use GitHub Pages as the primary URL for sharing.

### Common failure: "Project not found" (Manus only)

Causes:
- `webdev_save_checkpoint` called before `webdev_init_project` in a new session
- `webdev_init_project` called before `git clone` (directory doesn't exist yet)
- The project name passed to `webdev_init_project` doesn't match `embedded-kb`

### Article Inventory (current kb-data.ts IDs)

Use these exact IDs when referencing or adding articles. Do NOT invent new IDs without updating this list.

| Category | Subcategory ID | Article ID |
|---|---|---|
| `qcom` | `non-hlos-overview` | `what-is-non-hlos` |
| `qcom` | `bootloader` | `boot-chain-overview` |
| `qcom` | `sensor-subsystem` | `sensor-layered-arch` |
| `qcom` | `sensor-subsystem` | `slpi-ssc-detail` |
| `qcom` | `dsp-subsystems` | `adsp-lpass` |
| `qcom` | `dsp-subsystems` | `cdsp-compute` |
| `qcom` | `mcu-subsystems` | `smcu-safety` |
| `qcom` | `mcu-subsystems` | `cmcu-connectivity` |
| `qcom` | `camera-isp` | `camera-isp-pipeline` |
| `qcom` | `power-modem` | `aop-rpm-power` |
| `qcom` | `power-modem` | `mpss-modem` |
| `sensor` | `capsense` | `capsense-coming-soon` |
| `pmic` | `pmar2230` | `pmar2230-overview` |
| `pmic` | `pmar2230` | `pmar2230-power-rails` |
| `pmic` | `pmar2230` | `pmar2230-pon-sequence` |
| `pmic` | `pmar2230` | `pmar2230-gpio-spmi` |
| `pmic` | `pmar2230` | `pmar2230-linux-software` |
| `pmic` | `pmar2230` | `pmar2230-register-map` |
| `pmic` | `pmar2230` | `pmar2230-design-guidelines` |
| `sensor` | `hall-sensor` | `ak09973d-overview` |
| `sensor` | `hall-sensor` | `ak09973d-registers` |
| `sensor` | `hall-sensor` | `ak09973d-design-guidelines` |

Route pattern: `/article/:catId/:subId/:artId`

## Overview

This skill produces a **Technical Codex** style knowledge base:
- Dark sidebar (`#0F172A`) + light content area
- Three-column layout: sidebar navigation → article content → in-page TOC
- Fonts: Space Grotesk (headings), Inter (body), JetBrains Mono (code)
- All content stored as typed data in `kb-data.ts`; no CMS or backend needed

## Workflow

1. **Gather content** — Extract all Q&A topics from conversation history; group into categories and subcategories
2. **Initialize project** — `webdev_init_project` with `web-static` scaffold
3. **Write design plan** — Create `ideas.md` (Technical Codex approach)
4. **Build data layer** — Write `client/src/lib/kb-data.ts` with all articles as HTML strings
5. **Build UI** — Write CSS tokens, KBLayout, Home, CategoryPage, ArticlePage in one batch
6. **Checkpoint & deliver** — `webdev_save_checkpoint`, send `manus-webdev://` attachment

For full file templates, see `references/file-templates.md`.
For article HTML authoring conventions, see `references/article-conventions.md`.
For automatic QCOM / Sensor classification rules, see `references/domain-classification.md`.

## Content Extraction Rules

When reading conversation history to extract knowledge:
- Capture **both the question and the corrected/clarified answer** (not just the first answer)
- Mark common misconceptions explicitly with a `callout-warning` block
- One article per distinct concept; split if a topic has two clearly separable sub-topics
- Tag articles with short technical keywords (e.g., `["SLPI", "ADSP", "低功耗"]`)
- If content is missing for a subcategory, create a placeholder article with `callout-info` noting what will be added

## Automatic Domain Classification

Before writing any article, classify each topic using the decision tree in `references/domain-classification.md`:

1. **Determine the top-level category** — `qcom` (SoC internals, boot, DSP, MCU, camera), `sensor` (physical sensor IC, driver, calibration), or `pmic` (power management IC, regulators, power sequencing)
2. **Determine the subcategory** — match keywords to the subcategory table in the reference file
3. **Check for cross-domain topics** — SLPI/ADSP sensor processing belongs in `qcom › sensor-subsystem`; the Sensor category covers external IC drivers only; PMIC covers PMAR2230 and other Qualcomm PMICs
4. **Create placeholder subcategories** for any domain mentioned by the user but lacking content

The reference file also contains the complete `categories` skeleton array to use as a starting point in `kb-data.ts`.

## Design Decisions (commit to these)

| Element | Choice |
|---|---|
| Default theme | Light (`defaultTheme="light"` in App.tsx) |
| Sidebar background | `oklch(0.13 0.02 255)` ≈ `#0F172A` |
| Primary accent | Green `oklch(0.52 0.18 142)` ≈ `#22C55E` |
| Heading font | Space Grotesk (Google Fonts) |
| Code font | JetBrains Mono (Google Fonts) |
| Article content | Raw HTML string in `kb-data.ts`, rendered via `dangerouslySetInnerHTML` |
| Routing | Wouter: `/`, `/category/:catId`, `/article/:catId/:subId/:artId` |

## Category Color Tokens

Use `color` field in Category to drive card styling:
- `"green"` → green border/badge (hardware/firmware topics)
- `"blue"` → blue border/badge (software/driver topics)

## Article HTML Conventions

Articles use `.kb-prose` CSS class. Supported elements:

```html
<h1> <h2> <h3>          <!-- headings, auto-extracted for TOC -->
<p> <ul> <ol> <li>      <!-- body text -->
<strong> <code> <pre>   <!-- emphasis and code -->
<table><thead><tbody>   <!-- comparison tables -->

<!-- Callout boxes -->
<div class="callout-warning">⚠️ ...</div>
<div class="callout-info">ℹ️ ...</div>
<div class="callout-tip">✅ ...</div>
```

Keep `<pre>` content as plain text (no inner `<code>` tags for flow diagrams).

## Common Pitfalls

- **Site not updating after GitHub push**: Wait 1-2 minutes for GitHub Actions to finish. Check status at `github.com/clarkman-meta/embedded-knowledge/actions`
- **GitHub Actions build fails**: Check the Actions log. Common causes: TypeScript errors in `kb-data.ts`, missing required fields in article/category objects
- **webdev_save_checkpoint returns "Project not found"**: This is for the Manus secondary site only. For GitHub Pages, just `git push` — no checkpoint needed
- **Page shows 404 on GitHub Pages**: The app uses hash routing (`#/...`). Direct links to sub-paths without `#` will 404 — always use the hash-prefixed URLs

- **RegExpStringIterator TS error**: Use `Array.from(str.matchAll(...))` not `[...str.matchAll(...)]`
- **Missing pages cause build errors**: Create all pages referenced in `App.tsx` before saving
- **TOC not rendering**: Ensure `<h1>/<h2>/<h3>` in article HTML have no extra attributes breaking the regex
- **Sidebar not showing sub-items**: Default `expandedCats` state must include the first category ID set to `true`
- **Sidebar all collapsed by default**: `expandedCats` is initialized as `{}` (empty) — all categories start collapsed and users expand them manually

## Security: PAT Handling for manus-skill Repo

> **Important**: When syncing this skill to `github.com/clarkman-meta/manus-skill`, the `references/github-credentials.md` file must have its PAT replaced with the `<GITHUB_PAT>` placeholder before pushing, to avoid token leakage to the public repo.
>
> - **Sandbox local** (`/home/ubuntu/skills/embedded-knowledge/references/github-credentials.md`): Retains the **full token** for Manus runtime use.
> - **manus-skill repo** (`embedded-knowledge/references/github-credentials.md`): Contains only `<GITHUB_PAT>` as a placeholder — never the real token.
>
> When updating the skill and pushing to manus-skill, always sanitize `github-credentials.md` in the manus-skill copy before committing.
