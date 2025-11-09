<!-- b072d6f8-3cbd-4a3f-9091-84d3ac99435d b2f955bc-5634-4f67-af2c-9b7716815644 -->
# SynthBench Implementation Plan

## Overview

SynthBench is a synthetic data generation and fuzzing platform that creates challenging, diverse, and adversarial datasets for evaluating LLM robustness. It combines AI-powered generation (via Vercel AI SDK), DSL-based fuzzing engines, and semantic validation to produce edge-case prompts that can be directly exported to EvalForge for evaluation.

## Project Structure

```
apps/synth-bench/
├── app/
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── page.tsx                    # Main dashboard
│   │   ├── datasets/
│   │   │   ├── page.tsx                # Dataset list
│   │   │   ├── [id]/
│   │   │   │   ├── page.tsx            # Dataset detail
│   │   │   │   └── edit/
│   │   │   │       └── page.tsx        # Dataset editor
│   │   │   └── new/
│   │   │       └── page.tsx            # Create dataset
│   │   ├── generators/
│   │   │   ├── page.tsx                # Generator templates list
│   │   │   ├── [id]/
│   │   │   │   └── page.tsx            # Generator detail
│   │   │   └── new/
│   │   │       └── page.tsx            # Create generator
│   │   ├── jobs/
│   │   │   ├── page.tsx                # Generation jobs list
│   │   │   └── [id]/
│   │   │       └── page.tsx            # Job detail with progress
│   │   └── export/
│   │       ├── page.tsx                # Export interface
│   │       └── [datasetId]/
│   │           └── page.tsx            # Export to EvalForge
│   ├── api/
│   │   ├── datasets/
│   │   │   ├── route.ts                # CRUD for datasets
│   │   │   └── [id]/
│   │   │       └── route.ts            # Dataset operations
│   │   ├── generators/
│   │   │   ├── route.ts                # Generator CRUD
│   │   │   └── [id]/
│   │   │       └── route.ts            # Generator operations
│   │   ├── generate/
│   │   │   └── route.ts                # Edge function for data generation
│   │   ├── fuzz/
│   │   │   └── route.ts                # Edge function for DSL fuzzing
│   │   ├── validate/
│   │   │   └── route.ts                # Semantic validation endpoint
│   │   ├── jobs/
│   │   │   ├── route.ts                # Create/list jobs
│   │   │   └── [id]/
│   │   │       └── route.ts            # Job status/operations
│   │   └── export/
│   │       ├── route.ts                # Export dataset
│   │       └── [datasetId]/
│   │           └── route.ts            # Export to EvalForge
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── dataset/
│   │   ├── datasetForm.tsx             # Dataset creation/editing form
│   │   ├── datasetList.tsx             # Dataset list component
│   │   ├── datasetViewer.tsx         # Dataset preview with samples
│   │   └── exportDialog.tsx            # Export to EvalForge dialog
│   ├── generator/
│   │   ├── generatorForm.tsx           # Generator template form
│   │   ├── generatorList.tsx           # Generator list
│   │   ├── promptTemplateEditor.tsx    # Prompt template editor
│   │   └── fuzzerConfig.tsx            # DSL fuzzer configuration UI
│   ├── generation/
│   │   ├── jobCard.tsx                 # Job status card
│   │   ├── jobProgress.tsx            # Job progress indicator
│   │   ├── samplePreview.tsx           # Generated sample preview
│   │   └── validationStatus.tsx       # Validation results display
│   └── shared/
│       ├── dataTable.tsx               # Reusable data table
│       └── chart.tsx                   # Dataset statistics charts
├── lib/
│   ├── supabase/
│   │   ├── client.ts                   # Supabase client
│   │   └── server.ts                   # Server-side client
│   ├── types/
│   │   └── generation.ts               # TypeScript types
│   └── utils/
│       ├── deduplication.ts            # Semantic deduplication utilities
│       └── formatting.ts               # Export formatting
├── package.json
├── next.config.ts
├── tsconfig.json
└── tailwind.config.ts
```

## Core Features Implementation

### 1. Database Schema (packages/db)

Add to Prisma schema:

```prisma
model Dataset {
  id          String   @id @default(cuid())
  name        String
  description String?
  domain      String   // e.g., "math", "reasoning", "entity-extraction"
  samples     Sample[]
  generators  Generator[]
  jobs        GenerationJob[]
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([domain, createdAt])
}

model Sample {
  id          String   @id @default(cuid())
  datasetId   String
  dataset     Dataset  @relation(fields: [datasetId], references: [id], onDelete: Cascade)
  prompt      String
  metadata    Json?    // Additional context, tags, etc.
  validated   Boolean  @default(false)
  validationErrors Json? // Validation error details
  createdAt   DateTime @default(now())
  
  @@index([datasetId, validated])
}

model Generator {
  id          String   @id @default(cuid())
  name        String
  description String?
  type        String   // "ai", "fuzz", "hybrid"
  config      Json     // Generator-specific configuration
  template    String?  // Prompt template for AI generation
  fuzzerRules Json?    // DSL fuzzer rules (regex, JSON, SQL grammars)
  datasetId   String?
  dataset     Dataset? @relation(fields: [datasetId], references: [id])
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([datasetId])
}

model GenerationJob {
  id          String   @id @default(cuid())
  datasetId   String
  dataset     Dataset  @relation(fields: [datasetId], references: [id])
  generatorId String
  generator   Generator @relation(fields: [generatorId], references: [id])
  status      String   // pending, running, completed, failed
  targetCount Int      // Target number of samples
  generatedCount Int   @default(0)
  validatedCount Int   @default(0)
  metadata    Json?    // Job configuration, progress, errors
  createdAt   DateTime @default(now())
  completedAt DateTime?
  
  @@index([datasetId, status])
}
```

### 2. Core Data Package (packages/core-data)

Implement data generation and fuzzing logic:

- `src/fuzzer/regex.ts` - Regex-based fuzzing engine
- `src/fuzzer/json.ts` - JSON grammar fuzzer
- `src/fuzzer/sql.ts` - SQL-like grammar fuzzer
- `src/fuzzer/dsl.ts` - Mini-DSL parser and generator
- `src/generator/ai.ts` - AI-powered generation using Vercel AI SDK
- `src/generator/counterfactual.ts` - Counterfactual generation strategies
- `src/generator/edgecase.ts` - Edge-case generation strategies
- `src/validation/semantic.ts` - Semantic validation and deduplication
- `src/validation/deduplication.ts` - Similarity-based deduplication
- `src/export/evalforge.ts` - Export utilities for EvalForge integration

### 3. API Routes

**POST /api/datasets** - Create dataset

- Validate dataset configuration
- Store in Supabase

**GET /api/datasets** - List all datasets

- Pagination support
- Filtering by domain, date range

**POST /api/generators** - Create generator

- Validate generator type and configuration
- Validate fuzzer rules if type is "fuzz" or "hybrid"
- Store in Supabase

**POST /api/generate** - Edge function for AI-powered generation

- Use Vercel AI SDK to call LLM (OpenAI/HuggingFace)
- Generate samples based on template and config
- Return generated samples

**POST /api/fuzz** - Edge function for DSL fuzzing

- Use core-data fuzzer engines
- Generate samples based on fuzzer rules
- Return fuzzed samples

**POST /api/validate** - Semantic validation endpoint

- Validate samples for semantic correctness
- Perform deduplication
- Return validation results

**POST /api/jobs** - Create generation job

- Accept datasetId, generatorId, targetCount
- Queue or run generation immediately
- Track progress

**GET /api/jobs/[id]** - Get job status

- Return current progress, generated count, validation status

**POST /api/export/[datasetId]** - Export dataset to EvalForge

- Format dataset samples for EvalForge
- Create evaluation task in EvalForge (if integrated)
- Return export metadata

### 4. UI Components

**Dataset Management**

- Dataset form with domain selection, description
- Dataset list with search and filters
- Dataset detail view showing samples, statistics, export options

**Generator Configuration**

- Generator form with type selector (AI/Fuzz/Hybrid)
- Prompt template editor for AI generators
- Fuzzer rules editor for DSL-based generators
- Generator list with preview

**Generation Jobs**

- Job creation interface with generator selection
- Job list with status indicators
- Job detail view with progress, sample preview, validation status

**Export Interface**

- Export dialog with format selection
- EvalForge integration UI
- Export history and status

### 5. Integration Points

- Use `@ai-reliability-suite/core-data` for fuzzing and generation logic
- Use `@ai-reliability-suite/db` for database access
- Use `@ai-reliability-suite/ui` for shared components
- Vercel AI SDK for LLM calls in Edge functions
- Supabase for data persistence
- Integration with EvalForge via export API

## Implementation Steps

1. Initialize Next.js app with TypeScript and App Router
2. Set up Tailwind CSS and shadcn/ui components
3. Create database schema in packages/db
4. Implement core-data package with fuzzing engines and AI generation
5. Build API routes for datasets, generators, and jobs
6. Create Edge functions for generation and fuzzing using Vercel AI SDK
7. Implement semantic validation and deduplication logic
8. Build UI components for dataset management
9. Build UI components for generator configuration
10. Implement generation job tracking and progress UI
11. Add export functionality for EvalForge integration
12. Add error handling and loading states
13. Add data visualization for dataset statistics
14. Test end-to-end generation workflow

## Key Dependencies

- next, react, react-dom
- @vercel/ai (Vercel AI SDK)
- @supabase/supabase-js
- zod (validation)
- openai (OpenAI API client)
- @huggingface/inference (optional, for HuggingFace models)
- @ai-reliability-suite/core-data (workspace)
- @ai-reliability-suite/db (workspace)
- @ai-reliability-suite/ui (workspace)
- recharts (charts for statistics)
- react-syntax-highlighter (for template/fuzzer rule display)

### To-dos

- [ ] Initialize Next.js app structure with App Router, TypeScript, and Tailwind CSS configuration
- [ ] Set up Supabase client configuration and database schema for Dataset, Sample, Generator, and GenerationJob models
- [ ] Implement core-data package with fuzzing engines (regex, JSON, SQL), AI generation utilities, and semantic validation/deduplication
- [ ] Create API routes for dataset CRUD operations (POST /api/datasets, GET /api/datasets, GET /api/datasets/[id])
- [ ] Create API routes for generator CRUD operations (POST /api/generators, GET /api/generators, GET /api/generators/[id])
- [ ] Build Edge function for AI-powered generation (POST /api/generate) using Vercel AI SDK to call LLM
- [ ] Build Edge function for DSL fuzzing (POST /api/fuzz) using core-data fuzzer engines
- [ ] Create API routes for generation jobs (POST /api/jobs, GET /api/jobs, GET /api/jobs/[id]) with progress tracking
- [ ] Create semantic validation endpoint (POST /api/validate) with deduplication logic
- [ ] Build dataset management UI components (datasetForm, datasetList, datasetViewer, exportDialog)
- [ ] Build generator configuration UI components (generatorForm, generatorList, promptTemplateEditor, fuzzerConfig)
- [ ] Create generation job UI components (jobCard, jobProgress, samplePreview, validationStatus)
- [ ] Implement export functionality for EvalForge integration (POST /api/export/[datasetId] and export UI)