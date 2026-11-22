# Codebase State

Living map of `momakoding.github.io`. **This document is English-only.** All entries must be written in English.

> **What lives here vs. in code.**
> This file is the project *map*: stack, structure, pointers, and architectural conventions. It is **not** a parallel registry — canonical truth is always the code. Module internals and design rationale live in source code and [`docs/DECISION_LOG.md`](./docs/DECISION_LOG.md). When this file conflicts with code, trust the code and update this file during a user-initiated sync.
>
> **Independent numbering.** Sections below use `1.`, `2.`, `3.` numbering and do not share `AGENTS.md`'s `§` sequence.
>
> Read freely; edit **only** during a user-initiated sync (see `AGENTS.md` §0).
>
> *Last synced: 2026-11-22.*

---

## 1. Project Snapshot

- **Name:** `momakoding.github.io`
- **Type:** Momakoding organization website (static Nuxt 4 site)
- **Stage:** Initial scaffold deployed to `https://momakoding.github.io/`; page design and content sections are pending template handoff.
- **Deployment:** Automated GitHub Pages build and deploy via GitHub Actions on push to `main`.

---

## 2. Technology Stack

> Exact dependency versions live in `package.json` and `pnpm-lock.yaml`. This table maps layers only.

| Layer | Technology | Notes |
| --- | --- | --- |
| **Framework** | Nuxt 4 (Vue 3 `<script setup lang="ts">`) | `app/` source root, file-based routing, static prerendering |
| **Language** | TypeScript (`vue-tsc` + `nuxt typecheck`) | Pinned `<7` to match Volar peer requirements |
| **Bundler** | Vite (managed by Nuxt) | `~` alias resolves to `app/` |
| **Styling** | Tailwind CSS v4 (`@tailwindcss/vite`) | Entry stylesheet at `app/assets/css/main.css` |
| **Static Build** | Nitro `github_pages` preset | `nuxt generate` outputs `.output/public` with `.nojekyll` |
| **Package Manager** | pnpm 12 | `pnpm-workspace.yaml` approves `esbuild` build script |
| **CI / Hosting** | GitHub Actions + GitHub Pages | Workflow at `.github/workflows/deploy.yml` (Node 24) |

**Do not add new top-level dependencies without appending a row to [`docs/DECISION_LOG.md`](./docs/DECISION_LOG.md).**

---

## 3. Directory Map

*Folder-level map plus key entry files; code is canonical.*

```text
momakoding.github.io/
├── .github/
│   └── workflows/
│       └── deploy.yml        # Build, typecheck, generate, and deploy to GitHub Pages
├── app/
│   ├── app.vue               # Root shell (<NuxtRouteAnnouncer /> + <NuxtPage />)
│   ├── assets/
│   │   └── css/main.css      # Tailwind CSS v4 entry (@import "tailwindcss";)
│   └── pages/
│       └── index.vue         # Home page (placeholder until template design lands)
├── docs/
│   ├── DECISION_LOG.md       # Append-only architecture and tooling decisions
│   └── TROUBLESHOOTING.md    # Known issues and fixes
└── public/                   # Unprocessed static assets (favicon.ico, robots.txt)
```

Root configuration files (`nuxt.config.ts`, `package.json`, `pnpm-lock.yaml`, `pnpm-workspace.yaml`, `tsconfig.json`, `AGENTS.md`, `CODEBASE_STATE.md`, `CHANGELOG`, `README.md`) are omitted from the tree as self-evident.

---

## 4. Source-of-Truth Pointers

| Looking for… | Read here |
| --- | --- |
| Nuxt, Vite, Nitro, and `<head>` config | `nuxt.config.ts` |
| Scripts, dependencies, and `packageManager` | `package.json` |
| pnpm build-script approvals | `pnpm-workspace.yaml` |
| Global styles, `@theme` tokens, `@utility` classes | `app/assets/css/main.css` |
| Routes and pages | `app/pages/**` |
| GitHub Pages CI/CD pipeline | `.github/workflows/deploy.yml` |
| Architecture and tooling rationale | `docs/DECISION_LOG.md` |
| Environment and build troubleshooting | `docs/TROUBLESHOOTING.md` |

---

## 5. Styling & State Conventions

- **Styling:** Tailwind CSS v4 utility-first via `@tailwindcss/vite`. Meaningful component parts use BEM classes composed with `@apply` inside `<style scoped>` with `@reference "~/assets/css/main.css"`. Shared tokens and utilities live in `app/assets/css/main.css`.
- **State:** Component-local `ref` / `reactive` by default; cross-component shared state uses Nuxt `useState` or `provide` / `inject`. No external state store is installed.
- **Environment variables:** None required for the static site yet. If introduced later, document public `NUXT_PUBLIC_*` keys here and keep real `.env` files untracked.
