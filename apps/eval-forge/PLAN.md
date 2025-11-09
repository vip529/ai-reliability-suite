# 🧠 EvalForge — Implementation Plan

## 📋 Overview

This document outlines the comprehensive implementation plan for the EvalForge module, a structured LLM evaluation framework that enables developers to define evaluation tasks with prompts, rubrics, and expected schemas, then automatically compute weighted scores and generate reproducible reports.

**Core Objectives:**
- Build a structured evaluation system with rubric-based scoring
- Implement JSON Schema and Zod-based output validation
- Create evaluation run management and tracking
- Develop report generation with Markdown and JSON formats
- Integrate with Supabase for data persistence
- Provide a modern dashboard UI for task and run management

---

## 🏗️ Architecture Overview

```
eval-forge/
├── app/                          # Next.js App Router
│   ├── (dashboard)/             # Main dashboard routes
│   │   ├── page.tsx             # Dashboard home
│   │   ├── tasks/               # Task management
│   │   │   ├── page.tsx         # Task list
│   │   │   ├── [id]/
│   │   │   │   ├── page.tsx     # Task detail
│   │   │   │   └── edit/
│   │   │   │       └── page.tsx # Task editor
│   │   │   └── new/
│   │   │       └── page.tsx     # Create task
│   │   ├── runs/                # Evaluation runs
│   │   │   ├── page.tsx         # Run list
│   │   │   └── [id]/
│   │   │       └── page.tsx     # Run detail with results
│   │   └── reports/              # Generated reports
│   │       ├── page.tsx         # Report list
│   │       └── [id]/
│   │           └── page.tsx     # Report viewer
│   ├── api/                     # API routes
│   │   ├── tasks/               # Task CRUD endpoints
│   │   ├── runs/                # Run management endpoints
│   │   ├── evaluate/            # Edge function for evaluation
│   │   └── reports/             # Report generation endpoints
│   ├── layout.tsx               # Root layout
│   └── globals.css              # Global styles
├── components/                   # React components
│   ├── task/                    # Task management components
│   ├── evaluation/              # Evaluation display components
│   ├── reports/                 # Report components
│   └── shared/                  # Shared UI components
├── lib/                         # Core logic
│   ├── supabase/                # Supabase client utilities
│   ├── types/                   # TypeScript types
│   └── utils/                   # Utility functions
└── hooks/                       # React hooks
```

---

## 📦 Phase 1: Project Foundation & Setup

### 1.1 Monorepo Integration
- [ ] Create `package.json` with Next.js, Vercel AI SDK, React dependencies
- [ ] Set up TypeScript configuration extending root config
- [ ] Configure Next.js with App Router
- [ ] Set up Tailwind CSS + shadcn/ui
- [ ] Create workspace dependencies to `packages/core-eval`, `packages/db`, `packages/ui`
- [ ] Configure Next.js config for Edge runtime support

### 1.2 Database Schema (Supabase/Prisma)
- [ ] Define `EvaluationTask` model (id, name, description, prompt, rubric, schema, model, timestamps)
- [ ] Define `EvaluationRun` model (id, taskId, status, input, output, scores, schemaValid, schemaErrors, metadata, timestamps)
- [ ] Define `EvaluationReport` model (id, taskId, format, content, runIds, createdAt)
- [ ] Create Prisma schema and migrations
- [ ] Set up Supabase client utilities (client.ts, server.ts)
- [ ] Add database indexes for performance

### 1.3 Environment & Configuration
- [ ] Create `.env.example` with required variables
- [ ] Set up environment variable types
- [ ] Configure Vercel Edge runtime settings
- [ ] Set up API route structure
- [ ] Configure Tailwind CSS theme

---

## 🎯 Phase 2: Core Evaluation Package Integration

### 2.1 Core-Eval Package Setup
- [ ] Verify `packages/core-eval` structure exists
- [ ] Implement rubric scoring logic (`src/rubric/scorer.ts`)
- [ ] Create rubric types (`src/rubric/types.ts`)
- [ ] Implement schema validation (`src/schema/validator.ts`)
- [ ] Build report generation utilities (`src/report/generator.ts`)
- [ ] Create report formatting utilities (`src/report/formatter.ts`)
- [ ] Export all utilities from package index

### 2.2 Rubric System
- [ ] Define rubric criteria structure (name, description, weight, scoring function)
- [ ] Implement weighted score calculation
- [ ] Create rubric validation utilities
- [ ] Build rubric scoring engine
- [ ] Add support for custom scoring functions

### 2.3 Schema Validation
- [ ] Integrate JSON Schema validation (using ajv)
- [ ] Integrate Zod schema validation
- [ ] Create unified validation interface
- [ ] Implement validation error reporting
- [ ] Add schema compliance scoring

---

## 🔌 Phase 3: API Routes & Edge Functions

### 3.1 Task Management API
- [ ] Create `POST /api/tasks` - Create evaluation task
  - [ ] Validate rubric structure
  - [ ] Validate JSON Schema if provided
  - [ ] Store in Supabase
  - [ ] Return created task
- [ ] Create `GET /api/tasks` - List all tasks
  - [ ] Add pagination support
  - [ ] Add filtering and sorting
  - [ ] Return task list with metadata
- [ ] Create `GET /api/tasks/[id]` - Get task by ID
  - [ ] Return task with all details
  - [ ] Include related runs count
- [ ] Create `PUT /api/tasks/[id]` - Update task
  - [ ] Validate input
  - [ ] Update in Supabase
- [ ] Create `DELETE /api/tasks/[id]` - Delete task
  - [ ] Handle cascade deletion of runs

### 3.2 Evaluation Run API
- [ ] Create `POST /api/runs` - Create evaluation run
  - [ ] Accept taskId and input
  - [ ] Validate task exists
  - [ ] Create run record with pending status
  - [ ] Queue evaluation (or trigger immediately)
  - [ ] Return run ID
- [ ] Create `GET /api/runs` - List runs
  - [ ] Add pagination
  - [ ] Add filtering by task, status
  - [ ] Add sorting options
- [ ] Create `GET /api/runs/[id]` - Get run details
  - [ ] Return full run data with scores
  - [ ] Include schema validation results

### 3.3 Evaluation Edge Function
- [ ] Create `POST /api/evaluate` - Edge function for evaluation
  - [ ] Configure Edge runtime
  - [ ] Accept runId or taskId + input
  - [ ] Use Vercel AI SDK to call LLM
  - [ ] Compute rubric scores using core-eval
  - [ ] Validate schema using core-eval
  - [ ] Store results in Supabase
  - [ ] Return structured response
  - [ ] Handle errors gracefully
  - [ ] Add timeout handling

### 3.4 Report Generation API
- [ ] Create `POST /api/reports` - Generate report
  - [ ] Accept taskId and runIds array
  - [ ] Accept format (markdown/json)
  - [ ] Aggregate run results
  - [ ] Generate Markdown or JSON report using core-eval
  - [ ] Store in Supabase
  - [ ] Return report ID
- [ ] Create `GET /api/reports` - List reports
  - [ ] Add pagination
  - [ ] Filter by task
- [ ] Create `GET /api/reports/[id]` - Get report
  - [ ] Return report content
  - [ ] Support different formats

---

## 🎨 Phase 4: UI Components — Task Management

### 4.1 Task Form Component
- [ ] Create `taskForm.tsx` component
  - [ ] Name and description fields
  - [ ] Prompt editor (textarea with syntax highlighting)
  - [ ] Model selector dropdown
  - [ ] Rubric editor integration
  - [ ] Schema editor integration
  - [ ] Form validation
  - [ ] Submit handler
  - [ ] Loading states
  - [ ] Error handling

### 4.2 Rubric Editor Component
- [ ] Create `rubricEditor.tsx` component
  - [ ] Add/remove rubric criteria
  - [ ] Edit criterion name, description, weight
  - [ ] Weight validation (must sum to 1.0 or 100%)
  - [ ] Visual weight distribution display
  - [ ] Drag-and-drop reordering (optional)
  - [ ] Preview rubric structure

### 4.3 Schema Editor Component
- [ ] Create `schemaEditor.tsx` component
  - [ ] JSON Schema editor (monaco editor or textarea)
  - [ ] Schema validation feedback
  - [ ] Syntax highlighting
  - [ ] Format/beautify JSON
  - [ ] Schema preview
  - [ ] Support for Zod schema input (optional)

### 4.4 Task List Component
- [ ] Create `taskList.tsx` component
  - [ ] Display tasks in table or card layout
  - [ ] Search functionality
  - [ ] Filter by model, date
  - [ ] Sort by name, date, run count
  - [ ] Pagination
  - [ ] Link to task detail
  - [ ] Quick actions (edit, delete, run)

### 4.5 Task Detail Page
- [ ] Create task detail view
  - [ ] Display full task information
  - [ ] Show rubric breakdown
  - [ ] Display schema if present
  - [ ] List all runs for this task
  - [ ] Show aggregate metrics
  - [ ] Quick run creation
  - [ ] Edit/delete actions

---

## 📊 Phase 5: UI Components — Evaluation Dashboard

### 5.1 Run Card Component
- [ ] Create `runCard.tsx` component
  - [ ] Display run summary
  - [ ] Status indicator (pending, running, completed, failed)
  - [ ] Overall score display
  - [ ] Schema validation status
  - [ ] Timestamp information
  - [ ] Link to detailed view

### 5.2 Result Viewer Component
- [ ] Create `resultViewer.tsx` component
  - [ ] Display input prompt
  - [ ] Display model output
  - [ ] Show score breakdown
  - [ ] Display schema validation results
  - [ ] Show metadata
  - [ ] Copy to clipboard functionality
  - [ ] Syntax highlighting for code outputs

### 5.3 Score Breakdown Component
- [ ] Create `scoreBreakdown.tsx` component
  - [ ] Display individual rubric scores
  - [ ] Show weighted contributions
  - [ ] Visual score bars
  - [ ] Overall weighted score
  - [ ] Score explanations (if available)
  - [ ] Color coding (green/yellow/red)

### 5.4 Schema Validation Component
- [ ] Create `schemaValidation.tsx` component
  - [ ] Display validation status (pass/fail)
  - [ ] Show validation errors
  - [ ] Highlight invalid fields
  - [ ] Display expected vs actual schema
  - [ ] Expandable error details

### 5.5 Run List Page
- [ ] Create run list view
  - [ ] Filter by task, status, date range
  - [ ] Sort options
  - [ ] Pagination
  - [ ] Bulk actions (optional)
  - [ ] Export functionality

---

## 📈 Phase 6: UI Components — Reports

### 6.1 Report Generator Component
- [ ] Create `reportGenerator.tsx` component
  - [ ] Task selector
  - [ ] Run selection (multi-select or date range)
  - [ ] Format selector (Markdown/JSON)
  - [ ] Generate button
  - [ ] Loading state
  - [ ] Preview before saving

### 6.2 Report Viewer Component
- [ ] Create `reportViewer.tsx` component
  - [ ] Markdown rendering (react-markdown)
  - [ ] JSON syntax highlighting
  - [ ] Copy to clipboard
  - [ ] Export as file
  - [ ] Print-friendly styling
  - [ ] Share functionality (optional)

### 6.3 Report List Page
- [ ] Create report list view
  - [ ] Display all reports
  - [ ] Filter by task, format, date
  - [ ] Link to report viewer
  - [ ] Delete action

---

## 🎯 Phase 7: Dashboard & Analytics

### 7.1 Dashboard Layout
- [ ] Create main dashboard page
  - [ ] Overview statistics (total tasks, runs, reports)
  - [ ] Recent runs widget
  - [ ] Quick actions
  - [ ] Navigation sidebar
  - [ ] Header with user info (if auth added)
  - [ ] Responsive design

### 7.2 Data Visualization
- [ ] Create `chart.tsx` component using Recharts
  - [ ] Score distribution chart
  - [ ] Score trends over time
  - [ ] Rubric criteria comparison
  - [ ] Schema validation success rate
  - [ ] Model performance comparison
  - [ ] Export chart as image

### 7.3 Metrics & Analytics
- [ ] Task metrics display
  - [ ] Average scores per task
  - [ ] Run count per task
  - [ ] Success rate
  - [ ] Schema compliance rate
- [ ] Run analytics
  - [ ] Score distribution
  - [ ] Latency metrics (if tracked)
  - [ ] Error rate
- [ ] Aggregate views
  - [ ] Cross-task comparisons
  - [ ] Model performance analysis

---

## 🛠️ Phase 8: Shared Components & Utilities

### 8.1 Data Table Component
- [ ] Create `dataTable.tsx` reusable component
  - [ ] Sortable columns
  - [ ] Filterable columns
  - [ ] Pagination
  - [ ] Row selection
  - [ ] Custom cell rendering
  - [ ] Responsive design

### 8.2 Loading & Error States
- [ ] Create loading skeleton components
- [ ] Create error boundary component
- [ ] Create toast notification system
- [ ] Create empty state components
- [ ] Create error message components

### 8.3 Form Utilities
- [ ] Create form validation utilities
- [ ] Create form field components
- [ ] Create form error display
- [ ] Integrate with react-hook-form (optional)

### 8.4 Type Definitions
- [ ] Create `evaluation.ts` types file
  - [ ] EvaluationTask type
  - [ ] EvaluationRun type
  - [ ] EvaluationReport type
  - [ ] Rubric types
  - [ ] API response types
  - [ ] Component prop types

---

## 🧪 Phase 9: Testing & Quality

### 9.1 Unit Tests
- [ ] Test rubric scoring logic
- [ ] Test schema validation
- [ ] Test report generation
- [ ] Test API route handlers
- [ ] Test utility functions

### 9.2 Integration Tests
- [ ] Test task CRUD operations
- [ ] Test evaluation run workflow
- [ ] Test report generation workflow
- [ ] Test database operations
- [ ] Test Edge function execution

### 9.3 E2E Tests (Optional)
- [ ] Test task creation workflow
- [ ] Test evaluation run creation and execution
- [ ] Test report generation and viewing
- [ ] Test dashboard interactions

### 9.4 Performance Testing
- [ ] Test evaluation latency
- [ ] Test dashboard load times
- [ ] Test report generation performance
- [ ] Optimize database queries
- [ ] Add query result caching where appropriate

---

## 📚 Phase 10: Documentation & Polish

### 10.1 Code Documentation
- [ ] Add JSDoc comments to core functions
- [ ] Document API endpoints
- [ ] Create architecture diagrams
- [ ] Document component props
- [ ] Document type definitions

### 10.2 User Documentation
- [ ] Update README with setup instructions
- [ ] Create user guide for dashboard
- [ ] Document task creation process
- [ ] Document rubric definition
- [ ] Document schema validation
- [ ] Add example tasks and rubrics
- [ ] Create troubleshooting guide

### 10.3 Developer Documentation
- [ ] Document API contract
- [ ] Create contribution guidelines
- [ ] Document extension points
- [ ] Add code examples
- [ ] Document environment variables

---

## 🚀 Phase 11: Deployment & Optimization

### 11.1 Vercel Configuration
- [ ] Configure Vercel project settings
- [ ] Set up Edge function regions
- [ ] Configure environment variables
- [ ] Set up build and deploy scripts
- [ ] Configure preview deployments

### 11.2 Supabase Setup
- [ ] Configure Supabase project
- [ ] Run database migrations
- [ ] Configure Row Level Security (RLS) if needed
- [ ] Set up database backups
- [ ] Configure connection pooling

### 11.3 Performance Optimization
- [ ] Optimize database queries (add indexes)
- [ ] Implement response caching
- [ ] Optimize bundle size
- [ ] Add lazy loading for components
- [ ] Optimize images and assets

### 11.4 Monitoring & Observability
- [ ] Set up error tracking (Sentry or similar)
- [ ] Add performance monitoring
- [ ] Configure logging
- [ ] Set up alerts for critical errors
- [ ] Add analytics (optional)

---

## 📋 Implementation Priority

### MVP (Minimum Viable Product)
**Phases 1-3, 4.1-4.3, 5.1-5.2, 8.1-8.2**: Core evaluation engine, basic task management, run execution, simple result display

### Enhanced Version
**Phases 4.4-4.5, 5.3-5.5, 6, 7**: Full UI components, report generation, analytics, comprehensive dashboard

### Production Ready
**Phases 9-11**: Testing, documentation, optimization, deployment

---

## 🔗 Dependencies & Integrations

### Core Packages (workspace)
- `@ai-reliability-suite/core-eval`: Rubric scoring, schema validation, report generation
- `@ai-reliability-suite/db`: Prisma schema, database client
- `@ai-reliability-suite/ui`: Shared UI components

### External Dependencies
- `next`: ^14.0.0
- `@vercel/ai`: Latest
- `react`, `react-dom`: ^18.0.0
- `@supabase/supabase-js`: Latest
- `zod`: Latest
- `ajv`: Latest (JSON Schema validation)
- `recharts`: Latest (charts)
- `react-markdown`: Latest (report rendering)
- `tailwindcss`: Latest
- `shadcn/ui`: Latest

---

## 🎯 Success Criteria

1. ✅ Users can create evaluation tasks with prompts, rubrics, and schemas
2. ✅ Evaluation runs execute successfully using Vercel AI SDK
3. ✅ Rubric scores are computed correctly with weighted calculations
4. ✅ Schema validation works for both JSON Schema and Zod
5. ✅ Reports are generated in Markdown and JSON formats
6. ✅ Dashboard displays tasks, runs, and results clearly
7. ✅ System handles errors gracefully
8. ✅ Performance is acceptable (<3s for evaluation runs)
9. ✅ Code is well-documented and maintainable
10. ✅ System is deployable to Vercel + Supabase

---

## 📝 Notes

- Start with MVP features and iterate
- Prioritize type safety and error handling
- Keep Edge runtime compatibility in mind
- Design for extensibility (custom rubrics, schemas)
- Focus on developer experience and reproducibility
- Ensure evaluation results are deterministic where possible
- Consider batch evaluation for multiple inputs
- Plan for future features: metamorphic testing, comparison views, API access

---

## 🔄 Future Enhancements (Post-MVP)

- [ ] Batch evaluation support (multiple inputs at once)
- [ ] Metamorphic testing integration
- [ ] Model comparison views
- [ ] API access for programmatic evaluation
- [ ] Evaluation templates and presets
- [ ] Collaborative features (sharing tasks, reports)
- [ ] Advanced analytics and insights
- [ ] Export/import evaluation tasks
- [ ] Integration with CI/CD pipelines
- [ ] Webhook support for evaluation completion

