# framer-motion-animator-refurbished

An enhanced version of [`patricio0312rev/skills@framer-motion-animator`](https://skills.sh/patricio0312rev/skills/framer-motion-animator) (5.1K installs), merged with performance and modernization guidelines from [`mindrally/skills@framer-motion`](https://skills.sh/mindrally/skills/framer-motion).

## What changed from the original

### 1. Updated package import (breaking change in newer projects)

The Framer Motion library was renamed to **Motion**. All imports have been updated:

```tsx
// Before (original skill)
import { motion } from 'framer-motion';

// After (this skill)
import { motion } from 'motion/react';
```

The old `framer-motion` package still works for existing projects, but new projects should use `motion`.

### 2. Updated installation instructions

```bash
# Recommended (new)
npm install motion

# Legacy (still works)
npm install framer-motion
```

### 3. Added Performance-First section

A new section explaining **which properties are safe to animate** (GPU-accelerated) vs which ones cause layout thrashing:

- ✅ Safe: `x`, `y`, `scale`, `rotate`, `opacity`, `filter`, `clipPath`
- ❌ Avoid: `width`, `height`, `top`, `left`, `margin`, `padding`

### 4. Added `will-change` guidance

Explains how to use `will-change` correctly and — critically — **how to avoid creating new objects on every render**:

```tsx
// Bad — creates new object on every render
<motion.div style={{ willChange: "transform" }} />

// Good — define outside component
const transformStyle = { willChange: "transform" };
<motion.div style={transformStyle} />
```

### 5. Added memoization patterns

New section on memoizing variants and callbacks to prevent unnecessary re-renders:

```tsx
const variants = useMemo(() => ({
  hidden: { opacity: 0 },
  visible: { opacity: 1 }
}), []);

const handleComplete = useCallback(() => { ... }, []);
```

### 6. Added Performance Debugging section

Practical checklist for diagnosing animation jank:
- Use React DevTools to inspect re-renders
- Use Chrome DevTools Performance tab
- Target 60fps minimum, 120fps on high refresh rate displays
- Test on real mid-range Android devices

### 7. Expanded Best Practices

The original had 8 best practices. This version has 9, with the `motion/react` import as rule #1 and memoization added explicitly.

## What was kept from the original

Everything else remains intact from the original `framer-motion-animator`:

- Core Workflow (6-step process)
- Basic animations, exit animations with `AnimatePresence`
- Variants pattern (staggered children, interactive variants)
- Page transitions (Next.js App Router)
- Shared layout animations with `LayoutGroup`
- Gesture animations (drag, swipe to dismiss)
- Scroll animations (`useInView`, `useScroll`, `useTransform`, parallax)
- Animation hooks (`useAnimate`, `useMotionValue`)
- Reusable components (`AnimatedContainer`, `AnimatedList`)
- Transition presets
- Reduced motion support
- Output checklist

## Installation

```bash
npx skills add emekui/skills@framer-motion-animator-refurbished -g
```

## Credits

- Original skill: [patricio0312rev/skills@framer-motion-animator](https://skills.sh/patricio0312rev/skills/framer-motion-animator)
- Performance guidelines: [mindrally/skills@framer-motion](https://skills.sh/mindrally/skills/framer-motion)
