<!-- 4b375999-0b3e-407e-b80a-12f413b72c49 d267bf1e-0311-4961-8b2f-65f94bcb8532 -->
# AgentReliabilityLab Implementation Plan

## Overview

AgentReliabilityLab is a sandbox for testing and analyzing the reliability of tool-using, multi-step AI agents. It provides planner-executor agent execution, schema-based output validation with self-repair, retry logic, interactive trace visualization, and reliability dashboards with metrics and analytics.

## Project Structure

```
apps/agent-reliability-lab/
├── app/
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── page.tsx                    # Main dashboard
│   │   ├── runs/
│   │   │   ├── page.tsx                # Run history list
│   │   │   └── [id]/
│   │   │       └── page.tsx            # Run detail with trace visualization
│   │   ├── analytics/
│   │   │   ├── page.tsx                # Metrics dashboard
│   │   │   └── reliability/
│   │   │       └── page.tsx            # Reliability score trends
│   │   └── config/
│   │       ├── page.tsx                # Agent configuration presets
│   │       └── [id]/
│   │           └── page.tsx            # Edit agent config
│   ├── api/
│   │   ├── agent/
│   │   │   └── run/
│   │   │       └── route.ts            # POST /api/agent/run (Edge function)
│   │   ├── tools/
│   │   │   └── execute/
│   │   │       └── route.ts            # POST /api/tools/execute
│   │   ├── traces/
│   │   │   ├── [runId]/
│   │   │   │   ├── route.ts            # GET /api/traces/:runId
│   │   │   │   └── graph/
│   │   │   │       └── route.ts        # GET /api/traces/:runId/graph (React Flow format)
│   │   │   └── route.ts                # GET /api/traces (list traces)
│   │   └── metrics/
│   │       ├── route.ts                # GET /api/metrics/runs
│   │       └── aggregate/
│   │           └── route.ts            # GET /api/metrics/aggregate
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── agent/
│   │   ├── agentConfigForm.tsx         # Agent configuration form
│   │   ├── agentExecutor.tsx           # Agent execution UI
│   │   ├── runStatus.tsx               # Real-time run status display
│   │   └── toolSelector.tsx             # Tool selection interface
│   ├── trace/
│   │   ├── traceVisualization.tsx      # React Flow trace graph
│   │   ├── traceNode.tsx               # Custom node component
│   │   ├── traceDetails.tsx            # Node details panel
│   │   └── traceTimeline.tsx           # Timeline view component
│   ├── dashboard/
│   │   ├── metricsDashboard.tsx        # Main metrics dashboard
│   │   ├── runList.tsx                 # Run history table
│   │   ├── reliabilityChart.tsx        # Reliability score visualization
│   │   ├── toolUsageChart.tsx          # Tool usage statistics
│   │   └── latencyChart.tsx            # Latency distribution
│   └── shared/
│       ├── dataTable.tsx               # Reusable data table
│       └── chart.tsx                  # Chart wrapper component
├── lib/
│   ├── agent/
│   │   ├── executor.ts                 # AgentExecutor class
│   │   ├── planner.ts                 # Planner class
│   │   ├── executor.ts                 # Executor class
│   │   └── types.ts                    # Agent types
│   ├── tools/
│   │   ├── registry.ts                 # ToolRegistry class
│   │   ├── calculator.ts              # Calculator tool
│   │   ├── search.ts                   # Search tool
│   │   ├── validator.ts                # Validator tool
│   │   └── types.ts                    # Tool types
│   ├── validation/
│   │   ├── validator.ts                # SchemaValidator class
│   │   ├── repair.ts                   # RepairEngine class
│   │   └── retry.ts                    # Retry logic utilities
│   ├── trace/
│   │   ├── recorder.ts                 # TraceRecorder class
│   │   ├── processor.ts                # Trace processing utilities
│   │   └── types.ts                    # Trace types
│   ├── supabase/
│   │   ├── client.ts                   # Supabase client
│   │   └── server.ts                   # Server-side client
│   └── utils/
│       ├── metrics.ts                  # Metrics calculation utilities
│       └── formatting.ts               # Data formatting
├── package.json
├── next.config.ts
├── tsconfig.json
└── tailwind.config.ts
```

## Core Features Implementation

### 1. Database Schema (packages/db)

Add to Prisma schema:

```prisma
model AgentRun {
  id          String   @id @default(uuid())
  userId      String?
  task        String
  config      Json     // AgentConfig
  status      String   // pending, running, completed, failed, cancelled
  startedAt   DateTime @default(now())
  completedAt DateTime?
  metrics     Json     // RunMetrics
  traceId     String   @unique
  traces      AgentTrace[]
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([userId, createdAt])
  @@index([status])
}

model AgentTrace {
  id          String   @id @default(uuid())
  runId       String
  run         AgentRun @relation(fields: [runId], references: [id], onDelete: Cascade)
  stepNumber  Int
  nodeType    String   // plan, tool_call, validation, repair, error, retry
  nodeData    Json     // TraceNodeData
  parentId    String?
  timestamp   DateTime @default(now())
  latency     Int?     // milliseconds
  createdAt   DateTime @default(now())
  
  @@index([runId, stepNumber])
  @@index([runId, nodeType])
}
```

### 2. Core Agent Package (packages/core-agent)

Implement agent execution engine:

- `src/executor.ts` - AgentExecutor class with planner-executor loop
- `src/planner.ts` - Planner class using Vercel AI SDK
- `src/executor.ts` - Executor class for step execution
- `src/tool-registry.ts` - ToolRegistry for tool management
- `src/types.ts` - Agent, Plan, StepResult types
- `src/retry.ts` - Retry logic with exponential backoff
- `src/context.ts` - Execution context management

### 3. API Routes

**POST /api/agent/run** - Execute agent run (Edge function)

- Accept task string and AgentConfig
- Initialize AgentExecutor with tools and config
- Execute agent with trace recording
- Stream updates via SSE (optional) or return complete result
- Store run and traces in Supabase
- Return runId, status, metrics, traceId

**POST /api/tools/execute** - Execute tool call

- Validate tool name and input parameters
- Execute tool via ToolRegistry
- Return tool result with latency
- Handle errors gracefully

**GET /api/traces/:runId** - Get trace data

- Retrieve trace nodes and edges from Supabase
- Return TraceGraph structure
- Include metadata and metrics

**GET /api/traces/:runId/graph** - Get trace graph for React Flow

- Transform trace data to React Flow format
- Calculate node positions (hierarchical layout)
- Return nodes and edges array

**GET /api/metrics/runs** - Get run metrics

- Query AgentRun records with filters
- Aggregate metrics (success rate, latency, retry count)
- Return runs list and aggregate statistics

**GET /api/metrics/aggregate** - Get aggregated metrics

- Calculate time-series metrics
- Aggregate by date range, agent config
- Return reliability scores, tool usage stats

### 4. UI Components

**Agent Configuration**

- Agent config form with model selection, temperature, max steps
- Tool selection multi-select interface
- Schema editor (JSON Schema or Zod)
- Retry configuration form
- Preset selection dropdown

**Agent Execution**

- Run creation form with task input
- Real-time execution status display
- Streaming output viewer (if streaming enabled)
- Run cancellation button
- Progress indicator

**Trace Visualization**

- React Flow graph with custom node types
- Interactive node details panel
- Zoom, pan, minimap controls
- Node search and filtering
- Timeline view alternative

**Metrics Dashboard**

- Success rate over time (line chart)
- Latency distribution (histogram)
- Tool usage statistics (bar chart)
- Error breakdown (pie chart)
- Reliability score trend (line chart)
- Run list with filtering and sorting

### 5. Built-in Tools

**Calculator Tool**

- Safe mathematical expression evaluation
- Zod schema: `z.object({ expression: z.string() })`
- Returns: `{ result: number }`

**Search Tool**

- Web search simulation (mock or real API)
- Zod schema: `z.object({ query: z.string(), limit: z.number().optional() })`
- Returns: `{ results: Array<{ title, url, snippet }> }`

**Validator Tool**

- Schema validation for outputs
- Zod schema: `z.object({ data: z.unknown(), schema: z.string() })`
- Returns: `{ valid: boolean, errors?: ZodError }`

### 6. Integration Points

- Use `@ai-reliability-suite/core-agent` for agent execution engine
- Use `@ai-reliability-suite/db` for database access
- Use `@ai-reliability-suite/ui` for shared components
- Vercel AI SDK for LLM calls in Edge functions
- Supabase for data persistence
- React Flow for trace visualization
- Recharts for metrics charts

## Implementation Steps

1. Initialize Next.js app with TypeScript and App Router
2. Set up Tailwind CSS and shadcn/ui components
3. Create database schema in packages/db for AgentRun and AgentTrace
4. Implement core-agent package with AgentExecutor, Planner, Executor classes
5. Build ToolRegistry with calculator, search, validator tools
6. Implement SchemaValidator and RepairEngine in core-agent
7. Create TraceRecorder class for trace logging
8. Build API routes for agent execution (POST /api/agent/run)
9. Build API routes for trace retrieval (GET /api/traces/:runId)
10. Create React Flow trace visualization component
11. Build agent configuration form UI
12. Build agent execution UI with real-time status
13. Create metrics dashboard with charts
14. Implement trace details panel and interactive features
15. Add error handling and loading states
16. Test end-to-end agent execution workflow

## Key Dependencies

- next, react, react-dom
- @vercel/ai (Vercel AI SDK)
- @xyflow/react (React Flow)
- @supabase/supabase-js
- zod (schema validation)
- recharts (charts)
- @ai-reliability-suite/core-agent (workspace)
- @ai-reliability-suite/db (workspace)
- @ai-reliability-suite/ui (workspace)

### To-dos

- [ ] Initialize Next.js app structure with App Router, TypeScript, and Tailwind CSS configuration
- [ ] Set up Supabase client configuration and database schema for AgentRun and AgentTrace models in packages/db
- [ ] Implement core-agent package with AgentExecutor, Planner, Executor classes and tool orchestration logic
- [ ] Build ToolRegistry with built-in tools (calculator, search, validator) and tool execution logic
- [ ] Implement SchemaValidator and RepairEngine classes with Zod validation and LLM-based repair logic
- [ ] Create TraceRecorder class for capturing execution traces and storing in Supabase
- [ ] Build Edge function for agent execution (POST /api/agent/run) using Vercel AI SDK
- [ ] Create API routes for trace retrieval (GET /api/traces/:runId and GET /api/traces/:runId/graph)
- [ ] Build tool execution API endpoint (POST /api/tools/execute)
- [ ] Create metrics API endpoints (GET /api/metrics/runs and GET /api/metrics/aggregate)
- [ ] Create React Flow trace visualization component with custom node types and interactive features
- [ ] Build agent configuration form UI with model selection, tool selection, schema editor, and retry config
- [ ] Create agent execution UI with real-time status display, progress indicator, and cancellation
- [ ] Build metrics dashboard with charts (success rate, latency, tool usage, reliability scores) using Recharts
- [ ] Create trace details panel with node information, tool call data, and error messages