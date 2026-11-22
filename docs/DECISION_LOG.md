# Decision Log for momakoding.github.io

Append-only record of non-obvious architecture, dependency, and tooling decisions. Never silently re-decide a logged choice — raise it in chat first if a new approach contradicts an entry below.

| Date | Decision | Rationale |
| --- | --- | --- |
| 2026-11-22 | **Initialize the site with Nuxt 4 (`app/` directory) and Nitro `github_pages` preset, deployed via GitHub Actions (`.github/workflows/deploy.yml` on Node 24).** | `momakoding.github.io` is a root organization GitHub Pages site (`/` base URL). Setting `nitro.preset: 'github_pages'` ensures `pnpm run generate` emits `.output/public` with `.nojekyll`, `200.html`, and `404.html` identically both locally and in CI, while GitHub Actions runs `typecheck` and `generate` on a supported Node 24 runtime. |
| 2026-11-22 | **Wire Tailwind CSS v4 directly through `@tailwindcss/vite` and `app/assets/css/main.css` instead of `@nuxtjs/tailwindcss`.** | Tailwind CSS v4 provides first-class Vite integration with zero extra Nuxt module indirection, faster CSS builds, and native `@theme` / `@utility` / `@reference` support. |
| 2026-11-22 | **Pin `typescript` to `<7` (`^6.0.3`) alongside `vue-tsc`.** | `@volar/typescript@2.4.30` and `@volar/language-service@2.4.30` declare a `<7` peer dependency on TypeScript; installing TypeScript 7 triggers unmet peer warnings under `pnpm peers check`. Pinning to 6.x keeps `nuxt typecheck` and peer checks clean. |
| 2026-11-22 | **Approve `esbuild` build scripts in `pnpm-workspace.yaml` (`allowBuilds.esbuild: true`) and declare `"packageManager": "pnpm@12.6.0"` in `package.json`.** | pnpm 12 blocks unapproved dependency build scripts with `ERR_PNPM_IGNORED_BUILDS` and validates `packageManager` alignment during `pnpm install --frozen-lockfile`. Recording the approval in `pnpm-workspace.yaml` and pinning `packageManager` lets both local installs and `pnpm/action-setup@v4` in CI resolve deterministically. |
