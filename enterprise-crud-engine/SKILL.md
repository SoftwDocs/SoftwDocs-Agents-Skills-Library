---
name: enterprise-crud-engine
description: Generate production-grade CRUD modules with TanStack Query, Shadcn/UI DataTables, Server Actions, optimistic updates, and full audit trails. Enterprise-ready for ERP, admin dashboards, and data-heavy applications.
tags: [crud, tanstack-query, shadcn-ui, server-actions, optimistic-updates, enterprise]
version: 2.0.0
author: SoftwDocs
---

# Enterprise CRUD Engine

## Instructions
1. **State Management**: Use `TanStack Query` for caching and optimistic updates.
2. **UI Pattern**: Use `Shadcn/UI` DataTable with server-side pagination and filtering.
3. **Safe Actions**: Wrap every DB mutation in a `try-catch` with standardized error logging.
4. **Audit Trail**: Every update must include 'updatedAt' and 'updatedBy' logic.

## Overview

A comprehensive skill for building scalable, enterprise-grade CRUD (Create, Read, Update, Delete) interfaces with advanced features like optimistic updates, server-side pagination, real-time synchronization, and complete audit trails.

## Architecture Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │  DataTable   │  │   Forms      │  │  Action Menus   │  │
│  │ (Shadcn/UI)  │  │ (React Hook) │  │  (Dropdowns)    │  │
│  └──────────────┘  └──────────────┘  └─────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│                    STATE MANAGEMENT LAYER                    │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              TanStack Query (React Query)              │ │
│  │  • Caching  • Optimistic Updates  • Background Sync    │ │
│  │  • Pagination  • Filtering  • Sorting                │ │
│  └────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                    SERVER LAYER                              │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │   Actions    │  │   Zod        │  │    Prisma       │  │
│  │  (Next.js)   │  │ Validation   │  │   (Database)    │  │
│  └──────────────┘  └──────────────┘  └─────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│                    INFRASTRUCTURE LAYER                      │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │  Audit Log   │  │ Rate Limiter │  │  Error Handler  │  │
│  └──────────────┘  └──────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Server Actions Architecture

**Complete Server Action Pattern:**
```typescript
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';
import { prisma } from '@/lib/prisma';
import { auth } from '@/lib/auth';
import { createAuditLog } from '@/lib/audit';

// Zod Schema Definition
const CreateRecordSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  status: z.enum(['active', 'inactive', 'pending']),
  metadata: z.record(z.unknown()).optional(),
});

type CreateRecordInput = z.infer<typeof CreateSchema>;

interface ActionResult<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: Record<string, string[]>;
  };
  meta?: {
    timestamp: string;
    requestId: string;
    executionTimeMs: number;
  };
}

export async function createRecord(
  data: CreateRecordInput
): Promise<ActionResult<Record>> {
  const startTime = Date.now();
  const requestId = crypto.randomUUID();
  
  try {
    // 1. Authentication
    const session = await auth();
    if (!session?.user) {
      return {
        success: false,
        error: { code: 'UNAUTHORIZED', message: 'Authentication required' },
        meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
      };
    }
    
    // 2. Authorization
    if (!session.user.permissions.includes('records:create')) {
      return {
        success: false,
        error: { code: 'FORBIDDEN', message: 'Insufficient permissions' },
        meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
      };
    }
    
    // 3. Input Validation
    const validated = CreateRecordSchema.safeParse(data);
    if (!validated.success) {
      return {
        success: false,
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid input data',
          details: validated.error.flatten().fieldErrors
        },
        meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
      };
    }
    
    // 4. Database Operation with Transaction
    const record = await prisma.$transaction(async (tx) => {
      // Create record
      const newRecord = await tx.record.create({
        data: {
          ...validated.data,
          createdBy: session.user.id,
          updatedBy: session.user.id,
          createdAt: new Date(),
          updatedAt: new Date(),
        },
      });
      
      // Create audit log
      await createAuditLog(tx, {
        action: 'CREATE',
        entityType: 'Record',
        entityId: newRecord.id,
        userId: session.user.id,
        oldValues: null,
        newValues: newRecord,
        ipAddress: session.ip,
        userAgent: session.userAgent,
      });
      
      return newRecord;
    });
    
    // 5. Cache Revalidation
    revalidatePath('/dashboard/records');
    revalidatePath(`/api/records/${record.id}`);
    
    return {
      success: true,
      data: record,
      meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
    };
    
  } catch (error) {
    console.error(`[${requestId}] Create record failed:`, error);
    
    return {
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: process.env.NODE_ENV === 'production' 
          ? 'An unexpected error occurred' 
          : error.message
      },
      meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
    };
  }
}

// Update Action with Optimistic Locking
export async function updateRecord(
  id: string,
  data: Partial<CreateRecordInput>,
  version?: number // For optimistic locking
): Promise<ActionResult<Record>> {
  const startTime = Date.now();
  const requestId = crypto.randomUUID();
  
  try {
    const session = await auth();
    if (!session?.user) {
      return {
        success: false,
        error: { code: 'UNAUTHORIZED', message: 'Authentication required' },
        meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
      };
    }
    
    const existing = await prisma.record.findUnique({ where: { id } });
    if (!existing) {
      return {
        success: false,
        error: { code: 'NOT_FOUND', message: 'Record not found' },
        meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
      };
    }
    
    // Optimistic locking check
    if (version !== undefined && existing.version !== version) {
      return {
        success: false,
        error: { code: 'CONFLICT', message: 'Record was modified by another user' },
        meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
      };
    }
    
    const updated = await prisma.$transaction(async (tx) => {
      const record = await tx.record.update({
        where: { id },
        data: {
          ...data,
          updatedBy: session.user.id,
          updatedAt: new Date(),
          version: { increment: 1 },
        },
      });
      
      await createAuditLog(tx, {
        action: 'UPDATE',
        entityType: 'Record',
        entityId: id,
        userId: session.user.id,
        oldValues: existing,
        newValues: record,
        ipAddress: session.ip,
        userAgent: session.userAgent,
      });
      
      return record;
    });
    
    revalidatePath('/dashboard/records');
    revalidatePath(`/api/records/${id}`);
    
    return {
      success: true,
      data: updated,
      meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
    };
    
  } catch (error) {
    console.error(`[${requestId}] Update record failed:`, error);
    return {
      success: false,
      error: { code: 'INTERNAL_ERROR', message: 'An unexpected error occurred' },
      meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
    };
  }
}

// Soft Delete Action
export async function deleteRecord(id: string): Promise<ActionResult<void>> {
  const startTime = Date.now();
  const requestId = crypto.randomUUID();
  
  try {
    const session = await auth();
    if (!session?.user?.permissions.includes('records:delete')) {
      return {
        success: false,
        error: { code: 'FORBIDDEN', message: 'Insufficient permissions' },
        meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
      };
    }
    
    const existing = await prisma.record.findUnique({ where: { id } });
    if (!existing) {
      return {
        success: false,
        error: { code: 'NOT_FOUND', message: 'Record not found' },
        meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
      };
    }
    
    await prisma.$transaction(async (tx) => {
      // Soft delete
      await tx.record.update({
        where: { id },
        data: {
          deletedAt: new Date(),
          deletedBy: session.user.id,
          isActive: false,
        },
      });
      
      await createAuditLog(tx, {
        action: 'DELETE',
        entityType: 'Record',
        entityId: id,
        userId: session.user.id,
        oldValues: existing,
        newValues: null,
        ipAddress: session.ip,
        userAgent: session.userAgent,
      });
    });
    
    revalidatePath('/dashboard/records');
    
    return {
      success: true,
      meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
    };
    
  } catch (error) {
    console.error(`[${requestId}] Delete record failed:`, error);
    return {
      success: false,
      error: { code: 'INTERNAL_ERROR', message: 'An unexpected error occurred' },
      meta: { timestamp: new Date().toISOString(), requestId, executionTimeMs: Date.now() - startTime }
    };
  }
}

// List Action with Filtering, Sorting, Pagination
export async function listRecords(params: {
  page?: number;
  pageSize?: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
  filters?: Record<string, unknown>;
  search?: string;
}): Promise<ActionResult<{ records: Record[]; meta: PaginationMeta }>> {
  const { page = 1, pageSize = 10, sortBy = 'createdAt', sortOrder = 'desc', filters = {}, search } = params;
  
  const where: Prisma.RecordWhereInput = {
    deletedAt: null, // Exclude soft-deleted
    ...buildFilters(filters),
    ...(search && {
      OR: [
        { name: { contains: search, mode: 'insensitive' } },
        { email: { contains: search, mode: 'insensitive' } },
      ],
    }),
  };
  
  const [records, total] = await Promise.all([
    prisma.record.findMany({
      where,
      orderBy: { [sortBy]: sortOrder },
      skip: (page - 1) * pageSize,
      take: pageSize,
      include: {
        createdByUser: { select: { id: true, name: true } },
        updatedByUser: { select: { id: true, name: true } },
      },
    }),
    prisma.record.count({ where }),
  ]);
  
  return {
    success: true,
    data: {
      records,
      meta: {
        page,
        pageSize,
        total,
        totalPages: Math.ceil(total / pageSize),
      },
    },
    meta: { timestamp: new Date().toISOString(), requestId: crypto.randomUUID(), executionTimeMs: 0 }
  };
}
```

### 2. TanStack Query Integration

**Complete Query Hooks Pattern:**
```typescript
// hooks/use-records.ts
'use client';

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { createRecord, updateRecord, deleteRecord, listRecords } from '@/lib/actions';

// Query Keys Factory - Essential for cache management
const recordsKeys = {
  all: ['records'] as const,
  lists: () => [...recordsKeys.all, 'list'] as const,
  list: (filters: Record<string, unknown>) => [...recordsKeys.lists(), filters] as const,
  details: () => [...recordsKeys.all, 'detail'] as const,
  detail: (id: string) => [...recordsKeys.details(), id] as const,
};

interface UseRecordsOptions {
  page?: number;
  pageSize?: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
  filters?: Record<string, unknown>;
  search?: string;
}

export function useRecords(options: UseRecordsOptions = {}) {
  return useQuery({
    queryKey: recordsKeys.list(options),
    queryFn: () => listRecords(options),
    staleTime: 30 * 1000,
    gcTime: 5 * 60 * 1000,
    placeholderData: (previousData) => previousData,
  });
}

export function useRecord(id: string) {
  return useQuery({
    queryKey: recordsKeys.detail(id),
    queryFn: () => getRecordById(id),
    enabled: !!id,
    staleTime: 60 * 1000,
  });
}

// Optimistic Create Mutation
export function useCreateRecord() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: createRecord,
    
    onMutate: async (newRecord) => {
      await queryClient.cancelQueries({ queryKey: recordsKeys.lists() });
      
      const previousRecords = queryClient.getQueryData(recordsKeys.lists());
      
      queryClient.setQueryData(recordsKeys.lists(), (old: any) => ({
        ...old,
        data: {
          records: [
            { ...newRecord, id: `temp-${Date.now()}`, isOptimistic: true },
            ...(old?.data?.records || [])
          ],
          meta: {
            ...old?.data?.meta,
            total: (old?.data?.meta?.total || 0) + 1,
          }
        },
      }));
      
      return { previousRecords };
    },
    
    onError: (err, newRecord, context) => {
      if (context?.previousRecords) {
        queryClient.setQueryData(recordsKeys.lists(), context.previousRecords);
      }
    },
    
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: recordsKeys.lists() });
    },
  });
}

// Optimistic Update Mutation
export function useUpdateRecord() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: any }) => updateRecord(id, data),
    
    onMutate: async ({ id, data }) => {
      await queryClient.cancelQueries({ queryKey: recordsKeys.detail(id) });
      await queryClient.cancelQueries({ queryKey: recordsKeys.lists() });
      
      const previousRecord = queryClient.getQueryData(recordsKeys.detail(id));
      const previousRecords = queryClient.getQueryData(recordsKeys.lists());
      
      queryClient.setQueryData(recordsKeys.detail(id), (old: any) => ({
        ...old,
        ...data,
        isOptimistic: true,
      }));
      
      queryClient.setQueryData(recordsKeys.lists(), (old: any) => ({
        ...old,
        data: {
          ...old?.data,
          records: old?.data?.records?.map((record: any) =>
            record.id === id ? { ...record, ...data, isOptimistic: true } : record
          ),
        },
      }));
      
      return { previousRecord, previousRecords };
    },
    
    onError: (err, variables, context) => {
      if (context?.previousRecord) {
        queryClient.setQueryData(recordsKeys.detail(variables.id), context.previousRecord);
      }
      if (context?.previousRecords) {
        queryClient.setQueryData(recordsKeys.lists(), context.previousRecords);
      }
    },
    
    onSettled: (data, error, variables) => {
      queryClient.invalidateQueries({ queryKey: recordsKeys.detail(variables.id) });
      queryClient.invalidateQueries({ queryKey: recordsKeys.lists() });
    },
  });
}

// Optimistic Delete Mutation
export function useDeleteRecord() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: deleteRecord,
    
    onMutate: async (id) => {
      await queryClient.cancelQueries({ queryKey: recordsKeys.lists() });
      
      const previousRecords = queryClient.getQueryData(recordsKeys.lists());
      
      queryClient.setQueryData(recordsKeys.lists(), (old: any) => ({
        ...old,
        data: {
          records: old?.data?.records?.filter((record: any) => record.id !== id),
          meta: {
            ...old?.data?.meta,
            total: Math.max(0, (old?.data?.meta?.total || 0) - 1),
          },
        },
      }));
      
      return { previousRecords };
    },
    
    onError: (err, id, context) => {
      if (context?.previousRecords) {
        queryClient.setQueryData(recordsKeys.lists(), context.previousRecords);
      }
    },
    
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: recordsKeys.lists() });
    },
  });
}
```

### 3. DataTable with Shadcn/UI

**Advanced DataTable Implementation:**
```typescript
// components/data-table/data-table.tsx
'use client';

import {
  useReactTable,
  getCoreRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  getFilteredRowModel,
  flexRender,
} from '@tanstack/react-table';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Skeleton } from '@/components/ui/skeleton';
import { useState } from 'react';
import { useDebounce } from '@/hooks/use-debounce';

export function DataTable({ columns, data, meta, isLoading, onPaginationChange, onSortingChange }) {
  const [globalFilter, setGlobalFilter] = useState('');
  const debouncedFilter = useDebounce(globalFilter, 300);
  
  const table = useReactTable({
    data,
    columns,
    pageCount: meta.totalPages,
    manualPagination: true,
    manualSorting: true,
    state: {
      pagination: { pageIndex: meta.page - 1, pageSize: meta.pageSize },
    },
    onSortingChange: (updater) => {
      const newSorting = typeof updater === 'function' ? updater(table.getState().sorting) : updater;
      if (newSorting.length > 0) {
        onSortingChange(newSorting[0].id, newSorting[0].desc ? 'desc' : 'asc');
      }
    },
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
  });
  
  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <Input
          placeholder="Search..."
          value={globalFilter}
          onChange={(e) => setGlobalFilter(e.target.value)}
          className="max-w-sm"
        />
        <div className="flex gap-2">
          <Button
            variant="outline"
            size="sm"
            onClick={() => onPaginationChange(meta.page - 1, meta.pageSize)}
            disabled={meta.page <= 1}
          >
            Previous
          </Button>
          <span className="text-sm">Page {meta.page} of {meta.totalPages}</span>
          <Button
            variant="outline"
            size="sm"
            onClick={() => onPaginationChange(meta.page + 1, meta.pageSize)}
            disabled={meta.page >= meta.totalPages}
          >
            Next
          </Button>
        </div>
      </div>
      
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => (
                  <TableHead key={header.id}>
                    {header.isPlaceholder ? null : (
                      <div
                        className={header.column.getCanSort() ? 'cursor-pointer' : ''}
                        onClick={header.column.getToggleSortingHandler()}
                      >
                        {flexRender(header.column.columnDef.header, header.getContext())}
                        {header.column.getIsSorted() ? (header.column.getIsSorted() === 'asc' ? ' ↑' : ' ↓') : null}
                      </div>
                    )}
                  </TableHead>
                ))}
              </TableRow>
            ))}
          </TableHeader>
          
          <TableBody>
            {isLoading ? (
              Array.from({ length: meta.pageSize }).map((_, i) => (
                <TableRow key={i}>
                  {columns.map((_, j) => (
                    <TableCell key={j}><Skeleton className="h-4 w-full" /></TableCell>
                  ))}
                </TableRow>
              ))
            ) : table.getRowModel().rows?.length ? (
              table.getRowModel().rows.map((row) => (
                <TableRow
                  key={row.id}
                  data-state={row.getIsSelected() && 'selected'}
                  className={row.original.isOptimistic ? 'opacity-50' : ''}
                >
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(cell.column.columnDef.cell, cell.getContext())}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell colSpan={columns.length} className="h-24 text-center">
                  No results.
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}
```

## Audit Trail System

```typescript
// lib/audit.ts
import { PrismaClient } from '@prisma/client';

interface AuditLogEntry {
  action: 'CREATE' | 'UPDATE' | 'DELETE' | 'VIEW';
  entityType: string;
  entityId: string;
  userId: string;
  oldValues: Record<string, unknown> | null;
  newValues: Record<string, unknown> | null;
  ipAddress?: string;
  userAgent?: string;
}

export async function createAuditLog(
  tx: PrismaClient,
  entry: AuditLogEntry
): Promise<void> {
  await tx.auditLog.create({
    data: { ...entry, timestamp: new Date() },
  });
}

export async function getAuditTrail(entityType: string, entityId: string) {
  return await prisma.auditLog.findMany({
    where: { entityType, entityId },
    orderBy: { timestamp: 'desc' },
    include: {
      user: { select: { id: true, name: true, email: true } },
    },
  });
}
```

## Best Practices

1. **Always use database transactions** for multi-table operations
2. **Implement soft deletes** for data integrity (deletedAt, deletedBy fields)
3. **Use optimistic updates** for better perceived performance
4. **Implement proper error boundaries** at component level
5. **Add loading states** for all async operations
6. **Use debouncing** for search inputs (300ms default)
7. **Implement optimistic locking** for concurrent edit scenarios
8. **Add retry logic** for failed mutations with exponential backoff
9. **Use query key factories** for predictable cache management
10. **Implement proper pagination** with cursor-based for large datasets

## Security Checklist

- [ ] All actions verify authentication
- [ ] Authorization checks on every operation
- [ ] Input validation with Zod schemas
- [ ] Rate limiting on mutation actions
- [ ] SQL injection prevention (use ORM)
- [ ] XSS prevention (sanitize outputs)
- [ ] CSRF protection (Next.js handles this)
- [ ] Audit logging for all mutations
- [ ] Soft deletes instead of hard deletes
- [ ] Field-level permissions for sensitive data
