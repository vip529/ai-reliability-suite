# ⚙️ SynthBench — Technical Specification

## 📐 Data Models & Types

### Dataset
```typescript
type Dataset = {
  id: string;
  name: string;
  description?: string;
  domain: string; // e.g., "math", "reasoning", "entity-extraction"
  samples: Sample[];
  generators: Generator[];
  jobs: GenerationJob[];
  createdAt: Date;
  updatedAt: Date;
};
```

### Sample
```typescript
type Sample = {
  id: string;
  datasetId: string;
  dataset: Dataset;
  prompt: string;
  metadata?: Record<string, unknown>; // Additional context, tags, etc.
  validated: boolean;
  validationErrors?: ValidationError[];
  createdAt: Date;
};
```

### Generator
```typescript
type Generator = {
  id: string;
  name: string;
  description?: string;
  type: 'ai' | 'fuzz' | 'hybrid';
  config: GeneratorConfig;
  template?: string; // Prompt template for AI generation
  fuzzerRules?: FuzzerRules; // DSL fuzzer rules
  datasetId?: string;
  dataset?: Dataset;
  createdAt: Date;
  updatedAt: Date;
};
```

### GeneratorConfig
```typescript
type GeneratorConfig = {
  // AI Generator Config
  model?: string; // 'gpt-4', 'claude-3-opus', etc.
  temperature?: number;
  maxTokens?: number;
  batchSize?: number;
  
  // Fuzzer Config
  fuzzerType?: 'regex' | 'json' | 'sql';
  iterations?: number;
  seed?: number;
  
  // Hybrid Config
  aiWeight?: number; // 0-1, balance between AI and fuzzing
};
```

### FuzzerRules
```typescript
type FuzzerRules = {
  type: 'regex' | 'json' | 'sql';
  rules: FuzzerRule[];
  constraints?: FuzzerConstraints;
};

type FuzzerRule = 
  | { type: 'regex'; pattern: string; flags?: string }
  | { type: 'json'; schema: JSONSchema; variations: string[] }
  | { type: 'sql'; grammar: SQLGrammar; templates: string[] };

type FuzzerConstraints = {
  minLength?: number;
  maxLength?: number;
  allowedChars?: string;
  forbiddenPatterns?: string[];
};
```

### GenerationJob
```typescript
type GenerationJob = {
  id: string;
  datasetId: string;
  dataset: Dataset;
  generatorId: string;
  generator: Generator;
  status: 'pending' | 'running' | 'completed' | 'failed' | 'cancelled';
  targetCount: number;
  generatedCount: number;
  validatedCount: number;
  metadata?: JobMetadata;
  createdAt: Date;
  completedAt?: Date;
};
```

### JobMetadata
```typescript
type JobMetadata = {
  progress: number; // 0-100
  currentBatch?: number;
  totalBatches?: number;
  errors?: JobError[];
  samples?: SamplePreview[]; // Preview of generated samples
  validationStats?: ValidationStats;
};

type JobError = {
  message: string;
  timestamp: Date;
  sampleId?: string;
};

type SamplePreview = {
  id: string;
  prompt: string;
  validated: boolean;
};
```

### ValidationStats
```typescript
type ValidationStats = {
  total: number;
  valid: number;
  invalid: number;
  duplicate: number;
  duplicateRemoved: number;
  averageSimilarity?: number;
};
```

### ValidationError
```typescript
type ValidationError = {
  type: 'semantic' | 'syntax' | 'duplicate' | 'custom';
  message: string;
  severity: 'error' | 'warning';
  details?: Record<string, unknown>;
};
```

---

## 🏗️ Core Components

### AIGenerator

**Purpose**: Generate synthetic data using LLM via Vercel AI SDK

**Interface**:
```typescript
class AIGenerator {
  constructor(config: AIGeneratorConfig);
  
  async generate(template: string, count: number, context?: Record<string, unknown>): Promise<string[]>;
  async generateBatch(template: string, batchSize: number, totalCount: number): Promise<string[]>;
  generateCounterfactual(basePrompt: string, variations: string[]): Promise<string[]>;
  generateEdgeCase(domain: string, constraints: string[]): Promise<string[]>;
}
```

**Strategies**:
- **Template-based**: Use prompt templates with variable substitution
- **Counterfactual**: Generate variations by negating or modifying base prompts
- **Edge-case**: Generate boundary cases and adversarial examples

**Responsibilities**:
- Call LLM using Vercel AI SDK
- Handle template variable substitution
- Generate diverse samples
- Handle errors and retries
- Support batch generation

### FuzzerEngine

**Purpose**: Generate synthetic data using DSL-based fuzzing

**Interface**:
```typescript
class FuzzerEngine {
  constructor(rules: FuzzerRules);
  
  generate(count: number, seed?: number): Promise<string[]>;
  validateRule(rule: FuzzerRule): ValidationResult;
}
```

**Fuzzer Types**:
- **Regex Fuzzer**: Generate strings matching regex patterns
- **JSON Fuzzer**: Generate valid/invalid JSON based on schema
- **SQL Fuzzer**: Generate SQL-like queries from grammar

**Responsibilities**:
- Parse fuzzer rules
- Generate samples based on rules
- Ensure generated samples meet constraints
- Support seeding for reproducibility

### SemanticValidator

**Purpose**: Validate and deduplicate generated samples

**Interface**:
```typescript
class SemanticValidator {
  constructor(config: ValidatorConfig);
  
  validate(samples: string[]): Promise<ValidationResult[]>;
  deduplicate(samples: string[], threshold?: number): Promise<DeduplicationResult>;
  calculateSimilarity(sample1: string, sample2: string): Promise<number>;
}
```

**Validation Types**:
- **Semantic**: Check if sample makes sense
- **Syntax**: Check if sample is well-formed
- **Duplicate**: Check for similar samples

**Responsibilities**:
- Validate sample quality
- Calculate semantic similarity
- Remove duplicates based on threshold
- Report validation errors

### GenerationJobExecutor

**Purpose**: Execute generation jobs with progress tracking

**Interface**:
```typescript
class GenerationJobExecutor {
  constructor(generator: AIGenerator | FuzzerEngine, validator: SemanticValidator);
  
  async execute(job: GenerationJob): Promise<GenerationJob>;
  async generateSamples(job: GenerationJob): Promise<Sample[]>;
  async validateSamples(samples: Sample[]): Promise<Sample[]>;
  updateProgress(jobId: string, progress: number): Promise<void>;
}
```

**Responsibilities**:
- Orchestrate generation and validation
- Track job progress
- Handle errors and retries
- Store samples in database
- Update job status

### EvalForgeExporter

**Purpose**: Export datasets to EvalForge format

**Interface**:
```typescript
class EvalForgeExporter {
  constructor();
  
  export(dataset: Dataset, format: 'json' | 'csv'): Promise<ExportResult>;
  exportToEvalForge(dataset: Dataset, taskConfig: EvalForgeTaskConfig): Promise<ExportResult>;
  formatSamples(samples: Sample[]): Promise<FormattedSample[]>;
}
```

**Export Formats**:
- **JSON**: Structured JSON format
- **CSV**: CSV format for spreadsheet tools
- **EvalForge**: Direct integration with EvalForge API

**Responsibilities**:
- Format samples for export
- Create EvalForge evaluation tasks
- Handle export errors
- Return export metadata

---

## 📊 API Endpoints

### POST /api/datasets
**Purpose**: Create a new dataset

**Request**:
```typescript
{
  name: string;
  description?: string;
  domain: string;
}
```

**Response**:
```typescript
{
  id: string;
  dataset: Dataset;
}
```

### GET /api/datasets
**Purpose**: List all datasets

**Query Parameters**:
- `page`: number (default: 1)
- `limit`: number (default: 20)
- `domain`: string (filter by domain)
- `search`: string (search by name/description)
- `sortBy`: 'name' | 'createdAt' | 'updatedAt'
- `order`: 'asc' | 'desc'

**Response**:
```typescript
{
  datasets: Dataset[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

### GET /api/datasets/[id]
**Purpose**: Get dataset by ID with samples

**Response**:
```typescript
{
  dataset: Dataset;
  sampleCount: number;
  validatedCount: number;
  statistics: DatasetStatistics;
}
```

### POST /api/generators
**Purpose**: Create a new generator

**Request**:
```typescript
{
  name: string;
  description?: string;
  type: 'ai' | 'fuzz' | 'hybrid';
  config: GeneratorConfig;
  template?: string;
  fuzzerRules?: FuzzerRules;
  datasetId?: string;
}
```

**Response**:
```typescript
{
  id: string;
  generator: Generator;
}
```

**Validation**:
- If type is 'ai', template must be provided
- If type is 'fuzz', fuzzerRules must be provided
- If type is 'hybrid', both template and fuzzerRules must be provided

### POST /api/generate
**Purpose**: AI-powered generation (Edge function)

**Request**:
```typescript
{
  generatorId: string;
  count: number;
  context?: Record<string, unknown>;
}
```

**Response**:
```typescript
{
  samples: string[];
  metadata: {
    generated: number;
    timeMs: number;
  };
}
```

### POST /api/fuzz
**Purpose**: DSL fuzzing (Edge function)

**Request**:
```typescript
{
  fuzzerRules: FuzzerRules;
  count: number;
  seed?: number;
}
```

**Response**:
```typescript
{
  samples: string[];
  metadata: {
    generated: number;
    timeMs: number;
  };
}
```

### POST /api/validate
**Purpose**: Semantic validation endpoint

**Request**:
```typescript
{
  samples: string[];
  options?: {
    deduplicate?: boolean;
    similarityThreshold?: number;
  };
}
```

**Response**:
```typescript
{
  results: ValidationResult[];
  statistics: ValidationStats;
}
```

### POST /api/jobs
**Purpose**: Create generation job

**Request**:
```typescript
{
  datasetId: string;
  generatorId: string;
  targetCount: number;
}
```

**Response**:
```typescript
{
  id: string;
  job: GenerationJob;
}
```

### GET /api/jobs/[id]
**Purpose**: Get job status

**Response**:
```typescript
{
  job: GenerationJob;
  progress: number;
  samples: SamplePreview[];
  statistics: ValidationStats;
}
```

### POST /api/export/[datasetId]
**Purpose**: Export dataset

**Request**:
```typescript
{
  format: 'json' | 'csv';
  options?: {
    includeMetadata?: boolean;
    validatedOnly?: boolean;
  };
}
```

**Response**:
```typescript
{
  exportId: string;
  format: string;
  downloadUrl?: string;
  data?: string; // For small exports
}
```

### POST /api/export/[datasetId]/evalforge
**Purpose**: Export dataset to EvalForge

**Request**:
```typescript
{
  taskName: string;
  taskDescription?: string;
  rubric: RubricCriteria[];
  schema?: JSONSchema;
}
```

**Response**:
```typescript
{
  exportId: string;
  evalForgeTaskId?: string;
  status: 'success' | 'partial' | 'failed';
  message?: string;
}
```

---

## 🎨 UI Components

### DatasetForm
**Props**:
```typescript
type DatasetFormProps = {
  initialDataset?: Dataset;
  onSubmit: (dataset: Dataset) => void;
  onCancel?: () => void;
};
```

**Fields**:
- Name input
- Description textarea
- Domain selector dropdown

### GeneratorForm
**Props**:
```typescript
type GeneratorFormProps = {
  initialGenerator?: Generator;
  onSubmit: (generator: Generator) => void;
  onCancel?: () => void;
};
```

**Fields**:
- Name and description
- Type selector (AI/Fuzz/Hybrid)
- Conditional fields based on type
- Template editor (for AI/Hybrid)
- Fuzzer rules editor (for Fuzz/Hybrid)

### PromptTemplateEditor
**Props**:
```typescript
type PromptTemplateEditorProps = {
  template: string;
  onChange: (template: string) => void;
};
```

**Features**:
- Syntax highlighting
- Variable placeholder support (e.g., `{{variable}}`)
- Template preview
- Template validation

### FuzzerConfig
**Props**:
```typescript
type FuzzerConfigProps = {
  rules: FuzzerRules;
  onChange: (rules: FuzzerRules) => void;
};
```

**Features**:
- Fuzzer type selector
- Rule editor with syntax highlighting
- Rule validation
- Rule preview/test

### JobProgress
**Props**:
```typescript
type JobProgressProps = {
  job: GenerationJob;
  onCancel?: () => void;
};
```

**Features**:
- Progress bar
- Percentage display
- Generated/validated counts
- ETA calculation
- Real-time updates

### ExportDialog
**Props**:
```typescript
type ExportDialogProps = {
  dataset: Dataset;
  onExport: (format: string, options: ExportOptions) => void;
  onClose: () => void;
};
```

**Features**:
- Format selector
- Export options
- EvalForge integration UI
- Export progress

---

## 🔄 Generation Flow

### Generation Job Lifecycle

1. **Job Creation**
   - User creates job via POST /api/jobs
   - Job record created with 'pending' status
   - Returns job ID

2. **Generation Execution**
   - Fetch generator and dataset from database
   - Update job status to 'running'
   - For each batch:
     - Generate samples (AI or Fuzz)
     - Validate samples
     - Deduplicate if enabled
     - Store valid samples
     - Update progress

3. **Validation Phase**
   - For each generated sample:
     - Semantic validation
     - Syntax validation
     - Duplicate check
   - Mark samples as validated/invalid
   - Update validation statistics

4. **Completion**
   - Update job status to 'completed'
   - Set completedAt timestamp
   - Calculate final statistics
   - Return job results

### AI Generation Process

1. **Template Processing**
   - Parse template for variables
   - Substitute variables with context values
   - Generate prompt variations

2. **LLM Call**
   - Call LLM via Vercel AI SDK
   - Use configured model and parameters
   - Handle streaming if supported

3. **Response Processing**
   - Parse LLM response
   - Extract generated samples
   - Clean and normalize samples

### Fuzzing Process

1. **Rule Parsing**
   - Parse fuzzer rules
   - Validate rule syntax
   - Build generation grammar

2. **Sample Generation**
   - Generate samples based on rules
   - Apply constraints
   - Ensure diversity

3. **Validation**
   - Validate generated samples
   - Check against constraints
   - Filter invalid samples

---

## 📈 Validation & Deduplication

### Similarity Calculation
```typescript
async function calculateSimilarity(sample1: string, sample2: string): Promise<number> {
  // Use embedding-based similarity (e.g., OpenAI embeddings)
  const embedding1 = await getEmbedding(sample1);
  const embedding2 = await getEmbedding(sample2);
  
  // Cosine similarity
  return cosineSimilarity(embedding1, embedding2);
}
```

### Deduplication Algorithm
```typescript
async function deduplicate(samples: string[], threshold: number = 0.9): Promise<string[]> {
  const unique: string[] = [];
  const seen: string[] = [];
  
  for (const sample of samples) {
    let isDuplicate = false;
    
    for (const seenSample of seen) {
      const similarity = await calculateSimilarity(sample, seenSample);
      if (similarity >= threshold) {
        isDuplicate = true;
        break;
      }
    }
    
    if (!isDuplicate) {
      unique.push(sample);
      seen.push(sample);
    }
  }
  
  return unique;
}
```

---

## 🔐 Security Considerations

1. **LLM API Keys**
   - Store API keys in environment variables
   - Never expose keys in client-side code
   - Use Edge functions for LLM calls

2. **Input Validation**
   - Validate all user inputs (templates, rules)
   - Sanitize templates to prevent injection
   - Limit template complexity

3. **Fuzzer Rules**
   - Validate fuzzer rules before execution
   - Prevent malicious rule definitions
   - Limit rule complexity

4. **Rate Limiting**
   - Implement rate limiting on generation endpoints
   - Prevent abuse of LLM API calls
   - Queue large generation jobs

5. **Data Privacy**
   - Don't log sensitive generated data
   - Allow users to delete datasets and samples
   - Implement data retention policies

---

## 🚀 Performance Targets

- **AI Generation**: < 2s per sample for simple templates
- **Fuzzing**: < 100ms per sample for regex/JSON fuzzing
- **Validation**: < 500ms per sample for semantic validation
- **Deduplication**: < 1s per sample comparison
- **Job Execution**: < 5s for small jobs (< 10 samples)
- **Dashboard Load**: < 1s for initial load
- **API Response**: < 200ms for non-generation endpoints

---

## 📝 Notes

- All timestamps should use ISO 8601 format
- Use UUIDs (cuid) for all IDs
- Support both AI and fuzzing generation methods
- Ensure generated samples are diverse and challenging
- Implement proper error boundaries in React
- Use React Suspense for async data loading
- Consider batch processing for large generations
- Support seeding for reproducible fuzzing




