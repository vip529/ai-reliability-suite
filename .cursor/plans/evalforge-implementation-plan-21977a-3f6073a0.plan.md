<!-- 3f6073a0-98ad-4c24-b2b5-f76e4290dbae 23041924-14c6-47c5-b070-35160f56d983 -->
# EvalForge Implementation Plan

## Overview

EvalForge is a structured LLM evaluation framework that enables developers to define evaluation tasks with prompts, rubrics, and expected schemas, then automatically compute weighted scores and generate reproducible reports.

## Project Structure

```
apps/eval-forge/
├── app/
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── page.tsx                    # Main dashboard
│   │   ├── tasks/
│   │   │   ├── page.tsx                # Task list
│   │   │   ├── [id]/
│   │   │   │   ├── page.tsx            # Task detail
│   │   │   │   └── edit/
│   │   │   │       └── page.tsx        # Task editor
│   │   │   └── new/
│   │   │       └── page.tsx            # Create task
│   │   ├── runs/
│   │   │   ├── page.tsx                # Evaluation runs list
│   │   │   └── [id]/
│   │   │       └── page.tsx            # Run detail with results
│   │   └── reports/
│   │       ├── page.tsx                # Generated reports
│   │       └── [id]/
│   │           └── page.tsx            # Report viewer
│   ├── api/
│   │   ├── tasks/
│   │   │   ├── route.ts                # CRUD for tasks
│   │   │   └── [id]/
│   │   │       └── route.ts            # Task operations
│   │   ├── runs/
│   │   │   ├── route.ts                # Create/list runs
│   │   │   └── [id]/
│   │   │       └── route.ts            # Run operations
│   │   ├── evaluate/
│   │   │   └── route.ts                # Edge function for evaluation
│   │   └── reports/
│   │       ├── route.ts                # Generate reports
│   │       └── [id]/
│   │           └── route.ts            # Report operations
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── task/
│   │   ├── taskForm.tsx                # Task creation/editing form
│   │   ├── taskList.tsx                # Task list component
│   │   ├── rubricEditor.tsx            # Rubric definition UI
│   │   └── schemaEditor.tsx            # JSON Schema editor
│   ├── evaluation/
│   │   ├── runCard.tsx                 # Run summary card
│   │   ├── resultViewer.tsx            # Detailed result display
│   │   ├── scoreBreakdown.tsx          # Rubric score visualization
│   │   └── schemaValidation.tsx        # Schema validation status
│   ├── reports/
│   │   ├── reportGenerator.tsx         # Report generation UI
│   │   └── reportViewer.tsx            # Markdown/JSON report display
│   └── shared/
│       ├── dataTable.tsx               # Reusable data table
│       └── chart.tsx                   # Score visualization charts
├── lib/
│   ├── supabase/
│   │   ├── client.ts                   # Supabase client
│   │   └── server.ts                   # Server-side client
│   ├── types/
│   │   └── evaluation.ts               # TypeScript types
│   └── utils/
│       ├── validation.ts               # Schema validation utilities
│       └── formatting.ts               # Report formatting
├── package.json
├── next.config.ts
├── tsconfig.json
└── tailwind.config.ts
```

## Core Features Implementation

### 1. Database Schema (packages/db)

Add to Prisma schema:

```prisma
model EvaluationTask {
  id          String   @id @default(cuid())
  name        String
  description String?
  prompt      String
  rubric      Json     // Array of rubric criteria with weights
  schema      Json?    // JSON Schema for structured output validation
  model       String   // Model identifier (gpt-4, claude-3, etc.)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  runs        EvaluationRun[]
  
  @@index([createdAt])
}

model EvaluationRun {
  id          String   @id @default(cuid())
  taskId      String
  task        EvaluationTask @relation(fields: [taskId], references: [id])
  status      String   // pending, running, completed, failed
  input       String   // Test input/prompt variant
  output      String?  // Model response
  scores      Json?    // Rubric scores breakdown
  schemaValid Boolean? // Schema validation result
  schemaErrors Json?   // Schema validation errors
  metadata    Json?    // Additional metadata
  createdAt   DateTime @default(now())
  completedAt DateTime?
  
  @@index([taskId, createdAt])
}

model EvaluationReport {
  id          String   @id @default(cuid())
  taskId      String
  task        EvaluationTask @relation(fields: [taskId], references: [id])
  format      String   // markdown, json
  content     String   // Report content
  runIds      String[] // Array of run IDs included
  createdAt   DateTime @default(now())
  
  @@index([taskId])
}
```

### 2. Core Evaluation Package (packages/core-eval)

Implement evaluation logic:

- `src/rubric/scorer.ts` - Compute weighted rubric scores
- `src/rubric/types.ts` - Rubric criteria types
- `src/schema/validator.ts` - JSON Schema + Zod validation
- `src/report/generator.ts` - Markdown/JSON report generation
- `src/report/formatter.ts` - Formatting utilities

### 3. API Routes

**POST /api/tasks** - Create evaluation task

- Validate rubric structure
- Validate JSON Schema if provided
- Store in Supabase

**GET /api/tasks** - List all tasks

- Pagination support
- Filtering and sorting

**POST /api/runs** - Create evaluation run

- Accept taskId and input
- Queue evaluation (or run immediately)

**POST /api/evaluate** - Edge function for evaluation

- Use Vercel AI SDK to call LLM
- Compute rubric scores using core-eval
- Validate schema using core-eval
- Store results in Supabase
- Return structured response

**POST /api/reports** - Generate report

- Aggregate run results
- Generate Markdown or JSON report
- Store in Supabase

### 4. UI Components

**Task Management**

- Task form with prompt editor, rubric builder, schema editor
- Task list with search and filters
- Task detail view showing all runs and metrics

**Evaluation Dashboard**

- Run list with status indicators
- Result viewer with score breakdown
- Schema validation status display
- Charts showing score distributions

**Report Generation**

- Report format selector (Markdown/JSON)
- Report viewer with syntax highlighting
- Export functionality

### 5. Integration Points

- Use `@ai-reliability-suite/core-eval` for scoring logic
- Use `@ai-reliability-suite/db` for database access
- Use `@ai-reliability-suite/ui` for shared components
- Vercel AI SDK for LLM calls in Edge functions
- Supabase for data persistence

## Implementation Steps

1. Initialize Next.js app with TypeScript and App Router
2. Set up Tailwind CSS and shadcn/ui components
3. Create database schema in packages/db
4. Implement core-eval package with rubric scoring and schema validation
5. Build API routes for tasks, runs, and evaluation
6. Create Edge function for evaluation using Vercel AI SDK
7. Build UI components for task management
8. Build UI components for evaluation dashboard
9. Implement report generation and viewing
10. Add error handling and loading states
11. Add data visualization for scores and metrics
12. Test end-to-end evaluation workflow

## Key Dependencies

- next, react, react-dom
- @vercel/ai (Vercel AI SDK)
- @supabase/supabase-js
- zod (schema validation)
- ajv (JSON Schema validation)
- recharts (charts)
- react-markdown (report rendering)
- @ai-reliability-suite/core-eval (workspace)
- @ai-reliability-suite/db (workspace)
- @ai-reliability-suite/ui (workspace)

### To-dos

- [ ] Initialize Next.js app structure with App Router, TypeScript, and Tailwind CSS configuration
- [ ] Set up Supabase client configuration and database schema for EvaluationTask, EvaluationRun, and EvaluationReport models
- [ ] Implement core-eval package with rubric scoring logic, schema validation, and report generation utilities
- [ ] Create API routes for task CRUD operations (POST /api/tasks, GET /api/tasks, GET /api/tasks/[id])
- [ ] Build Edge function for evaluation (POST /api/evaluate) using Vercel AI SDK to call LLM and compute scores
- [ ] Create API routes for evaluation runs (POST /api/runs, GET /api/runs, GET /api/runs/[id])
- [ ] Build task management UI components (taskForm, taskList, rubricEditor, schemaEditor)
- [ ] Create evaluation dashboard UI (runCard, resultViewer, scoreBreakdown, schemaValidation components)
- [ ] Implement report generation API and UI (reportGenerator, reportViewer components with Markdown/JSON support)
- [ ] Add data visualization components using Recharts for score distributions and metrics