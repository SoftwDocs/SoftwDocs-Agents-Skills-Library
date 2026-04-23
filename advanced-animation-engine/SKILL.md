---
name: advanced-animation-engine
description: Professional animation system using GSAP and Framer Motion for complex, performant web animations. Includes timeline orchestration, scroll-based animations, physics simulations, and micro-interactions.
tags: [animation, gsap, framer-motion, ui, motion, performance]
version: 1.0.0
author: SoftwDocs
---

# Advanced Animation Engine

## Overview

A comprehensive skill for building professional-grade animations in web applications using GSAP and Framer Motion. Covers timeline orchestration, scroll-based animations, physics simulations, micro-interactions, and performance optimization for production use.

## When to Use This Skill

- Building interactive landing pages with complex animations
- Creating scroll-triggered storytelling experiences
- Implementing physics-based UI interactions
- Adding micro-interactions for enhanced UX
- Building animated data visualizations
- Creating animated transitions in SPAs

## Core Technologies

### GSAP (GreenSock Animation Platform)
Industry-standard animation library with:
- Timeline-based sequencing
- ScrollTrigger for scroll animations
- Draggable for interactive elements
- MotionPathPlugin for path animations
- MorphSVGPlugin for shape morphing

### Framer Motion
React animation library with:
- Declarative animation API
- Gesture support (drag, hover, tap)
- Layout animations
- Shared element transitions
- Animation variants

## Implementation Patterns

### Pattern 1: GSAP Timeline Orchestration

Complex multi-step animations with precise timing control:

```typescript
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

// Master timeline for hero section
const heroTimeline = gsap.timeline({
  defaults: { ease: 'power3.out' },
  scrollTrigger: {
    trigger: '.hero-section',
    start: 'top top',
    end: '+=200%',
    scrub: 1,
    pin: true,
  },
});

// Sequence animations
heroTimeline
  .from('.hero-title', {
    y: 100,
    opacity: 0,
    duration: 1,
  })
  .from('.hero-subtitle', {
    y: 50,
    opacity: 0,
    duration: 0.8,
  }, '-=0.5')
  .from('.hero-cta', {
    scale: 0.8,
    opacity: 0,
    duration: 0.6,
  }, '-=0.3')
  .from('.hero-visual', {
    x: 100,
    opacity: 0,
    duration: 1.2,
  }, '-=0.8');

// Parallax background
gsap.to('.hero-bg', {
  yPercent: 50,
  ease: 'none',
  scrollTrigger: {
    trigger: '.hero-section',
    start: 'top top',
    end: 'bottom top',
    scrub: true,
  },
});
```

### Pattern 2: Framer Motion Layout Animations

Smooth layout transitions in React:

```typescript
import { motion, AnimatePresence } from 'framer-motion';

const variants = {
  hidden: { opacity: 0, y: 20 },
  visible: { 
    opacity: 1, 
    y: 0,
    transition: {
      duration: 0.5,
      staggerChildren: 0.1,
    },
  },
  exit: { opacity: 0, y: -20 },
};

export function AnimatedList({ items }: { items: string[] }) {
  return (
    <motion.ul
      variants={variants}
      initial="hidden"
      animate="visible"
      exit="exit"
    >
      <AnimatePresence>
        {items.map((item) => (
          <motion.li
            key={item}
            variants={variants}
            layout
            whileHover={{ scale: 1.02, x: 5 }}
            whileTap={{ scale: 0.98 }}
          >
            {item}
          </motion.li>
        ))}
      </AnimatePresence>
    </motion.ul>
  );
}

// Shared element transitions
export function Card({ id, content }: { id: string; content: string }) {
  return (
    <motion.div
      layoutId={id}
      initial={{ opacity: 0, scale: 0.9 }}
      animate={{ opacity: 1, scale: 1 }}
      exit={{ opacity: 0, scale: 0.9 }}
      transition={{ type: 'spring', stiffness: 300, damping: 30 }}
    >
      {content}
    </motion.div>
  );
}
```

### Pattern 3: Scroll-Based Animations

Parallax and scroll-triggered effects:

```typescript
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

// Parallax sections
function createParallaxEffect() {
  const sections = gsap.utils.toArray('.parallax-section');
  
  sections.forEach((section) => {
    const image = section.querySelector('.parallax-image');
    const text = section.querySelector('.parallax-text');
    
    gsap.to(image, {
      yPercent: 30,
      ease: 'none',
      scrollTrigger: {
        trigger: section,
        start: 'top bottom',
        end: 'bottom top',
        scrub: true,
      },
    });
    
    gsap.from(text, {
      y: 50,
      opacity: 0,
      duration: 1,
      scrollTrigger: {
        trigger: section,
        start: 'top 80%',
        toggleActions: 'play none none reverse',
      },
    });
  });
}

// Reveal animations on scroll
function createRevealAnimations() {
  const reveals = gsap.utils.toArray('.reveal');
  
  reveals.forEach((element) => {
    gsap.from(element, {
      y: 50,
      opacity: 0,
      duration: 1,
      ease: 'power3.out',
      scrollTrigger: {
        trigger: element,
        start: 'top 85%',
        toggleActions: 'play none none reverse',
      },
    });
  });
}

// Text reveal animation
function createTextReveal() {
  const textReveals = gsap.utils.toArray('.text-reveal');
  
  textReveals.forEach((element) => {
    const text = element.textContent || '';
    element.innerHTML = '';
    
    text.split('').forEach((char) => {
      const span = document.createElement('span');
      span.textContent = char === ' ' ? '\u00A0' : char;
      span.className = 'char';
      element.appendChild(span);
    });
    
    gsap.from(element.querySelectorAll('.char'), {
      y: 100,
      opacity: 0,
      duration: 0.5,
      stagger: 0.02,
      ease: 'back.out(1.7)',
      scrollTrigger: {
        trigger: element,
        start: 'top 80%',
      },
    });
  });
}
```

### Pattern 4: Physics-Based Animations

Realistic physics simulations:

```typescript
import { motion, useMotionValue, useSpring, useTransform } from 'framer-motion';

// Spring physics for smooth interactions
export function DraggableCard() {
  const x = useMotionValue(0);
  const y = useMotionValue(0);
  
  const springConfig = { damping: 25, stiffness: 300 };
  const springX = useSpring(x, springConfig);
  const springY = useSpring(y, springConfig);
  
  const rotateX = useTransform(springY, [-200, 200], [25, -25]);
  const rotateY = useTransform(springX, [-200, 200], [-25, 25]);
  
  return (
    <motion.div
      style={{ x: springX, y: springY, rotateX, rotateY }}
      drag
      dragElastic={0.2}
      dragConstraints={{ left: -100, right: 100, top: -100, bottom: 100 }}
      whileHover={{ scale: 1.05 }}
      whileTap={{ scale: 0.95 }}
    >
      <div className="card-content">
        {/* Card content */}
      </div>
    </motion.div>
  );
}

// Magnetic button effect
export function MagneticButton({ children }: { children: React.ReactNode }) {
  const ref = useRef<HTMLButtonElement>(null);
  const x = useMotionValue(0);
  const y = useMotionValue(0);
  
  const handleMouseMove = (e: MouseEvent) => {
    const rect = ref.current?.getBoundingClientRect();
    if (!rect) return;
    
    const centerX = rect.left + rect.width / 2;
    const centerY = rect.top + rect.height / 2;
    
    x.set(e.clientX - centerX);
    y.set(e.clientY - centerY);
  };
  
  const handleMouseLeave = () => {
    x.set(0);
    y.set(0);
  };
  
  return (
    <motion.button
      ref={ref}
      style={{ x, y }}
      onMouseMove={handleMouseMove}
      onMouseLeave={handleMouseLeave}
      whileHover={{ scale: 1.1 }}
      whileTap={{ scale: 0.9 }}
    >
      {children}
    </motion.button>
  );
}
```

### Pattern 5: Micro-Interactions

Subtle animations for enhanced UX:

```typescript
import { motion } from 'framer-motion';

// Button hover effects
export function AnimatedButton({ children }: { children: React.ReactNode }) {
  return (
    <motion.button
      whileHover={{ scale: 1.05 }}
      whileTap={{ scale: 0.95 }}
      className="px-6 py-3 bg-blue-500 text-white rounded-lg"
    >
      {children}
    </motion.button>
  );
}

// Icon animations
export function AnimatedIcon({ icon }: { icon: React.ReactNode }) {
  return (
    <motion.div
      whileHover={{ rotate: 15, scale: 1.2 }}
      whileTap={{ rotate: -15, scale: 0.9 }}
      transition={{ type: 'spring', stiffness: 400, damping: 10 }}
    >
      {icon}
    </motion.div>
  );
}

// Input focus animations
export function AnimatedInput() {
  return (
    <motion.input
      initial={{ width: '100%' }}
      whileFocus={{ 
        scale: 1.02,
        boxShadow: '0 0 0 3px rgba(59, 130, 246, 0.5)',
      }}
      transition={{ type: 'spring', stiffness: 300, damping: 20 }}
      className="px-4 py-2 border rounded-lg"
    />
  );
}

// Loading spinner
export function LoadingSpinner() {
  return (
    <motion.div
      animate={{ rotate: 360 }}
      transition={{ duration: 1, repeat: Infinity, ease: 'linear' }}
      className="w-8 h-8 border-4 border-blue-500 border-t-transparent rounded-full"
    />
  );
}
```

## Performance Optimization

### 1. Use Hardware Acceleration

```typescript
// Good - uses GPU
gsap.to(element, {
  x: 100,
  y: 100,
  opacity: 0.5,
  scale: 1.2,
});

// Avoid - triggers layout
gsap.to(element, {
  width: 200,
  height: 200,
  top: 100,
  left: 100,
});
```

### 2. Debounce Scroll Events

```typescript
import { debounce } from 'lodash';

const handleScroll = debounce(() => {
  // Scroll animation logic
}, 16); // ~60fps

window.addEventListener('scroll', handleScroll);
```

### 3. Use will-change Sparingly

```css
.animated-element {
  will-change: transform, opacity;
}

/* Remove when not animating */
.animated-element:not(.animating) {
  will-change: auto;
}
```

### 4. Reduce Paint Areas

```typescript
// Use transforms instead of changing layout properties
gsap.to(element, {
  x: 100, // Good - composite layer
  // width: 200, // Bad - triggers repaint
});
```

## Best Practices

### ✅ Do

- Use `transform` and `opacity` for smooth animations
- Implement animation cleanup on unmount
- Use reduced motion for accessibility
- Test animations on low-end devices
- Provide fallbacks for browsers without animation support

### ❌ Don't

- Animate layout properties (width, height, top, left)
- Create infinite loops without user control
- Use animations that cause motion sickness
- Overuse animations (less is more)
- Ignore performance budgets

## Accessibility

```typescript
// Respect user preferences
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');

if (!prefersReducedMotion.matches) {
  // Enable animations
  gsap.to(element, { opacity: 1, duration: 0.5 });
} else {
  // Skip animations
  element.style.opacity = '1';
}

// Framer Motion reduced motion
<motion.div
  animate={!prefersReducedMotion.matches ? { opacity: 1 } : { opacity: 1 }}
  transition={prefersReducedMotion.matches ? { duration: 0 } : undefined}
>
  Content
</motion.div>
```

## Common Pitfalls

1. **Memory Leaks**: Always clean up animations on component unmount
2. **Layout Thrashing**: Batch DOM reads and writes
3. **Over-animating**: Use animations purposefully, not decoratively
4. **Mobile Performance**: Test on actual devices, not just simulators
5. **Scroll Jank**: Use `requestAnimationFrame` for scroll handlers

## Integration Examples

### With Next.js

```typescript
'use client';

import { useEffect, useRef } from 'react';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

export default function AnimatedSection() {
  const sectionRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    gsap.registerPlugin(ScrollTrigger);
    
    const ctx = gsap.context(() => {
      gsap.from(sectionRef.current, {
        opacity: 0,
        y: 50,
        duration: 1,
        scrollTrigger: {
          trigger: sectionRef.current,
          start: 'top 80%',
        },
      });
    }, sectionRef);
    
    return () => ctx.revert();
  }, []);
  
  return <div ref={sectionRef}>Animated content</div>;
}
```

### With Tailwind CSS

```typescript
<motion.div
  className="p-6 bg-white rounded-lg shadow-lg"
  whileHover={{ 
    scale: 1.05,
    boxShadow: '0 20px 25px -5px rgba(0, 0, 0, 0.1)',
  }}
  transition={{ type: 'spring', stiffness: 300, damping: 20 }}
>
  Content
</motion.div>
```

## Resources

- [GSAP Documentation](https://greensock.com/docs/)
- [Framer Motion Documentation](https://www.framer.com/motion/)
- [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API)
- [Animation Performance Guide](https://web.dev/animations-guide/)
