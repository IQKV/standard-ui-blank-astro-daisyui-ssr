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

## Minimal Change Discipline

Solve the whole problem with the least new code. The best code is the code never written.

### Scope before you write

Read the task and the code it touches. List every place the change must reach: callers, tests, fixtures, config, exports. Check what it could break for users: data it would destroy or expose, callers that stop working. That is scope. Extra features are not.

### Pick the smallest complete change

Take the first option that fully works:

1. Does it need to exist? Drop features, options and flexibility nobody asked for — name them in one line so the user can decide.
2. Already in this codebase (helper, component, service, pattern)? Use it the way the surrounding code does.
3. Standard library or platform feature? Use it, unless the project has its own.
4. An installed dependency? Use it. Never add a dependency for a few lines.
5. Can it be one line a reader gets at a glance? One line.
6. Otherwise: the minimum code that works.

Be lazy about the solution, never about the change itself:

- Finish every part the task needs, including the callers, tests and fixtures the change breaks.
- No abstraction, wrapper, option, config, boilerplate or "for later" code nobody asked for. Keep values in the form the platform already gives you. Deletion beats addition. Keep the layers, interfaces and conventions the codebase already has.
- The shortest working diff wins, once you know everything it must touch. A one-liner that needs decoding is not short.
- Comment only the why the code cannot show, in one line.
- Bug fix: grep every caller of the function you touch, then fix the root cause once in the shared code.
- Code you move or merge keeps its error handling and validation.
- Between options of equal size, take the one that is correct on edge cases.
- New non-trivial logic (a branch, a loop, a parser, money or security, or a whole new script) leaves one small test or an assert-based self-check. Trivial changes need none.
- A shortcut with a known limit gets a one-line comment: `shortcut: <the limit>, <when to upgrade>`.
- Push back when the need does not justify a part: state the smaller option in one line instead of building the bigger one silently.

Never cut: validation at trust boundaries, error handling that prevents data loss, security, accessibility, the calibration real hardware needs, anything the user explicitly asked for.

### Token discipline

- Locate with Grep/Glob, then read only the ranges you need — not whole trees or whole files.
- Prefer editing existing files to creating new ones; reuse code that is already there before writing.
- Never paste file contents back in a reply. Cite the path and lines.
- Keep replies to the outcome: what changed, what passed, any risk (see AI Communication Standards).

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

## Evaluation

How we judge whether an agent task in this repository is actually done. Evaluation is evidence-based — a claim without the command and its observed result is not a pass.

> This section starts minimal and grows with the repository. Add a gate only when a change type repeatedly slips through without it.

### Definition of done

A task is evaluated as done only when all of the following hold:

- Every gate required by the change type below has run and passed.
- The final report names the exact command(s) run and the observed result (pass/fail) — per AI Agent Guidelines.
- No required gate was skipped silently. A gate that could not run is reported as a gap, never as a pass.

### Required gates by change type

| Change type                 | Required gates                                                                                   |
| --------------------------- | ------------------------------------------------------------------------------------------------ |
| Any change                  | `pnpm build` (runs `astro check`) and `pnpm lint`                                                |
| New page or API endpoint    | `pnpm build`; validate input with Zod and sanitize user HTML before sending                      |
| Env var added or renamed    | `pnpm build`; server-side vars go through `src/lib/env.ts`, client vars use the `PUBLIC_` prefix |
| Dependency or config change | `pnpm build`                                                                                     |
| Docs or `AGENTS.md` only    | `pnpm formatter:check`; no build gate — state that none applies                                  |

### Ordering

Run the narrowest relevant gate first, then the full set for the change type. On the first failure, stop and report it — do not continue on an unproven base (see Execution Discipline).

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
