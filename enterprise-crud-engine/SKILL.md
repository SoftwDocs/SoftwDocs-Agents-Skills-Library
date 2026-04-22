---
name: enterprise-crud-engine
description: Generate standardized enterprise CRUD modules with TanStack Query, Shadcn UI Tables, and Server Actions.
---

# Enterprise CRUD Engine

## Instructions
1. **State Management**: Use `TanStack Query` for caching and optimistic updates.
2. **UI Pattern**: Use `Shadcn/UI` DataTable with server-side pagination and filtering.
3. **Safe Actions**: Wrap every DB mutation in a `try-catch` with standardized error logging.
4. **Audit Trail**: Every update must include 'updatedAt' and 'updatedBy' logic.

## Example Pattern
```typescript
export async function updateRecord(id: string, data: any) {
  // 1. Auth check
  // 2. Data validation
  // 3. Database operation
  // 4. RevalidatePath('/')
}
