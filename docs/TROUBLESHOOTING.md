# Troubleshooting

Known issues and fixes for `momakoding.github.io`, recorded as **Symptom → Root Cause → Fix → Prevention**.

---

## 1. `@reference` in `<style scoped>` throws `ENOENT` or `@apply` emits no CSS

- **Symptom:** Using `@apply` or custom `@theme` / `@utility` tokens inside a Vue SFC `<style scoped>` block either silently outputs no styles or fails during Vite build with `ENOENT: no such file or directory` when writing `@reference "@/assets/css/main.css";`.
- **Root Cause:** Vue compiles `<style scoped>` blocks in isolation, so Tailwind CSS v4 does not see global tokens unless imported via `@reference`. In this Nuxt 4 setup, Tailwind v4 resolves Nuxt's `~` alias (`→ app/`) but does not resolve `@` inside `@reference`.
- **Fix:** Add `@reference "~/assets/css/main.css";` at the top of the `<style scoped>` block.
- **Prevention:** Always use `~` (never `@`) for `@reference` in SFC scoped styles per `AGENTS.md` §2.3.2.

---

## 2. pnpm 12 fails with `ERR_PNPM_IGNORED_BUILDS` or `ERR_PNPM_FROZEN_LOCKFILE_WITH_OUTDATED_LOCKFILE`

- **Symptom:** Running `pnpm install` fails with `Ignored build scripts: esbuild@...`, or `pnpm install --frozen-lockfile` fails with `Cannot update packageManagerDependencies with "frozen-lockfile" because the lockfile is not up to date`.
- **Root Cause:** pnpm 12 requires explicit `allowBuilds` entries in `pnpm-workspace.yaml` for packages with native/install build scripts, and stores `packageManager` metadata inside `pnpm-lock.yaml`.
- **Fix:** Confirm the needed package is approved in `pnpm-workspace.yaml`, then run `pnpm install` once to refresh `pnpm-lock.yaml` before verifying `pnpm install --frozen-lockfile`.
- **Prevention:** Whenever `packageManager` or dependencies change in `package.json`, sync `pnpm-lock.yaml` and test `pnpm install --frozen-lockfile` before pushing to CI.

---

## 3. Nuxt prints an unsupported Node.js version warning locally

- **Symptom:** Running Nuxt commands on Node `22.19.0` prints `Current version of Node.js (22.19.0) is unsupported and might cause issues. Please upgrade to a compatible version ^22.21.0 || ^24.11.0 || >=26.0.0.`
- **Root Cause:** Nuxt 4.6+ targets newer Node 22/24 patch releases.
- **Fix:** Upgrade local Node.js to `>=22.21.0` or Node 24 LTS if runtime issues occur.
- **Prevention:** GitHub Actions CI (`.github/workflows/deploy.yml`) is already configured to use Node 24.
