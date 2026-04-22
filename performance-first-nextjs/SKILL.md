---
name: performance-first-nextjs
description: Optimize Next.js applications for Core Web Vitals. Use when building production-ready apps that need 90+ Lighthouse scores.
---

# Performance First Skill

## Instructions
1. **Image Optimization**: Use `next/image` with proper `priority` for LCP elements.
2. **Dynamic Imports**: Use `next/dynamic` for heavy components (e.g., Maps, Charts).
3. **Data Fetching**: Prefer Server Components for data fetching to reduce bundle size.
4. **Caching**: Implement `revalidate` tags for ISR (Incremental Static Regeneration).

## Code Standards
- No unused heavy libraries.
- Use `lucide-react` or SVGs for icons.
- Font optimization via `next/font`.
