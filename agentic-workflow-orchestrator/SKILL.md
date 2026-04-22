---
name: agentic-workflow-orchestrator
description: Manage complex multi-step AI tasks by breaking them into logical sub-tasks with state tracking, dependency management, and autonomous execution. Use for enterprise-grade feature development or multi-agent systems.
tags: [agentic, workflow, orchestration, multi-agent, project-management]
version: 2.0.0
author: SoftwDocs
---

# Agentic Workflow Orchestrator

## Overview

This skill transforms Claude into a sophisticated project orchestrator capable of managing complex, multi-phase development tasks through intelligent task decomposition, sub-agent delegation, and state-aware execution.

## Core Principles

1. **Single Responsibility**: Each sub-agent has ONE clearly defined goal
2. **State Immutability**: Task states are append-only with full audit trail
3. **Dependency-Driven Execution**: Tasks execute based on dependency graph, not sequence
4. **Fail-Fast Validation**: Every output verified before propagation to downstream tasks

## Workflow Architecture

### Phase 1: Request Analysis & Decomposition

```
Input: Complex user request
↓
Semantic Analysis → Intent classification → Scope boundaries
↓
Task Map Generation (hierarchical decomposition)
```

**Task Map Structure:**
```typescript
interface Task {
  id: string;
  name: string;
  description: string;
  agentType: 'architect' | 'frontend' | 'backend' | 'database' | 'testing' | 'devops';
  dependencies: string[];
  estimatedComplexity: 'low' | 'medium' | 'high' | 'critical';
  acceptanceCriteria: string[];
  deliverables: string[];
  contextRequired: string[];
  contextProduced: string[];
}
```

### Phase 2: Sub-Agent Delegation Matrix

| Agent Type | Expertise Domain | Input Context | Output Deliverable |
|------------|-----------------|---------------|-------------------|
| **Architect** | System design, patterns | Requirements, constraints | Architecture doc, component diagram |
| **Frontend** | UI/UX, components, state | Design system, API contracts | Component library, page implementations |
| **Backend** | API design, business logic | Schema, requirements | Route handlers, services, validators |
| **Database** | Schema, migrations, queries | Data model requirements | Prisma schema, migrations, seeders |
| **Testing** | Test plans, coverage | Implementation code | Unit tests, integration tests, E2E |
| **DevOps** | CI/CD, deployment, infra | Architecture, environment | GitHub Actions, Docker, Terraform |

### Phase 3: Context Preservation Protocol

**Context Chain Rules:**
1. Each sub-agent receives ONLY the context relevant to its task
2. Context is passed as structured data, not free-form text
3. Context versioning tracks what each agent "saw"
4. Cross-cutting concerns (auth, logging) injected via context providers

**Context Propagation Example:**
```
Main Agent (Task Map)
├── Sub-Agent A (Schema Design)
│   └── Produces: Entity relationships, field types
│       └── Context passed to → Sub-Agent B
├── Sub-Agent B (API Routes)
│   └── Receives: Schema from A + Requirements
│   └── Produces: OpenAPI spec, route handlers
│       └── Context passed to → Sub-Agent C
└── Sub-Agent C (Frontend Components)
    └── Receives: API spec from B + Design system
    └── Produces: React components, forms
```

### Phase 4: Validation Gates

**Pre-Execution Validation:**
- [ ] All dependencies exist and are complete
- [ ] Required context keys are present
- [ ] Agent has sufficient context to begin

**Post-Execution Validation:**
- [ ] All acceptance criteria met
- [ ] Output format matches specification
- [ ] No undefined/null values in critical paths
- [ ] Security scan passed (no secrets, SQL injection checks)

**Integration Validation:**
- [ ] Components compose correctly
- [ ] TypeScript types align across boundaries
- [ ] API contracts match between consumer and provider

## Execution Patterns

### Pattern 1: Sequential Pipeline
For tasks with strict dependencies:
```
Auth System Build
├── Schema → API → UI → Tests
└── Each waits for previous completion
```

### Pattern 2: Parallel Fan-Out
For independent modules:
```
Dashboard Feature
├── Sidebar Component (Agent A)
├── Analytics Widget (Agent B)
├── User Profile Card (Agent C)
└── Main Agent composes when all complete
```

### Pattern 3: Dynamic Branching
For conditional execution:
```
E-commerce Checkout
├── Payment Gateway Check
│   ├── If Stripe → Stripe Agent
│   ├── If PayPal → PayPal Agent
│   └── If Custom → Custom Gateway Agent
└── All branches converge to Receipt Generator
```

## State Management

**Task State Machine:**
```
PENDING → ASSIGNED → IN_PROGRESS → REVIEW → COMPLETED
                    ↓
                   FAILED → RETRY/ABORT
```

**State Persistence:**
```typescript
interface WorkflowState {
  workflowId: string;
  startedAt: DateTime;
  tasks: Map<string, TaskState>;
  globalContext: Record<string, unknown>;
  checkpoints: Checkpoint[];
}

interface TaskState {
  status: TaskStatus;
  assignedAgent: string;
  startedAt?: DateTime;
  completedAt?: DateTime;
  attempts: number;
  outputs: Record<string, unknown>;
  validationResults: ValidationResult[];
  errorLog: ErrorEntry[];
}
```

## Error Handling & Recovery

**Retry Strategies:**
- **Immediate Retry**: Network flakes, transient errors (max 3 attempts)
- **Backoff Retry**: Rate limits, resource contention (exponential backoff)
- **Escalation**: Persistent failures escalate to Main Agent for replanning

**Circuit Breaker Pattern:**
```typescript
if (agentFailureRate > 0.5) {
  // Route to alternative agent or degrade gracefully
  await escalateToAlternativeAgent(task);
}
```

## Advanced Techniques

### Technique 1: Meta-Cognitive Planning
Before delegation, ask:
1. "What could go wrong in each sub-task?"
2. "What edge cases exist at integration boundaries?"
3. "What rollback plan exists if a task fails?"

### Technique 2: Context Window Optimization
- Compress completed task outputs to summaries
- Maintain only active task full context
- Archive completed workflow states

### Technique 3: Agent Specialization Drift Detection
Monitor for agents performing outside their domain. If Frontend Agent starts writing SQL, trigger re-delegation.

## Example: Full E-Commerce Build

**User Request:** "Build a complete e-commerce platform with inventory management"

**Generated Task Map:**
```
1. [ARCHITECT] System Design (0 deps)
   → Produces: Tech stack, component diagram, API contracts

2. [DATABASE] Schema Design (depends: 1)
   → Produces: Product, Order, User, Inventory entities

3. [BACKEND] Auth Service (depends: 2)
   → Produces: JWT auth, password reset, session management

4. [BACKEND] Product API (depends: 2)
   → Produces: CRUD endpoints, search, filtering

5. [BACKEND] Order API (depends: 2, 4)
   → Produces: Cart, checkout, payment webhooks

6. [BACKEND] Inventory API (depends: 2, 4)
   → Produces: Stock tracking, low-stock alerts

7. [FRONTEND] Auth UI (depends: 3)
   → Produces: Login, register, password forms

8. [FRONTEND] Product Catalog (depends: 4)
   → Produces: Product grid, filters, product detail

9. [FRONTEND] Checkout Flow (depends: 5, 7)
   → Produces: Cart, shipping, payment, confirmation

10. [FRONTEND] Admin Dashboard (depends: 3, 4, 5, 6)
    → Produces: Inventory management, order tracking, analytics

11. [TESTING] E2E Suite (depends: 7, 8, 9, 10)
    → Produces: Playwright tests for critical user flows

12. [DEVOPS] Deployment Pipeline (depends: 1)
    → Produces: CI/CD, staging/production environments
```

**Execution Order (dependency-resolved):**
```
Phase 1: 1 → 2
Phase 2: 3, 4 (parallel)
Phase 3: 5, 6, 7, 8 (parallel)
Phase 4: 9, 10 (parallel)
Phase 5: 11, 12 (parallel)
```

## Best Practices

1. **Never Delegate Without Acceptance Criteria**: Every task must have verifiable completion conditions
2. **Maintain Dependency Graph**: Always visualize task relationships before execution
3. **Checkpoint Frequently**: Save state after each completed task for recovery
4. **Validate Types at Boundaries**: Ensure type contracts between agents
5. **Document Decisions**: Log why certain architectural choices were made
6. **Monitor Token Usage**: Track context window consumption, summarize when needed

## Anti-Patterns to Avoid

❌ **God Agent**: Single agent handling everything
❌ **Context Flooding**: Passing entire conversation history to every sub-agent
❌ **Silent Failures**: Tasks marked complete without validation
❌ **Circular Dependencies**: Tasks depending on each other (detect and break cycles)
❌ **Premature Optimization**: Optimizing before architecture is complete

## Integration with Other Skills

Combine with:
- `enterprise-crud-engine` for data layer implementation
- `api-security-shield` for authentication flows
- `auto-test-architect` for validation testing
- `performance-first-nextjs` for frontend optimization
