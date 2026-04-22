---
name: agentic-workflow-orchestrator
description: Manage complex multi-step AI tasks by breaking them into logical sub-tasks. Use for autonomous agent development or complex feature builds.
---

# Agentic Workflow Orchestrator

## Instructions
1. **Analyze Phase**: Break down the user's request into a 'Task Map'.
2. **Delegation**: Identify which parts need a specialized Sub-Agent (e.g., UI, Backend, Testing).
3. **Context Preservation**: Ensure the main agent tracks the state of each sub-task.
4. **Validation**: Each output must be verified against the initial 'Task Map' before proceeding.

## Example Workflow
- User wants a full Auth system.
- Skill instructs:
  1. Generate Schema (Sub-agent A)
  2. Write API Routes (Sub-agent B)
  3. Build UI Components (Sub-agent C)
  4. Integration Testing (Main Agent)
