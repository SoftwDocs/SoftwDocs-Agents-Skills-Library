---
name: tailwind-glass-ui
description: Create premium glassmorphism UI components with Tailwind CSS. Advanced translucent effects, backdrop filters, and modern frosted glass designs for high-end applications.
tags: [tailwind, glassmorphism, ui, design, css, modern]
version: 2.0.0
author: SoftwDocs
---

# Tailwind Glass UI

## Overview

A comprehensive skill for creating stunning glassmorphism (frosted glass) UI components using Tailwind CSS. Glassmorphism combines transparency, blur effects, and subtle borders to create modern, premium interfaces that feel both light and sophisticated.

## Core Glassmorphism Principles

```
┌─────────────────────────────────────────────────────────────────┐
│                    GLASSMORPHISM FOUNDATIONS                    │
├─────────────────────────────────────────────────────────────────┤
│  1. TRANSPARENCY    │  background-opacity: 10-25%               │
│  2. BACKDROP BLUR   │  backdrop-blur: sm (4px) to 3xl (64px)   │
│  3. SUBTLE BORDERS  │  border-white/20 or border-white/10     │
│  4. SOFT SHADOWS    │  shadow-lg with spread                  │
│  5. SATURATION      │  backdrop-saturate for vibrancy           │
└─────────────────────────────────────────────────────────────────┘
```

## Essential Tailwind Classes

### Basic Glass Card
```tsx
<div className="
  bg-white/10              /* Semi-transparent white */
  backdrop-blur-xl          /* Heavy blur (24px) */
  backdrop-saturate-150     /* Slightly increase saturation */
  border border-white/20    /* Subtle white border */
  rounded-2xl               /* Rounded corners */
  shadow-2xl                /* Deep shadow for depth */
  shadow-black/10           /* Subtle dark shadow */
">
  <h2 className="text-white font-semibold">Glass Card</h2>
  <p className="text-white/80">Content with frosted glass effect</p>
</div>
```

### Gradient Glass Background
```tsx
<div className="
  relative
  bg-gradient-to-br
  from-purple-500/20        /* Gradient with low opacity */
  via-blue-500/10
  to-pink-500/20
  backdrop-blur-2xl
  border border-white/10
  rounded-3xl
">
  <div className="absolute inset-0 bg-white/5 rounded-3xl" /> {/* Inner glow */}
  <div className="relative z-10 p-6">
    <h3>Gradient Glass</h3>
  </div>
</div>
```

## Advanced Glass Components

### 1. Navigation Bar (Glass Header)
```tsx
// components/glass-navbar.tsx
'use client';

import { useState, useEffect } from 'react';
import { cn } from '@/lib/utils';

export function GlassNavbar() {
  const [scrolled, setScrolled] = useState(false);

  useEffect(() => {
    const handleScroll = () => setScrolled(window.scrollY > 20);
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  return (
    <nav
      className={cn(
        'fixed top-0 left-0 right-0 z-50',
        'transition-all duration-500 ease-out',
        scrolled
          ? 'bg-white/10 backdrop-blur-xl backdrop-saturate-150 border-b border-white/10 shadow-lg shadow-black/5'
          : 'bg-transparent'
      )}
    >
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex items-center justify-between h-16">
          {/* Logo */}
          <div className="flex items-center space-x-2">
            <div className="w-8 h-8 rounded-lg bg-gradient-to-br from-cyan-400 to-blue-600" />
            <span className="text-white font-bold text-xl">Brand</span>
          </div>

          {/* Nav Links */}
          <div className="hidden md:flex items-center space-x-1">
            {['Features', 'Pricing', 'About', 'Contact'].map((item) => (
              <a
                key={item}
                href={`#${item.toLowerCase()}`}
                className="
                  px-4 py-2 rounded-lg
                  text-white/80 hover:text-white
                  hover:bg-white/10
                  transition-all duration-200
                "
              >
                {item}
              </a>
            ))}
          </div>

          {/* CTA Button */}
          <button className="
            px-6 py-2 rounded-full
            bg-white/20 hover:bg-white/30
            backdrop-blur-md
            border border-white/30
            text-white font-medium
            transition-all duration-200
            hover:scale-105
            active:scale-95
          ">
            Get Started
          </button>
        </div>
      </div>
    </nav>
  );
}
```

### 2. Glass Card with Hover Effects
```tsx
// components/glass-card.tsx
import { cn } from '@/lib/utils';

interface GlassCardProps {
  children: React.ReactNode;
  className?: string;
  variant?: 'default' | 'gradient' | 'dark';
  hover?: boolean;
}

export function GlassCard({
  children,
  className,
  variant = 'default',
  hover = true,
}: GlassCardProps) {
  const variants = {
    default: '
      bg-white/10
      border-white/20
    ',
    gradient: '
      bg-gradient-to-br from-white/20 to-white/5
      border-white/30
    ',
    dark: '
      bg-black/30
      backdrop-blur-2xl
      border-white/10
    ',
  };

  return (
    <div
      className={cn(
        'relative overflow-hidden',
        'rounded-2xl',
        'backdrop-blur-xl backdrop-saturate-150',
        variants[variant],
        hover && '
          hover:bg-white/15
          hover:scale-[1.02]
          hover:shadow-2xl hover:shadow-black/20
          transition-all duration-300 ease-out
        ',
        'group',
        className
      )}
    >
      {/* Shine Effect on Hover */}
      <div className="
        absolute inset-0
        bg-gradient-to-r from-transparent via-white/10 to-transparent
        -translate-x-full group-hover:translate-x-full
        transition-transform duration-1000 ease-out
      " />
      
      {/* Corner Glow */}
      <div className="
        absolute -top-20 -right-20
        w-40 h-40
        bg-gradient-to-br from-cyan-400/30 to-transparent
        rounded-full blur-3xl
        group-hover:opacity-100 opacity-50
        transition-opacity duration-500
      " />

      <div className="relative z-10 p-6">{children}</div>
    </div>
  );
}

// Usage
<GlassCard variant="gradient" className="w-80">
  <h3 className="text-xl font-semibold text-white mb-2">Premium Plan</h3>
  <p className="text-white/70 mb-4">Everything you need to get started</p>
  <div className="flex items-baseline gap-1">
    <span className="text-4xl font-bold text-white">$29</span>
    <span className="text-white/60">/month</span>
  </div>
</GlassCard>
```

### 3. Glass Modal/Dialog
```tsx
// components/glass-modal.tsx
'use client';

import { useEffect } from 'react';
import { X } from 'lucide-react';
import { cn } from '@/lib/utils';

interface GlassModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
  size?: 'sm' | 'md' | 'lg' | 'xl';
}

export function GlassModal({
  isOpen,
  onClose,
  title,
  children,
  size = 'md',
}: GlassModalProps) {
  useEffect(() => {
    if (isOpen) {
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = 'unset';
    }
    return () => { document.body.style.overflow = 'unset'; };
  }, [isOpen]);

  if (!isOpen) return null;

  const sizes = {
    sm: 'max-w-md',
    md: 'max-w-lg',
    lg: 'max-w-2xl',
    xl: 'max-w-4xl',
  };

  return (
    <div
      className="fixed inset-0 z-50 flex items-center justify-center p-4"
      onClick={onClose}
    >
      {/* Backdrop */}
      <div className="absolute inset-0 bg-black/60 backdrop-blur-sm" />

      {/* Modal */}
      <div
        className={cn(
          'relative w-full',
          sizes[size],
          'animate-in fade-in zoom-in-95 duration-200'
        )}
        onClick={(e) => e.stopPropagation()}
      >
        <div className="
          relative
          bg-white/10
          backdrop-blur-2xl backdrop-saturate-150
          border border-white/20
          rounded-3xl
          shadow-2xl shadow-black/30
          overflow-hidden
        ">
          {/* Header */}
          <div className="
            flex items-center justify-between
            px-6 py-4
            border-b border-white/10
          ">
            <h2 className="text-xl font-semibold text-white">{title}</h2>
            <button
              onClick={onClose}
              className="
                p-2 rounded-lg
                text-white/60 hover:text-white
                hover:bg-white/10
                transition-colors
              "
            >
              <X className="w-5 h-5" />
            </button>
          </div>

          {/* Content */}
          <div className="p-6">
            {children}
          </div>

          {/* Decorative Glow */}
          <div className="
            absolute -top-40 -right-40
            w-80 h-80
            bg-gradient-to-br from-purple-500/30 to-cyan-500/10
            rounded-full blur-3xl
            pointer-events-none
          " />
        </div>
      </div>
    </div>
  );
}
```

### 4. Glass Input Fields
```tsx
// components/glass-input.tsx
import { cn } from '@/lib/utils';
import { forwardRef } from 'react';

export interface GlassInputProps
  extends React.InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
  icon?: React.ReactNode;
}

export const GlassInput = forwardRef<HTMLInputElement, GlassInputProps>(
  ({ className, label, error, icon, ...props }, ref) => {
    return (
      <div className="w-full">
        {label && (
          <label className="block text-sm font-medium text-white/80 mb-2">
            {label}
          </label>
        )}
        <div className="relative">
          {icon && (
            <div className="absolute left-4 top-1/2 -translate-y-1/2 text-white/40">
              {icon}
            </div>
          )}
          <input
            ref={ref}
            className={cn(
              'w-full',
              'px-4 py-3',
              icon && 'pl-12',
              'bg-white/5',
              'backdrop-blur-sm',
              'border border-white/10',
              'rounded-xl',
              'text-white placeholder:text-white/40',
              'focus:outline-none focus:ring-2 focus:ring-white/20 focus:border-white/30',
              'transition-all duration-200',
              error && 'border-red-400/50 focus:ring-red-400/20',
              className
            )}
            {...props}
          />
        </div>
        {error && (
          <p className="mt-1 text-sm text-red-300">{error}</p>
        )}
      </div>
    );
  }
);
GlassInput.displayName = 'GlassInput';
```

### 5. Glass Button Variants
```tsx
// components/glass-button.tsx
import { cn } from '@/lib/utils';
import { ButtonHTMLAttributes, forwardRef } from 'react';

interface GlassButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
}

export const GlassButton = forwardRef<HTMLButtonElement, GlassButtonProps>(
  ({ className, variant = 'primary', size = 'md', isLoading, children, ...props }, ref) => {
    const variants = {
      primary: `
        bg-gradient-to-r from-cyan-500/80 to-blue-600/80
        hover:from-cyan-500 hover:to-blue-600
        text-white
        border border-white/20
        shadow-lg shadow-cyan-500/25
      `,
      secondary: `
        bg-white/10
        hover:bg-white/20
        text-white
        border border-white/20
        shadow-lg shadow-black/10
      `,
      ghost: `
        bg-transparent
        hover:bg-white/10
        text-white/80 hover:text-white
        border border-transparent hover:border-white/10
      `,
      danger: `
        bg-gradient-to-r from-red-500/80 to-pink-600/80
        hover:from-red-500 hover:to-pink-600
        text-white
        border border-white/20
        shadow-lg shadow-red-500/25
      `,
    };

    const sizes = {
      sm: 'px-4 py-2 text-sm',
      md: 'px-6 py-3 text-base',
      lg: 'px-8 py-4 text-lg',
    };

    return (
      <button
        ref={ref}
        className={cn(
          'relative overflow-hidden',
          'inline-flex items-center justify-center gap-2',
          'font-medium rounded-xl',
          'backdrop-blur-md backdrop-saturate-150',
          'transition-all duration-200',
          'hover:scale-105 active:scale-95',
          'disabled:opacity-50 disabled:cursor-not-allowed',
          variants[variant],
          sizes[size],
          className
        )}
        disabled={isLoading || props.disabled}
        {...props}
      >
        {/* Shine effect */}
        <span className="
          absolute inset-0
          bg-gradient-to-r from-transparent via-white/20 to-transparent
          -translate-x-full
          group-hover:animate-shine
        " />
        
        {isLoading ? (
          <>
            <span className="w-4 h-4 border-2 border-white/30 border-t-white rounded-full animate-spin" />
            <span>Loading...</span>
          </>
        ) : (
          children
        )}
      </button>
    );
  }
);
GlassButton.displayName = 'GlassButton';
```

## Glass Layout Patterns

### Hero Section with Glass Elements
```tsx
// sections/glass-hero.tsx
export function GlassHero() {
  return (
    <section className="relative min-h-screen overflow-hidden">
      {/* Background */}
      <div className="absolute inset-0 bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900" />
      
      {/* Animated Background Shapes */}
      <div className="absolute inset-0 overflow-hidden">
        <div className="absolute top-1/4 -left-20 w-96 h-96 bg-purple-500/30 rounded-full blur-3xl animate-pulse" />
        <div className="absolute bottom-1/4 -right-20 w-96 h-96 bg-cyan-500/30 rounded-full blur-3xl animate-pulse delay-1000" />
        <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[600px] h-[600px] bg-blue-500/20 rounded-full blur-3xl" />
      </div>

      {/* Content */}
      <div className="relative z-10 max-w-7xl mx-auto px-4 py-32 flex items-center">
        <div className="grid lg:grid-cols-2 gap-12 items-center">
          {/* Text Content */}
          <div className="space-y-6">
            <div className="
              inline-flex items-center gap-2
              px-4 py-2 rounded-full
              bg-white/10 backdrop-blur-md
              border border-white/20
            ">
              <span className="w-2 h-2 rounded-full bg-green-400 animate-pulse" />
              <span className="text-sm text-white/80">Now in Beta</span>
            </div>

            <h1 className="text-5xl md:text-7xl font-bold text-white leading-tight">
              Build the
              <span className="text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 to-purple-400">
                Future
              </span>
            </h1>

            <p className="text-xl text-white/70 max-w-lg">
              Create stunning interfaces with our glassmorphism design system.
            </p>

            <div className="flex gap-4">
              <GlassButton size="lg">Get Started</GlassButton>
              <GlassButton variant="secondary" size="lg">Learn More</GlassButton>
            </div>
          </div>

          {/* Glass Cards Grid */}
          <div className="relative">
            <div className="absolute inset-0 bg-gradient-to-r from-cyan-500/20 to-purple-500/20 rounded-3xl blur-3xl" />
            
            <div className="relative grid gap-4">
              <GlassCard className="transform rotate-3 translate-x-8">
                <div className="flex items-center gap-4">
                  <div className="w-12 h-12 rounded-xl bg-gradient-to-br from-cyan-400 to-blue-500" />
                  <div>
                    <p className="text-white font-medium">Analytics</p>
                    <p className="text-white/60 text-sm">Real-time insights</p>
                  </div>
                </div>
              </GlassCard>

              <GlassCard className="transform -rotate-2 -translate-x-4">
                <div className="flex items-center gap-4">
                  <div className="w-12 h-12 rounded-xl bg-gradient-to-br from-purple-400 to-pink-500" />
                  <div>
                    <p className="text-white font-medium">Security</p>
                    <p className="text-white/60 text-sm">Enterprise-grade</p>
                  </div>
                </div>
              </GlassCard>

              <GlassCard className="transform rotate-1 translate-x-12">
                <div className="flex items-center gap-4">
                  <div className="w-12 h-12 rounded-xl bg-gradient-to-br from-green-400 to-emerald-500" />
                  <div>
                    <p className="text-white font-medium">Performance</p>
                    <p className="text-white/60 text-sm">Lightning fast</p>
                  </div>
                </div>
              </GlassCard>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
}
```

## Advanced Effects

### Animated Glass Background
```tsx
// components/animated-glass-bg.tsx
'use client';

export function AnimatedGlassBackground() {
  return (
    <div className="fixed inset-0 -z-10 overflow-hidden">
      {/* Base Gradient */}
      <div className="absolute inset-0 bg-gradient-to-br from-slate-950 via-purple-950 to-slate-950" />
      
      {/* Animated Orbs */}
      <div className="absolute top-0 left-0 w-full h-full">
        {[...Array(5)].map((_, i) => (
          <div
            key={i}
            className="absolute rounded-full blur-3xl opacity-30 animate-float"
            style={{
              width: `${200 + i * 100}px`,
              height: `${200 + i * 100}px`,
              background: `linear-gradient(135deg, 
                hsl(${260 + i * 30}, 70%, 50%), 
                hsl(${200 + i * 20}, 70%, 50%))`,
              left: `${10 + i * 20}%`,
              top: `${10 + i * 15}%`,
              animationDelay: `${i * 2}s`,
              animationDuration: `${10 + i * 5}s`,
            }}
          />
        ))}
      </div>

      {/* Noise Texture Overlay */}
      <div 
        className="absolute inset-0 opacity-20"
        style={{
          backgroundImage: `url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E")`,
        }}
      />
    </div>
  );
}
```

## Tailwind Configuration Extension

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: ['./app/**/*.{js,ts,jsx,tsx,mdx}'],
  theme: {
    extend: {
      backdropBlur: {
        xs: '2px',
      },
      backdropSaturate: {
        125: '1.25',
        150: '1.5',
      },
      animation: {
        'float': 'float 6s ease-in-out infinite',
        'shine': 'shine 2s ease-in-out infinite',
      },
      keyframes: {
        float: {
          '0%, 100%': { transform: 'translateY(0)' },
          '50%': { transform: 'translateY(-20px)' },
        },
        shine: {
          '0%': { transform: 'translateX(-100%)' },
          '100%': { transform: 'translateX(100%)' },
        },
      },
    },
  },
  plugins: [],
};

export default config;
```

## Best Practices

1. **Contrast**: Ensure text has sufficient contrast against the glass background
2. **Layering**: Don't stack too many glass elements - max 3-4 layers
3. **Performance**: Use `will-change: transform` sparingly for animations
4. **Mobile**: Test on mobile - blur effects can be expensive on low-end devices
5. **Fallbacks**: Provide solid color fallbacks for browsers without backdrop-filter support

## Browser Support

| Feature | Support |
|---------|---------|
| `backdrop-filter` | Chrome 76+, Firefox 103+, Safari 9+ |
| `backdrop-blur` | All modern browsers |
| `backdrop-saturate` | Chrome/Safari only |

## Accessibility Considerations

- Ensure 4.5:1 contrast ratio for text on glass backgrounds
- Test with reduced motion preferences
- Provide focus indicators for interactive elements
- Don't rely solely on blur effects for visual hierarchy
