# AI Reliability Suite — Claude Context

This file provides essential context for working with the AI Reliability Suite codebase. Keep it concise and actionable.

## Project Overview

Full-stack monorepo for evaluating, stress-testing, and improving LLM and agentic AI system reliability.

**Apps:**
- `apps/eval-forge/` — Structured LLM evaluation framework (rubric-based scoring, schema validation)
- `apps/synth-bench/` — Synthetic data and edge-case generator
- `apps/agent-reliability-lab/` — Agent reliability sandbox with tool-calling and trace visualization

**Tech Stack:** Next.js (App Router), Vercel AI SDK, TypeScript, Zod, Supabase, Prisma, pnpm workspaces

## Bash Commands

- `pnpm install` — Install all dependencies (run at root)
- `pnpm --filter eval-forge dev` — Run specific app in dev mode
- `pnpm --filter <app-name> <command>` — Run command in specific app/package
- `pnpm --filter db prisma migrate dev` — Run database migrations
- `pnpm --filter db prisma generate` — Generate Prisma client
- `pnpm --filter db prisma studio` — Open Prisma Studio

## Code Style

- **React Components**: Filename `camelCase.tsx` (e.g., `userProfile.tsx`), folder `kebab-case` (e.g., `user-profile/`)
- **Utilities**: `kebab-case.ts` (e.g., `format-date.ts`, `validate-schema.ts`)
- **Constants**: `UPPER_SNAKE_CASE` for exported constants
- **Imports**: Use ES modules (`import/export`), destructure when possible
- **TypeScript**: Strict mode, always type function parameters and return types
- **Comments**: Avoid unless essential for complex business logic or non-obvious decisions

## Core Files & Utilities

**Documentation (per app):**
- `README.md` — Overview and features
- `SPEC.md` — Technical specification (data models, API endpoints, components)
- `PLAN.md` — Implementation plan with phases
- `QUICKSTART.md` — Setup guide

**Configuration:**
- `.cursor/rules/` — Workspace coding rules (always applied)
- `packages/db/prisma/schema.prisma` — Database schema
- Root `.env` — Shared environment variables
- App `.env.local` — App-specific variables

**Key Directories:**
- `apps/*/app/` — Next.js App Router routes
- `apps/*/components/` — React components
- `apps/*/lib/` — Core logic and utilities
- `packages/` — Shared packages (core-eval, db, ui, etc.)

## Workflow

1. **Before making changes**: Review `SPEC.md` and `PLAN.md` for the relevant app
2. **Type checking**: Run typecheck after making code changes
3. **Database changes**: Update Prisma schema, create migration, generate client
4. **API routes**: Use Edge runtime for LLM calls (`export const runtime = 'edge'`)
5. **Testing**: Prefer running single tests for performance, not whole test suite
6. **Documentation**: Update SPEC.md, PLAN.md, or README.md when adding features

## Environment Variables

Required variables:
- `OPENAI_API_KEY` — OpenAI API key for LLM calls
- `SUPABASE_URL` — Supabase project URL
- `SUPABASE_ANON_KEY` — Supabase anonymous key
- `DATABASE_URL` — PostgreSQL connection string

## Database

- **ORM**: Prisma
- **Hosting**: Supabase (Postgres)
- **Schema location**: `packages/db/prisma/schema.prisma`
- **Migrations**: Use `prisma migrate dev` to create and apply migrations
- **Client generation**: Run `prisma generate` after schema changes

## API Patterns

- **Edge functions**: Use `export const runtime = 'edge'` for LLM routes
- **Validation**: Always validate inputs with Zod schemas
- **Error handling**: Return proper HTTP status codes, never silently fail
- **Database**: Use Supabase client from `lib/supabase/` (server.ts for server, client.ts for client)

## EvalForge Core Concepts

- **EvaluationTask**: Prompt + rubric criteria + optional schema + model
- **EvaluationRun**: Single execution with input, output, scores, validation results
- **EvaluationReport**: Aggregated reports (Markdown/JSON)
- **Key components**: `RubricScorer`, `SchemaValidator`, `ReportGenerator`, `EvaluationExecutor`
- **API routes**: `/api/tasks`, `/api/runs`, `/api/evaluate`, `/api/reports`

## Important Rules

- **IMPORTANT**: Only add code when 95% confident in correctness
- **IMPORTANT**: All Edge functions must use Edge-compatible dependencies
- **IMPORTANT**: Never commit API keys or sensitive data
- **IMPORTANT**: Always validate inputs and outputs with Zod
- **IMPORTANT**: Use workspace protocol (`workspace:*`) for internal package dependencies
- Optimize for Edge-first execution (low latency, minimal dependencies)
- Use server components by default, client components only when needed
- Implement proper loading states and error boundaries
- Follow DRY principle — extract common logic into utilities

## Common Tasks

**Adding a feature:**
1. Check SPEC.md and PLAN.md
2. Follow file naming conventions
3. Add TypeScript types
4. Update documentation

**Creating a component:**
1. Use `camelCase.tsx` filename
2. Place in appropriate folder (`components/task/`, `components/shared/`)
3. Define TypeScript prop types
4. Include error boundaries and loading states
5. Use Tailwind CSS + shadcn/ui

**Database operations:**
1. Update `packages/db/prisma/schema.prisma`
2. Create migration: `pnpm --filter db prisma migrate dev`
3. Generate client: `pnpm --filter db prisma generate`
4. Use Prisma client for type-safe queries

## Repository Etiquette

- Use conventional commits
- Keep commits focused and atomic
- Update documentation when adding features
- Test changes before committing

## Unexpected Behaviors

- Edge runtime has limited Node.js API support — check compatibility
- Prisma client must be regenerated after schema changes
- pnpm workspace dependencies require `workspace:*` protocol
- Next.js App Router uses server components by default

