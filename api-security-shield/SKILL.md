---
name: api-security-shield
description: Enterprise-grade API security for Next.js and FastAPI. JWT authentication, rate limiting, CORS, input sanitization, SQL injection prevention, and OWASP compliance.
tags: [security, api, jwt, rate-limiting, cors, owasp, authentication]
version: 2.0.0
author: EticTech
---

# API Security Shield

## Overview

A comprehensive security skill for building fortress-grade APIs. Implements defense-in-depth strategies covering authentication, authorization, input validation, rate limiting, and OWASP Top 10 protection.

## Security Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEFENSE IN DEPTH LAYERS                      │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 1: PERIMETER                                             │
│  ├─ HTTPS/TLS enforcement                                       │
│  ├─ CORS configuration                                          │
│  ├─ Security headers (CSP, HSTS, X-Frame-Options)              │
│  └─ DDoS protection (rate limiting)                             │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 2: AUTHENTICATION                                        │
│  ├─ JWT with RS256 (asymmetric)                                 │
│  ├─ Refresh token rotation                                      │
│  ├─ Session management                                          │
│  └─ Multi-factor authentication (MFA)                         │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 3: AUTHORIZATION                                         │
│  ├─ RBAC (Role-Based Access Control)                          │
│  ├─ ABAC (Attribute-Based Access Control)                     │
│  ├─ Resource-level permissions                                  │
│  └─ API key management                                          │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 4: INPUT VALIDATION                                      │
│  ├─ Schema validation (Zod)                                     │
│  ├─ SQL injection prevention                                    │
│  ├─ XSS prevention                                                │
│  ├─ CSRF tokens                                                   │
│  └─ File upload validation                                        │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 5: DATA PROTECTION                                       │
│  ├─ Encryption at rest (AES-256)                              │
│  ├─ Encryption in transit (TLS 1.3)                           │
│  ├─ PII masking                                                   │
│  └─ Audit logging                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 1. JWT Authentication (Next.js + FastAPI)

### JWT Configuration
```typescript
// lib/auth/jwt.ts
import jwt from 'jsonwebtoken';
import { cookies } from 'next/headers';

// Use RS256 for production (asymmetric keys)
const ACCESS_TOKEN_SECRET = process.env.JWT_ACCESS_SECRET!;
const REFRESH_TOKEN_SECRET = process.env.JWT_REFRESH_SECRET!;
const ACCESS_TOKEN_EXPIRY = '15m';
const REFRESH_TOKEN_EXPIRY = '7d';

export interface TokenPayload {
  sub: string;      // user id
  email: string;
  roles: string[];
  iat: number;
  exp: number;
}

export function generateTokens(user: { id: string; email: string; roles: string[] }) {
  const accessToken = jwt.sign(
    { sub: user.id, email: user.email, roles: user.roles },
    ACCESS_TOKEN_SECRET,
    { algorithm: 'HS256', expiresIn: ACCESS_TOKEN_EXPIRY }
  );

  const refreshToken = jwt.sign(
    { sub: user.id, type: 'refresh' },
    REFRESH_TOKEN_SECRET,
    { algorithm: 'HS256', expiresIn: REFRESH_TOKEN_EXPIRY }
  );

  return { accessToken, refreshToken };
}

export function verifyAccessToken(token: string): TokenPayload {
  return jwt.verify(token, ACCESS_TOKEN_SECRET, { algorithms: ['HS256'] }) as TokenPayload;
}

export function verifyRefreshToken(token: string): { sub: string; type: string } {
  return jwt.verify(token, REFRESH_TOKEN_SECRET, { algorithms: ['HS256'] }) as any;
}

// Server-side auth helper
export async function getServerSession() {
  const cookieStore = await cookies();
  const token = cookieStore.get('access_token')?.value;
  
  if (!token) return null;
  
  try {
    return verifyAccessToken(token);
  } catch {
    return null;
  }
}
```

### Protected API Route
```typescript
// app/api/protected/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { verifyAccessToken } from '@/lib/auth/jwt';
import { rateLimit } from '@/lib/rate-limit';

export async function GET(request: NextRequest) {
  try {
    // 1. Rate limiting
    const rateLimitResult = await rateLimit.check(request, { max: 100, window: 60 });
    if (!rateLimitResult.success) {
      return NextResponse.json(
        { error: 'Rate limit exceeded' },
        { status: 429, headers: { 'Retry-After': String(rateLimitResult.retryAfter) } }
      );
    }

    // 2. Token extraction
    const authHeader = request.headers.get('authorization');
    if (!authHeader?.startsWith('Bearer ')) {
      return NextResponse.json(
        { error: 'Missing authorization header' },
        { status: 401 }
      );
    }

    const token = authHeader.slice(7);

    // 3. Token verification
    const payload = verifyAccessToken(token);

    // 4. Authorization check
    if (!payload.roles.includes('admin')) {
      return NextResponse.json(
        { error: 'Insufficient permissions' },
        { status: 403 }
      );
    }

    // 5. Process request
    const data = await fetchSensitiveData(payload.sub);
    
    return NextResponse.json({ data, user: payload });

  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return NextResponse.json(
        { error: 'Token expired', code: 'TOKEN_EXPIRED' },
        { status: 401 }
      );
    }
    if (error.name === 'JsonWebTokenError') {
      return NextResponse.json(
        { error: 'Invalid token' },
        { status: 401 }
      );
    }
    
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

## 2. Rate Limiting

### Redis-Based Rate Limiter
```typescript
// lib/rate-limit.ts
import { Redis } from '@upstash/redis';
import { NextRequest } from 'next/server';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

interface RateLimitOptions {
  max: number;        // Max requests
  window: number;     // Time window in seconds
  keyPrefix?: string;
}

export async function rateLimit(
  request: NextRequest,
  options: RateLimitOptions
) {
  const ip = request.ip ?? '127.0.0.1';
  const key = `${options.keyPrefix || 'ratelimit'}:${ip}`;
  const window = options.window;
  
  const current = await redis.incr(key);
  
  if (current === 1) {
    await redis.expire(key, window);
  }
  
  const ttl = await redis.ttl(key);
  
  return {
    success: current <= options.max,
    limit: options.max,
    remaining: Math.max(0, options.max - current),
    reset: ttl,
    retryAfter: current > options.max ? ttl : 0,
  };
}

// Sliding window rate limiter (more accurate)
export async function slidingWindowRateLimit(
  identifier: string,
  options: RateLimitOptions
) {
  const key = `sliding:${options.keyPrefix || 'ratelimit'}:${identifier}`;
  const now = Date.now();
  const windowMs = options.window * 1000;
  
  // Remove old entries
  await redis.zremrangebyscore(key, 0, now - windowMs);
  
  // Count current entries
  const count = await redis.zcard(key);
  
  if (count >= options.max) {
    const oldest = await redis.zrange(key, 0, 0, { withScores: true });
    const resetTime = Math.ceil((oldest[0]?.score + windowMs - now) / 1000);
    
    return {
      success: false,
      limit: options.max,
      remaining: 0,
      reset: resetTime,
      retryAfter: resetTime,
    };
  }
  
  // Add current request
  await redis.zadd(key, { score: now, member: `${now}-${Math.random()}` });
  await redis.expire(key, options.window);
  
  return {
    success: true,
    limit: options.max,
    remaining: options.max - count - 1,
    reset: options.window,
    retryAfter: 0,
  };
}
```

## 3. CORS Configuration

### Next.js Middleware CORS
```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const allowedOrigins = [
  'https://app.example.com',
  'https://admin.example.com',
  process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : '',
].filter(Boolean);

export function middleware(request: NextRequest) {
  const origin = request.headers.get('origin') ?? '';
  const isAllowed = allowedOrigins.includes(origin);
  
  // Handle preflight
  if (request.method === 'OPTIONS') {
    const response = new NextResponse(null, { status: 204 });
    
    if (isAllowed) {
      response.headers.set('Access-Control-Allow-Origin', origin);
    }
    response.headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
    response.headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Requested-With');
    response.headers.set('Access-Control-Allow-Credentials', 'true');
    response.headers.set('Access-Control-Max-Age', '86400');
    
    return response;
  }
  
  const response = NextResponse.next();
  
  if (isAllowed) {
    response.headers.set('Access-Control-Allow-Origin', origin);
  }
  response.headers.set('Access-Control-Allow-Credentials', 'true');
  
  // Security headers
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-XSS-Protection', '1; mode=block');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  // CSP Header
  response.headers.set(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self' https://api.example.com;"
  );
  
  return response;
}

export const config = {
  matcher: ['/api/:path*', '/((?!_next/static|_next/image|favicon.ico).*)'],
};
```

## 4. Input Validation & Sanitization

### Zod Schema Validation
```typescript
// lib/validation/schemas.ts
import { z } from 'zod';

// User registration schema
export const registerSchema = z.object({
  email: z
    .string()
    .min(1, 'Email is required')
    .email('Invalid email format')
    .transform((email) => email.toLowerCase().trim()),
  password: z
    .string()
    .min(12, 'Password must be at least 12 characters')
    .max(128, 'Password must not exceed 128 characters')
    .regex(/[A-Z]/, 'Must contain uppercase letter')
    .regex(/[a-z]/, 'Must contain lowercase letter')
    .regex(/[0-9]/, 'Must contain number')
    .regex(/[^A-Za-z0-9]/, 'Must contain special character'),
  name: z
    .string()
    .min(2, 'Name must be at least 2 characters')
    .max(100, 'Name must not exceed 100 characters')
    .regex(/^[a-zA-Z\s'-]+$/, 'Name contains invalid characters')
    .transform((name) => name.trim()),
});

// SQL injection prevention - Sanitize search inputs
export const searchSchema = z.object({
  query: z
    .string()
    .max(100)
    .transform((q) => q.replace(/[<>\"']/g, ''))  // Remove dangerous chars
    .transform((q) => q.trim()),
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
});

// File upload validation
export const fileUploadSchema = z.object({
  file: z
    .instanceof(File)
    .refine((f) => f.size <= 5 * 1024 * 1024, 'Max 5MB file size')
    .refine(
      (f) => ['image/jpeg', 'image/png', 'image/webp', 'application/pdf'].includes(f.type),
      'Only images and PDFs allowed'
    ),
});

// API key validation
export const apiKeySchema = z.object({
  key: z.string().regex(/^pk_[a-zA-Z0-9]{32}$/, 'Invalid API key format'),
});
```

### Validation Middleware
```typescript
// lib/validation/middleware.ts
import { z } from 'zod';
import { NextRequest, NextResponse } from 'next/server';

type ValidationSchemas = {
  body?: z.ZodSchema;
  query?: z.ZodSchema;
  params?: z.ZodSchema;
};

export function validate(schemas: ValidationSchemas) {
  return async (request: NextRequest) => {
    try {
      const result: Record<string, unknown> = {};

      if (schemas.body) {
        const body = await request.json();
        result.body = schemas.body.parse(body);
      }

      if (schemas.query) {
        const searchParams = Object.fromEntries(request.nextUrl.searchParams);
        result.query = schemas.query.parse(searchParams);
      }

      return { success: true, data: result };
    } catch (error) {
      if (error instanceof z.ZodError) {
        return {
          success: false,
          error: {
            message: 'Validation failed',
            issues: error.issues.map((issue) => ({
              path: issue.path.join('.'),
              message: issue.message,
            })),
          },
        };
      }
      throw error;
    }
  };
}
```

## 5. SQL Injection Prevention

### Safe Database Queries (Prisma)
```typescript
// lib/db/queries.ts
import { prisma } from './client';

// ❌ NEVER DO THIS - Vulnerable to SQL injection
// const users = await prisma.$queryRaw`SELECT * FROM users WHERE email = '${email}'`;

// ✅ SAFE - Parameterized query
export async function findUserByEmail(email: string) {
  return await prisma.user.findUnique({
    where: { email },
  });
}

// ✅ SAFE - Using Prisma's query engine
export async function searchUsers(searchTerm: string) {
  return await prisma.user.findMany({
    where: {
      OR: [
        { name: { contains: searchTerm, mode: 'insensitive' } },
        { email: { contains: searchTerm, mode: 'insensitive' } },
      ],
    },
  });
}

// ✅ SAFE - Raw query with tagged template (Prisma handles escaping)
export async function getUserStats(userId: string) {
  // Prisma's $queryRawTagged automatically escapes parameters
  const result = await prisma.$queryRaw`
    SELECT COUNT(*) as count FROM posts WHERE author_id = ${userId}
  `;
  return result;
}

// ✅ SAFE - Using query builder
export async function complexSearch(filters: {
  status?: string;
  minDate?: Date;
  maxDate?: Date;
}) {
  return await prisma.post.findMany({
    where: {
      AND: [
        filters.status && { status: filters.status },
        filters.minDate && { createdAt: { gte: filters.minDate } },
        filters.maxDate && { createdAt: { lte: filters.maxDate } },
      ].filter(Boolean),
    },
  });
}
```

## 6. XSS Prevention

### Output Sanitization
```typescript
// lib/security/xss.ts
import DOMPurify from 'isomorphic-dompurify';

// Sanitize HTML content
export function sanitizeHtml(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p', 'br', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: [],
  });
}

// Sanitize for plain text (remove all HTML)
export function stripHtml(dirty: string): string {
  return DOMPurify.sanitize(dirty, { ALLOWED_TAGS: [] });
}

// React component with sanitized content
export function SafeHtml({ content }: { content: string }) {
  const sanitized = sanitizeHtml(content);
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}
```

## 7. CSRF Protection

### CSRF Token Implementation
```typescript
// lib/csrf.ts
import { randomBytes } from 'crypto';
import { cookies } from 'next/headers';

const CSRF_COOKIE_NAME = 'csrf_token';
const CSRF_HEADER_NAME = 'x-csrf-token';

export function generateCsrfToken(): string {
  return randomBytes(32).toString('hex');
}

export async function setCsrfCookie() {
  const token = generateCsrfToken();
  const cookieStore = await cookies();
  
  cookieStore.set(CSRF_COOKIE_NAME, token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    path: '/',
    maxAge: 3600, // 1 hour
  });
  
  return token;
}

export async function validateCsrfToken(request: Request): Promise<boolean> {
  const cookieStore = await cookies();
  const cookieToken = cookieStore.get(CSRF_COOKIE_NAME)?.value;
  const headerToken = request.headers.get(CSRF_HEADER_NAME);
  
  if (!cookieToken || !headerToken) return false;
  
  // Timing-safe comparison
  try {
    return crypto.timingSafeEqual(
      Buffer.from(cookieToken),
      Buffer.from(headerToken)
    );
  } catch {
    return false;
  }
}
```

## 8. Security Headers Configuration

### Next.js Config
```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-DNS-Prefetch-Control',
            value: 'on',
          },
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload',
          },
          {
            key: 'X-Frame-Options',
            value: 'SAMEORIGIN',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin',
          },
          {
            key: 'Permissions-Policy',
            value: 'camera=(), microphone=(), geolocation=(self)',
          },
          {
            key: 'Content-Security-Policy',
            value: "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self' https://*.example.com;",
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

## 9. API Key Management

### Secure API Key Generation
```typescript
// lib/api-keys.ts
import { createHash, randomBytes } from 'crypto';

export interface ApiKeyConfig {
  name: string;
  userId: string;
  permissions: string[];
  expiresAt?: Date;
}

export function generateApiKey(): { key: string; hash: string } {
  // Generate random key
  const key = `pk_${randomBytes(32).toString('hex')}`;
  
  // Store only hash
  const hash = createHash('sha256').update(key).digest('hex');
  
  return { key, hash };
}

export async function createApiKey(config: ApiKeyConfig) {
  const { key, hash } = generateApiKey();
  
  await prisma.apiKey.create({
    data: {
      name: config.name,
      userId: config.userId,
      keyHash: hash,
      permissions: config.permissions,
      expiresAt: config.expiresAt,
      lastUsedAt: null,
    },
  });
  
  // Return key only once - it cannot be retrieved later
  return { key, prefix: key.slice(0, 8) };
}

export async function validateApiKey(key: string): Promise<ApiKey | null> {
  const hash = createHash('sha256').update(key).digest('hex');
  
  const apiKey = await prisma.apiKey.findFirst({
    where: {
      keyHash: hash,
      isActive: true,
      OR: [
        { expiresAt: null },
        { expiresAt: { gt: new Date() } },
      ],
    },
  });
  
  if (apiKey) {
    // Update last used
    await prisma.apiKey.update({
      where: { id: apiKey.id },
      data: { lastUsedAt: new Date() },
    });
  }
  
  return apiKey;
}
```

## 10. FastAPI Security Implementation

### Python Security Setup
```python
# main.py
from fastapi import FastAPI, Depends, HTTPException, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
import jwt
from datetime import datetime, timedelta

app = FastAPI()
security = HTTPBearer()
limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.example.com"],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)

# Trusted hosts
app.add_middleware(TrustedHostMiddleware, allowed_hosts=["api.example.com"])

# JWT verification
def verify_token(credentials: HTTPAuthorizationCredentials = Security(security)):
    token = credentials.credentials
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")

@app.get("/protected")
@limiter.limit("100/minute")
async def protected_route(request: Request, user: dict = Depends(verify_token)):
    return {"message": "Access granted", "user": user}
```

## Security Checklist

### Authentication & Authorization
- [ ] JWT with RS256 for production
- [ ] Token expiration and refresh mechanism
- [ ] Role-based access control (RBAC)
- [ ] API key rotation capability
- [ ] Session invalidation on logout

### Input Security
- [ ] All inputs validated with Zod/schemas
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (output encoding)
- [ ] CSRF tokens for state-changing operations
- [ ] File upload type and size validation

### Infrastructure
- [ ] HTTPS everywhere
- [ ] Security headers configured
- [ ] Rate limiting on all endpoints
- [ ] CORS properly configured
- [ ] DDoS protection enabled

### Monitoring & Logging
- [ ] Failed authentication attempts logged
- [ ] Suspicious activity alerts
- [ ] Audit trail for sensitive operations
- [ ] PII data encrypted at rest
- [ ] Regular security audits scheduled
