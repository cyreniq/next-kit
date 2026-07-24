# next-kit

A minimal Next.js starter kit for building landing pages — hardened, typed, and ready to clone.

## Stack

- **Next.js 16** — App Router
- **React 19**
- **TypeScript 5** — Strict mode
- **Tailwind CSS 4** — Via PostCSS plugin, inline `@theme` configuration
- **Inter** — Sans font, self-hosted at build time via `next/font/google`
- **ESLint 9** — Next.js Core Web Vitals + TypeScript rules (flat config)
- **Prettier** — With Tailwind CSS class sorting

## Getting Started

Requires **Node.js 22+**.

```bash
npm install
npm run dev
```

Dev server runs on [http://localhost:3000](http://localhost:3000).

## Scripts

| Command                | Description                        |
| ---------------------- | ---------------------------------- |
| `npm run dev`          | Start dev server on port 3000      |
| `npm run build`        | Production build                   |
| `npm run start`        | Start production server            |
| `npm run lint`         | Run ESLint                         |
| `npm run type-check`   | TypeScript type checking (no emit) |
| `npm run format`       | Format with Prettier               |
| `npm run format:check` | Check formatting without writing   |

No test framework is configured.

## Project Structure

```
src/
  app/                  # Pages and layouts (App Router)
  components/
    sections/           # Full-width landing page sections
    common/             # Reusable utility components
  fonts/                # Optional self-hosted local fonts (next/font/local)
public/
  .well-known/          # security.txt
  images/
  videos/
  vectors/
```

## Path Alias

`@/*` maps to `src/*` — e.g. `import Foo from "@/components/common/Foo"`.

## Fonts

Inter is loaded via `next/font/google` in `src/app/layout.tsx`. Next self-hosts it
at build time, so there's no runtime request to Google and the strict `font-src 'self'`
CSP is unaffected. It's exposed as the `--font-inter` CSS variable and mapped to
`--font-sans` in `globals.css`. To swap fonts, change the import in `layout.tsx`
(or use `next/font/local` with files in `src/fonts/`) and update the `--font-*`
references in `globals.css`.

## Theming

Colors and fonts are configured via CSS variables in `src/app/globals.css` using
Tailwind CSS 4's `@theme inline` directive. Edit `--background` / `--foreground`
in `:root` to reskin.

## Environment

Copy `.env.example` to `.env.local` and fill in values as needed.

| Variable     | Description                                                                            |
| ------------ | -------------------------------------------------------------------------------------- |
| `DEV_ORIGIN` | Optional. Tunneled dev origin (e.g. an ngrok URL) added to Next's `allowedDevOrigins`. |

## Security

Pre-configured with hardened HTTP security headers (`next.config.ts`) — HSTS, a strict
Content Security Policy, COOP/CORP/COEP, and a locked-down Permissions-Policy — plus a
`security.txt` disclosure contact. **Indexing is blocked by default** via `robots.txt`,
`robots` metadata, and the `X-Robots-Tag` header; remove all three before launch. See
the Security section of `CLAUDE.md` for the details and trade-offs.

Dependency advisories are watched automatically: Dependabot opens security-only PRs
(version-update PRs are disabled) and a weekly GitHub Actions job runs `npm audit`
against the lockfile. Both require Dependabot alerts/security updates to be enabled
in the repo settings — see the Automation section of `CLAUDE.md`.

## Editor

VS Code workspace config lives in `.vscode/`: recommended extensions (ESLint, Prettier,
Tailwind CSS IntelliSense) and settings for format-on-save with Prettier and ESLint
auto-fix.

## Setting Up a New Project

See `CLAUDE.md` for a step-by-step checklist of what to update when starting a new
project from this kit (name, metadata, fonts, theme, security contact, and lifting the
no-index guards).
