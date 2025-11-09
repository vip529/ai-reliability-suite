<!-- 6114f088-c279-4ea5-b89e-2321e33f780e ecf3b9e2-a0ed-48f9-817e-f1c46d553c24 -->
# Main Repository Setup Plan

## Overview

Establish the foundational monorepo structure with pnpm workspaces, shared packages, development tooling, and configuration files to support three Next.js apps (EvalForge, SynthBench, AgentReliabilityLab).

## Monorepo Structure

```
ai-reliability-suite/
├── apps/
│   ├── eval-forge/          # Next.js app (to be set up)
│   ├── synth-bench/         # Next.js app (to be set up)
│   └── agent-reliability-lab/ # Next.js app (to be set up)
├── packages/
│   ├── core-eval/           # Rubric & schema scoring logic
│   ├── core-data/           # DSL fuzzers & validation utilities
│   ├── core-agent/          # Planner, executor, tool orchestration
│   ├── db/                  # Prisma + Supabase schema
│   └── ui/                  # Shared React & styling components
├── pnpm-workspace.yaml      # pnpm workspace configuration
├── package.json             # Root package.json with scripts
├── tsconfig.json            # Base TypeScript config
├── .gitignore               # Git ignore rules
├── .env.example             # Environment variables template
├── turbo.json               # Turborepo config (optional, for build orchestration)
└── README.md                # Already exists
```

## Implementation Steps

### 1. Root Package Configuration

- Create `package.json` with:
  - pnpm workspace configuration
  - Root-level scripts (`dev`, `build`, `lint`, `type-check`)
  - Shared dev dependencies (TypeScript, ESLint, Prettier, etc.)
  - Workspace protocol references to packages

### 2. pnpm Workspace Setup

- Create `pnpm-workspace.yaml` defining:
  - `apps/*` workspace pattern
  - `packages/*` workspace pattern

### 3. TypeScript Base Configuration

- Create root `tsconfig.json` with:
  - Base compiler options (strict mode, ES2022, etc.)
  - Path aliases for shared packages
  - References to workspace packages

### 4. Shared Packages Structure

Create skeleton packages with minimal setup:

**packages/core-eval/**

- `package.json` with workspace protocol
- `tsconfig.json` extending root config
- `src/index.ts` (barrel export)
- Basic rubric scoring utilities structure

**packages/core-data/**

- `package.json` with workspace protocol
- `tsconfig.json` extending root config
- `src/index.ts` (barrel export)
- DSL fuzzer utilities structure

**packages/core-agent/**

- `package.json` with workspace protocol
- `tsconfig.json` extending root config
- `src/index.ts` (barrel export)
- Agent orchestration utilities structure

**packages/db/**

- `package.json` with Prisma dependencies
- `prisma/schema.prisma` (initial schema)
- `tsconfig.json` extending root config
- `src/index.ts` (Prisma client export)

**packages/ui/**

- `package.json` with React, Tailwind, shadcn/ui dependencies
- `tsconfig.json` extending root config
- `tailwind.config.ts` (base config)
- `src/components/` directory structure
- `src/lib/utils.ts` (shadcn utilities)

### 5. Development Tooling

- `.gitignore` with Node.js, Next.js, IDE, and environment file patterns
- `.env.example` with:
  - `OPENAI_API_KEY`
  - `SUPABASE_URL`
  - `SUPABASE_ANON_KEY`
  - `DATABASE_URL`
  - `NEXT_PUBLIC_*` variables for each app
- ESLint configuration (optional, can be added per app)
- Prettier configuration (optional)

### 6. Vercel Configuration

- `vercel.json` at root for monorepo deployment detection
- Configure build settings for each app

### 7. Documentation

- Update root `README.md` with setup instructions (already exists, may need minor updates)

## Key Files to Create

1. `pnpm-workspace.yaml` - Workspace definition
2. `package.json` - Root package with scripts and dev deps
3. `tsconfig.json` - Base TypeScript configuration
4. `.gitignore` - Git ignore patterns
5. `.env.example` - Environment variables template
6. `vercel.json` - Vercel monorepo configuration
7. `packages/*/package.json` - Package manifests (5 packages)
8. `packages/*/tsconfig.json` - Package TypeScript configs
9. `packages/db/prisma/schema.prisma` - Initial Prisma schema
10. `packages/ui/tailwind.config.ts` - Tailwind base config

## Dependencies Strategy

- **Root level**: Dev tools (TypeScript, ESLint, Prettier, pnpm)
- **Packages**: Production dependencies specific to each package
- **Apps**: Next.js, Vercel AI SDK, and app-specific dependencies
- **Shared deps**: Use workspace protocol (`workspace:*`) for internal packages

## Next Steps

After main repo setup, each app will be initialized as a Next.js project with:

- App Router structure
- TypeScript configuration extending root
- Integration with shared packages
- Vercel AI SDK setup
- Supabase client configuration

### To-dos

- [ ] Create pnpm-workspace.yaml and root package.json with workspace configuration and scripts
- [ ] Create root tsconfig.json with base compiler options and path aliases
- [ ] Create skeleton structure for all 5 shared packages (core-eval, core-data, core-agent, db, ui) with package.json and tsconfig.json
- [ ] Set up packages/db with Prisma schema and initial database models
- [ ] Set up packages/ui with Tailwind config and shadcn/ui base structure
- [ ] Create .gitignore, .env.example, and vercel.json configuration files
- [ ] Verify monorepo structure and workspace dependencies resolve correctly