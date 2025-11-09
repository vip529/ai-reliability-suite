# ⚙️ SynthBench — Implementation Plan

## 📋 Overview

This document outlines the comprehensive implementation plan for the SynthBench module, a synthetic data generation and fuzzing platform that creates challenging, diverse, and adversarial datasets for evaluating LLM robustness. It combines AI-powered generation (via Vercel AI SDK), DSL-based fuzzing engines, and semantic validation to produce edge-case prompts that can be directly exported to EvalForge for evaluation.

**Core Objectives:**
- Build AI-powered synthetic data generation system
- Implement DSL-based fuzzing engines (regex, JSON, SQL)
- Create semantic validation and deduplication
- Develop generation job tracking and progress monitoring
- Integrate with EvalForge for seamless dataset export
- Provide a modern dashboard UI for dataset and generator management

---

## 🏗️ Architecture Overview

```
synth-bench/
├── app/                          # Next.js App Router
│   ├── (dashboard)/             # Main dashboard routes
│   │   ├── page.tsx             # Dashboard home
│   │   ├── datasets/            # Dataset management
│   │   │   ├── page.tsx         # Dataset list
│   │   │   ├── [id]/
│   │   │   │   ├── page.tsx     # Dataset detail
│   │   │   │   └── edit/
│   │   │   │       └── page.tsx # Dataset editor
│   │   │   └── new/
│   │   │       └── page.tsx     # Create dataset
│   │   ├── generators/          # Generator templates
│   │   │   ├── page.tsx         # Generator list
│   │   │   ├── [id]/
│   │   │   │   └── page.tsx     # Generator detail
│   │   │   └── new/
│   │   │       └── page.tsx     # Create generator
│   │   ├── jobs/                # Generation jobs
│   │   │   ├── page.tsx         # Job list
│   │   │   └── [id]/
│   │   │       └── page.tsx     # Job detail with progress
│   │   └── export/              # Export interface
│   │       ├── page.tsx         # Export dashboard
│   │       └── [datasetId]/
│   │           └── page.tsx     # Export to EvalForge
│   ├── api/                     # API routes
│   │   ├── datasets/            # Dataset CRUD endpoints
│   │   ├── generators/          # Generator CRUD endpoints
│   │   ├── generate/            # AI generation Edge function
│   │   ├── fuzz/                # DSL fuzzing Edge function
│   │   ├── validate/            # Semantic validation endpoint
│   │   ├── jobs/                # Job management endpoints
│   │   └── export/              # Export endpoints
│   ├── layout.tsx               # Root layout
│   └── globals.css              # Global styles
├── components/                   # React components
│   ├── dataset/                 # Dataset management components
│   ├── generator/               # Generator configuration components
│   ├── generation/              # Generation job components
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
- [ ] Create workspace dependencies to `packages/core-data`, `packages/db`, `packages/ui`
- [ ] Configure Next.js config for Edge runtime support

### 1.2 Database Schema (Supabase/Prisma)
- [ ] Define `Dataset` model (id, name, description, domain, timestamps)
- [ ] Define `Sample` model (id, datasetId, prompt, metadata, validated, validationErrors)
- [ ] Define `Generator` model (id, name, description, type, config, template, fuzzerRules)
- [ ] Define `GenerationJob` model (id, datasetId, generatorId, status, targetCount, progress)
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

## 🎯 Phase 2: Core Data Package Integration

### 2.1 Core-Data Package Setup
- [ ] Verify `packages/core-data` structure exists
- [ ] Implement regex fuzzer (`src/fuzzer/regex.ts`)
- [ ] Implement JSON grammar fuzzer (`src/fuzzer/json.ts`)
- [ ] Implement SQL-like grammar fuzzer (`src/fuzzer/sql.ts`)
- [ ] Create DSL parser and generator (`src/fuzzer/dsl.ts`)
- [ ] Implement AI-powered generator (`src/generator/ai.ts`)
- [ ] Implement counterfactual generation (`src/generator/counterfactual.ts`)
- [ ] Implement edge-case generation (`src/generator/edgecase.ts`)
- [ ] Create semantic validation (`src/validation/semantic.ts`)
- [ ] Create deduplication utilities (`src/validation/deduplication.ts`)
- [ ] Create EvalForge export utilities (`src/export/evalforge.ts`)
- [ ] Export all utilities from package index

### 2.2 Fuzzing Engines
- [ ] Regex fuzzer with pattern generation
- [ ] JSON grammar fuzzer with valid/invalid variations
- [ ] SQL-like grammar fuzzer for query generation
- [ ] DSL parser for custom fuzzer rules
- [ ] Fuzzer rule validation
- [ ] Fuzzer configuration interface

### 2.3 AI Generation
- [ ] Template-based generation using Vercel AI SDK
- [ ] Counterfactual generation strategies
- [ ] Edge-case generation strategies
- [ ] Batch generation support
- [ ] Generation quality controls

### 2.4 Validation & Deduplication
- [ ] Semantic validation logic
- [ ] Similarity-based deduplication
- [ ] Embedding-based similarity calculation
- [ ] Validation error reporting
- [ ] Deduplication threshold configuration

---

## 🔌 Phase 3: API Routes & Edge Functions

### 3.1 Dataset Management API
- [ ] Create `POST /api/datasets` - Create dataset
  - [ ] Validate dataset configuration
  - [ ] Store in Supabase
  - [ ] Return created dataset
- [ ] Create `GET /api/datasets` - List all datasets
  - [ ] Add pagination support
  - [ ] Add filtering by domain, date range
  - [ ] Add sorting options
  - [ ] Return dataset list with metadata
- [ ] Create `GET /api/datasets/[id]` - Get dataset by ID
  - [ ] Return dataset with sample count
  - [ ] Include statistics
- [ ] Create `PUT /api/datasets/[id]` - Update dataset
  - [ ] Validate input
  - [ ] Update in Supabase
- [ ] Create `DELETE /api/datasets/[id]` - Delete dataset
  - [ ] Handle cascade deletion of samples

### 3.2 Generator Management API
- [ ] Create `POST /api/generators` - Create generator
  - [ ] Validate generator type and configuration
  - [ ] Validate fuzzer rules if type is "fuzz" or "hybrid"
  - [ ] Store in Supabase
  - [ ] Return created generator
- [ ] Create `GET /api/generators` - List generators
  - [ ] Add pagination
  - [ ] Filter by type, dataset
  - [ ] Add sorting options
- [ ] Create `GET /api/generators/[id]` - Get generator details
  - [ ] Return full generator configuration
- [ ] Create `PUT /api/generators/[id]` - Update generator
- [ ] Create `DELETE /api/generators/[id]` - Delete generator

### 3.3 Generation Edge Functions
- [ ] Create `POST /api/generate` - AI-powered generation
  - [ ] Configure Edge runtime
  - [ ] Accept generator config and template
  - [ ] Use Vercel AI SDK to call LLM
  - [ ] Generate samples based on template
  - [ ] Return generated samples
  - [ ] Handle errors gracefully
- [ ] Create `POST /api/fuzz` - DSL fuzzing
  - [ ] Configure Edge runtime
  - [ ] Accept fuzzer rules and config
  - [ ] Use core-data fuzzer engines
  - [ ] Generate fuzzed samples
  - [ ] Return fuzzed samples
  - [ ] Handle errors gracefully

### 3.4 Validation API
- [ ] Create `POST /api/validate` - Semantic validation
  - [ ] Accept samples array
  - [ ] Validate semantic correctness
  - [ ] Perform deduplication
  - [ ] Return validation results
  - [ ] Mark samples as validated/invalid

### 3.5 Job Management API
- [ ] Create `POST /api/jobs` - Create generation job
  - [ ] Accept datasetId, generatorId, targetCount
  - [ ] Validate inputs
  - [ ] Create job record
  - [ ] Queue or trigger generation
  - [ ] Return job ID
- [ ] Create `GET /api/jobs` - List jobs
  - [ ] Add pagination
  - [ ] Filter by dataset, status
  - [ ] Add sorting options
- [ ] Create `GET /api/jobs/[id]` - Get job status
  - [ ] Return current progress
  - [ ] Include generated count, validated count
  - [ ] Include sample previews
- [ ] Create `PUT /api/jobs/[id]` - Update job (cancel, pause)
- [ ] Create `DELETE /api/jobs/[id]` - Delete job

### 3.6 Export API
- [ ] Create `POST /api/export/[datasetId]` - Export dataset
  - [ ] Accept format and options
  - [ ] Format samples for export
  - [ ] Generate export file or data
  - [ ] Return export metadata
- [ ] Create `POST /api/export/[datasetId]/evalforge` - Export to EvalForge
  - [ ] Format dataset for EvalForge
  - [ ] Create evaluation task in EvalForge (if integrated)
  - [ ] Return export status
- [ ] Create `GET /api/export/[datasetId]` - Get export history

---

## 🎨 Phase 4: UI Components — Dataset Management

### 4.1 Dataset Form Component
- [ ] Create `datasetForm.tsx` component
  - [ ] Name and description fields
  - [ ] Domain selector dropdown
  - [ ] Form validation
  - [ ] Submit handler
  - [ ] Loading states
  - [ ] Error handling

### 4.2 Dataset List Component
- [ ] Create `datasetList.tsx` component
  - [ ] Display datasets in table or card layout
  - [ ] Search functionality
  - [ ] Filter by domain, date
  - [ ] Sort options
  - [ ] Pagination
  - [ ] Link to dataset detail
  - [ ] Quick actions (edit, delete, export)

### 4.3 Dataset Viewer Component
- [ ] Create `datasetViewer.tsx` component
  - [ ] Display dataset information
  - [ ] Show sample list with pagination
  - [ ] Sample preview
  - [ ] Statistics display
  - [ ] Export options
  - [ ] Validation status overview

### 4.4 Export Dialog Component
- [ ] Create `exportDialog.tsx` component
  - [ ] Format selector (JSON, CSV, EvalForge)
  - [ ] Export options configuration
  - [ ] EvalForge integration UI
  - [ ] Export progress indicator
  - [ ] Download/export button

---

## 🛠️ Phase 5: UI Components — Generator Configuration

### 5.1 Generator Form Component
- [ ] Create `generatorForm.tsx` component
  - [ ] Name and description fields
  - [ ] Type selector (AI/Fuzz/Hybrid)
  - [ ] Conditional fields based on type
  - [ ] Form validation
  - [ ] Submit handler
  - [ ] Loading states

### 5.2 Prompt Template Editor Component
- [ ] Create `promptTemplateEditor.tsx` component
  - [ ] Template textarea with syntax highlighting
  - [ ] Variable placeholder support
  - [ ] Template preview
  - [ ] Template validation
  - [ ] Save/load templates

### 5.3 Fuzzer Config Component
- [ ] Create `fuzzerConfig.tsx` component
  - [ ] Fuzzer type selector (regex, JSON, SQL)
  - [ ] Rule editor with syntax highlighting
  - [ ] Rule validation
  - [ ] Rule preview/test
  - [ ] Configuration options

### 5.4 Generator List Component
- [ ] Create `generatorList.tsx` component
  - [ ] Display generators in list
  - [ ] Filter by type, dataset
  - [ ] Search functionality
  - [ ] Link to generator detail
  - [ ] Quick actions (edit, delete, use)

---

## 📊 Phase 6: UI Components — Generation Jobs

### 6.1 Job Card Component
- [ ] Create `jobCard.tsx` component
  - [ ] Display job summary
  - [ ] Status indicator (pending, running, completed, failed)
  - [ ] Progress display
  - [ ] Generated/validated counts
  - [ ] Timestamp information
  - [ ] Link to detailed view

### 6.2 Job Progress Component
- [ ] Create `jobProgress.tsx` component
  - [ ] Progress bar
  - [ ] Percentage display
  - [ ] ETA calculation
  - [ ] Real-time updates
  - [ ] Cancel/pause controls

### 6.3 Sample Preview Component
- [ ] Create `samplePreview.tsx` component
  - [ ] Display generated samples
  - [ ] Sample validation status
  - [ ] Sample metadata
  - [ ] Pagination for large sets
  - [ ] Copy to clipboard

### 6.4 Validation Status Component
- [ ] Create `validationStatus.tsx` component
  - [ ] Validation statistics
  - [ ] Pass/fail breakdown
  - [ ] Validation error display
  - [ ] Deduplication stats
  - [ ] Re-validate button

### 6.5 Job Detail Page
- [ ] Create job detail view
  - [ ] Full job information
  - [ ] Progress tracking
  - [ ] Sample previews
  - [ ] Validation results
  - [ ] Job controls (cancel, retry)

---

## 📈 Phase 7: Dashboard & Analytics

### 7.1 Dashboard Layout
- [ ] Create main dashboard page
  - [ ] Overview statistics (total datasets, samples, jobs)
  - [ ] Recent jobs widget
  - [ ] Quick actions
  - [ ] Navigation sidebar
  - [ ] Header with user info (if auth added)
  - [ ] Responsive design

### 7.2 Data Visualization
- [ ] Create `chart.tsx` component using Recharts
  - [ ] Dataset size distribution
  - [ ] Generation job success rate
  - [ ] Sample validation rate over time
  - [ ] Domain distribution chart
  - [ ] Generator performance comparison
  - [ ] Export chart as image

### 7.3 Statistics & Analytics
- [ ] Dataset statistics display
  - [ ] Total samples per dataset
  - [ ] Validation rate
  - [ ] Domain breakdown
- [ ] Generation analytics
  - [ ] Job success rate
  - [ ] Average generation time
  - [ ] Samples per job
- [ ] Export analytics
  - [ ] Export history
  - [ ] Export success rate
  - [ ] EvalForge integration stats

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
- [ ] Create `generation.ts` types file
  - [ ] Dataset type
  - [ ] Sample type
  - [ ] Generator type
  - [ ] GenerationJob type
  - [ ] API response types
  - [ ] Component prop types

---

## 🧪 Phase 9: Testing & Quality

### 9.1 Unit Tests
- [ ] Test fuzzer engines (regex, JSON, SQL)
- [ ] Test AI generation logic
- [ ] Test validation and deduplication
- [ ] Test export utilities
- [ ] Test API route handlers
- [ ] Test utility functions

### 9.2 Integration Tests
- [ ] Test dataset CRUD operations
- [ ] Test generator CRUD operations
- [ ] Test generation job workflow
- [ ] Test validation workflow
- [ ] Test export workflow
- [ ] Test database operations

### 9.3 E2E Tests (Optional)
- [ ] Test dataset creation workflow
- [ ] Test generator creation and usage
- [ ] Test generation job execution
- [ ] Test export to EvalForge
- [ ] Test dashboard interactions

### 9.4 Performance Testing
- [ ] Test generation latency
- [ ] Test validation performance
- [ ] Test dashboard load times
- [ ] Test export performance
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
- [ ] Document fuzzer DSL syntax

### 10.2 User Documentation
- [ ] Update README with setup instructions
- [ ] Create user guide for dashboard
- [ ] Document dataset creation process
- [ ] Document generator configuration
- [ ] Document fuzzer DSL syntax
- [ ] Add example generators and datasets
- [ ] Create troubleshooting guide

### 10.3 Developer Documentation
- [ ] Document API contract
- [ ] Create contribution guidelines
- [ ] Document extension points
- [ ] Add code examples
- [ ] Document environment variables
- [ ] Document fuzzer engine architecture

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
- [ ] Optimize generation batch sizes
- [ ] Implement job queue for large generations

### 11.4 Monitoring & Observability
- [ ] Set up error tracking (Sentry or similar)
- [ ] Add performance monitoring
- [ ] Configure logging
- [ ] Set up alerts for critical errors
- [ ] Add analytics (optional)
- [ ] Monitor generation job success rates

---

## 📋 Implementation Priority

### MVP (Minimum Viable Product)
**Phases 1-3, 4.1-4.2, 5.1, 6.1-6.2, 8.1-8.2**: Core generation engine, basic dataset management, simple generator config, job tracking, basic UI

### Enhanced Version
**Phases 4.3-4.4, 5.2-5.4, 6.3-6.5, 7**: Full UI components, advanced generators, detailed job views, analytics, export functionality

### Production Ready
**Phases 9-11**: Testing, documentation, optimization, deployment

---

## 🔗 Dependencies & Integrations

### Core Packages (workspace)
- `@ai-reliability-suite/core-data`: Fuzzing engines, AI generation, validation
- `@ai-reliability-suite/db`: Prisma schema, database client
- `@ai-reliability-suite/ui`: Shared UI components

### External Dependencies
- `next`: ^14.0.0
- `@vercel/ai`: Latest
- `react`, `react-dom`: ^18.0.0
- `@supabase/supabase-js`: Latest
- `zod`: Latest
- `openai`: Latest
- `@huggingface/inference`: Latest (optional)
- `recharts`: Latest (charts)
- `react-syntax-highlighter`: Latest (syntax highlighting)
- `tailwindcss`: Latest
- `shadcn/ui`: Latest

---

## 🎯 Success Criteria

1. ✅ Users can create datasets and generators
2. ✅ AI-powered generation works with templates
3. ✅ DSL fuzzing engines generate valid samples
4. ✅ Semantic validation and deduplication work correctly
5. ✅ Generation jobs track progress accurately
6. ✅ Datasets can be exported to EvalForge
7. ✅ Dashboard displays datasets, generators, and jobs clearly
8. ✅ System handles errors gracefully
9. ✅ Performance is acceptable (<5s for small generation jobs)
10. ✅ Code is well-documented and maintainable
11. ✅ System is deployable to Vercel + Supabase

---

## 📝 Notes

- Start with MVP features and iterate
- Prioritize type safety and error handling
- Keep Edge runtime compatibility in mind
- Design for extensibility (custom fuzzers, generators)
- Focus on developer experience and dataset quality
- Ensure generated samples are diverse and challenging
- Consider batch processing for large generations
- Plan for future features: advanced fuzzing strategies, custom DSLs, real-time generation

---

## 🔄 Future Enhancements (Post-MVP)

- [ ] Advanced fuzzing strategies (mutation-based, grammar-based)
- [ ] Custom DSL for fuzzer rules
- [ ] Real-time generation streaming
- [ ] Collaborative dataset sharing
- [ ] Dataset versioning
- [ ] Advanced analytics and insights
- [ ] Integration with more LLM providers
- [ ] Template library and marketplace
- [ ] Automated quality scoring
- [ ] Dataset comparison and merging




