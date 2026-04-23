---
name: caching-strategies
description: Complete caching strategies implementation with Redis, Memcached, CDN, browser caching, database query caching, and cache invalidation patterns for high-performance applications.
tags: [caching, redis, memcached, cdn, performance, optimization]
version: 1.0.0
author: SoftwDocs
---

# Caching Strategies

## Overview

A comprehensive skill for implementing effective caching strategies in applications. Covers Redis, Memcached, CDN integration, browser caching, database query caching, cache invalidation patterns, and performance optimization techniques.

## When to Use This Skill

- Reducing database load
- Improving API response times
- Implementing session storage
- Caching expensive computations
- Building high-performance applications
- Reducing infrastructure costs

## Core Technologies

### In-Memory Caching
- Redis for advanced caching
- Memcached for simple caching
- Node-cache for local caching

### CDN Caching
- Cloudflare CDN
- AWS CloudFront
- Fastly CDN

### Browser Caching
- HTTP cache headers
- Service Workers
- Local Storage

## Implementation Patterns

### Pattern 1: Redis Caching

Advanced caching with Redis:

```typescript
// cache/redis.ts
import { createClient, RedisClientType } from 'redis';

class RedisCache {
  private client: RedisClientType;

  constructor() {
    this.client = createClient({
      url: process.env.REDIS_URL!,
    });
    
    this.client.on('error', (error) => {
      console.error('Redis error:', error);
    });
  }

  async connect() {
    await this.client.connect();
  }

  async disconnect() {
    await this.client.disconnect();
  }

  async get<T>(key: string): Promise<T | null> {
    const value = await this.client.get(key);
    return value ? JSON.parse(value) : null;
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    const serialized = JSON.stringify(value);
    
    if (ttl) {
      await this.client.setEx(key, ttl, serialized);
    } else {
      await this.client.set(key, serialized);
    }
  }

  async del(key: string): Promise<void> {
    await this.client.del(key);
  }

  async delPattern(pattern: string): Promise<void> {
    const keys = await this.client.keys(pattern);
    if (keys.length > 0) {
      await this.client.del(keys);
    }
  }

  async exists(key: string): Promise<boolean> {
    return (await this.client.exists(key)) === 1;
  }

  async ttl(key: string): Promise<number> {
    return await this.client.ttl(key);
  }

  async incr(key: string): Promise<number> {
    return await this.client.incr(key);
  }

  async expire(key: string, seconds: number): Promise<void> {
    await this.client.expire(key, seconds);
  }
}

export const redisCache = new RedisCache();

// Cache decorator
export function Cache(ttl: number = 3600) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      const cacheKey = `${target.constructor.name}:${propertyKey}:${JSON.stringify(args)}`;
      
      // Try to get from cache
      const cached = await redisCache.get(cacheKey);
      if (cached !== null) {
        return cached;
      }
      
      // Execute original method
      const result = await originalMethod.apply(this, args);
      
      // Cache the result
      await redisCache.set(cacheKey, result, ttl);
      
      return result;
    };
    
    return descriptor;
  };
}

// Usage
class UserService {
  @Cache(300) // 5 minutes
  async getUser(id: string) {
    return await fetchUserFromDatabase(id);
  }
}
```

### Pattern 2: Multi-Level Caching

Implementing cache hierarchy:

```typescript
// cache/multi-level.ts
import LRUCache from 'lru-cache';

const localCache = new LRUCache<string, any>({
  max: 1000,
  ttl: 1000 * 60 * 5, // 5 minutes
});

class MultiLevelCache {
  async get<T>(key: string): Promise<T | null> {
    // Level 1: Local memory cache
    const localValue = localCache.get(key);
    if (localValue !== undefined) {
      return localValue;
    }
    
    // Level 2: Redis cache
    const redisValue = await redisCache.get<T>(key);
    if (redisValue !== null) {
      // Populate local cache
      localCache.set(key, redisValue);
      return redisValue;
    }
    
    return null;
  }

  async set(key: string, value: any, ttl: number = 3600): Promise<void> {
    // Set in both caches
    localCache.set(key, value);
    await redisCache.set(key, value, ttl);
  }

  async del(key: string): Promise<void> {
    localCache.delete(key);
    await redisCache.del(key);
  }

  async invalidatePattern(pattern: string): Promise<void> {
    // Clear local cache
    localCache.clear();
    // Clear Redis cache
    await redisCache.delPattern(pattern);
  }
}

export const multiLevelCache = new MultiLevelCache();
```

### Pattern 3: Database Query Caching

Caching database queries:

```typescript
// cache/query-cache.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function getCachedUser(id: string) {
  const cacheKey = `user:${id}`;
  
  // Try cache first
  const cached = await redisCache.get(cacheKey);
  if (cached) {
    return cached;
  }
  
  // Query database
  const user = await prisma.user.findUnique({
    where: { id },
  });
  
  if (user) {
    // Cache for 1 hour
    await redisCache.set(cacheKey, user, 3600);
  }
  
  return user;
}

export async function invalidateUserCache(id: string) {
  await redisCache.del(`user:${id}`);
  await redisCache.delPattern(`user:${id}:*`);
}

// Cache invalidation middleware
export function withCacheInvalidation<T extends { id: string }>(
  model: string
) {
  return async (data: T) => {
    // Perform database operation
    const result = await prisma[model].create({ data });
    
    // Invalidate cache
    await redisCache.del(`${model}:${result.id}`);
    
    return result;
  };
}

// Usage
export async function createUser(data: any) {
  return withCacheInvalidation('user')(data);
}
```

### Pattern 4: CDN Caching

Implementing CDN caching strategies:

```typescript
// middleware/cache-headers.ts
import { NextResponse } from 'next/server';

export function setCacheHeaders(
  response: NextResponse,
  options: {
    maxAge?: number;
    sMaxAge?: number;
    staleWhileRevalidate?: number;
    mustRevalidate?: boolean;
  }
) {
  const {
    maxAge = 3600,
    sMaxAge = 86400,
    staleWhileRevalidate = 60,
    mustRevalidate = false,
  } = options;

  const cacheControl = [
    `public`,
    `max-age=${maxAge}`,
    `s-maxage=${sMaxAge}`,
    `stale-while-revalidate=${staleWhileRevalidate}`,
    mustRevalidate ? 'must-revalidate' : '',
  ]
    .filter(Boolean)
    .join(', ');

  response.headers.set('Cache-Control', cacheControl);
  response.headers.set('CDN-Cache-Control', cacheControl);
  
  return response;
}

// API route with CDN caching
// app/api/products/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { setCacheHeaders } from '@/middleware/cache-headers';

export async function GET(request: NextRequest) {
  const products = await getProducts();
  
  const response = NextResponse.json(products);
  
  // Cache for 1 hour, CDN for 1 day
  return setCacheHeaders(response, {
    maxAge: 3600,
    sMaxAge: 86400,
    staleWhileRevalidate: 300,
  });
}

// Static asset caching
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/static/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
    ];
  },
};
```

### Pattern 5: Cache Invalidation Strategies

Implementing cache invalidation patterns:

```typescript
// cache/invalidation.ts
export class CacheInvalidator {
  // Time-based expiration
  async invalidateByTime(key: string, ttl: number) {
    await redisCache.set(key, null, ttl);
  }

  // Event-based invalidation
  async invalidateOnEvent(event: string, data: any) {
    switch (event) {
      case 'user_updated':
        await this.invalidateUser(data.userId);
        break;
      case 'post_created':
        await this.invalidatePosts();
        break;
      case 'post_deleted':
        await this.invalidatePost(data.postId);
        break;
    }
  }

  async invalidateUser(userId: string) {
    await redisCache.del(`user:${userId}`);
    await redisCache.delPattern(`user:${userId}:*`);
    await redisCache.delPattern(`users:*`);
  }

  async invalidatePost(postId: string) {
    await redisCache.del(`post:${postId}`);
    await redisCache.delPattern(`post:${postId}:*`);
    await redisCache.delPattern(`posts:*`);
  }

  async invalidatePosts() {
    await redisCache.delPattern('posts:*');
  }

  // Tag-based invalidation
  async setWithTags(key: string, value: any, tags: string[], ttl?: number) {
    await redisCache.set(key, value, ttl);
    
    // Add key to each tag
    for (const tag of tags) {
      await redisCache.sadd(`tag:${tag}`, key);
    }
  }

  async invalidateByTag(tag: string) {
    const keys = await redisCache.smembers(`tag:${tag}`);
    
    if (keys.length > 0) {
      await redisCache.del(keys);
      await redisCache.del(`tag:${tag}`);
    }
  }

  // Write-through cache
  async writeThrough(key: string, value: any, ttl: number) {
    // Write to cache first
    await redisCache.set(key, value, ttl);
    
    // Write to database
    await writeToDatabase(key, value);
  }

  // Write-back cache
  async writeBack(key: string, value: any, ttl: number) {
    // Write to cache only
    await redisCache.set(key, value, ttl);
    
    // Schedule write to database
    setTimeout(async () => {
      await writeToDatabase(key, value);
    }, 1000);
  }
}

export const cacheInvalidator = new CacheInvalidator();
```

## Cache Strategies

### Cache-Aside (Lazy Loading)

```typescript
async function getWithCacheAside(key: string) {
  // Check cache
  let value = await redisCache.get(key);
  
  if (value === null) {
    // Cache miss - load from database
    value = await loadFromDatabase(key);
    
    // Populate cache
    await redisCache.set(key, value, 3600);
  }
  
  return value;
}
```

### Read-Through

```typescript
async function getWithReadThrough(key: string) {
  // Cache handles loading on miss
  return await redisCache.get(key) || await loadFromDatabase(key);
}
```

### Write-Through

```typescript
async function setWithWriteThrough(key: string, value: any) {
  // Write to cache and database synchronously
  await Promise.all([
    redisCache.set(key, value, 3600),
    writeToDatabase(key, value),
  ]);
}
```

### Write-Behind (Write-Back)

```typescript
async function setWithWriteBehind(key: string, value: any) {
  // Write to cache immediately
  await redisCache.set(key, value, 3600);
  
  // Write to database asynchronously
  setImmediate(async () => {
    await writeToDatabase(key, value);
  });
}
```

## Best Practices

### ✅ Do

- Use appropriate TTL values
- Implement cache invalidation
- Monitor cache hit rates
- Use cache keys consistently
- Implement fallback logic
- Use compression for large values
- Monitor Redis memory usage
- Implement cache warming

### ❌ Don't

- Cache everything
- Use infinite TTL
- Ignore cache invalidation
- Forget to monitor performance
- Use inconsistent key formats
- Skip error handling
- Cache sensitive data without encryption
- Ignore memory limits

## Monitoring

```typescript
// cache/monitoring.ts
export class CacheMonitor {
  async getStats() {
    const info = await redisCache.client.info('stats');
    const memory = await redisCache.client.info('memory');
    
    return {
      info,
      memory,
    };
  }

  async getHitRate() {
    const stats = await this.getStats();
    const keyspaceHits = stats.info.match(/keyspace_hits:(\d+)/)?.[1] || '0';
    const keyspaceMisses = stats.info.match(/keyspace_misses:(\d+)/)?.[1] || '0';
    
    const hits = parseInt(keyspaceHits);
    const misses = parseInt(keyspaceMisses);
    const total = hits + misses;
    
    return total > 0 ? (hits / total) * 100 : 0;
  }
}

export const cacheMonitor = new CacheMonitor();
```

## Resources

- [Redis Documentation](https://redis.io/docs/)
- [Memcached Documentation](https://memcached.org/)
- [CDN Best Practices](https://developers.google.com/web/fundamentals/performance/http-caching)
