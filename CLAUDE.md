# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Kit Detection

If the root directory name is **not** `next-kit`, this repo has been cloned for a new project. Before doing anything else, walk the user through the **Project Setup Checklist** at the bottom of this file, prompting for each value (project name, metadata, font choice, theme colors, etc.) and applying the changes. Skip any items the user has already completed.

## Overview

Minimal Next.js starter kit for building landing pages. This is a template repo — see the setup checklist below when cloning for a new project.

**Stack:** Next.js 16 (App Router), React 19, TypeScript 5 (strict), Tailwind CSS 4 (PostCSS plugin, inline `@theme`), Inter (sans, via `next/font/google`), ESLint 9 (flat config with `defineConfig`), Prettier (with Tailwind CSS class sorting)

## Commands

- `npm run dev` — Dev server on port 3000
- `npm run build` — Production build
- `npm run start` — Start production server
- `npm run lint` — ESLint
- `npm run type-check` — TypeScript type checking (`tsc --noEmit`)
- `npm run format` — Prettier formatting
- `npm run format:check` — Check formatting

No test framework is configured — there is no `test` script and no test runner installed.

## Architecture

- **App Router** — Pages and layouts in `src/app/`
- **Layout** — `layout.tsx` wraps `{children}` in `<main>` so every route gets a top-level landmark. Don't add another `<main>` inside individual pages.
- **Metadata** — `layout.tsx` sets `metadataBase` to `https://example.com` (placeholder — replace with the production URL per project, see setup checklist). Required for OG/Twitter image URL resolution.
- **Path alias** — `@/*` maps to `src/*`
- **Tailwind CSS 4** — Theme configured inline via `@theme` in `src/app/globals.css`; colors use CSS custom properties (`--background`, `--foreground`) defined in `:root`
- **Fonts** — Inter loaded via `next/font/google` in `layout.tsx` (Next self-hosts it at build time — no runtime request to Google, so the strict `font-src 'self'` CSP is unaffected), `latin` subset, `display: "swap"`, exposed as the `--font-inter` CSS variable and mapped to `--font-sans` in `@theme` in `globals.css`. Only a sans family is shipped — there is no `--font-mono`; add one in `@theme` if a project needs monospace. The `body` `font-family` intentionally restates the same `--font-inter` chain — some mobile browsers need the explicit declaration. `src/fonts/` is empty (`.gitkeep`), available for self-hosted local fonts via `next/font/local`.
- **Components** — `src/components/sections/` for page sections, `src/components/common/` for reusable utility components
- **Static assets** — `public/images/`, `public/videos/`, `public/vectors/`
- **Custom font files** — `src/fonts/` (empty; for self-hosted `next/font/local` fonts)
- **ESLint** — Flat config in `eslint.config.mjs` using `defineConfig` from `eslint/config` with `eslint-config-next/core-web-vitals` and `eslint-config-next/typescript`

## Security

- **No indexing** — Crawling/indexing blocked via `robots.txt`, `robots` metadata in `layout.tsx`, and `X-Robots-Tag` header in `next.config.ts` (remove before launch)
- **Security headers** — Comprehensive HTTP security headers in `next.config.ts`: HSTS, CSP, COOP, CORP, COEP, X-Content-Type-Options, X-Frame-Options, X-DNS-Prefetch-Control, X-Permitted-Cross-Domain-Policies, Referrer-Policy, Permissions-Policy. The kit intentionally sets both CSP `frame-ancestors 'none'` and `X-Frame-Options: DENY` — modern browsers honor `frame-ancestors`, XFO covers legacy browsers.
- **CSP** — The Content Security Policy in `next.config.ts` is strict (`default-src 'self'`; `img-src` allows only `'self' blob:`, so `data:` URI images are blocked). When adding external resources (analytics, fonts, images, APIs), update the CSP directives accordingly or they will be blocked. `'unsafe-inline'` is required in `script-src` because Next.js App Router injects inline scripts for RSC payloads — removing it breaks hydration; `style-src` also carries `'unsafe-inline'`. Dev mode additionally includes `'unsafe-eval'`. For stricter CSP, implement nonce-based CSP via middleware (requires dynamic rendering).
- **COEP** — `Cross-Origin-Embedder-Policy: require-corp` blocks cross-origin resources even when the CSP allows them, unless the resource is served with CORP/CORS headers. When adding external resources, updating the CSP alone is not enough — also relax COEP or ensure the resource supports `crossorigin`.
- **`poweredByHeader: false`** — Next.js framework identification disabled
- **`security.txt`** — Vulnerability reporting contact at `public/.well-known/security.txt`. Update the `Contact`, `Canonical`, and `Expires` fields per project; renew `Expires` before it lapses (RFC 9116) or reporters' tooling treats the file as stale.

## Automation

The kit has no push-triggered CI (local checks plus the deploy build cover that); automation exists to watch the repo while it sits idle between projects:

- **Dependabot, security-only** — `.github/dependabot.yml` sets `open-pull-requests-limit: 0`, which disables version-update PRs; only security updates open PRs. Requires "Dependabot alerts" and "Dependabot security updates" enabled in the GitHub repo settings (`gh api -X PUT repos/<owner>/<repo>/vulnerability-alerts` and `.../automated-security-fixes`) — the yml alone does nothing without them.
- **Weekly audit** — `.github/workflows/audit.yml` runs `npm audit` Mondays 06:00 UTC (and via manual dispatch). It fails on advisories of _any_ severity, matching the kit's practice of fixing even moderate ones, and catches transitive advisories that Dependabot can't cleanly PR (the `overrides` cases). It runs against the lockfile only — no `npm ci`, so install scripts never run in CI.

## Environment Variables

- **`DEV_ORIGIN`** — Optional. Set in `.env.local` to allow a custom dev origin (e.g., a tunneled URL) via `allowedDevOrigins` in `next.config.ts`

## Editor

VS Code workspace config in `.vscode/`: recommended extensions (ESLint, Prettier, Tailwind CSS IntelliSense) and settings that suppress unknown `@` rule warnings from Tailwind CSS 4 syntax, enable format-on-save, pin Prettier as the default formatter, and auto-apply ESLint fixes on save (`source.fixAll.eslint`).

`.claude/` is gitignored — project-level Claude Code settings, permissions, or skills placed there stay local to this machine. Remove the `.gitignore` entry first if that config should be committed and shared.

## Dependency Notes

- **`overrides`** — `package.json` pins five transitive deps for security fixes:
  - `@babel/core@^7.29.7` — fixes arbitrary file read via sourceMappingURL (GHSA-4x5r-pxfx-6jf8) pulled in via `eslint-plugin-react-hooks`. Keep until `eslint-config-next` updates its dependency tree.
  - `js-yaml@^4.2.1` — fixes quadratic-complexity DoS in merge-key handling (GHSA-h67p-54hq-rp68, and GHSA-52cp-r559-cp3m which covered up to 4.2.0) pulled in via `@eslint/eslintrc`. Keep until ESLint updates its dependency tree.
  - `minimatch@^10` — fixes security advisories in transitive ESLint dependencies. Keep until `eslint-config-next` updates its dependency tree. Forces a semver-major jump (ESLint-family packages declare `^3.1.x`) — if ESLint glob/ignore matching misbehaves, suspect this first.
  - `postcss@^8.5.10` — fixes CVE in postcss pulled in by both Next.js and `@tailwindcss/postcss`. Keep until both upstreams ship 8.5.10+.
  - `sharp@^0.35.3` — fixes inherited libvips CVEs (GHSA-f88m-g3jw-g9cj); Next 16 declares `^0.34.5` as an optional dep. A 0.x minor jump over Next's declared range — if `next/image` optimization misbehaves, suspect this first. Keep until Next bumps its sharp range.
- **`allowScripts`** — `package.json` records reviewed install scripts per npm's `approve-scripts` feature: `sharp` (prebuilt-binary check) and `unrs-resolver` (napi postinstall check), both pinned to the installed version. When either package's version changes, `npm install` warns again — review and re-run `npm approve-scripts <pkg>` to update the pin.
- **`engines`** — Kit declares `node >=22.0.0` (Node 22 LTS). `@types/node` is pinned to `^22` to match.

## Project Setup Checklist

When cloning this kit for a new project, update the following:

1. **`package.json`** — Change `name` from `"next-kit"` to the project name
2. **`src/app/layout.tsx`** — Update `metadata.title` from `"next-kit"`; update `metadata.metadataBase` from `https://example.com` to the production URL; add `description` and Open Graph tags as needed
3. **`src/app/page.tsx`** — Replace the placeholder landing page content
4. **`README.md`** — Update title, description, and any project-specific details
5. **Font** — Kit ships Inter via `next/font/google`. To swap it: change the `next/font/google` import and variable in `layout.tsx` (or use `next/font/local` with files in `src/fonts/`), then update the `--font-*` references in `globals.css`
6. **Theming** — Update `--background` and `--foreground` CSS variables in `globals.css`
7. **Metadata** — Add favicon, og-image, and other assets to `public/`
8. **Security** — Update `security.txt` contact, expiry, and `Canonical:` URL; adjust CSP in `next.config.ts` if needed; remove `X-Robots-Tag` header, `robots.txt` disallow, and `robots` metadata when ready to launch
9. **Environment** — Copy `.env.example` to `.env.local` and fill in values if needed
10. **`.gitkeep` files** — Remove from any folder once it has actual content
11. **Dependabot** — Enable "Dependabot alerts" and "Dependabot security updates" on the new GitHub repo (see Automation section); the checked-in `dependabot.yml` and audit workflow do the rest
