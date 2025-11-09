# 🚀 SynthBench — Quick Start Guide

## 📋 Overview

This guide provides a quick reference for implementing the SynthBench module. For detailed specifications, see [SPEC.md](./SPEC.md). For the full implementation plan, see [PLAN.md](./PLAN.md).

---

## 🎯 Core Concepts

### Generation Model
```
Generator (AI/Fuzz/Hybrid) → Generate Samples → Validate → Deduplicate → Store in Dataset
```

### Key Components
1. **Dataset**: Container for generated samples
2. **Generator**: Defines how to generate samples (AI template, fuzzer rules)
3. **GenerationJob**: Tracks generation progress and results
4. **AIGenerator**: Uses LLM to generate samples from templates
5. **FuzzerEngine**: Uses DSL rules to generate samples
6. **SemanticValidator**: Validates and deduplicates samples

---

## 🏗️ Implementation Order

### Week 1: Foundation
1. Set up Next.js app with App Router
2. Configure TypeScript, Tailwind, shadcn/ui
3. Set up Supabase/Prisma schema
4. Create basic folder structure
5. Set up core-data package structure

### Week 2: Core Generation Engine
1. Implement `AIGenerator` class with Vercel AI SDK
2. Implement basic fuzzer (regex first)
3. Build `SemanticValidator` with basic validation
4. Create generation job executor

### Week 3: API Routes
1. Create dataset CRUD endpoints
2. Create generator CRUD endpoints
3. Build AI generation Edge function
4. Build fuzzing Edge function
5. Create validation endpoint

### Week 4: Job Management
1. Create job management API
2. Implement job progress tracking
3. Build job execution logic
4. Test end-to-end generation workflow

### Week 5: Dataset Management UI
1. Build dataset form component
2. Create dataset list and detail pages
3. Build sample viewer
4. Create export dialog

### Week 6: Generator Configuration UI
1. Build generator form component
2. Create prompt template editor
3. Build fuzzer config UI
4. Create generator list

### Week 7: Job UI & Polish
1. Create job list and detail pages
2. Build job progress component
3. Add sample preview
4. Add error handling and testing

---

## 🛠️ Essential Components to Build First

### 1. AIGenerator (Core Feature)
```typescript
// Start with simple template-based generation
import { openai } from '@ai-sdk/openai';
import { generateText } from 'ai';

class AIGenerator {
  constructor(private config: AIGeneratorConfig) {}
  
  async generate(template: string, count: number): Promise<string[]> {
    const samples: string[] = [];
    
    for (let i = 0; i < count; i++) {
      // Substitute variables in template
      const prompt = this.substituteVariables(template, { index: i });
      
      // Call LLM
      const { text } = await generateText({
        model: openai(this.config.model || 'gpt-4'),
        prompt,
        temperature: this.config.temperature || 0.7,
      });
      
      samples.push(text);
    }
    
    return samples;
  }
  
  private substituteVariables(template: string, context: Record<string, unknown>): string {
    return template.replace(/\{\{(\w+)\}\}/g, (_, key) => {
      return String(context[key] || '');
    });
  }
}
```

### 2. RegexFuzzer (Simplest Fuzzer)
```typescript
// Start with regex-based fuzzing
class RegexFuzzer {
  constructor(private pattern: string) {}
  
  generate(count: number): string[] {
    const samples: string[] = [];
    
    for (let i = 0; i < count; i++) {
      // Simple regex fuzzing: generate strings matching pattern
      const sample = this.generateFromPattern(this.pattern);
      samples.push(sample);
    }
    
    return samples;
  }
  
  private generateFromPattern(pattern: string): string {
    // Simplified: generate random strings matching pattern
    // For MVP, use a library like 'randexp' or implement basic generation
    // This is a placeholder - implement based on your needs
    return this.basicRegexGeneration(pattern);
  }
}
```

### 3. SemanticValidator (Core Feature)
```typescript
// Start with basic validation
class SemanticValidator {
  async validate(samples: string[]): Promise<ValidationResult[]> {
    const results: ValidationResult[] = [];
    
    for (const sample of samples) {
      const result: ValidationResult = {
        valid: true,
        errors: [],
      };
      
      // Basic validation checks
      if (sample.length === 0) {
        result.valid = false;
        result.errors.push({ type: 'semantic', message: 'Empty sample' });
      }
      
      if (sample.length > 10000) {
        result.valid = false;
        result.errors.push({ type: 'semantic', message: 'Sample too long' });
      }
      
      // Add more validation rules as needed
      
      results.push(result);
    }
    
    return results;
  }
  
  async deduplicate(samples: string[], threshold: number = 0.9): Promise<string[]> {
    // For MVP, use simple string comparison
    // Later, add embedding-based similarity
    const unique: string[] = [];
    const seen = new Set<string>();
    
    for (const sample of samples) {
      const normalized = sample.toLowerCase().trim();
      if (!seen.has(normalized)) {
        unique.push(sample);
        seen.add(normalized);
      }
    }
    
    return unique;
  }
}
```

### 4. GenerationJobExecutor (Orchestrator)
```typescript
class GenerationJobExecutor {
  constructor(
    private generator: AIGenerator | FuzzerEngine,
    private validator: SemanticValidator
  ) {}
  
  async execute(job: GenerationJob): Promise<GenerationJob> {
    // Update status
    job.status = 'running';
    await this.updateJob(job);
    
    try {
      // Generate samples
      const rawSamples = await this.generator.generate(
        job.generator.template || '',
        job.targetCount
      );
      
      job.generatedCount = rawSamples.length;
      await this.updateJob(job);
      
      // Validate samples
      const validationResults = await this.validator.validate(rawSamples);
      const validSamples = rawSamples.filter((_, i) => validationResults[i].valid);
      
      // Deduplicate
      const uniqueSamples = await this.validator.deduplicate(validSamples);
      
      job.validatedCount = uniqueSamples.length;
      job.status = 'completed';
      job.completedAt = new Date();
      
      // Store samples in database
      await this.storeSamples(job.datasetId, uniqueSamples);
      
      await this.updateJob(job);
      return job;
    } catch (error) {
      job.status = 'failed';
      job.metadata = { ...job.metadata, error: error.message };
      await this.updateJob(job);
      throw error;
    }
  }
}
```

---

## 📊 Database Schema (Prisma)

```prisma
model Dataset {
  id          String   @id @default(cuid())
  name        String
  description String?
  domain      String
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
  metadata    Json?
  validated   Boolean  @default(false)
  validationErrors Json?
  createdAt   DateTime @default(now())
  
  @@index([datasetId, validated])
}

model Generator {
  id          String   @id @default(cuid())
  name        String
  description String?
  type        String   // "ai", "fuzz", "hybrid"
  config      Json
  template    String?
  fuzzerRules Json?
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
  status      String
  targetCount Int
  generatedCount Int   @default(0)
  validatedCount Int   @default(0)
  metadata    Json?
  createdAt   DateTime @default(now())
  completedAt DateTime?
  
  @@index([datasetId, status])
}
```

---

## 🔌 API Route Structure

```
app/api/
├── datasets/
│   ├── route.ts              # GET, POST /api/datasets
│   └── [id]/
│       └── route.ts          # GET, PUT, DELETE /api/datasets/[id]
├── generators/
│   ├── route.ts              # GET, POST /api/generators
│   └── [id]/
│       └── route.ts          # GET, PUT, DELETE /api/generators/[id]
├── generate/
│   └── route.ts              # POST /api/generate (Edge function)
├── fuzz/
│   └── route.ts              # POST /api/fuzz (Edge function)
├── validate/
│   └── route.ts              # POST /api/validate
├── jobs/
│   ├── route.ts              # GET, POST /api/jobs
│   └── [id]/
│       └── route.ts          # GET, PUT, DELETE /api/jobs/[id]
└── export/
    ├── route.ts              # GET /api/export
    └── [datasetId]/
        ├── route.ts          # POST /api/export/[datasetId]
        └── evalforge/
            └── route.ts      # POST /api/export/[datasetId]/evalforge
```

---

## 🎨 UI Component Structure

```
components/
├── dataset/
│   ├── datasetForm.tsx
│   ├── datasetList.tsx
│   ├── datasetViewer.tsx
│   └── exportDialog.tsx
├── generator/
│   ├── generatorForm.tsx
│   ├── generatorList.tsx
│   ├── promptTemplateEditor.tsx
│   └── fuzzerConfig.tsx
├── generation/
│   ├── jobCard.tsx
│   ├── jobProgress.tsx
│   ├── samplePreview.tsx
│   └── validationStatus.tsx
└── shared/
    ├── dataTable.tsx
    └── chart.tsx
```

---

## 🔑 Key Implementation Decisions

### 1. Generation Methods
- **Start with AI generation** (easier to get working)
- Add fuzzing later for more control
- Support hybrid mode for best of both worlds

### 2. Validation Strategy
- **Start with basic validation** (length, format)
- Add semantic validation later (LLM-based)
- Use simple deduplication first (exact match)
- Add embedding-based similarity later

### 3. Job Execution
- **Run jobs synchronously for MVP** (small batches)
- Add job queue later for large generations
- Track progress in database
- Support cancellation

### 4. Template System
- **Use simple variable substitution** (`{{variable}}`)
- Support context injection
- Validate templates before use

### 5. Fuzzer Rules
- **Start with regex fuzzing** (simplest)
- Add JSON and SQL fuzzers later
- Validate rules before execution
- Support rule preview/test

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
    "react-syntax-highlighter": "latest",
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
- Test AIGenerator with mock LLM calls
- Test fuzzer engines with various rules
- Test validation logic
- Test deduplication algorithms

### Integration Tests
- Test generation job execution end-to-end
- Test API endpoints
- Test database operations
- Test export functionality

### Manual Testing Checklist
- [ ] Can create dataset
- [ ] Can create AI generator with template
- [ ] Can create fuzzer generator with rules
- [ ] Can create and execute generation job
- [ ] Samples are generated correctly
- [ ] Validation works correctly
- [ ] Deduplication removes duplicates
- [ ] Can export dataset to EvalForge

---

## 🚨 Common Pitfalls to Avoid

1. **Don't generate too many samples at once** - Batch generation to avoid timeouts
2. **Don't forget to validate** - Always validate generated samples
3. **Don't skip deduplication** - Duplicate samples reduce dataset quality
4. **Don't block on LLM calls** - Use async/await properly, handle timeouts
5. **Don't forget error boundaries** - Wrap components in error boundaries
6. **Don't over-fetch data** - Use pagination for large datasets
7. **Don't ignore Edge runtime limits** - Keep functions lightweight
8. **Don't store sensitive data** - Sanitize generated samples before storage

---

## 📚 Resources

- [Vercel AI SDK Docs](https://sdk.vercel.ai/docs)
- [Zod Docs](https://zod.dev/)
- [JSON Schema Docs](https://json-schema.org/)
- [Next.js App Router Docs](https://nextjs.org/docs/app)
- [Supabase Docs](https://supabase.com/docs)
- [Recharts Docs](https://recharts.org/)

---

## 🎯 MVP Success Criteria

Your MVP is complete when:

1. ✅ Can create dataset
2. ✅ Can create AI generator with template
3. ✅ Can create and execute generation job
4. ✅ Samples are generated using LLM
5. ✅ Basic validation works
6. ✅ Can view generated samples
7. ✅ Can export dataset (JSON format)
8. ✅ All data persists to Supabase

---

## 🔄 Next Steps After MVP

1. Add fuzzing engines (regex, JSON, SQL)
2. Add semantic validation with embeddings
3. Add embedding-based deduplication
4. Add EvalForge export integration
5. Add job queue for large generations
6. Add advanced analytics
7. Add template library
8. Add batch generation optimization

---

## 💡 Pro Tips

1. **Start Simple**: Use AI generation first, add fuzzing later
2. **Template Variables**: Use `{{variable}}` syntax for easy substitution
3. **Batch Processing**: Generate samples in batches to avoid timeouts
4. **Validate Early**: Validate samples as they're generated
5. **Deduplicate**: Always deduplicate to ensure dataset quality
6. **Track Progress**: Update job progress frequently for better UX
7. **Error Handling**: Handle LLM API errors gracefully
8. **Test Templates**: Test templates before running large jobs

---

## 🆘 Getting Help

- Check [SPEC.md](./SPEC.md) for detailed type definitions
- Check [PLAN.md](./PLAN.md) for implementation phases
- Review Vercel AI SDK examples
- Check fuzzing libraries for inspiration

---

## 📝 Example Generator Templates

### AI Generator Template
```typescript
const exampleAIGenerator: Generator = {
  name: "Math Problem Generator",
  type: "ai",
  template: "Generate a challenging math problem about {{topic}}. The problem should be suitable for {{level}} students.",
  config: {
    model: "gpt-4",
    temperature: 0.8,
  },
};
```

### Fuzzer Generator Rules
```typescript
const exampleFuzzerGenerator: Generator = {
  name: "Email Fuzzer",
  type: "fuzz",
  fuzzerRules: {
    type: "regex",
    rules: [
      {
        type: "regex",
        pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
      },
    ],
  },
};
```

---

**Happy Building! 🚀**




