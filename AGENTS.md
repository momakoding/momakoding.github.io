# AGENTS.md

> Behavior rules for AI coding agents on **`momakoding.github.io`**. This file records **rules only**; project facts — tech stack, architecture, directory map, and pointers — live in [`CODEBASE_STATE.md`](./CODEBASE_STATE.md) (see §9).
>
> You **must** write in English only in `AGENTS.md`. (Conversational replies match the user's active language; only this document is English-only.)

---

## §0. Multi-agent Collaboration Protocol

### 0.1 Doc Edit Policy (read this first)

- **Do NOT modify `CODEBASE_STATE.md`, `CHANGELOG`, or `docs/*` during feature work.** Finish the code first. Do not bump dates, append change-log lines, touch `docs/DECISION_LOG.md`, or move WIP rows in `docs/WIP.md` in the same turn you wrote code.
- **These docs are edited only when the user explicitly asks**, typically right before a commit. Phrasings that unlock doc edits: _"update the codebase state"_, _"sync the docs"_, _"prep for commit"_, _"fill in the change log"_. Without an explicit request, leave the docs alone.
- **Proactive ask.** After finishing any add / modify / delete coding task, ask the user whether they want `CHANGELOG`, `CODEBASE_STATE.md`, and/or a git commit updated now (per §5's post-task prompting rule). Never do these silently, and don't just wait passively. Mention the `CCC` / `3C` shortcut (§8) so the user can trigger all three at once.
- **When the user does ask**, batch all deferred updates from the session into one coherent edit (see "Pre-commit doc sync" in 0.3).
- **`AGENTS.md` (this file) changes only when the user asks to change a rule.** Feature work never touches it.
- **Ignore markdownlint warnings in Markdown docs.** Do not spend turns chasing lint diagnostics (MD0xx: blank-line spacing, table pipe alignment, trailing newlines, etc.) on `*.md` files. Author content for clarity and correctness; leave style-only lint alone unless the user explicitly asks to fix it.

### 0.2 Git Commit Policy

- **All commit messages must be in English** (subject line and body).
- Use conventional commit format: `type(scope): description`
  - **Type selection rules.** Before writing any commit message, match the change against these definitions in order. Do not guess by exclusion or "feel"; positively match the primary intent of the commit.

    | Type | Definition | When to use |
    | --- | --- | --- |
    | `feat` | A new feature or capability for the user. | Adding a page, a new section, a new interactive component. |
    | `fix` | A bug fix — something was broken, now it works. | Fixing a crash, wrong behavior, or an unintended visual defect. |
    | `refactor` | Code restructuring that changes no external behavior. | Moving functions, extracting a helper, renaming internals. Neither adds a feature nor fixes a bug. |
    | `style` | Formatting-only changes — whitespace, semicolons, indentation, linting. **No logic changes at all.** | Running a formatter, fixing indentation, adding trailing commas. If any runtime value, default, or behavior changes, it is NOT style. |
    | `docs` | Documentation only — `*.md`, comments, JSDoc. | Updating CHANGELOG, CODEBASE_STATE, README, code comments. |
    | `chore` | Build process, tooling, CI, dependencies, config files. **Not runtime application code.** | Updating lockfiles, CI scripts, lint config, adding a dev dependency. |
    | `perf` | A performance improvement with no behavior change. | Optimizing rendering, reducing bundle size, caching. |
    | `test` | Adding or updating tests only. | New test files, test fixtures, test config. |
    | `build` | Changes to the build system or external dependencies. | Nuxt/Vite config, package.json scripts, deployment config. |

    **Decision procedure:** Ask "what is the primary intent?" → match the definition → use that type. If a commit genuinely spans two types (e.g. a feature that also fixes a bug), pick the dominant one. If they are truly independent, split into separate commits.

    **Common mistakes to avoid:**
    - Changing a default value or layout behavior is NOT `style` — it changes runtime output. It is `fix` if correcting a defect, or `feat` if introducing a new visual capability.
    - Removing dead code or restructuring without behavior change is `refactor`, not `chore`.
    - `chore` is for build/tooling/CI, not a catch-all for "not feat and not fix".

    Reference: [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)

  - Scope: optional, e.g. `(page)`, `(component)`, `(style)`, `(layout)`, `(ci)`, `(agents)`.
  - Description: imperative mood, lowercase, no trailing period.
- **Body: lead-in sentence + dense bullets.** Non-trivial changes carry a body: a one-line lead-in summarizing the change, then one bullet per area. Bullets explain **mechanisms and design decisions** — how it works, which trade-offs were made and why, what was deferred and where the deferral is recorded — not a file inventory. Wrap at ~72 columns.
- **Never add co-author trailers** (e.g. `Co-authored-by: ...`) to commit messages.
- **Atomic commits.** Each commit must represent one cohesive, self-contained logical change. Do not bundle unrelated fixes, refactors, or features into the same commit; split when changes touch independent areas.
- When the user asks for a commit, **confirm the commit message and scope with the user** before executing, per the confirmation rules in 0.6.
- **Commit status reporting.** When committing, focus only on this session's changes — files outside this session's scope are neither inspected nor mentioned. If a conflict appears, report the conflict itself. The post-commit report states exactly: 「本对话涉及到的文件已经全部提交，且不涉及冲突」 when clean, or 「本对话涉及到的文件已经全部提交，但目前出现了冲突」 when conflicts exist (then identify only the conflicting paths this session touched). Never enumerate other pending working-tree changes in the report.
- Examples:
  - `feat(page): add projects showcase section to home page`
  - `fix(style): resolve mobile navigation overflow on narrow viewports`
  - `docs: sync codebase state`

### 0.3 Coding & Doc-sync Lifecycle

1. **Before coding**, skim §1–§7 once per session, then read `CODEBASE_STATE.md` and `docs/WIP.md` (if present) for anything relevant to your task.
2. **While coding**, the canonical registry of names is the **code itself**. Before adding a new component name, route, composable, or exported type, confirm it doesn't already exist in the relevant files — don't rely on docs for collision checks.
3. **After coding**, **do not edit any docs.** Keep a short scratch list of what would need to change so you can produce a clean batch when the user asks — then offer the sync per §5's post-task prompting rule.
4. **Pre-commit doc sync (user-initiated only).** When the user explicitly asks to update the codebase state or sync docs, do all of:
   - Record one line per coherent change in `CHANGELOG`, following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/): file the entry under `Unreleased` in the matching category (`Added` / `Changed` / `Deprecated` / `Removed` / `Fixed` / `Security`), newest date first within each category, and keep the existing order among entries of the same day. Each entry carries a `YYYY-MM-DD HH:MM` timestamp prefix. (Create the file on first use.) **If a changelog-related skill is installed (see §6.1), load it via the native skill mechanism before writing entries.**
     - **Write for humans, not a commit log.** A `CHANGELOG` entry is one or two plain sentences stating the noteworthy difference — what a visitor or developer will notice, not how the code got there. Shape: `- YYYY-MM-DD HH:MM — <sentence>.` The category heading already carries the change type.
     - **Do not put in an entry:** a `type(scope):` prefix or bold lead (commit-message syntax), file or path lists, internal function / constant names (unless user-facing), or mechanism / trade-off detail (that belongs in the commit body or `docs/DECISION_LOG.md`).
     - **One entry may span several commits**, and one commit may need no entry (pure refactor, doc-only). Write from the reader's side, not the commit's.
     - **Record the net result, not the path.** If a thing went A → B → C within the unreleased period, the entry says A → C (or just C if A never shipped); never list the intermediate B. When a later change reworks something already recorded, edit the existing entry instead of adding a new one. A bug introduced and fixed inside the same unreleased period needs no `Fixed` entry.
     - **Skip what readers cannot notice:** internal refactors, comment-only edits, type or lint fixes, dev-server tuning, and doc / decision-log updates need no entry. Always list deprecations, removals, and breaking changes.
   - If a change introduced a non-obvious trade-off (dependency swap, architecture change, new tooling), add a row to `docs/DECISION_LOG.md`. Never silently re-decide a logged choice.
   - If the directory map in `CODEBASE_STATE.md` drifted at the _folder_ level, update it. Don't enumerate individual new files.
   - Update `docs/WIP.md` if the work claimed a multi-turn area: strike through the row when done. (Create the file on first use.)

### 0.4 Conflict-avoidance Rules

- **Namespaces are globally unique.** Route paths, component names, composable names, and exported type names must not collide. Grep the code before adding.
- **One owner per feature.** When you start a multi-turn feature, note it in chat so other agents know you're claiming that area. The `docs/WIP.md` row lands during the next user-initiated doc sync.
- **Code wins, doc follows.** `CODEBASE_STATE.md` is human-curated context; if it ever conflicts with the code, trust the code and update the doc at sync time.
- **No silent deletes.** If you delete a documented symbol or route, move its row to a `~~strikethrough~~` line rather than silently removing it.
- **Atomic doc updates.** One feature → one coherent doc edit at commit time. Don't fold unrelated refactors into the same doc patch.

### 0.5 Read-before-write Checklist (copy into your plan)

- [ ] Confirm the feature area in `CODEBASE_STATE.md` and `docs/WIP.md` (if present).
- [ ] Grep relevant files for any new route, component, composable, or type you plan to add — confirm no collision.
- [ ] If a relevant decision exists in `docs/DECISION_LOG.md`, follow it; if your approach contradicts one, raise it in chat before re-deciding.
- [ ] Remember: do NOT edit any docs in this turn. Save doc updates for when the user asks.

### 0.6 Commit Confirmation

- **Before executing any `git commit`, confirm the message with the user via the built-in question-asking tool.** **Batch all commits into one call**: when multiple commits are proposed, send them all as separate questions in a single tool invocation so the user reviews every commit at once, not one at a time.
- Each question follows this format (preserve line breaks exactly):

  ```text
  该提交通过吗：
  {{commit message including body}}

  描述：{{short Chinese description of the change}}
  ```

- Options per question (exact labels, in this order):
  1. `通过` — confirmation given; proceed with the commit.
  2. `不通过，暂缓提交` — rejected; do not commit, leave changes staged/unstaged as they are.
  3. `不通过，原因是` — rejected with a reason; let the user enter the reason via free-text input, then address the feedback before re-proposing.
- Never commit without an explicit `通过` answer. If the user rejects, fix or wait as instructed — do not retry the same commit unprompted.
- This applies to **every commit path**: direct commit requests, `CCC` / `3C` step 3, and any batch flow.

---

## §1. Role & Context

You are an AI frontend developer building and maintaining **`momakoding.github.io`** — the official static website for Momakoding, built with **Nuxt 4** (Vue 3 `<script setup lang="ts">`), **TypeScript**, and **Tailwind CSS v4**, and deployed to **GitHub Pages** via GitHub Actions.

Keep the site lightweight, fast to load, responsive across viewports, and statically prerenderable without relying on a runtime Node.js server.

---

## §2. Development Workflow

### 2.1 Scope of Responsibilities

Current mode is limited to:

- **confirming** — clarify requirements before coding
- **coding** — feature and page implementation
- **boilerplating** — scaffolding and template code
- **first-glimpse review** — initial code review

Do not proactively refactor or optimize unrelated code.

### 2.2 Operational Standards

- **Communicate before modifying.** Before changing existing files, inform the user what you are about to change.
- **Confirm dependencies.** Do not auto-install packages. List the required packages and wait for user confirmation.
- **Verification approach.** Let the user verify in the browser — do not run `pnpm run dev` / `build` / `generate` / `test` automatically after every coding turn since HMR is assumed to be running. Short-time linters and type-check-only commands (such as `pnpm run typecheck` / `vue-tsc`) are acceptable, and running a build when the user explicitly requests verification is allowed.
- **Clarify unclear requirements** before writing code.
- **Documentation sync.** See 0.1's proactive-ask rule — offer to sync docs and commit after finishing a coding task; don't wait to be asked and don't do it silently.
- **Session-start skill load.** At session start, load the humanizer skill matching the conversation language (e.g. `humanizer-zh` for Chinese, `humanizer` for English) via the native skill mechanism. If loading fails, report the reason instead of skipping silently; if it is not installed, remind the user to install it.
- **Don't thrash.** If an error or blocker resists, do not retry the same direction more than twice — stop and explain the situation.

### 2.3 Framework Integration Rules

#### 2.3.1 Vue & Nuxt 4

- **Vue 3 SFC + `<script setup lang="ts">` only.** No Options API.
- **Strict TypeScript.**
- **Directory layering.** Source lives under `app/`: pages in `app/pages/`, shared layouts in `app/layouts/`, UI components in `app/components/`, reactive composables in `app/composables/`, pure helpers in `app/utils/`, static assets in `app/assets/` and `public/`. Keep SFC templates focused on presentation and delegate non-trivial logic to composables or utilities.
- **Import paths.** Use the **`~` alias** (`→ app/` in Nuxt 4) for cross-folder imports. Relative `./` is allowed **only** for same-folder siblings. Avoid upward relative chains (`../`, `../..`) that reach outside the current folder — they break when files move. Nuxt auto-imports (`useState`, components, composables) need no explicit import.
- **Static prerender compatibility.** The site deploys via `nuxt generate` with Nitro preset `github_pages`. Guard browser-only APIs (`window`, `document`, `IntersectionObserver`, `localStorage`) inside `onMounted` or `<ClientOnly>`, and do not introduce runtime server API routes unless a deployment change is logged in `docs/DECISION_LOG.md`.

#### 2.3.2 CSS (Tailwind CSS v4)

This project uses **Tailwind CSS v4** wired via `@tailwindcss/vite` with entry stylesheet `app/assets/css/main.css`. v4 differs from v3 in real ways — read this whole subsection before writing styles.

**Principle: atomic-first, split by semantics.** Default to utilities, then decide by **whether the element carries real meaning**:

- **Meaningful component parts → BEM class + `@apply`.** When a class names a genuine part of a component (a hero banner, a project card header, a nav pill, a footer link group), give it a BEM class and compose it with `@apply` inside a scoped block. The name documents intent, keeps templates readable, and allows reuse within the component.

  ```vue
  <template>
    <article class="project-card project-card--featured">...</article>
  </template>

  <style scoped>
  @reference "~/assets/css/main.css";

  .project-card {
    @apply rounded-2xl border border-slate-800 bg-slate-900/60 p-6;
  }
  .project-card--featured {
    @apply border-sky-400/50;
  }
  </style>
  ```

- **Throwaway / rendered / non-semantic markup → inline utilities.** For markup with no standalone meaning — especially wrapper divs or dynamically rendered HTML — stack atomic classes inline without inventing a class name.
- **Add a component-scoped `<style>` only when utilities aren't enough** — a meaningful/repeated class list, a pseudo-element, a complex selector, or a keyframe animation.
- **Lift anything reused across components into `app/assets/css/main.css`** (a shared `@utility` or `@theme` token) rather than copy-pasting per file.
- Keep styles modular and portable — no undocumented coupling to global element selectors.

**Referencing the global stylesheet — use the `~` alias, NOT `@`.** A scoped `<style>` block is compiled in isolation, so to use `@apply`, custom utilities, or theme tokens inside it you must first pull in the global sheet with `@reference`. In this Nuxt project the path **must use the `~` alias** (`→ app/`), because Tailwind v4's `@reference` does **not** resolve Vite's `@` alias:

```vue
<style scoped>
@reference "~/assets/css/main.css";   /* ✅ correct */

.hero-card {
  @apply max-w-3xl px-6 py-8 rounded-2xl wrap-break-word;
}
</style>
```

```css
@reference "@/assets/css/main.css";   /* ❌ throws ENOENT: no such file or directory */
```

Forgetting `@reference` causes `@apply` and custom utilities inside `<style scoped>` to emit nothing silently. See `docs/TROUBLESHOOTING.md`.

**`@utility` — make a custom class `@apply`-able.** A plain `.foo {}` cannot be consumed by `@apply` from another file. To define a reusable class that *can* be `@apply`-ed, declare it with `@utility` in `app/assets/css/main.css`:

```css
/* app/assets/css/main.css */
@utility surface-panel {
  border-radius: 1rem;
  border: 1px solid var(--color-border-subtle);
  background-color: var(--color-surface);
}
```

**`@theme` — define reusable design tokens.** Declare shared colors, typography, and spacing tokens in `app/assets/css/main.css` under `@theme`; each becomes both a CSS variable and a Tailwind utility (`bg-*`, `text-*`, `border-*`):

```css
@theme {
  --color-surface: #0f172a;
  --color-border-subtle: rgba(148, 163, 184, 0.2);
}
```

**v3 → v4 gotchas:**

- **Opacity:** standalone `bg-opacity-*` / `text-opacity-*` classes are **gone**. Use slash syntax — `bg-white/50`, `text-slate-400/80` — or `opacity-50`.
- **Word wrap:** use built-in v4 utilities like `wrap-break-word` instead of custom `word-wrap` rules.
- **Plugin-gated classes:** `animate-in` / `fade-in` / `slide-in-*` need a dedicated animation plugin; `prose` / `prose-lg` need `@tailwindcss/typography`. Both are extra dependencies — **do not add them without a `docs/DECISION_LOG.md` entry** (§4) — then enable each via `@plugin "..."` in `app/assets/css/main.css`.

#### 2.3.3 State Management

- **Component-local state** uses `ref` / `reactive`.
- **Cross-component / cross-page state** uses Nuxt's `useState` (SSR-safe shared reactivity) or `provide` / `inject` scoped to a component tree.
- **No Pinia / Vuex** unless a decision is logged in `docs/DECISION_LOG.md`.

---

## §3. Code Standards

### 3.1 TypeScript Basics

- **No `var`.** `const` by default, `let` only when reassignment is real.
- **No `any`.** Use `unknown` + narrowing, or define a real type.
- **Typed props and emits.** All props typed; all events typed. Non-trivial payloads get a named type used at both ends.
- **Single Responsibility Principle.** Functions should do one thing; when a function grows multiple distinct stages, split it rather than letting it accrete.
- **Blank lines between stages.** Within a function body, separate distinct logical stages with a blank line so each phase reads as one visual block.
- **Doc comments on functions.** Every function carries a documentation comment (`/** ... */`) describing its purpose; use single-line `//` comments only for necessary clarifications inside the body.
- **Inline spec references.** When a comment describes behavior governed by a spec or design doc, put the reference at the end of the relevant sentence or block — never mid-sentence. Pattern: `…detail. See <doc> <section>.`
- **Public types survive autocomplete.** Prefer `LiteralUnion<T>` over `T | string` for open string unions.

### 3.2 Component Rules

- **Strict UI separation.** Components render UI and hold component-local UI state only; move reusable data formatting, filtering, or side effects into `app/composables/` or `app/utils/`.
- **v-model contract.** Custom input components take `modelValue` and emit `update:modelValue` (or use Vue 3.4+ typed `defineModel`).

### 3.3 Code Quality

- **No magic numbers.** Define named constants in `as const` objects.
- **Loop hygiene.** Pre-compute outside tight loops and avoid redundant reactive work during renders.
- **Modularity.** Separate types, static content data, pure helpers, composables, and SFC presentation cleanly.
- **Consistent naming.** Follow §7 naming conventions.

### 3.4 TypeScript Iteration Conventions

- **Array iteration.** Prefer chainable methods (`forEach`, `map`, `filter`, `reduce`, `some`, `every`, etc.) over raw `for` loops unless performance is critical in hot paths.
- **Object iteration.** Prefer `Object.entries()` combined with array methods over `for...in` loops to avoid prototype chain pitfalls; use `Object.keys()` or `Object.entries()` with explicit type guards when needed.
- **Plain `for` loops.** Only use when you need early exit, index arithmetic, or a performance-sensitive section. Document the reason if it is not obvious.

### 3.5 Feature Modules — barrel & public API

When a feature unit outgrows one file, split it into a folder module: data models, constants, and each pure-logic concern get their own files, and the folder exposes a single public surface through its `index.ts` (barrel).

- **Unified export.** The module's public API is exactly what its `index.ts` re-exports; anything not re-exported there is private to the folder.
- **Unified import.** Callers import from the folder root only (`~/<module>` cross-folder, `./<module>` same-folder siblings), never from files inside it. When a folder module lacks an `index.ts` and is not a framework-managed directory (such as Nuxt `app/pages/` or auto-imported `app/components/`), create one instead of deep-importing.
- **Enforcement.** Once ESLint is adopted (new dev dependency ⇒ `docs/DECISION_LOG.md`), enforce folder boundaries with restricted-import rules.

### 3.6 Diagrams

- **Box-and-line diagrams — flowcharts, architecture diagrams, sequence diagrams, and the like — use Mermaid.** Draw them as fenced code blocks with the `mermaid` language tag (`` ```mermaid ``), inline in the Markdown they illustrate. No ASCII art for these.
- **Directory trees** may use fenced plain-text blocks in `CODEBASE_STATE.md` or SVG image files when a visual diagram is needed.
- Prefer simple, readable layouts over exhaustive detail — a diagram that needs a legend is too complex.

---

## §4. Forbidden

- `var`.
- `any` — use `unknown` + narrowing, or define a real type.
- Direct DOM manipulation outside Vue's reactivity system.
- Unnamed magic numbers scattered in code.
- Blocking operations on the main thread.
- **Using `write` on existing files.** `write` is for creating new files only. Modifying an existing file — no matter how small the change — must go through a targeted `edit`, never a full-file `write` overwrite.
- **Running the project or opening files yourself after finishing a coding task — the user runs it.** (Exception: the user explicitly asks you to start the dev server or run a build; when starting a dev server, run it in the background so the call doesn't block. Short-time linters and type-check-only commands such as `pnpm run typecheck` are acceptable.)
- Adding a new dependency without a `docs/DECISION_LOG.md` entry.
- Auto-installing packages without user confirmation.
- Silent renames of exported symbols, route paths, composable names, or type names.
- Upward relative import paths (`../`, `../..`) that reach outside the current folder — use the `~` alias.
- Retrying the same failing approach more than twice without stopping to explain.
- Reading any config files (`.env`, `.yaml`, `.yml`, etc.) unless they are sample files with an `.example` suffix (e.g. `.yaml.example`, `.env.example`). Reading project config like `nuxt.config.ts` / `package.json` is fine.
- Compressing any doc documents.
- Co-author trailers in commit messages.

---

## §5. Interaction Protocol

- Before adding a feature, check `docs/WIP.md` (if present) for conflicts.
- When the user describes a **visual or interaction requirement**, translate it directly into implementation (CSS classes, layout structure, component props, composable logic) — don't over-engineer.
- When asked to research or explain, do not edit code.
- When asked to implement, follow the Read-before-write checklist in §0.
- **Post-task prompting.** Offer `CCC` / `3C` (§8) **if and only if** the finished task was a coding task. If the turn only changed development documents (`docs/*`, `AGENTS.md`, `CODEBASE_STATE.md`, etc. — not `app/assets/` files), you may still remind the user to commit, but there is nothing to sync: skip the `CODEBASE_STATE.md` update and the `CHANGELOG` entry. Never prompt after research- or explanation-only turns.

---

## §6. Skills

Skills use a **two-tier architecture**:

1. **Source of truth**: `.agents/skills/<name>/SKILL.md` — agent-agnostic, shared across all agents and tooling.
2. **Agent deployment**: On project initialization for a specific agent, skills are migrated to that agent's native discovery folder (e.g. `.kilo/skills/<name>/SKILL.md`). The agent then auto-discovers and loads them via its native mechanism (no manual file reading).

**IMPORTANT: Always use the agent's native skill registration and discovery mechanism to load skills. Do NOT directly read skill files as plain documents.**

**Rules:**

- When adding a new skill, create the file under `.agents/skills/<name>/SKILL.md` with `name` and `description` in YAML frontmatter.
- When initializing a project for an agent, copy `.agents/skills/` entries into the agent's native skill directory. The deployed copy is what the agent loads.
- Do NOT edit deployed copies directly — edit the source in `.agents/skills/` and re-deploy.
- A task may match multiple skills — load all matching ones before writing code.

### 6.1 Recommended skills

- **[changelog-principle](https://github.com/NBDatsuya/changelog-principle)** — write `CHANGELOG` entries from the net diff against the last release, not from `git log` history. Load it via the native skill mechanism before writing changelog entries (see 0.3).
- **[humanizer](https://github.com/blader/humanizer)** and its Chinese edition **[humanizer-zh](https://github.com/op7418/Humanizer-zh)** — rewrite AI-sounding prose so it reads naturally without changing what it says. Load the one matching the conversation language at session start (§2.2).
- **[qu-ai-wei](https://github.com/LifelongLazyLearner/qu-ai-wei)** — humanize Simplified Chinese while preserving facts, certainty, register, and the author's voice.

---

## §7. Naming Conventions

- **Files**: `kebab-case` (e.g. `project-card.vue`, `site-links.ts`). Exception: root `app.vue` and Nuxt dynamic route parameters (`[slug].vue`).
- **Component references in templates**: `PascalCase` (e.g. `<ProjectCard>`, `<SiteHeader>`).
- **Routes / Pages**: `kebab-case` under `app/pages/` (e.g. `app/pages/about.vue` → `/about`).
- **Types & Interfaces**: `PascalCase` (e.g. `ProjectItem`, `NavLink`).
- **Composables**: `camelCase` with `use` prefix (e.g. `useSiteTheme`, `useScrollSpy`).
- **Constants**: `UPPER_SNAKE_CASE` inside `as const` objects.
- **Folder modules**: when a unit outgrows one file, extract a folder and export its public surface via `index.ts` (rule details: §3.5).

---

## §8. Quick Commands

### 8.1 CCC / 3C

When the user inputs **`CCC`** or **`3C`**, execute the following three steps in order. **Scope: this session only.** Cover only the changes made in the current session; never stage, document, or mention unrelated pending changes. Broaden to the whole working tree only when the user explicitly asks for "all" changes.

**CCC is NOT a working-tree cleanup — anything unrelated to the current session is off-limits: do not read it, do not mention it, do not ask about it.** During CCC, **reading is restricted to files this session touched or created** — opening, inspecting, or investigating any other file (even via `git status` / `git diff` side curiosity about unrelated paths) is **forbidden**. Do NOT try to clean up the working tree, do NOT bring unrelated changes up in conversation, and do NOT ask the user what to do with them — leave them untouched and unmentioned. Stage by explicit path (the files this session touched), never by blanket `git add .` / `git add -A`.

1. **CHANGELOG** — Append entries per coherent change made in this session to `CHANGELOG` (newest first, `YYYY-MM-DD HH:MM` timestamp, Keep a Changelog format, plain reader-facing sentences per 0.3 — no commit-message syntax). Create on first use if absent.
2. **CODEBASE** — Sync `CODEBASE_STATE.md` (directory map at folder level), `docs/DECISION_LOG.md` (rows for non-obvious trade-offs), and `docs/WIP.md` (strikethrough completed rows) for this session's changes only. **Skip CODEBASE when the session's changes are code-quality work only** (refactors that do not touch architecture, module layout, or public APIs).
3. **COMMIT** — Confirm the commit message and scope with the user (per 0.2 and 0.6), then stage only this session's changes and commit using conventional commit format.

---

## §9. Documentation Map

This file covers agent _behavior_ only. Project _facts_ live elsewhere:

- [`CODEBASE_STATE.md`](./CODEBASE_STATE.md) — project snapshot, tech stack, directory map, source-of-truth pointers, and architecture notes. **Uses its own independent section numbering** — never shares this file's `§` sequence. Read freely; edit only during a user-initiated sync per §0.
- [`docs/DECISION_LOG.md`](./docs/DECISION_LOG.md) — architectural and tooling decisions (append-only).
- [`docs/WIP.md`](./docs/WIP.md) — claimed work-in-progress areas. Created on first use.
- [`docs/TROUBLESHOOTING.md`](./docs/TROUBLESHOOTING.md) — known issues and fixes (symptom → root cause → fix → prevention).
- [`CHANGELOG`](./CHANGELOG) — per-change history, newest first, in [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format with `YYYY-MM-DD HH:MM` timestamps.
