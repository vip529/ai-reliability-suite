# 🧠 EvalForge — Technical Specification

## 📐 Data Models & Types

### EvaluationTask
```typescript
type EvaluationTask = {
  id: string;
  name: string;
  description?: string;
  prompt: string;
  rubric: RubricCriteria[];
  schema?: JSONSchema | ZodSchema;
  model: 'gpt-4' | 'gpt-3.5-turbo' | 'claude-3-opus' | 'claude-3-sonnet' | string;
  createdAt: Date;
  updatedAt: Date;
};
```

### RubricCriteria
```typescript
type RubricCriteria = {
  id: string;
  name: string;
  description: string;
  weight: number; // 0-1, must sum to 1.0 across all criteria
  scoringFunction?: 'auto' | 'llm' | 'custom';
  maxScore?: number; // Default: 100
};
```

### EvaluationRun
```typescript
type EvaluationRun = {
  id: string;
  taskId: string;
  task: EvaluationTask;
  status: 'pending' | 'running' | 'completed' | 'failed';
  input: string; // Test input/prompt variant
  output?: string; // Model response
  scores?: RubricScores;
  overallScore?: number; // Weighted sum of rubric scores
  schemaValid?: boolean;
  schemaErrors?: SchemaValidationError[];
  metadata?: Record<string, unknown>;
  createdAt: Date;
  completedAt?: Date;
};
```

### RubricScores
```typescript
type RubricScores = {
  [criterionId: string]: {
    score: number; // 0-100 or 0-maxScore
    explanation?: string;
    weight: number;
    weightedScore: number; // score * weight
  };
};
```

### SchemaValidationError
```typescript
type SchemaValidationError = {
  path: string;
  message: string;
  value?: unknown;
  expected?: string;
};
```

### EvaluationReport
```typescript
type EvaluationReport = {
  id: string;
  taskId: string;
  task: EvaluationTask;
  format: 'markdown' | 'json';
  content: string; // Generated report content
  runIds: string[]; // Array of run IDs included
  summary: ReportSummary;
  createdAt: Date;
};
```

### ReportSummary
```typescript
type ReportSummary = {
  totalRuns: number;
  averageScore: number;
  scoreDistribution: {
    min: number;
    max: number;
    mean: number;
    median: number;
    stdDev: number;
  };
  schemaComplianceRate: number; // 0-1
  rubricBreakdown: {
    [criterionId: string]: {
      averageScore: number;
      weight: number;
      contribution: number; // averageScore * weight
    };
  };
};
```

---

## 🏗️ Core Components

### RubricScorer

**Purpose**: Compute weighted rubric scores from model outputs

**Interface**:
```typescript
class RubricScorer {
  constructor(rubric: RubricCriteria[]);
  
  async score(output: string, input: string, context?: Record<string, unknown>): Promise<RubricScores>;
  calculateOverallScore(scores: RubricScores): number;
  validateRubric(rubric: RubricCriteria[]): ValidationResult;
}
```

**Scoring Methods**:
- **auto**: Automatic scoring based on keyword matching or simple heuristics
- **llm**: Use LLM to score based on criterion description
- **custom**: User-provided scoring function

**Responsibilities**:
- Validate rubric weights sum to 1.0
- Execute scoring for each criterion
- Calculate weighted scores
- Generate explanations for scores

### SchemaValidator

**Purpose**: Validate model outputs against JSON Schema or Zod schema

**Interface**:
```typescript
class SchemaValidator {
  constructor();
  
  validate(data: unknown, schema: JSONSchema | ZodSchema): ValidationResult;
  validateJSONSchema(data: unknown, schema: JSONSchema): ValidationResult;
  validateZodSchema(data: unknown, schema: ZodSchema): ValidationResult;
}
```

**ValidationResult**:
```typescript
type ValidationResult = {
  valid: boolean;
  errors: SchemaValidationError[];
  score: number; // 0-100, compliance score
};
```

**Responsibilities**:
- Support both JSON Schema (via ajv) and Zod schemas
- Provide detailed error messages with paths
- Calculate compliance score based on errors
- Handle nested validation errors

### ReportGenerator

**Purpose**: Generate Markdown or JSON reports from evaluation runs

**Interface**:
```typescript
class ReportGenerator {
  constructor();
  
  generateMarkdown(task: EvaluationTask, runs: EvaluationRun[]): string;
  generateJSON(task: EvaluationTask, runs: EvaluationRun[]): string;
  calculateSummary(runs: EvaluationRun[]): ReportSummary;
}
```

**Report Structure** (Markdown):
- Header with task information
- Executive summary with key metrics
- Score distribution visualization (text-based)
- Rubric breakdown with average scores
- Schema compliance statistics
- Individual run details
- Recommendations and insights

**Responsibilities**:
- Aggregate scores across runs
- Calculate statistics (mean, median, std dev)
- Format output in requested format
- Include visualizations (text-based for Markdown)

### EvaluationExecutor

**Purpose**: Execute evaluation runs by calling LLM and computing scores

**Interface**:
```typescript
class EvaluationExecutor {
  constructor(model: ModelConfig, scorer: RubricScorer, validator: SchemaValidator);
  
  async execute(run: EvaluationRun, task: EvaluationTask): Promise<EvaluationRun>;
  async callLLM(prompt: string, model: string, schema?: JSONSchema): Promise<string>;
}
```

**Responsibilities**:
- Call LLM using Vercel AI SDK
- Handle structured outputs if schema provided
- Compute rubric scores
- Validate schema compliance
- Store results in database
- Handle errors and timeouts

---

## 📊 API Endpoints

### POST /api/tasks
**Purpose**: Create a new evaluation task

**Request**:
```typescript
{
  name: string;
  description?: string;
  prompt: string;
  rubric: RubricCriteria[];
  schema?: JSONSchema | ZodSchema;
  model: string;
}
```

**Response**:
```typescript
{
  id: string;
  task: EvaluationTask;
}
```

**Validation**:
- Rubric weights must sum to 1.0
- Schema must be valid JSON Schema or Zod schema
- Prompt cannot be empty

### GET /api/tasks
**Purpose**: List all evaluation tasks

**Query Parameters**:
- `page`: number (default: 1)
- `limit`: number (default: 20)
- `search`: string (search by name/description)
- `model`: string (filter by model)
- `sortBy`: 'name' | 'createdAt' | 'updatedAt'
- `order`: 'asc' | 'desc'

**Response**:
```typescript
{
  tasks: EvaluationTask[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

### GET /api/tasks/[id]
**Purpose**: Get task by ID with related runs

**Response**:
```typescript
{
  task: EvaluationTask;
  runCount: number;
  averageScore?: number;
  recentRuns: EvaluationRun[]; // Last 5 runs
}
```

### POST /api/runs
**Purpose**: Create a new evaluation run

**Request**:
```typescript
{
  taskId: string;
  input: string; // Optional: override task prompt or provide variant
  metadata?: Record<string, unknown>;
}
```

**Response**:
```typescript
{
  id: string;
  status: 'pending' | 'running';
  run: EvaluationRun;
}
```

### POST /api/evaluate
**Purpose**: Execute evaluation (Edge function)

**Request**:
```typescript
{
  runId: string; // Or taskId + input
}
```

**Response**:
```typescript
{
  runId: string;
  status: 'completed' | 'failed';
  output: string;
  scores: RubricScores;
  overallScore: number;
  schemaValid: boolean;
  schemaErrors?: SchemaValidationError[];
}
```

**Process**:
1. Fetch task and run from database
2. Call LLM with task prompt (or run input)
3. Compute rubric scores
4. Validate schema if provided
5. Update run record with results
6. Return structured response

### GET /api/runs
**Purpose**: List evaluation runs

**Query Parameters**:
- `taskId`: string (filter by task)
- `status`: string (filter by status)
- `page`: number
- `limit`: number
- `sortBy`: 'createdAt' | 'score'
- `order`: 'asc' | 'desc'

**Response**:
```typescript
{
  runs: EvaluationRun[];
  pagination: {
    page: number;
    limit: number;
    total: number;
  };
}
```

### GET /api/runs/[id]
**Purpose**: Get run details

**Response**:
```typescript
{
  run: EvaluationRun;
  task: EvaluationTask;
}
```

### POST /api/reports
**Purpose**: Generate evaluation report

**Request**:
```typescript
{
  taskId: string;
  runIds?: string[]; // If not provided, include all runs for task
  format: 'markdown' | 'json';
}
```

**Response**:
```typescript
{
  id: string;
  report: EvaluationReport;
}
```

### GET /api/reports/[id]
**Purpose**: Get report content

**Response**:
```typescript
{
  report: EvaluationReport;
  content: string; // Report content (Markdown or JSON)
}
```

---

## 🎨 UI Components

### TaskForm
**Props**:
```typescript
type TaskFormProps = {
  initialTask?: EvaluationTask;
  onSubmit: (task: EvaluationTask) => void;
  onCancel?: () => void;
};
```

**Fields**:
- Name input
- Description textarea
- Prompt editor (textarea with syntax highlighting)
- Model selector dropdown
- Rubric editor (integrated RubricEditor)
- Schema editor (integrated SchemaEditor)

### RubricEditor
**Props**:
```typescript
type RubricEditorProps = {
  rubric: RubricCriteria[];
  onChange: (rubric: RubricCriteria[]) => void;
};
```

**Features**:
- Add/remove criteria
- Edit criterion name, description, weight
- Weight validation (must sum to 1.0)
- Visual weight distribution display
- Drag-and-drop reordering (optional)

### SchemaEditor
**Props**:
```typescript
type SchemaEditorProps = {
  schema?: JSONSchema | ZodSchema;
  onChange: (schema: JSONSchema | ZodSchema | undefined) => void;
  mode: 'json-schema' | 'zod';
};
```

**Features**:
- JSON Schema editor (Monaco editor or textarea)
- Zod schema input
- Schema validation feedback
- Syntax highlighting
- Format/beautify JSON

### ResultViewer
**Props**:
```typescript
type ResultViewerProps = {
  run: EvaluationRun;
  task: EvaluationTask;
  showDetails?: boolean;
};
```

**Features**:
- Display input prompt
- Display model output with syntax highlighting
- Show score breakdown (ScoreBreakdown component)
- Display schema validation status
- Show metadata
- Copy to clipboard functionality

### ScoreBreakdown
**Props**:
```typescript
type ScoreBreakdownProps = {
  scores: RubricScores;
  overallScore: number;
  showExplanations?: boolean;
};
```

**Features**:
- Display individual rubric scores
- Show weighted contributions
- Visual score bars with color coding
- Overall weighted score display
- Score explanations (if available)

### ReportViewer
**Props**:
```typescript
type ReportViewerProps = {
  report: EvaluationReport;
  format: 'markdown' | 'json';
};
```

**Features**:
- Markdown rendering (react-markdown)
- JSON syntax highlighting
- Copy to clipboard
- Export as file
- Print-friendly styling

---

## 🔄 Evaluation Flow

### Evaluation Run Lifecycle

1. **Run Creation**
   - User creates run via POST /api/runs
   - Run record created with 'pending' status
   - Returns run ID

2. **Evaluation Execution** (POST /api/evaluate)
   - Fetch task and run from database
   - Update run status to 'running'
   - Call LLM with task prompt (or run input)
   - Receive model output

3. **Score Computation**
   - For each rubric criterion:
     - Execute scoring function (auto/llm/custom)
     - Calculate weighted score
     - Generate explanation (if available)
   - Calculate overall weighted score

4. **Schema Validation**
   - If schema provided:
     - Parse model output (JSON if structured)
     - Validate against schema
     - Collect validation errors
     - Calculate compliance score

5. **Result Storage**
   - Update run record with:
     - Output
     - Scores
     - Schema validation results
   - Set status to 'completed'
   - Set completedAt timestamp

6. **Error Handling**
   - If LLM call fails: set status to 'failed', store error
   - If scoring fails: mark criterion as failed, continue with others
   - If validation fails: store errors, continue with scoring

---

## 📈 Score Calculation

### Overall Score Formula
```typescript
function calculateOverallScore(scores: RubricScores): number {
  let total = 0;
  for (const criterionId in scores) {
    const criterion = scores[criterionId];
    total += criterion.weightedScore; // Already score * weight
  }
  return total; // Should be 0-100 if weights sum to 1.0
}
```

### Rubric Weight Validation
```typescript
function validateRubricWeights(rubric: RubricCriteria[]): boolean {
  const totalWeight = rubric.reduce((sum, c) => sum + c.weight, 0);
  return Math.abs(totalWeight - 1.0) < 0.001; // Allow small floating point errors
}
```

### Schema Compliance Score
```typescript
function calculateComplianceScore(errors: SchemaValidationError[]): number {
  if (errors.length === 0) return 100;
  
  // Penalize based on error count and severity
  const baseScore = 100;
  const penalty = errors.length * 10; // 10 points per error
  return Math.max(0, baseScore - penalty);
}
```

---

## 🔐 Security Considerations

1. **LLM API Keys**
   - Store API keys in environment variables
   - Never expose keys in client-side code
   - Use Edge functions for LLM calls

2. **Input Validation**
   - Validate all user inputs (prompts, rubrics, schemas)
   - Sanitize prompts to prevent injection attacks
   - Limit prompt length

3. **Schema Validation**
   - Validate JSON Schema before storing
   - Prevent malicious schema definitions
   - Limit schema complexity

4. **Rate Limiting**
   - Implement rate limiting on evaluation endpoints
   - Prevent abuse of LLM API calls
   - Queue evaluations if needed

5. **Data Privacy**
   - Don't log sensitive data from outputs
   - Allow users to delete runs and tasks
   - Implement data retention policies

---

## 🚀 Performance Targets

- **Evaluation Execution**: < 3s for simple evaluations (single LLM call)
- **Score Computation**: < 500ms for rubrics with < 10 criteria
- **Schema Validation**: < 200ms for typical schemas
- **Report Generation**: < 1s for reports with < 100 runs
- **Dashboard Load**: < 1s for initial load
- **API Response**: < 200ms for non-evaluation endpoints

---

## 📝 Notes

- All timestamps should use ISO 8601 format
- Use UUIDs (cuid) for all IDs
- Rubric weights must sum to 1.0 (with small tolerance for floating point)
- Support both JSON Schema and Zod schemas
- LLM outputs should be cached when possible (same prompt + model)
- Implement proper error boundaries in React
- Use React Suspense for async data loading
- Consider batch evaluation for multiple inputs

