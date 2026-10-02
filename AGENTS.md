# AI Agent Development Guide

## Project Overview

**standard-ui-blank-astro-daisyui-ssr** — Minimal Astro SSR starter with React islands, Tailwind CSS, DaisyUI, and Resend for transactional email. Output mode is `server` (SSR via Node.js standalone adapter). Use as a base for content-driven or marketing sites that need server-side rendering.

**Key characteristics:**

- Astro 7, `output: "server"`, `@astrojs/node` standalone adapter
- React 19 islands for interactive components
- Tailwind CSS v3 + DaisyUI v5
- Resend for email (`src/pages/api/` endpoints)
- Zod for env/request validation, `sanitize-html` for user content
- ESLint (with `eslint-plugin-astro`) + OxFmt enforced in pre-commit
- Zustand v5 (available, not pre-wired)

## Project Structure

```
src/
├── components/
│   ├── Footer.astro        # Static footer
│   └── TopNav.tsx          # Navigation React island
├── layouts/                # Page layout wrappers
├── lib/
│   ├── env.ts              # Typed env var access (PUBLIC_* + server-side)
│   ├── logger.ts           # Server-side logging
│   ├── sanitize.ts         # sanitize-html wrapper
│   └── schemas/            # Zod schemas for forms / API input
├── middleware.ts            # Astro middleware (auth, logging, etc.)
├── pages/
│   ├── index.astro
│   ├── contact.astro
│   ├── 404.astro
│   ├── 500.astro
│   └── api/                # Server endpoints (Resend email, etc.)
└── styles/
```

## Architecture Rules

- **Pages** — always `.astro`. No React pages.
- **Static content** — `.astro` components. Interactive UI — `.tsx` React islands (`client:load` / `client:only`).
- **Server-side secrets** — only accessible in `.astro` frontmatter, `src/pages/api/`, and `src/lib/`. Never expose them to the client.
- **`PUBLIC_*` env vars** — embedded in the client bundle. Never put secrets there.
- **Email** — Resend via API endpoints in `src/pages/api/`. Always validate and sanitize input with Zod + `sanitize-html` before sending.
- **`checkOrigin: false`** — intentionally disabled (ingress header issue); do not re-enable without verifying the infra fix.

## Environment Variables

```env
PUBLIC_SITE_URL=https://example.com     # Public — embedded in client

RESEND_API_KEY=re_xxx                   # Server-only secret
RESEND_FROM=noreply@example.com         # Server-only
RESEND_FROM_NAME=                       # Optional display name
RESEND_TO=you@example.com              # Server-only
```

Server-side vars are accessed via `src/lib/env.ts` — never via `import.meta.env` directly in components.

## Execution Discipline

- Root cause first. Read the relevant page/component/endpoint before editing.
- After two identical failures without new evidence, change approach — do not retry blindly.
- Run `pnpm build` (includes `astro check`) before presenting a result.
- No speculative additions — add only what the request directly requires.

## Security

- Server-side secrets (`RESEND_API_KEY`, etc.) must never appear in client code or `PUBLIC_*` vars.
- Always validate API endpoint input with Zod schemas before using it.
- Always sanitize user-supplied HTML with `sanitize-html` before rendering or emailing.
- Flag unusual package names before installing. Use exact or pinned versions.
- Never bypass `--no-verify` unless explicitly requested.

## AI Agent Guidelines

- **Ask before applying**: describe changes, list affected files, wait for approval.
- **Approval phrases**: "Yes", "Proceed", "Apply", "Do it", "Looks good"
- **Never create** summary or review markdown files automatically.
- After changes: run `pnpm build`, report in 2–3 sentences, generate a commit message.

## Commit Standards

Format: `type(scope): subject`

- Subject: imperative, lowercase, no trailing period, ≤ 72 chars
- Types: `feat`, `fix`, `improvement`, `refactor`, `docs`, `style`, `chore`, `ci`, `perf`, `revert`
- Scope: affected area (e.g., `contact`, `nav`, `email`, `middleware`, `layout`, `deps`)
- For `fix`: symptom + trigger, not the code change
  - ✅ `fix(email): contact form submissions lost when Resend API key is unset`
  - ❌ `fix(email): add env check before send`

Examples:
- `feat(contact): add file attachment support to contact form`
- `fix(middleware): auth redirect loops on root path after SSR hydration`
- `chore(deps): update astro to v7.2.0`

## Development Commands

```bash
pnpm dev              # Start SSR dev server
pnpm build            # astro check + build (dist/server/)
pnpm preview          # Preview built output
pnpm start            # Run production server (node dist/server/entry.mjs)
pnpm lint             # ESLint
pnpm formatter:check  # OxFmt check
pnpm formatter:write  # OxFmt write
```
