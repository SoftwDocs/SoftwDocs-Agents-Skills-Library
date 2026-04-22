---
name: structured-ai-output
description: Force LLMs to return strict, validated JSON output for agentic workflows. Zod schemas, structured generation, and type-safe AI responses for reliable automation.
tags: [ai, llm, structured-output, zod, json, agents, validation]
version: 2.0.0
author: SoftwDocs
---

# Structured AI Output

## Overview

A comprehensive skill for forcing Large Language Models (LLMs) to return strictly-typed, validated JSON output. Essential for building reliable agentic workflows where AI responses must conform to predictable schemas.

## Why Structured Output Matters

```
┌─────────────────────────────────────────────────────────────────┐
│           UNSTRUCTURED vs STRUCTURED AI OUTPUTS                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ❌ UNSTRUCTURED (Free-form text)                               │
│  "The weather in New York is sunny and 72 degrees today..."     │
│                                                                 │
│  Problems:                                                      │
│  • Requires regex/parsing to extract data                       │
│  • Inconsistent formatting between calls                        │
│  • No type safety guarantees                                    │
│  • Difficult to validate                                        │
│  • Breaks when model changes                                    │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ STRUCTURED (Strict JSON)                                  │
│  {                                                              │
│    "location": "New York",                                      │
│    "temperature": 72,                                           │
│    "unit": "fahrenheit",                                        │
│    "condition": "sunny",                                        │
│    "confidence": 0.95                                          │
│  }                                                              │
│                                                                 │
│  Benefits:                                                      │
│  • Native TypeScript types                                      │
│  • Runtime validation with Zod                                  │
│  • 100% consistent structure                                    │
│  • Easy error handling                                          │
│  • Reliable agent orchestration                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Core Principles

1. **Schema-First Design**: Define output structure before prompting
2. **Strict Validation**: Always validate LLM output against Zod schema
3. **Fallback Strategies**: Handle parsing failures gracefully
4. **Type Safety**: Generate TypeScript types from schemas
5. **Retry Logic**: Re-prompt on validation failures

## Zod Schema Patterns

### Basic Output Schema
```typescript
// schemas/ai-output.ts
import { z } from 'zod';

// Simple structured response
export const SentimentSchema = z.object({
  sentiment: z.enum(['positive', 'negative', 'neutral']),
  confidence: z.number().min(0).max(1),
  explanation: z.string().max(200),
});

export type SentimentOutput = z.infer<typeof SentimentSchema>;
```

### Complex Agent Output
```typescript
// schemas/agent-actions.ts
import { z } from 'zod';

// Individual action an agent can take
export const AgentActionSchema = z.object({
  id: z.string().uuid(),
  type: z.enum([
    'search_database',
    'call_api',
    'send_email',
    'create_task',
    'update_record',
    'generate_report',
    'ask_user',
  ]),
  description: z.string().max(500),
  parameters: z.record(z.unknown()).optional(),
  dependsOn: z.array(z.string().uuid()).optional(),
  estimatedDuration: z.number().int().min(1).max(3600).optional(), // seconds
});

// Complete agent plan
export const AgentPlanSchema = z.object({
  goal: z.string().max(1000),
  actions: z.array(AgentActionSchema).min(1).max(20),
  estimatedTotalDuration: z.number().int().min(1),
  requiredResources: z.array(z.string()).optional(),
  riskAssessment: z.object({
    level: z.enum(['low', 'medium', 'high']),
    mitigations: z.array(z.string()).optional(),
  }),
});

export type AgentAction = z.infer<typeof AgentActionSchema>;
export type AgentPlan = z.infer<typeof AgentPlanSchema>;
```

### Nested Structured Output
```typescript
// schemas/code-generation.ts
export const CodeFileSchema = z.object({
  path: z.string().regex(/^[^\s]+$/), // No spaces in path
  content: z.string().min(1),
  language: z.enum([
    'typescript',
    'javascript',
    'python',
    'css',
    'html',
    'json',
    'sql',
    'markdown',
  ]),
  description: z.string().max(500).optional(),
});

export const CodeReviewCommentSchema = z.object({
  line: z.number().int().min(1),
  severity: z.enum(['error', 'warning', 'info', 'suggestion']),
  message: z.string().max(500),
  suggestion: z.string().max(500).optional(),
});

export const CodeGenerationSchema = z.object({
  files: z.array(CodeFileSchema).min(1),
  dependencies: z.array(z.object({
    name: z.string(),
    version: z.string().regex(/^\^?\d+\.\d+\.\d+/),
    isDev: z.boolean().default(false),
  })).optional(),
  setupInstructions: z.array(z.string()).optional(),
  review: z.object({
    summary: z.string().max(1000),
    comments: z.array(CodeReviewCommentSchema).optional(),
    qualityScore: z.number().min(0).max(10),
  }),
});

export type CodeGeneration = z.infer<typeof CodeGenerationSchema>;
```

## LLM Integration with Structured Output

### OpenAI Structured Output (GPT-4)
```typescript
// lib/llm/openai.ts
import OpenAI from 'openai';
import { z } from 'zod';
import { zodToJsonSchema } from 'zod-to-json-schema';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

export async function generateStructured<T extends z.ZodSchema>(
  prompt: string,
  schema: T,
  options: {
    model?: string;
    temperature?: number;
    maxRetries?: number;
  } = {}
): Promise<z.infer<T>> {
  const { model = 'gpt-4-turbo-preview', temperature = 0.1, maxRetries = 3 } = options;
  
  const jsonSchema = zodToJsonSchema(schema, 'output');
  
  const response = await openai.chat.completions.create({
    model,
    temperature,
    messages: [
      {
        role: 'system',
        content: `You are a precise data extraction assistant. Always respond with valid JSON matching the provided schema exactly.`,
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    functions: [
      {
        name: 'extract_data',
        description: 'Extract structured data from the input',
        parameters: jsonSchema.definitions?.output || jsonSchema,
      },
    ],
    function_call: { name: 'extract_data' },
  });

  const functionCall = response.choices[0].message.function_call;
  if (!functionCall?.arguments) {
    throw new Error('No function call in response');
  }

  const parsed = JSON.parse(functionCall.arguments);
  
  // Validate with Zod
  const validated = schema.safeParse(parsed);
  if (!validated.success) {
    throw new Error(`Schema validation failed: ${validated.error.message}`);
  }

  return validated.data;
}

// Usage
const sentiment = await generateStructured(
  `Analyze the sentiment: "This product is amazing, I love it!"`,
  SentimentSchema
);
// sentiment: { sentiment: 'positive', confidence: 0.95, explanation: '...' }
```

### Anthropic Claude Structured Output
```typescript
// lib/llm/anthropic.ts
import Anthropic from '@anthropic-ai/sdk';
import { z } from 'zod';

const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

export async function generateStructuredClaude<T extends z.ZodSchema>(
  prompt: string,
  schema: T,
  options: { maxRetries?: number } = {}
): Promise<z.infer<T>> {
  const { maxRetries = 3 } = options;
  
  const schemaDescription = generateSchemaDescription(schema);
  
  const fullPrompt = `${prompt}

You must respond with ONLY valid JSON matching this exact schema:
${schemaDescription}

Do not include any other text, markdown formatting, or explanations outside the JSON.`;

  const response = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    temperature: 0,
    messages: [{ role: 'user', content: fullPrompt }],
  });

  const content = response.content[0].type === 'text' ? response.content[0].text : '';
  
  // Extract JSON from response (handles code blocks)
  const jsonMatch = content.match(/```json\n?([\s\S]*?)\n?```/) || 
                    content.match(/\{[\s\S]*\}/);
  
  if (!jsonMatch) {
    throw new Error('No JSON found in response');
  }

  const jsonStr = jsonMatch[1] || jsonMatch[0];
  const parsed = JSON.parse(jsonStr);
  
  return schema.parse(parsed); // Validates and returns typed data
}

function generateSchemaDescription(schema: z.ZodSchema): string {
  // Convert Zod schema to human-readable description
  // This helps Claude understand the expected structure
  return JSON.stringify(zodToJsonSchema(schema), null, 2);
}
```

### Gemini/Google Structured Output
```typescript
// lib/llm/gemini.ts
import { GoogleGenerativeAI } from '@google/generative-ai';
import { z } from 'zod';

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY!);

export async function generateStructuredGemini<T extends z.ZodSchema>(
  prompt: string,
  schema: T
): Promise<z.infer<T>> {
  const model = genAI.getGenerativeModel({
    model: 'gemini-pro',
    generationConfig: {
      temperature: 0.1,
      responseMimeType: 'application/json',
    },
  });

  const schemaInstruction = `Respond with JSON matching this schema: ${JSON.stringify(
    zodToJsonSchema(schema)
  )}`;

  const result = await model.generateContent([
    { text: schemaInstruction },
    { text: prompt },
  ]);

  const response = await result.response;
  const text = response.text();
  
  const parsed = JSON.parse(text);
  return schema.parse(parsed);
}
```

## Multi-Provider Abstraction

### Unified LLM Client
```typescript
// lib/llm/client.ts
import { z } from 'zod';

export type LLMProvider = 'openai' | 'anthropic' | 'gemini' | 'azure';

export interface LLMConfig {
  provider: LLMProvider;
  model?: string;
  temperature?: number;
  maxTokens?: number;
}

export class StructuredLLMClient {
  constructor(private defaultConfig: LLMConfig) {}

  async generate<T extends z.ZodSchema>(
    prompt: string,
    schema: T,
    config?: Partial<LLMConfig>
  ): Promise<{
    data: z.infer<T>;
    metadata: {
      provider: LLMProvider;
      model: string;
      tokensUsed: number;
      latency: number;
    };
  }> {
    const startTime = Date.now();
    const finalConfig = { ...this.defaultConfig, ...config };

    let result: { data: z.infer<T>; tokensUsed: number };

    switch (finalConfig.provider) {
      case 'openai':
        result = await this.callOpenAI(prompt, schema, finalConfig);
        break;
      case 'anthropic':
        result = await this.callAnthropic(prompt, schema, finalConfig);
        break;
      case 'gemini':
        result = await this.callGemini(prompt, schema, finalConfig);
        break;
      default:
        throw new Error(`Unknown provider: ${finalConfig.provider}`);
    }

    return {
      data: result.data,
      metadata: {
        provider: finalConfig.provider,
        model: finalConfig.model || 'default',
        tokensUsed: result.tokensUsed,
        latency: Date.now() - startTime,
      },
    };
  }

  private async callOpenAI<T extends z.ZodSchema>(
    prompt: string,
    schema: T,
    config: LLMConfig
  ): Promise<{ data: z.infer<T>; tokensUsed: number }> {
    // Implementation from above
    // Returns validated data + token count
  }

  // ... other provider implementations
}

// Usage
const client = new StructuredLLMClient({
  provider: 'openai',
  model: 'gpt-4-turbo-preview',
  temperature: 0.1,
});

const { data, metadata } = await client.generate(
  'Extract entities from: "Apple Inc. was founded by Steve Jobs in Cupertino, CA in 1976"',
  z.object({
    entities: z.array(z.object({
      name: z.string(),
      type: z.enum(['person', 'organization', 'location', 'date']),
      value: z.string(),
    })),
  })
);
```

## Advanced Patterns

### Retry with Schema Correction
```typescript
// lib/llm/retry.ts
import { z } from 'zod';

export async function generateWithRetry<T extends z.ZodSchema>(
  prompt: string,
  schema: T,
  generateFn: (prompt: string) => Promise<unknown>,
  maxRetries = 3
): Promise<z.infer<T>> {
  let lastError: Error | null = null;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const raw = await generateFn(
        attempt === 0 
          ? prompt 
          : `${prompt}\n\nPrevious attempt failed with error: ${lastError?.message}. Please fix and return valid JSON.`
      );

      const validated = schema.safeParse(raw);
      
      if (validated.success) {
        return validated.data;
      }

      lastError = new Error(validated.error.message);
    } catch (error) {
      lastError = error instanceof Error ? error : new Error(String(error));
    }
  }

  throw new Error(`Failed after ${maxRetries} attempts: ${lastError?.message}`);
}
```

### Streaming Structured Output
```typescript
// lib/llm/streaming.ts
import { z } from 'zod';
import OpenAI from 'openai';

export async function* streamStructured<T extends z.ZodSchema>(
  prompt: string,
  schema: T,
  options: { bufferSize?: number } = {}
): AsyncGenerator<Partial<z.infer<T>>> {
  const bufferSize = options.bufferSize || 100;
  let buffer = '';

  const stream = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    stream: true,
    messages: [
      {
        role: 'system',
        content: 'Respond with JSON. Stream the response.',
      },
      { role: 'user', content: prompt },
    ],
  });

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content || '';
    buffer += content;

    // Try to parse partial JSON
    try {
      const partial = JSON.parse(buffer);
      yield partial;
    } catch {
      // Continue accumulating
    }
  }

  // Final validation
  yield schema.parse(JSON.parse(buffer));
}

// Usage
for await (const partial of streamStructured(prompt, schema)) {
  console.log('Partial result:', partial);
  // Update UI progressively
}
```

## Real-World Use Cases

### 1. Document Processing Agent
```typescript
// agents/document-processor.ts
const DocumentExtractionSchema = z.object({
  documentType: z.enum(['invoice', 'receipt', 'contract', 'resume', 'other']),
  extractedFields: z.array(z.object({
    fieldName: z.string(),
    value: z.string(),
    confidence: z.number().min(0).max(1),
    pageNumber: z.number().int().optional(),
  })),
  summary: z.string().max(500),
  actionRequired: z.boolean(),
  suggestedActions: z.array(z.enum(['review', 'approve', 'reject', 'forward'])).optional(),
});

async function processDocument(file: File) {
  const base64Image = await fileToBase64(file);
  
  const result = await generateStructured(
    `Extract all relevant information from this document: ${base64Image}`,
    DocumentExtractionSchema
  );

  if (result.actionRequired) {
    await createTask(result.suggestedActions?.[0] || 'review', result);
  }

  return result;
}
```

### 2. Code Review Agent
```typescript
// agents/code-reviewer.ts
const CodeReviewSchema = z.object({
  overallAssessment: z.enum(['approve', 'request_changes', 'comment']),
  issues: z.array(z.object({
    severity: z.enum(['critical', 'major', 'minor', 'info']),
    category: z.enum(['security', 'performance', 'maintainability', 'style', 'bug']),
    file: z.string(),
    line: z.number().optional(),
    description: z.string(),
    suggestion: z.string().optional(),
  })),
  metrics: z.object({
    complexity: z.number().min(1).max(10),
    testCoverage: z.number().min(0).max(100).optional(),
    estimatedReviewTime: z.number().int(), // minutes
  }),
  positiveFindings: z.array(z.string()).optional(),
});

async function reviewPullRequest(diff: string) {
  const review = await generateStructured(
    `Review this code diff and provide structured feedback:\n\n${diff}`,
    CodeReviewSchema
  );

  // Auto-post critical issues
  for (const issue of review.issues.filter(i => i.severity === 'critical')) {
    await postComment(issue.file, issue.line, issue.description);
  }

  return review;
}
```

### 3. Data Migration Agent
```typescript
// agents/data-migration.ts
const DataMappingSchema = z.object({
  sourceFields: z.array(z.object({
    name: z.string(),
    type: z.enum(['string', 'number', 'boolean', 'date', 'json']),
    sample: z.unknown(),
    nullCount: z.number().int(),
  })),
  mapping: z.array(z.object({
    sourceField: z.string(),
    targetField: z.string(),
    transformation: z.enum(['direct', 'cast', 'concat', 'split', 'custom']),
    transformationLogic: z.string().optional(),
    confidence: z.number().min(0).max(1),
  })),
  warnings: z.array(z.object({
    field: z.string(),
    issue: z.string(),
    recommendation: z.string(),
  })).optional(),
});

async function generateDataMapping(sourceData: unknown[], targetSchema: string) {
  return await generateStructured(
    `Generate data mapping from source to target:\nSource samples: ${JSON.stringify(sourceData.slice(0, 5))}\nTarget schema: ${targetSchema}`,
    DataMappingSchema
  );
}
```

## Error Handling

### Structured Error Schema
```typescript
// schemas/errors.ts
export const LLMErrorSchema = z.object({
  type: z.enum([
    'parse_error',
    'validation_error',
    'timeout_error',
    'rate_limit_error',
    'content_policy_error',
    'unknown_error',
  ]),
  message: z.string(),
  details: z.record(z.unknown()).optional(),
  rawResponse: z.string().optional(),
  suggestedAction: z.enum(['retry', 'escalate', 'skip', 'modify_prompt']).optional(),
});

export class StructuredLLMError extends Error {
  constructor(
    public errorData: z.infer<typeof LLMErrorSchema>
  ) {
    super(errorData.message);
  }
}
```

## Best Practices

1. **Always Validate**: Never trust raw LLM output
2. **Type Everything**: Generate TypeScript types from Zod schemas
3. **Handle Failures**: Implement retry with error context
4. **Monitor Quality**: Track validation success rates
5. **Version Schemas**: Maintain backward compatibility
6. **Test Prompts**: Unit test prompts with expected outputs

## Testing Structured Outputs

```typescript
// tests/structured-output.test.ts
import { describe, it, expect } from 'vitest';
import { generateStructured } from '@/lib/llm/client';

describe('Structured LLM Output', () => {
  it('extracts sentiment correctly', async () => {
    const result = await generateStructured(
      'This is terrible, I hate it!',
      SentimentSchema
    );

    expect(result.sentiment).toBe('negative');
    expect(result.confidence).toBeGreaterThan(0.8);
  });

  it('validates complex agent plans', async () => {
    const result = await generateStructured(
      'Plan: Search database for user, then send welcome email',
      AgentPlanSchema
    );

    expect(result.actions).toHaveLength(2);
    expect(result.actions[0].type).toBe('search_database');
  });

  it('handles validation failures gracefully', async () => {
    await expect(
      generateStructured(
        'Return invalid data',
        z.object({ required: z.string() }),
        { maxRetries: 1 }
      )
    ).rejects.toThrow();
  });
});
```
