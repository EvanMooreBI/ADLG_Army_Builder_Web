# ADLG Army Builder Web

Web application for building legal army lists for L'Art de la Guerre (ADLG) v4, an ancients/medieval miniature wargaming ruleset. Users register, create army lists, validate them in real time, and export as one-page PDFs.

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
- Reference data (armies, troop groups, options, point costs) is read-only, seeded via a separate pipeline
- User data (army lists, commands, selections) is scoped by RLS to `auth.uid()`
- Supabase client SDK used directly from components and server actions — no custom REST API
- Reference data cached in React context per session; user data fetched on navigation

### Validation Engine
- Client-side: Reactive validation via `useReducer`/`useMemo` — single source of truth for all UI constraints
- Server-side: Supabase Edge Function `validate-army-list` for defense-in-depth before PDF export
- All validation rules are data-driven from the database, not hardcoded

### Key Database Tables
- `armies`, `army_command_rules`, `general_costs`, `points_tier_rules` — reference data
- `troop_groups`, `troop_options` — army composition catalog
- `condition_tags`, `option_conditions` — conditional visibility logic
- `exclusion_groups`, `exclusion_group_members` — mutual exclusion constraints
- `army_lists`, `commands`, `list_selections` — user-owned data (RLS-protected)

## Development Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server
npm run build        # Production build
npm run lint         # Run linter
npm run test         # Run tests
```

## Environment Variables

Managed via `vercel env pull` for local dev. Required:

- `NEXT_PUBLIC_SUPABASE_URL` — Supabase project URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` — Supabase anonymous/public key
- `SUPABASE_SERVICE_ROLE_KEY` — For Edge Functions only (never exposed to client)

Use `.env.local` for local development. Never hardcode secrets.

## Coding Conventions

- TypeScript throughout, strict mode
- Next.js App Router: `app/` directory, server components by default, `'use client'` only when needed
- Tailwind CSS for all styling — no CSS modules or styled-components
- Supabase client: server client for server components/actions, browser client for client components
- Validation logic centralized in the reactive engine — no validation in individual components
- Database types generated from Supabase schema

## Key Design Decisions

- Points limits fixed at 100 / 200 / 300 (no custom values in MVP)
- Alliance contingents out of scope for MVP
- Only ADLG v4 data supported (schema is edition-aware for future versions)
- PDF export requires passing server-side validation first
- Mixed units (e.g. ½ spearmen / ½ bowmen) count as one unit toward group totals

## Companion Repository

The project spec, reference data, and seeder tool live in `../Obsidian.HobbyBrain/ADLG_ArmyBuilder_Web/`. Always consult `spec/SPEC.md` there for authoritative requirements.
