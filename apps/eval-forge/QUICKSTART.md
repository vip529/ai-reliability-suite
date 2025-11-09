# 🚀 EvalForge — Quick Start Guide

## 📋 Overview

This guide provides a quick reference for implementing the EvalForge module. For detailed specifications, see [SPEC.md](./SPEC.md). For the full implementation plan, see [PLAN.md](./PLAN.md).

---

## 🎯 Core Concepts

### Evaluation Model
```
Task (Prompt + Rubric + Schema) → LLM Call → Output → Score Computation → Schema Validation → Results
```

### Key Components
1. **EvaluationTask**: Defines what to evaluate (prompt, rubric, schema)
2. **EvaluationRun**: Single evaluation execution with results
3. **RubricScorer**: Computes weighted scores from outputs
4. **SchemaValidator**: Validates outputs against JSON Schema or Zod
5. **ReportGenerator**: Generates Markdown or JSON reports
6. **EvaluationExecutor**: Orchestrates LLM calls and scoring

---

## 🏗️ Implementation Order

### Week 1: Foundation
1. Set up Next.js app with App Router
2. Configure TypeScript, Tailwind, shadcn/ui
3. Set up Supabase/Prisma schema
4. Create basic folder structure
5. Set up core-eval package structure

### Week 2: Core Evaluation Engine
1. Implement `RubricScorer` class
2. Implement `SchemaValidator` (JSON Schema + Zod)
3. Build `EvaluationExecutor` with Vercel AI SDK
4. Create basic scoring functions (auto scoring)

### Week 3: API Routes
1. Create task CRUD endpoints
2. Create run management endpoints
3. Build evaluation Edge function
4. Test end-to-end evaluation flow

### Week 4: Task Management UI
1. Build task form component
2. Create rubric editor
3. Build schema editor
4. Create task list and detail pages

### Week 5: Evaluation Dashboard
1. Create run list component
2. Build result viewer with score breakdown
3. Add schema validation display
4. Create dashboard layout

### Week 6: Reports & Analytics
1. Implement report generation (Markdown/JSON)
2. Build report viewer component
3. Add score visualization charts
4. Create analytics dashboard

### Week 7: Polish & Testing
1. Add error handling and edge cases
2. Write unit tests
3. Optimize performance
4. Add documentation

---

## 🛠️ Essential Components to Build First

### 1. RubricScorer (Core Feature)
```typescript
// Start with simple auto-scoring
class RubricScorer {
  async score(output: string, rubric: RubricCriteria[]): Promise<RubricScores> {
    const scores: RubricScores = {};
    
    for (const criterion of rubric) {
      // Simple keyword-based scoring for MVP
      const score = this.autoScore(output, criterion);
      scores[criterion.id] = {
        score,
        weight: criterion.weight,
        weightedScore: score * criterion.weight,
      };
    }
    
    return scores;
  }
  
  private autoScore(output: string, criterion: RubricCriteria): number {
    // Simple heuristic: check if output contains keywords from description
    const keywords = criterion.description.toLowerCase().split(/\s+/);
    const matches = keywords.filter(kw => output.toLowerCase().includes(kw));
    return (matches.length / keywords.length) * 100;
  }
}
```

### 2. SchemaValidator (Core Feature)
```typescript
// Start with JSON Schema validation
import Ajv from 'ajv';

class SchemaValidator {
  private ajv = new Ajv();
  
  validate(data: unknown, schema: JSONSchema): ValidationResult {
    try {
      const validate = this.ajv.compile(schema);
      const valid = validate(data);
      
      return {
        valid,
        errors: valid ? [] : this.formatErrors(validate.errors || []),
        score: valid ? 100 : this.calculateComplianceScore(validate.errors || []),
      };
    } catch (error) {
      return {
        valid: false,
        errors: [{ path: '', message: error.message }],
        score: 0,
      };
    }
  }
}
```

### 3. EvaluationExecutor (Orchestrator)
```typescript
// Use Vercel AI SDK for LLM calls
import { openai } from '@ai-sdk/openai';
import { generateText } from 'ai';

class EvaluationExecutor {
  async execute(run: EvaluationRun, task: EvaluationTask): Promise<EvaluationRun> {
    // Call LLM
    const { text } = await generateText({
      model: openai(task.model),
      prompt: task.prompt,
    });
    
    // Compute scores
    const scorer = new RubricScorer(task.rubric);
    const scores = await scorer.score(text, task.prompt);
    const overallScore = scorer.calculateOverallScore(scores);
    
    // Validate schema if provided
    let schemaValid = true;
    let schemaErrors: SchemaValidationError[] = [];
    if (task.schema) {
      const validator = new SchemaValidator();
      const result = validator.validate(JSON.parse(text), task.schema);
      schemaValid = result.valid;
      schemaErrors = result.errors;
    }
    
    // Update run
    run.output = text;
    run.scores = scores;
    run.overallScore = overallScore;
    run.schemaValid = schemaValid;
    run.schemaErrors = schemaErrors;
    run.status = 'completed';
    run.completedAt = new Date();
    
    return run;
  }
}
```

---

## 📊 Database Schema (Prisma)

```prisma
model EvaluationTask {
  id          String   @id @default(cuid())
  name        String
  description String?
  prompt      String
  rubric      Json     // Array of RubricCriteria
  schema      Json?    // JSON Schema or Zod schema
  model       String
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  runs        EvaluationRun[]
  reports     EvaluationReport[]
  
  @@index([createdAt])
  @@index([model])
}

model EvaluationRun {
  id          String   @id @default(cuid())
  taskId      String
  task        EvaluationTask @relation(fields: [taskId], references: [id])
  status      String   // pending, running, completed, failed
  input       String   // Test input/prompt variant
  output      String?
  scores      Json?    // RubricScores
  overallScore Float?
  schemaValid Boolean?
  schemaErrors Json?   // Array of SchemaValidationError
  metadata    Json?
  createdAt   DateTime @default(now())
  completedAt DateTime?
  
  @@index([taskId, createdAt])
  @@index([status])
}

model EvaluationReport {
  id          String   @id @default(cuid())
  taskId      String
  task        EvaluationTask @relation(fields: [taskId], references: [id])
  format      String   // markdown, json
  content     String
  runIds      String[] // Array of run IDs
  summary     Json     // ReportSummary
  createdAt   DateTime @default(now())
  
  @@index([taskId])
}
```

---

## 🔌 API Route Structure

```
app/api/
├── tasks/
│   ├── route.ts              # GET, POST /api/tasks
│   └── [id]/
│       └── route.ts          # GET, PUT, DELETE /api/tasks/[id]
├── runs/
│   ├── route.ts              # GET, POST /api/runs
│   └── [id]/
│       └── route.ts          # GET /api/runs/[id]
├── evaluate/
│   └── route.ts              # POST /api/evaluate (Edge function)
└── reports/
    ├── route.ts              # GET, POST /api/reports
    └── [id]/
        └── route.ts          # GET /api/reports/[id]
```

---

## 🎨 UI Component Structure

```
components/
├── task/
│   ├── taskForm.tsx
│   ├── taskList.tsx
│   ├── rubricEditor.tsx
│   └── schemaEditor.tsx
├── evaluation/
│   ├── runCard.tsx
│   ├── resultViewer.tsx
│   ├── scoreBreakdown.tsx
│   └── schemaValidation.tsx
├── reports/
│   ├── reportGenerator.tsx
│   └── reportViewer.tsx
└── shared/
    ├── dataTable.tsx
    └── chart.tsx
```

---

## 🔑 Key Implementation Decisions

### 1. Scoring Methods
- **Start with auto-scoring** (keyword matching, simple heuristics)
- Add LLM-based scoring later for more nuanced evaluation
- Support custom scoring functions for advanced use cases

### 2. Schema Validation
- **Support both JSON Schema and Zod** from the start
- Use ajv for JSON Schema validation
- Use Zod's native validation for Zod schemas
- Provide unified interface for both

### 3. LLM Integration
- Use Vercel AI SDK for all LLM calls
- Support structured outputs when schema provided
- Implement retry logic for failed calls
- Cache identical prompts when possible

### 4. Report Generation
- **Start with Markdown** (easier to read)
- Add JSON format for programmatic access
- Generate reports on-demand (not pre-computed)
- Include statistics and visualizations

### 5. Error Handling
- Use Result types (success/error) instead of throwing
- Log errors with context
- Provide user-friendly error messages
- Handle partial failures (some criteria fail, others succeed)

---

## 📦 Dependencies Checklist

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "@vercel/ai": "latest",
    "@ai-sdk/openai": "latest",
    "@ai-sdk/anthropic": "latest",
    "recharts": "latest",
    "zod": "latest",
    "ajv": "latest",
    "react-markdown": "latest",
    "@prisma/client": "latest",
    "@supabase/supabase-js": "latest",
    "tailwindcss": "latest"
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
- Test RubricScorer with various outputs and rubrics
- Test SchemaValidator with valid/invalid data
- Test score calculation logic
- Test report generation

### Integration Tests
- Test evaluation execution end-to-end
- Test API endpoints
- Test database operations
- Test LLM integration (with mocks)

### Manual Testing Checklist
- [ ] Can create evaluation task with rubric
- [ ] Can create and execute evaluation run
- [ ] Rubric scores compute correctly
- [ ] Schema validation works for both JSON Schema and Zod
- [ ] Reports generate in Markdown and JSON formats
- [ ] Dashboard displays tasks, runs, and results
- [ ] Error handling works gracefully

---

## 🚨 Common Pitfalls to Avoid

1. **Don't forget to validate rubric weights** - Must sum to 1.0
2. **Don't parse JSON without error handling** - Model outputs may not be valid JSON
3. **Don't block on LLM calls** - Use async/await properly, handle timeouts
4. **Don't forget error boundaries** - Wrap components in error boundaries
5. **Don't over-fetch data** - Use pagination for large lists
6. **Don't ignore Edge runtime limits** - Keep functions lightweight
7. **Don't store sensitive data** - Sanitize outputs before storage

---

## 📚 Resources

- [Vercel AI SDK Docs](https://sdk.vercel.ai/docs)
- [Zod Docs](https://zod.dev/)
- [JSON Schema Docs](https://json-schema.org/)
- [Ajv Docs](https://ajv.js.org/)
- [Next.js App Router Docs](https://nextjs.org/docs/app)
- [Supabase Docs](https://supabase.com/docs)
- [Recharts Docs](https://recharts.org/)

---

## 🎯 MVP Success Criteria

Your MVP is complete when:

1. ✅ Can create evaluation task with prompt, rubric, and optional schema
2. ✅ Can create and execute evaluation run
3. ✅ Rubric scores compute correctly with weighted calculation
4. ✅ Schema validation works for JSON Schema
5. ✅ Reports generate in Markdown format
6. ✅ Dashboard shows task list, run list, and results
7. ✅ Errors are handled gracefully
8. ✅ All data persists to Supabase

---

## 🔄 Next Steps After MVP

1. Add LLM-based scoring for rubrics
2. Support Zod schemas in addition to JSON Schema
3. Add batch evaluation (multiple inputs at once)
4. Enhance report generation with charts
5. Add score visualization and analytics
6. Implement run comparison
7. Add export functionality (CSV, JSON)
8. Create evaluation templates/presets
9. Add metamorphic testing integration

---

## 💡 Pro Tips

1. **Start Simple**: Use auto-scoring (keyword matching) first, add LLM scoring later
2. **Validate Early**: Validate rubric weights and schemas when creating tasks
3. **Cache When Possible**: Cache LLM outputs for identical prompts
4. **Test Incrementally**: Test each component (scorer, validator) independently
5. **Document Rubrics**: Encourage users to write clear rubric descriptions
6. **Handle Edge Cases**: Model outputs may not be valid JSON, handle gracefully
7. **Visualize Scores**: Charts help users understand score distributions

---

## 🆘 Getting Help

- Check [SPEC.md](./SPEC.md) for detailed type definitions
- Check [PLAN.md](./PLAN.md) for implementation phases
- Review Vercel AI SDK examples
- Check Recharts examples for visualization patterns
- Review JSON Schema examples for schema definitions

---

## 📝 Example Evaluation Task

```typescript
const exampleTask: EvaluationTask = {
  name: "Code Explanation Quality",
  description: "Evaluate how well an LLM explains code snippets",
  prompt: "Explain what this code does: ${code}",
  rubric: [
    {
      id: "clarity",
      name: "Clarity",
      description: "Explanation is clear and easy to understand",
      weight: 0.4,
    },
    {
      id: "completeness",
      name: "Completeness",
      description: "Explanation covers all important aspects",
      weight: 0.3,
    },
    {
      id: "accuracy",
      name: "Accuracy",
      description: "Explanation is technically correct",
      weight: 0.3,
    },
  ],
  schema: {
    type: "object",
    properties: {
      explanation: { type: "string" },
      complexity: { type: "string", enum: ["low", "medium", "high"] },
    },
    required: ["explanation"],
  },
  model: "gpt-4",
};
```

---

**Happy Building! 🚀**

