---
name: performance-first-nextjs
description: Build production-grade Next.js 14+ applications achieving 95+ Lighthouse scores. Covers Core Web Vitals optimization, bundle analysis, image optimization, streaming, partial prerendering, and advanced caching strategies.
tags: [nextjs, performance, lighthouse, core-web-vitals, optimization, production]
version: 2.0.0
author: SoftwDocs
---

# Performance-First Next.js

## Overview

A comprehensive skill for building blazing-fast Next.js 14+ applications that consistently achieve 95+ Lighthouse scores. Covers every aspect of performance optimization from initial load to runtime efficiency.

## Core Web Vitals Targets

| Metric | Target | Description |
|--------|--------|-------------|
| **LCP** (Largest Contentful Paint) | < 2.5s | Main content loaded |
| **INP** (Interaction to Next Paint) | < 200ms | Visual response to interaction |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Visual stability |
| **TTFB** (Time to First Byte) | < 600ms | Server response time |
| **FCP** (First Contentful Paint) | < 1.8s | First visible content |

## Architecture for Performance

```
┌─────────────────────────────────────────────────────────────────┐
│                    PERFORMANCE OPTIMIZATION STACK              │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 1: ASSET OPTIMIZATION                                    │
│  ├─ next/image (WebP, AVIF, responsive sizes)                  │
│  ├─ next/font (subsetting, preload)                            │
│  ├─ Compression (brotli, gzip)                                   │
│  └─ Critical CSS extraction                                    │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 2: CODE OPTIMIZATION                                     │
│  ├─ Tree shaking & dead code elimination                         │
│  ├─ Dynamic imports (route + component level)                    │
│  ├─ Module federation for micro-frontends                      │
│  └─ SWC compilation (no Babel)                                   │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 3: RENDERING STRATEGY                                    │
│  ├─ Server Components (default)                                │
│  ├─ Partial Prerendering (PPR)                                 │
│  ├─ Streaming with Suspense                                    │
│  ├─ Edge Runtime for API routes                                │
│  └─ ISR + On-demand Revalidation                               │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 4: DATA & CACHING                                        │
│  ├─ React.cache() for deduplication                            │
│  ├─ fetch cache tags & revalidate                              │
│  ├─ unstable_cache for expensive operations                    │
│  ├─ Prisma Accelerate (connection pooling)                     │
│  └─ Redis/CDN for API responses                              │
└─────────────────────────────────────────────────────────────────┘
```

## 1. Image Optimization (Critical for LCP)

### Next/Image Configuration
```typescript
// next.config.js
const nextConfig = {
  images: {
    formats: ['image/webp', 'image/avif'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    minimumCacheTTL: 60,
    dangerouslyAllowSVG: true,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },
};
module.exports = nextConfig;
```

### Hero Image (LCP Element)
```typescript
import Image from 'next/image';
import { getPlaiceholder } from 'plaiceholder'; // For blur placeholder

// Server Component - Fetches blur data
export async function HeroImage() {
  const src = '/hero-image.jpg';
  const buffer = await fetch(src).then(async (res) =>
    Buffer.from(await res.arrayBuffer())
  );
  const { base64 } = await getPlaiceholder(buffer);

  return (
    <Image
      src={src}
      alt="Hero"
      width={1920}
      height={1080}
      priority              // Preload LCP image
      fetchPriority="high"  // Critical for LCP
      quality={90}
      placeholder="blur"
      blurDataURL={base64}
      sizes="100vw"
      className="object-cover"
    />
  );
}
```

### Responsive Image Gallery
```typescript
function ProductGallery({ images }: { images: string[] }) {
  return (
    <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
      {images.map((src, index) => (
        <Image
          key={src}
          src={src}
          alt={`Product ${index + 1}`}
          width={400}
          height={400}
          loading={index < 4 ? 'eager' : 'lazy'}  // First 4 eager
          quality={75}
          sizes="(max-width: 768px) 50vw, 25vw"
          className="rounded-lg"
        />
      ))}
    </div>
  );
}
```

## 2. Font Optimization (Eliminate Layout Shift)

### Next/Font Strategy
```typescript
// app/layout.tsx
import { Inter, Space_Grotesk } from 'next/font/google';
import localFont from 'next/font/local';

// Google Font with optimization
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',        // Prevent FOIT
  variable: '--font-inter',
  preload: true,
  fallback: ['system-ui', 'sans-serif'],
});

// Variable font for multiple weights
const spaceGrotesk = Space_Grotesk({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-space',
  weight: ['300', '400', '500', '600', '700'],
});

// Local font for custom branding
const calSans = localFont({
  src: '../public/fonts/CalSans-SemiBold.woff2',
  variable: '--font-cal',
  display: 'swap',
});

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={`${inter.variable} ${spaceGrotesk.variable} ${calSans.variable}`}>
      <body className="font-sans antialiased">{children}</body>
    </html>
  );
}
```

### Tailwind Configuration
```typescript
// tailwind.config.ts
export default {
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-inter)', 'system-ui', 'sans-serif'],
        display: ['var(--font-space)', 'var(--font-inter)', 'sans-serif'],
        cal: ['var(--font-cal)', 'var(--font-space)', 'sans-serif'],
      },
    },
  },
};
```

## 3. Dynamic Imports & Code Splitting

### Route-Level Dynamic Import
```typescript
// Heavy component loaded on demand
import dynamic from 'next/dynamic';
import { Skeleton } from '@/components/ui/skeleton';

const HeavyChart = dynamic(
  () => import('@/components/analytics/heavy-chart'),
  {
    loading: () => <Skeleton className="h-[400px] w-full" />,
    ssr: false,  // Disable SSR for client-only libraries
  }
);

const MapComponent = dynamic(
  () => import('@/components/map/interactive-map'),
  {
    loading: () => <div className="h-[500px] bg-muted animate-pulse" />,
  }
);

// Usage in page
export default function AnalyticsPage() {
  return (
    <div>
      <h1>Analytics Dashboard</h1>
      <HeavyChart data={data} />
      <MapComponent locations={locations} />
    </div>
  );
}
```

### Component-Level Dynamic Import (Inside Client Component)
```typescript
'use client';

import { useState } from 'react';
import dynamic from 'next/dynamic';

const EmojiPicker = dynamic(() => import('emoji-picker-react'), {
  ssr: false,
  loading: () => <div>Loading emoji picker...</div>,
});

export function CommentForm() {
  const [showEmoji, setShowEmoji] = useState(false);

  return (
    <form>
      <textarea />
      <button onClick={() => setShowEmoji(!showEmoji)}>
        Add Emoji
      </button>
      {showEmoji && <EmojiPicker onEmojiClick={handleEmoji} />}
    </form>
  );
}
```

## 4. Server Components & Data Fetching

### Server Component Pattern (Preferred)
```typescript
// app/products/page.tsx - Server Component by default
import { Suspense } from 'react';
import { ProductGrid, ProductGridSkeleton } from '@/components/products';

// Parallel data fetching
async function getProducts() {
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600, tags: ['products'] }
  });
  if (!res.ok) throw new Error('Failed to fetch');
  return res.json();
}

async function getCategories() {
  const res = await fetch('https://api.example.com/categories', {
    next: { revalidate: 86400 }  // Cache for 24 hours
  });
  return res.json();
}

export default async function ProductsPage() {
  // Both fetch in parallel (no waterfall)
  const [products, categories] = await Promise.all([
    getProducts(),
    getCategories(),
  ]);

  return (
    <div>
      <CategoryFilter categories={categories} />
      <Suspense fallback={<ProductGridSkeleton />}>
        <ProductGrid products={products} />
      </Suspense>
    </div>
  );
}
```

### React.cache for Deduplication
```typescript
import { cache } from 'react';

// Cache expensive database calls
const getUser = cache(async (id: string) => {
  return await prisma.user.findUnique({
    where: { id },
    include: { posts: true },
  });
});

// Multiple calls, single database query
export async function UserProfile({ id }: { id: string }) {
  const user = await getUser(id);  // First call hits DB
  return <div>{user.name}</div>;
}

export async function UserPosts({ id }: { id: string }) {
  const user = await getUser(id);  // Second call uses cache
  return <PostList posts={user.posts} />;
}
```

## 5. Advanced Caching Strategies

### Fetch Cache Configuration
```typescript
// Time-based revalidation
fetch('https://api.example.com/data', {
  next: { revalidate: 3600 }  // 1 hour
});

// On-demand revalidation with tags
fetch('https://api.example.com/data', {
  next: { tags: ['collection'] }
});

// Never cache (dynamic data)
fetch('https://api.example.com/data', {
  cache: 'no-store'
});

// Force fresh every request
fetch('https://api.example.com/data', {
  next: { revalidate: 0 }
});
```

### unstable_cache for Complex Operations
```typescript
import { unstable_cache } from 'next/cache';

const getCachedAnalytics = unstable_cache(
  async (userId: string) => {
    // Expensive aggregation query
    return await prisma.$queryRaw`
      SELECT * FROM analytics_summary WHERE user_id = ${userId}
    `;
  },
  ['analytics'],  // Cache key prefix
  { revalidate: 300, tags: ['analytics'] }  // 5 minutes
);
```

### Route Handler Caching
```typescript
// app/api/products/route.ts
import { NextResponse } from 'next/server';

export const dynamic = 'force-dynamic';  // No caching
// OR
export const dynamic = 'force-static';   // Static generation
export const revalidate = 60;            // ISR

export async function GET() {
  const products = await fetchProducts();
  return NextResponse.json(products);
}
```

## 6. Streaming & Suspense

### Progressive Loading Pattern
```typescript
import { Suspense } from 'react';
import { Skeleton } from '@/components/ui/skeleton';

function Page() {
  return (
    <main>
      {/* Non-critical, stream immediately */}
      <Header />
      
      {/* Critical content - wait for it */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductDetails id={params.id} />
      </Suspense>
      
      {/* Non-critical, can stream later */}
      <Suspense fallback={<Skeleton className="h-96" />}>
        <RelatedProducts id={params.id} />
      </Suspense>
      
      <Suspense fallback={<Skeleton className="h-64" />}>
        <Reviews id={params.id} />
      </Suspense>
    </main>
  );
}
```

### Loading UI (Route Level)
```typescript
// app/products/loading.tsx
import { Skeleton } from '@/components/ui/skeleton';

export default function Loading() {
  return (
    <div className="space-y-4">
      <Skeleton className="h-8 w-48" />  {/* Title */}
      <div className="grid grid-cols-3 gap-4">
        {Array.from({ length: 6 }).map((_, i) => (
          <Skeleton key={i} className="h-64" />
        ))}
      </div>
    </div>
  );
}
```

## 7. Bundle Optimization

### Analyzing Bundle Size
```bash
# Install analyzer
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer(nextConfig);

# Run analysis
ANALYZE=true npm run build
```

### Tree Shaking Configuration
```typescript
// next.config.js
module.exports = {
  webpack: (config, { isServer }) => {
    // Optimize lodash imports (use lodash-es instead)
    config.resolve.alias = {
      ...config.resolve.alias,
      lodash: 'lodash-es',
    };

    // Remove moment.js locale files (if using moment)
    if (!isServer) {
      config.plugins.push(
        new webpack.IgnorePlugin({
          resourceRegExp: /^\.\/locale$/,
          contextRegExp: /moment$/,
        })
      );
    }

    return config;
  },
};
```

### Import Optimization
```typescript
// ❌ Bad - Imports entire library
import _ from 'lodash';
_.debounce(fn, 300);

// ✅ Good - Imports only what you need
import debounce from 'lodash/debounce';
// or
import { debounce } from 'lodash-es';

// ❌ Bad - Imports all icons
import * as Icons from 'lucide-react';

// ✅ Good - Import specific icons
import { Search, Menu, User } from 'lucide-react';
```

## 8. Edge Runtime & Global Performance

### Edge API Routes
```typescript
// app/api/edge/route.ts
export const runtime = 'edge';
export const preferredRegion = 'iad1';  // US East

export async function GET(request: Request) {
  // Runs at edge, minimal cold start
  return new Response(JSON.stringify({ time: Date.now() }), {
    headers: { 'content-type': 'application/json' },
  });
}
```

### Middleware Optimization
```typescript
// middleware.ts
import { NextResponse } from 'next/server';

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};

export function middleware(request) {
  // Add security headers
  const response = NextResponse.next();
  response.headers.set('X-DNS-Prefetch-Control', 'on');
  return response;
}
```

## 9. Performance Monitoring

### Web Vitals Reporting
```typescript
// app/_components/web-vitals.tsx
'use client';

import { useReportWebVitals } from 'next/web-vitals';

export function WebVitals() {
  useReportWebVitals((metric) => {
    // Send to analytics
    console.log(metric);
    
    // Example: Send to Vercel Analytics
    // vercelAnalytics.track(metric);
    
    // Example: Send to custom endpoint
    fetch('/api/analytics/web-vitals', {
      method: 'POST',
      body: JSON.stringify(metric),
    });
  });

  return null;
}
```

## Performance Checklist

### Before Deployment:
- [ ] All images use `next/image` with proper sizing
- [ ] LCP image has `priority` and `fetchPriority="high"`
- [ ] Fonts use `next/font` with `display: 'swap'`
- [ ] Heavy components use dynamic imports
- [ ] Server Components for data fetching
- [ ] Proper Suspense boundaries with skeletons
- [ ] Fetch caching configured appropriately
- [ ] Bundle analyzer run and reviewed
- [ ] No unused dependencies in package.json
- [ ] Lucide icons instead of full icon libraries
- [ ] CSS optimized (purge unused Tailwind classes)
- [ ] Compression enabled (brotli/gzip)
- [ ] CDN configured for static assets

### Lighthouse Targets:
- **Performance**: 95+
- **Accessibility**: 100
- **Best Practices**: 100
- **SEO**: 100

## Common Anti-Patterns to Avoid

❌ **Using `<img>` instead of `<Image>`** - No optimization
❌ **Loading entire icon libraries** - Massive bundle bloat
❌ **No loading states** - Poor perceived performance
❌ **Client-side data fetching** - Unnecessary JavaScript
❌ **No image placeholders** - Layout shifts
❌ **Synchronous font loading** - FOIT (Flash of Invisible Text)
❌ **Unoptimized videos** - Use lazy loading, compression
❌ **No preconnect hints** - Slow third-party resource loading
