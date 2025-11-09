# 🚀 AgentReliabilityLab — Quick Start Guide

## 📋 Overview

This guide provides a quick reference for implementing the AgentReliabilityLab module. For detailed specifications, see [SPEC.md](./SPEC.md). For the full implementation plan, see [PLAN.md](./PLAN.md).

---

## 🎯 Core Concepts

### Agent Execution Model
```
Task → Planner → Plan → Executor → Tool Calls → Validation → Output
         ↓                                    ↓
    Trace Recorder ←──────────────────────────┘
```

### Key Components
1. **AgentExecutor**: Main orchestrator
2. **Planner**: Generates execution plans
3. **Executor**: Executes steps and tool calls
4. **ToolRegistry**: Manages available tools
5. **SchemaValidator**: Validates and repairs outputs
6. **TraceRecorder**: Records execution traces

---

## 🏗️ Implementation Order

### Week 1: Foundation
1. Set up Next.js app with App Router
2. Configure TypeScript, Tailwind, shadcn/ui
3. Set up Supabase/Prisma schema
4. Create basic folder structure

### Week 2: Core Engine
1. Implement `AgentExecutor` class
2. Build `Planner` with Vercel AI SDK
3. Build `Executor` with tool calling
4. Create `ToolRegistry` with 2-3 basic tools

### Week 3: Validation & Retry
1. Implement `SchemaValidator` with Zod
2. Build `RepairEngine` for self-repair
3. Add retry logic with exponential backoff
4. Test validation and repair flows

### Week 4: Trace System
1. Create `TraceRecorder` class
2. Implement trace storage in Supabase
3. Build trace retrieval API
4. Create basic trace data model

### Week 5: Visualization
1. Set up React Flow
2. Create custom node components
3. Build trace graph visualization
4. Add interactive features (click, zoom, pan)

### Week 6: Dashboard
1. Create dashboard layout
2. Build metrics components with Recharts
3. Implement run list and detail pages
4. Add filtering and search

### Week 7: Polish & Testing
1. Add error handling and edge cases
2. Write unit tests
3. Optimize performance
4. Add documentation

---

## 🛠️ Essential Tools to Build First

### 1. Calculator Tool (Simplest)
```typescript
// Good starting point - no external dependencies
const calculatorTool = {
  name: 'calculator',
  description: 'Evaluates mathematical expressions',
  parameters: z.object({ expression: z.string() }),
  executor: async ({ expression }) => {
    // Safe eval logic
    return { result: evaluate(expression) };
  },
};
```

### 2. Validator Tool (Core Feature)
```typescript
// Essential for schema validation testing
const validatorTool = {
  name: 'validator',
  description: 'Validates data against schema',
  parameters: z.object({
    data: z.unknown(),
    schema: z.string(), // JSON Schema string
  }),
  executor: async ({ data, schema }) => {
    // Validation logic
    return { valid: true, errors: [] };
  },
};
```

### 3. Search Tool (Mock First)
```typescript
// Start with mock data, add real API later
const searchTool = {
  name: 'search',
  description: 'Searches for information',
  parameters: z.object({ query: z.string() }),
  executor: async ({ query }) => {
    // Mock search results
    return { results: mockSearchResults(query) };
  },
};
```

---

## 📊 Database Schema (Prisma)

```prisma
model AgentRun {
  id          String   @id @default(uuid())
  userId      String?
  task        String
  config      Json     // AgentConfig
  status      String
  startedAt   DateTime @default(now())
  completedAt DateTime?
  metrics     Json     // RunMetrics
  traceId     String   @unique
  traces      AgentTrace[]
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

model AgentTrace {
  id          String   @id @default(uuid())
  runId       String
  run         AgentRun @relation(fields: [runId], references: [id])
  stepNumber  Int
  nodeType    String
  nodeData    Json
  parentId    String?
  timestamp   DateTime @default(now())
  latency     Int?     // milliseconds
  createdAt   DateTime @default(now())
}
```

---

## 🔌 API Route Structure

```
app/api/
├── agent/
│   └── run/
│       └── route.ts          # POST /api/agent/run
├── tools/
│   └── execute/
│       └── route.ts          # POST /api/tools/execute
├── traces/
│   ├── [runId]/
│   │   └── route.ts          # GET /api/traces/:runId
│   └── [runId]/graph/
│       └── route.ts          # GET /api/traces/:runId/graph
└── metrics/
    └── route.ts              # GET /api/metrics
```

---

## 🎨 UI Component Structure

```
components/
├── agent/
│   ├── agentConfigForm.tsx
│   ├── agentExecutor.tsx
│   └── runStatus.tsx
├── trace/
│   ├── traceVisualization.tsx
│   ├── traceNode.tsx
│   └── traceDetails.tsx
├── dashboard/
│   ├── metricsDashboard.tsx
│   ├── runList.tsx
│   └── reliabilityChart.tsx
└── ui/                        # shadcn/ui components
```

---

## 🔑 Key Implementation Decisions

### 1. Streaming vs Non-Streaming
- **Start with non-streaming** for MVP
- Add streaming later for better UX
- Use Server-Sent Events (SSE) for streaming

### 2. Trace Storage
- Store traces in Supabase (JSON columns)
- Keep trace graph structure in memory during execution
- Persist after run completion

### 3. Tool Execution
- Execute tools in Edge Functions for security
- Validate all inputs with Zod
- Cache tool results when appropriate

### 4. Error Handling
- Use Result types (success/error) instead of throwing
- Log errors with context
- Provide user-friendly error messages

### 5. Visualization
- Use React Flow for graph visualization
- Pre-calculate node positions (hierarchical layout)
- Lazy load large traces

---

## 📦 Dependencies Checklist

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "@vercel/ai": "latest",
    "@xyflow/react": "latest",
    "recharts": "latest",
    "zod": "latest",
    "@prisma/client": "latest",
    "@supabase/supabase-js": "latest",
    "tailwindcss": "latest",
    "zustand": "latest"
  },
  "devDependencies": {
    "@types/node": "latest",
    "@types/react": "latest",
    "typescript": "latest",
    "prisma": "latest"
  }
}
```

---

## 🧪 Testing Strategy

### Unit Tests
- Test each core class independently
- Mock external dependencies (AI SDK, Supabase)
- Test error cases and edge conditions

### Integration Tests
- Test agent execution end-to-end
- Test API endpoints
- Test database operations

### Manual Testing Checklist
- [ ] Simple agent run completes successfully
- [ ] Tool calls execute correctly
- [ ] Schema validation works
- [ ] Retry logic triggers on errors
- [ ] Trace visualization displays correctly
- [ ] Dashboard shows accurate metrics
- [ ] Error handling works gracefully

---

## 🚨 Common Pitfalls to Avoid

1. **Don't execute untrusted code** - Always sandbox tool execution
2. **Don't store sensitive data in traces** - Sanitize before storage
3. **Don't block on tool calls** - Use async/await properly
4. **Don't forget error boundaries** - Wrap components in error boundaries
5. **Don't over-fetch data** - Use pagination for large traces
6. **Don't ignore Edge runtime limits** - Keep functions lightweight

---

## 📚 Resources

- [Vercel AI SDK Docs](https://sdk.vercel.ai/docs)
- [React Flow Docs](https://reactflow.dev/)
- [Zod Docs](https://zod.dev/)
- [Next.js App Router Docs](https://nextjs.org/docs/app)
- [Supabase Docs](https://supabase.com/docs)

---

## 🎯 MVP Success Criteria

Your MVP is complete when:

1. ✅ Can create and execute a simple agent run
2. ✅ Agent can call at least 2 tools successfully
3. ✅ Schema validation works and shows errors
4. ✅ Basic trace visualization displays execution flow
5. ✅ Dashboard shows run list and basic metrics
6. ✅ Errors are handled gracefully
7. ✅ All data persists to Supabase

---

## 🔄 Next Steps After MVP

1. Add more built-in tools
2. Implement streaming responses
3. Add custom tool creation UI
4. Enhance trace visualization (timeline view)
5. Add reliability scoring and alerts
6. Implement run comparison
7. Add export functionality (JSON, CSV)
8. Create agent presets library

---

## 💡 Pro Tips

1. **Start Simple**: Build the calculator tool first - it's the easiest
2. **Mock Early**: Use mock data for tools initially, add real APIs later
3. **Trace Everything**: Record more data than you think you need
4. **Visualize Early**: Get trace visualization working early for debugging
5. **Test Incrementally**: Test each component as you build it
6. **Document As You Go**: Write JSDoc comments while code is fresh

---

## 🆘 Getting Help

- Check [SPEC.md](./SPEC.md) for detailed type definitions
- Check [PLAN.md](./PLAN.md) for implementation phases
- Review Vercel AI SDK examples
- Check React Flow examples for visualization patterns

---

**Happy Building! 🚀**

