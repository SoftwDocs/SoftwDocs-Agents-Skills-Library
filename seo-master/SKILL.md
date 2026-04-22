---
name: seo-master
description: Enterprise SEO automation for Next.js. Dynamic metadata generation, OpenGraph images, sitemaps, structured data (JSON-LD), Core Web Vitals optimization, and search ranking strategies.
tags: [seo, nextjs, metadata, opengraph, sitemap, structured-data, json-ld]
version: 2.0.0
author: SoftwDocs
---

# SEO Master

## Overview

A comprehensive SEO skill for Next.js 14+ applications. Covers automatic metadata generation, dynamic OpenGraph images, structured data implementation, sitemap automation, and search engine optimization best practices for maximum visibility.

## SEO Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    SEO OPTIMIZATION LAYERS                    │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 1: TECHNICAL SEO                                         │
│  ├─ Dynamic metadata (Next.js 14)                                 │
│  ├─ OpenGraph & Twitter Cards                                   │
│  ├─ Canonical URLs                                              │
│  ├─ robots.txt & sitemap.xml                                    │
│  └─ Structured data (JSON-LD)                                     │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 2: CONTENT SEO                                             │
│  ├─ Semantic HTML structure                                       │
│  ├─ Heading hierarchy (H1-H6)                                   │
│  ├─ Internal linking strategy                                     │
│  ├─ Image alt text optimization                                   │
│  └─ URL structure optimization                                    │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 3: PERFORMANCE SEO                                         │
│  ├─ Core Web Vitals optimization                                  │
│  ├─ Mobile-first indexing                                         │
│  ├─ AMP (if applicable)                                           │
│  └─ Lazy loading implementation                                   │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 4: SOCIAL & SHARING                                        │
│  ├─ Dynamic OG image generation                                   │
│  ├─ Social preview optimization                                   │
│  └─ Rich snippets configuration                                   │
└─────────────────────────────────────────────────────────────────┘
```

## 1. Dynamic Metadata Generation

### Base Metadata Configuration
```typescript
// app/layout.tsx
import type { Metadata, Viewport } from 'next';

export const viewport: Viewport = {
  width: 'device-width',
  initialScale: 1,
  maximumScale: 5,
  themeColor: [
    { media: '(prefers-color-scheme: light)', color: '#ffffff' },
    { media: '(prefers-color-scheme: dark)', color: '#0f172a' },
  ],
};

export const metadata: Metadata = {
  metadataBase: new URL('https://example.com'),
  title: {
    default: 'Your Brand - Tagline Here',
    template: '%s | Your Brand',
  },
  description: 'Your comprehensive description here (150-160 characters optimal)',
  keywords: ['keyword1', 'keyword2', 'keyword3'],
  authors: [{ name: 'Your Name', url: 'https://example.com/about' }],
  creator: 'Your Brand',
  publisher: 'Your Brand',
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://example.com',
    siteName: 'Your Brand',
    title: 'Your Brand - Tagline',
    description: 'Your comprehensive description',
    images: [
      {
        url: '/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'Your Brand Description',
      },
    ],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Your Brand - Tagline',
    description: 'Your comprehensive description',
    images: ['/twitter-image.jpg'],
    creator: '@yourhandle',
    site: '@yourhandle',
  },
  alternates: {
    canonical: 'https://example.com',
    languages: {
      'en-US': 'https://example.com',
      'es-ES': 'https://example.com/es',
      'fr-FR': 'https://example.com/fr',
    },
  },
  verification: {
    google: 'your-google-verification-code',
    yandex: 'your-yandex-verification-code',
    yahoo: 'your-yahoo-verification-code',
    other: {
      'msvalidate.01': 'your-bing-verification-code',
    },
  },
  category: 'technology',
  classification: 'business',
  referrer: 'origin-when-cross-origin',
  formatDetection: {
    telephone: false,
    date: false,
    address: false,
    email: false,
  },
  itunes: {
    appId: 'your-app-id',
  },
  other: {
    'facebook-domain-verification': 'your-facebook-code',
  },
};
```

### Dynamic Page Metadata
```typescript
// app/blog/[slug]/page.tsx
import type { Metadata, ResolvingMetadata } from 'next';
import { getPostBySlug } from '@/lib/posts';

interface Props {
  params: { slug: string };
}

export async function generateMetadata(
  { params }: Props,
  parent: ResolvingMetadata
): Promise<Metadata> {
  const post = await getPostBySlug(params.slug);
  
  if (!post) {
    return {
      title: 'Post Not Found',
      robots: { index: false, follow: false },
    };
  }

  const previousImages = (await parent).openGraph?.images || [];
  const ogImage = post.coverImage || '/default-og.jpg';

  return {
    title: post.title,
    description: post.excerpt,
    keywords: post.tags,
    authors: [{ name: post.author.name, url: post.author.url }],
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author.name],
      tags: post.tags,
      images: [
        {
          url: ogImage,
          width: 1200,
          height: 630,
          alt: post.title,
        },
        ...previousImages,
      ],
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [ogImage],
    },
    alternates: {
      canonical: `https://example.com/blog/${params.slug}`,
    },
    robots: {
      index: post.published,
      follow: true,
      'max-image-preview': 'large',
    },
  };
}

// Product Page Metadata
export async function generateProductMetadata(
  product: Product
): Promise<Metadata> {
  const price = formatPrice(product.price);
  
  return {
    title: `${product.name} - Buy Online | Your Store`,
    description: `Buy ${product.name} for ${price}. ${product.shortDescription}`,
    keywords: [product.name, product.category, 'buy online', 'best price'],
    openGraph: {
      title: product.name,
      description: product.shortDescription,
      type: 'product',
      images: product.images.map(img => ({
        url: img.url,
        width: img.width,
        height: img.height,
        alt: img.alt,
      })),
    },
    other: {
      // Additional product-specific meta tags
      'product:price:amount': String(product.price),
      'product:price:currency': 'USD',
      'product:availability': product.inStock ? 'in stock' : 'out of stock',
      'product:condition': 'new',
    },
  };
}
```

## 2. Dynamic OG Image Generation

### OG Image Route Handler
```typescript
// app/api/og/route.tsx
import { ImageResponse } from 'next/og';
import { NextRequest } from 'next/server';

export const runtime = 'edge';

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    
    const title = searchParams.get('title')?.slice(0, 100) || 'Your Brand';
    const description = searchParams.get('description')?.slice(0, 200) || '';
    const type = searchParams.get('type') || 'website';
    
    // Load fonts
    const interSemiBold = await fetch(
      new URL('../../../public/fonts/Inter-SemiBold.ttf', import.meta.url)
    ).then((res) => res.arrayBuffer());

    return new ImageResponse(
      (
        <div
          style={{
            height: '100%',
            width: '100%',
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'flex-start',
            justifyContent: 'center',
            backgroundImage: 'linear-gradient(to bottom right, #0f172a, #1e293b)',
            padding: '60px',
          }}
        >
          {/* Logo */}
          <div style={{ display: 'flex', alignItems: 'center', marginBottom: '40px' }}>
            <div
              style={{
                width: '60px',
                height: '60px',
                borderRadius: '12px',
                background: 'linear-gradient(135deg, #3b82f6, #8b5cf6)',
              }}
            />
            <span
              style={{
                marginLeft: '20px',
                fontSize: '32px',
                color: 'white',
                fontWeight: 600,
              }}
            >
              Your Brand
            </span>
          </div>

          {/* Type Badge */}
          <div
            style={{
              display: 'flex',
              backgroundColor: 'rgba(59, 130, 246, 0.2)',
              borderRadius: '8px',
              padding: '8px 16px',
              marginBottom: '20px',
            }}
          >
            <span style={{ color: '#60a5fa', fontSize: '18px', textTransform: 'uppercase' }}>
              {type}
            </span>
          </div>

          {/* Title */}
          <h1
            style={{
              fontSize: '64px',
              fontWeight: 700,
              color: 'white',
              lineHeight: 1.2,
              margin: '0 0 20px 0',
              maxWidth: '900px',
            }}
          >
            {title}
          </h1>

          {/* Description */}
          {description && (
            <p
              style={{
                fontSize: '32px',
                color: '#94a3b8',
                lineHeight: 1.4,
                maxWidth: '800px',
                margin: 0,
              }}
            >
              {description}
            </p>
          )}

          {/* URL */}
          <div
            style={{
              position: 'absolute',
              bottom: '60px',
              right: '60px',
              display: 'flex',
              alignItems: 'center',
            }}
          >
            <span style={{ color: '#64748b', fontSize: '24px' }}>
              example.com
            </span>
          </div>
        </div>
      ),
      {
        width: 1200,
        height: 630,
        fonts: [
          {
            name: 'Inter',
            data: interSemiBold,
            style: 'normal',
            weight: 600,
          },
        ],
      }
    );
  } catch (e) {
    console.log(`${e.message}`);
    return new Response('Failed to generate image', { status: 500 });
  }
}
```

### Using Dynamic OG Images
```typescript
// In your metadata export
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug);
  
  const ogImageUrl = `/api/og?title=${encodeURIComponent(post.title)}&description=${encodeURIComponent(post.excerpt)}&type=article`;

  return {
    openGraph: {
      images: [{ url: ogImageUrl, width: 1200, height: 630 }],
    },
    twitter: {
      images: [ogImageUrl],
    },
  };
}
```

## 3. Structured Data (JSON-LD)

### Organization Schema
```typescript
// components/json-ld/organization.tsx
export function OrganizationSchema() {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Your Brand',
    url: 'https://example.com',
    logo: 'https://example.com/logo.png',
    sameAs: [
      'https://twitter.com/yourhandle',
      'https://linkedin.com/company/yourbrand',
      'https://github.com/yourorg',
    ],
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+1-234-567-8900',
      contactType: 'customer service',
      availableLanguage: ['English', 'Spanish'],
    },
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}
```

### Article Schema
```typescript
// components/json-ld/article.tsx
interface ArticleSchemaProps {
  title: string;
  description: string;
  author: { name: string; url?: string };
  publishedAt: string;
  modifiedAt?: string;
  image: string;
  url: string;
  tags: string[];
}

export function ArticleSchema({
  title,
  description,
  author,
  publishedAt,
  modifiedAt,
  image,
  url,
  tags,
}: ArticleSchemaProps) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: title,
    description,
    image,
    datePublished: publishedAt,
    dateModified: modifiedAt || publishedAt,
    author: {
      '@type': 'Person',
      name: author.name,
      url: author.url,
    },
    publisher: {
      '@type': 'Organization',
      name: 'Your Brand',
      logo: {
        '@type': 'ImageObject',
        url: 'https://example.com/logo.png',
      },
    },
    mainEntityOfPage: {
      '@type': 'WebPage',
      '@id': url,
    },
    keywords: tags.join(', '),
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}
```

### Product Schema (E-commerce)
```typescript
// components/json-ld/product.tsx
interface ProductSchemaProps {
  name: string;
  description: string;
  image: string;
  sku: string;
  brand: string;
  price: number;
  currency: string;
  availability: 'InStock' | 'OutOfStock' | 'PreOrder';
  rating?: { value: number; count: number };
  reviews?: Array<{
    author: string;
    date: string;
    rating: number;
    text: string;
  }>;
}

export function ProductSchema(props: ProductSchemaProps) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: props.name,
    image: props.image,
    description: props.description,
    sku: props.sku,
    brand: {
      '@type': 'Brand',
      name: props.brand,
    },
    offers: {
      '@type': 'Offer',
      url: `https://example.com/products/${props.sku}`,
      priceCurrency: props.currency,
      price: props.price,
      availability: `https://schema.org/${props.availability}`,
      priceValidUntil: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toISOString(),
    },
    aggregateRating: props.rating && {
      '@type': 'AggregateRating',
      ratingValue: props.rating.value,
      reviewCount: props.rating.count,
    },
    review: props.reviews?.map((review) => ({
      '@type': 'Review',
      author: { '@type': 'Person', name: review.author },
      datePublished: review.date,
      reviewRating: {
        '@type': 'Rating',
        ratingValue: review.rating,
      },
      reviewBody: review.text,
    })),
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}
```

### BreadcrumbList Schema
```typescript
// components/json-ld/breadcrumbs.tsx
interface Breadcrumb {
  name: string;
  item: string;
}

export function BreadcrumbSchema({ items }: { items: Breadcrumb[] }) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: items.map((item, index) => ({
      '@type': 'ListItem',
      position: index + 1,
      name: item.name,
      item: item.item,
    })),
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}

// Usage in page
export default function ProductPage({ product }) {
  const breadcrumbs = [
    { name: 'Home', item: 'https://example.com' },
    { name: 'Products', item: 'https://example.com/products' },
    { name: product.category, item: `https://example.com/categories/${product.categorySlug}` },
    { name: product.name, item: `https://example.com/products/${product.slug}` },
  ];

  return (
    <>
      <BreadcrumbSchema items={breadcrumbs} />
      <ProductSchema {...product} />
      {/* ... */}
    </>
  );
}
```

## 4. Sitemap Generation

### Dynamic Sitemap
```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next';
import { getAllPosts } from '@/lib/posts';
import { getAllProducts } from '@/lib/products';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://example.com';

  // Static routes
  const staticRoutes = [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: 'daily' as const,
      priority: 1,
    },
    {
      url: `${baseUrl}/about`,
      lastModified: new Date(),
      changeFrequency: 'monthly' as const,
      priority: 0.8,
    },
    {
      url: `${baseUrl}/blog`,
      lastModified: new Date(),
      changeFrequency: 'daily' as const,
      priority: 0.9,
    },
    {
      url: `${baseUrl}/products`,
      lastModified: new Date(),
      changeFrequency: 'daily' as const,
      priority: 0.9,
    },
  ];

  // Dynamic blog posts
  const posts = await getAllPosts();
  const postRoutes = posts.map((post) => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: new Date(post.updatedAt),
    changeFrequency: 'weekly' as const,
    priority: 0.7,
  }));

  // Dynamic products
  const products = await getAllProducts();
  const productRoutes = products.map((product) => ({
    url: `${baseUrl}/products/${product.slug}`,
    lastModified: new Date(product.updatedAt),
    changeFrequency: 'daily' as const,
    priority: 0.8,
    images: product.images?.map((img) => ({
      loc: img.url,
      caption: img.alt,
      title: product.name,
    })),
  }));

  return [...staticRoutes, ...postRoutes, ...productRoutes];
}
```

### Multi-Sitemap (Large Sites)
```typescript
// app/sitemap.xml/route.ts
export const dynamic = 'force-dynamic';

export async function GET() {
  const sitemapIndex = `<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://example.com/sitemap-pages.xml</loc>
    <lastmod>${new Date().toISOString()}</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://example.com/sitemap-posts.xml</loc>
    <lastmod>${new Date().toISOString()}</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://example.com/sitemap-products.xml</loc>
    <lastmod>${new Date().toISOString()}</lastmod>
  </sitemap>
</sitemapindex>`;

  return new Response(sitemapIndex, {
    headers: {
      'Content-Type': 'application/xml',
    },
  });
}
```

## 5. robots.txt

```typescript
// app/robots.ts
import { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: [
          '/api/',
          '/admin/',
          '/_next/',
          '/private/',
          '/drafts/',
          '/*.json$',
          '/*.xml$',
        ],
      },
      {
        userAgent: 'Googlebot',
        allow: '/',
        disallow: ['/nogooglebot/'],
      },
      {
        userAgent: 'Googlebot-Image',
        allow: ['/images/', '/uploads/'],
      },
    ],
    sitemap: 'https://example.com/sitemap.xml',
    host: 'https://example.com',
  };
}
```

## 6. SEO-Friendly Components

### Semantic HTML Structure
```typescript
// components/seo/article-layout.tsx
export function ArticleLayout({
  children,
  title,
  author,
  publishedAt,
  category,
}: ArticleLayoutProps) {
  return (
    <article itemScope itemType="https://schema.org/Article">
      <header>
        <nav aria-label="Breadcrumb">
          {/* Breadcrumb navigation */}
        </nav>
        
        <h1 itemProp="headline" className="text-4xl font-bold">
          {title}
        </h1>
        
        <div className="flex items-center gap-4 text-sm text-gray-600">
          <address itemProp="author" itemScope itemType="https://schema.org/Person">
            By <a href={author.url} itemProp="url">
              <span itemProp="name">{author.name}</span>
            </a>
          </address>
          
          <time itemProp="datePublished" dateTime={publishedAt}>
            {formatDate(publishedAt)}
          </time>
          
          <span itemProp="articleSection">{category}</span>
        </div>
      </header>

      <div itemProp="articleBody" className="prose lg:prose-xl">
        {children}
      </div>

      <footer>
        {/* Tags, related articles, etc. */}
      </footer>
    </article>
  );
}
```

### Image SEO Component
```typescript
// components/seo/optimized-image.tsx
import Image from 'next/image';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  caption?: string;
  priority?: boolean;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  caption,
  priority = false,
}: OptimizedImageProps) {
  return (
    <figure itemScope itemType="https://schema.org/ImageObject">
      <Image
        src={src}
        alt={alt}
        width={width}
        height={height}
        priority={priority}
        itemProp="contentUrl"
      />
      {caption && (
        <figcaption itemProp="description">{caption}</figcaption>
      )}
      <meta itemProp="name" content={alt} />
    </figure>
  );
}
```

## 7. SEO Hooks & Utilities

### useCanonicalUrl Hook
```typescript
// hooks/use-canonical-url.ts
import { usePathname } from 'next/navigation';
import { useMemo } from 'react';

export function useCanonicalUrl(queryParams?: string[]) {
  const pathname = usePathname();
  
  return useMemo(() => {
    const url = new URL(pathname, 'https://example.com');
    
    // Only keep specified query params for canonical
    if (queryParams) {
      const params = new URLSearchParams(window.location.search);
      const filtered = new URLSearchParams();
      queryParams.forEach((key) => {
        const value = params.get(key);
        if (value) filtered.set(key, value);
      });
      url.search = filtered.toString();
    }
    
    return url.toString();
  }, [pathname, queryParams]);
}
```

### SEO Utils
```typescript
// lib/seo/utils.ts
export function generateMetaDescription(text: string, maxLength = 160): string {
  // Remove HTML tags
  const cleanText = text.replace(/<[^>]*>/g, '');
  
  // Truncate with ellipsis
  if (cleanText.length <= maxLength) return cleanText;
  
  return cleanText.slice(0, maxLength - 3).trim() + '...';
}

export function slugify(text: string): string {
  return text
    .toLowerCase()
    .trim()
    .replace(/[^\w\s-]/g, '')
    .replace(/[\s_-]+/g, '-')
    .replace(/^-+|-+$/g, '');
}

export function generateKeywords(title: string, content: string): string[] {
  // Extract keywords from title and content
  const words = [...title.split(' '), ...content.split(' ')];
  const frequency: Record<string, number> = {};
  
  words.forEach((word) => {
    const clean = word.toLowerCase().replace(/[^a-z]/g, '');
    if (clean.length > 3) {
      frequency[clean] = (frequency[clean] || 0) + 1;
    }
  });
  
  // Return top 10 keywords
  return Object.entries(frequency)
    .sort(([, a], [, b]) => b - a)
    .slice(0, 10)
    .map(([word]) => word);
}
```

## 8. International SEO (i18n)

```typescript
// app/[locale]/layout.tsx
import { locales, defaultLocale } from '@/lib/i18n';

export async function generateMetadata({ params: { locale } }): Promise<Metadata> {
  const t = await getTranslations(locale);
  
  return {
    title: t('metadata.title'),
    description: t('metadata.description'),
    alternates: {
      canonical: `/${locale}`,
      languages: Object.fromEntries(
        locales.map((loc) => [loc, `/${loc}`])
      ),
    },
    openGraph: {
      locale: locale.replace('-', '_'),
    },
  };
}
```

## SEO Checklist

### Technical SEO
- [ ] Dynamic metadata on all pages
- [ ] OG images generated dynamically
- [ ] Sitemap generated automatically
- [ ] robots.txt configured
- [ ] Canonical URLs on all pages
- [ ] Structured data (JSON-LD) implemented
- [ ] HTTPS enforced
- [ ] Mobile-friendly design
- [ ] Fast page load times

### Content SEO
- [ ] Unique title tags (< 60 chars)
- [ ] Meta descriptions (< 160 chars)
- [ ] Semantic HTML structure
- [ ] Proper heading hierarchy (H1 → H6)
- [ ] Image alt text optimized
- [ ] Internal linking strategy
- [ ] URL structure optimized
- [ ] Content quality and relevance

### Performance
- [ ] Core Web Vitals optimized
- [ ] Images optimized (WebP/AVIF)
- [ ] Lazy loading implemented
- [ ] Font optimization (next/font)
- [ ] Minification enabled

### Monitoring
- [ ] Google Search Console connected
- [ ] Google Analytics implemented
- [ ] Rank tracking set up
- [ ] Regular SEO audits scheduled
