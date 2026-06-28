# ADLG Army Builder Web

Web application for building legal army lists for L'Art de la Guerre (ADLG) v4, an ancients/medieval miniature wargaming ruleset. Users register, create army lists, validate them in real time, and export as one-page PDFs.

## Mandatory Codex Workflow

- At the start of every conversation, invoke the Superpowers startup skill first. Prefer `superpowers:using-superpowers` when plugin-namespaced skills are available; otherwise use the global mirror at `C:\Users\crazy\.codex\skills\using-superpowers\SKILL.md`.
- State one short reason whenever selecting a skill, bundle, or major reference.
- Use `reference-routing` before loading or editing instruction files, skills, bundles, specs, runbooks, or large references.
- Use `master-workflow` before planning, editing, validating, or finishing any feature, fix, refactor, behavior change, non-trivial repo change, or non-trivial instruction/workflow documentation change. If skipped, state why the task does not qualify.
- Use the global skill root `C:\Users\crazy\.codex\skills` for cross-repo workflow skills. `setup-master-workflow` is installed there and should be used only to install, repair, or adapt the workflow for a repo or machine.
- For non-trivial implementation or workflow-instruction edits, follow `master-workflow` worktree isolation from local `main` or `master`, using `.worktrees/` for detached worktrees. Detached worktrees are isolation only, not durable storage.
- Run the applicable `master-workflow` gates before handoff: requirements reconciliation, code review/simplification for code changes, writing/docs review and drift checks for docs, visual/artifact validation when relevant, final verification, and finish-state reporting.
- Keep tracked `AGENTS.md` as repo defaults. Put machine-local or temporary preferences only in ignored local overlays such as `AGENTS.local.md`.
- Use the global `adlg-armybuilder-context` skill for any ADLG development, requirements work, specs, seeder work, Supabase schema work, validation rules, point calculations, army list parsing, or implementation planning.
- Use the global `doc-drift-detector` skill whenever changing documentation, changing code that documentation describes, preparing releases, validating README/API accuracy, or checking documentation links.
- Keep this file as the single root Codex instruction source. Do not recreate `AGENT.md`.

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js (App Router) |
| Auth | Supabase Auth (email/password) |
| Database | Supabase PostgreSQL with Row Level Security |
| Frontend | React + Tailwind CSS |
| PDF Generation | @react-pdf/renderer |
| Hosting | Vercel |

## Architecture Overview

### Data Flow

- Reference data (armies, troop groups, options, point costs) is read-only, seeded via a separate pipeline.
- User data (army lists, commands, selections) is scoped by RLS to `auth.uid()`.
- Supabase client SDK is used directly from components and server actions; there is no custom REST API.
- Reference data is cached in React context per session; user data is fetched on navigation.

### Validation Engine

- Client-side validation uses reactive `useReducer`/`useMemo` state as the single source of truth for UI constraints.
- Server-side validation uses the Supabase Edge Function `validate-army-list` for defense in depth before PDF export.
- All validation rules are data-driven from the database, not hardcoded in individual components.

### Key Database Tables

- `armies`, `army_command_rules`, `general_costs`, `points_tier_rules`: reference data.
- `troop_groups`, `troop_options`: army composition catalog.
- `condition_tags`, `option_conditions`: conditional visibility logic.
- `exclusion_groups`, `exclusion_group_members`: mutual exclusion constraints.
- `army_lists`, `commands`, `list_selections`: user-owned data protected by RLS.

## Development Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server
npm run build        # Production build
npm run lint         # Run linter
npm run test         # Run tests
```

## Environment Variables

Managed via `vercel env pull` for local development. Required:

- `NEXT_PUBLIC_SUPABASE_URL`: Supabase project URL.
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Supabase anonymous/public key.
- `SUPABASE_SERVICE_ROLE_KEY`: for Edge Functions only; never expose this to the client.

Use `.env.local` for local development. Never hardcode secrets.

## Coding Conventions

- TypeScript throughout, strict mode.
- Next.js App Router uses the `app/` directory, server components by default, and `'use client'` only when needed.
- Tailwind CSS for all styling; do not add CSS modules or styled-components.
- Use the Supabase server client for server components/actions and the browser client for client components.
- Keep validation logic centralized in the reactive engine; do not put validation in individual components.
- Database types are generated from the Supabase schema.

## Key Design Decisions

- Points limits are fixed at 100, 200, and 300; custom values are out of scope for MVP.
- Alliance contingents are out of scope for MVP.
- Only ADLG v4 data is supported; the schema is edition-aware for future versions.
- PDF export requires passing server-side validation first.
- Mixed units, such as 1/2 spearmen and 1/2 bowmen, count as one unit toward group totals.

## Companion Repository

The project spec, reference data, and seeder tool live in `../Obsidian.HobbyBrain/ADLG_ArmyBuilder_Web/`. Always consult `spec/SPEC.md` there for authoritative requirements before implementation decisions.
