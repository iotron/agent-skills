---
name: gsap-setup
description: >
  GSAP setup and configuration for any framework — Vue 3, Nuxt 3, React, or Next.js. Detects
  framework from project files and provides the correct setup pattern. Covers: plugin/provider
  creation, registerPlugin calls, registerEffect registrations, gsap.defaults and gsap.config,
  CSS progressive enhancement, accessibility setup (useReducedMotion composable/hook),
  and prefers-reduced-motion integration.
  Triggers: GSAP setup, GSAP install, gsap plugin, GSAP Nuxt, GSAP Vue setup, GSAP React setup,
  GSAP Next.js setup, gsap config, GSAP accessibility setup, reduced motion setup, animation
  project setup, useGSAP, gsap registerPlugin.
---

# GSAP Setup

> **Flow**: **gsap-setup** → gsap-animate → gsap-optimise → gsap-test
> Detect the framework from project files (nuxt.config, next.config, package.json) and use the matching section below.

---

## 1. Installation

```bash
npm install gsap

# React projects: also install the official hook
npm install @gsap/react
```

GSAP plugins (ScrollTrigger, SplitText, etc.) are included in the `gsap` package — import directly.

---

## 2. Framework Setup

### Nuxt 3

Register as a plugin. Nuxt auto-imports `defineNuxtPlugin`; the `process.client` guard inside handles SSR safety.

```js
// plugins/gsap.js
import { gsap } from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

export default defineNuxtPlugin(() => {
  gsap.registerPlugin(ScrollTrigger)

  return { provide: { gsap, ScrollTrigger } }
})
```

Access in components:

```js
const { $gsap: gsap, $ScrollTrigger: ScrollTrigger } = useNuxtApp()
```

### Vue 3 (Vite)

Register plugins once in app entry.

```js
// src/main.js
import { gsap } from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

app.provide('gsap', gsap)
app.provide('ScrollTrigger', ScrollTrigger)
```

Access in components:

```js
import { inject } from 'vue'
const gsap = inject('gsap')

// Or import directly (tree-shakes fine with Vite)
import { gsap } from 'gsap'
```

### React (Vite / CRA)

Use the official `@gsap/react` hook for automatic cleanup.

```jsx
// src/App.jsx — register once at top level
import { gsap } from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'
import { useGSAP } from '@gsap/react'

gsap.registerPlugin(ScrollTrigger)

// In any component:
function Component() {
  const containerRef = useRef(null)

  useGSAP(() => {
    gsap.to('.box', { x: 200, duration: 1 })
  }, { scope: containerRef }) // scopes selectors to container

  return <div ref={containerRef}><div className="box">Animated</div></div>
}
```

Manual cleanup (without `@gsap/react`): wrap in `gsap.context()`, call `ctx.revert()` in the cleanup function.

### Next.js (App Router)

Register in a `'use client'` module. Never import GSAP in Server Components.

```jsx
// lib/gsap.js
'use client'
import { gsap } from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'
import { useGSAP } from '@gsap/react'
gsap.registerPlugin(ScrollTrigger)
export { gsap, ScrollTrigger, useGSAP }
```

---

## 3. Global Defaults

Set once at app initialisation — inherited by every tween.

```js
gsap.defaults({ overwrite: 'auto' })
gsap.ticker.lagSmoothing(500, 33)
```

- `overwrite: 'auto'` — kills only conflicting properties on the same target, preventing tween fights.
- `lagSmoothing(500, 33)` — caps lag compensation so big frame drops don't cause a jarring jump.

---

## 4. registerEffect — Reusable Animations

Register effects once at app level so they're available everywhere.

```js
// Wrapper appearance — fade + translate
gsap.registerEffect({
  name: 'reveal',
  effect: (targets, config) =>
    gsap.fromTo(targets,
      { y: 20, autoAlpha: 0 },
      { y: 0, autoAlpha: 1, ...config },
    ),
  defaults: { duration: 0.8, ease: 'power2.out', stagger: 0 },
  extendTimeline: true,
})

// SplitText masked word slide-up
gsap.registerEffect({
  name: 'textReveal',
  effect: (targets, config) =>
    gsap.to(targets, { y: '0%', ...config }),
  defaults: { duration: 0.6, ease: 'power3.out', stagger: 0.06, force3D: true },
  extendTimeline: true,
})
```

Usage: `gsap.effects.reveal('.cards')` or on timelines: `tl.reveal('.cards', {}, '-=0.2')`

---

## 5. CSS Pre-hide with `visibility: hidden`

Elements animated by GSAP should be pre-hidden in CSS to prevent the SSR-to-hydration flash (CLS). GSAP's `autoAlpha` tween sets `visibility: visible` when it runs, so no JS class toggling is needed.

```css
/* Pre-hide until GSAP animates them in via autoAlpha */
.reveal,
.text-reveal {
  visibility: hidden;
}
```

If JS never loads or the animation never fires, `visibility: hidden` keeps the element in layout flow (no layout shift) while hiding it — a safe progressive-enhancement baseline for animation targets.

---

## 6. Accessibility Setup

### Vue / Nuxt composable

```js
// composables/useReducedMotion.js
export const useReducedMotion = () => {
  const prefersReduced = ref(false)
  onMounted(() => {
    const mql = window.matchMedia('(prefers-reduced-motion: reduce)')
    prefersReduced.value = mql.matches
    mql.addEventListener('change', (e) => { prefersReduced.value = e.matches })
  })
  return { prefersReduced }
}
```

### React hook

```jsx
// hooks/useReducedMotion.js
import { useState, useEffect } from 'react'

export function useReducedMotion() {
  const [prefersReduced, setPrefersReduced] = useState(false)
  useEffect(() => {
    const mql = window.matchMedia('(prefers-reduced-motion: reduce)')
    setPrefersReduced(mql.matches)
    const handler = (e) => setPrefersReduced(e.matches)
    mql.addEventListener('change', handler)
    return () => mql.removeEventListener('change', handler)
  }, [])
  return prefersReduced
}
```

**Prefer `gsap.matchMedia()`** when animations should auto-revert on preference change — see **gsap-animate** skill.

---

## 7. Full Nuxt Plugin Example

```js
// plugins/gsap.js
import { gsap } from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'
import TextPlugin from 'gsap/TextPlugin'

/** Lazy-loader factory: import once, register once, cache forever. */
function createLazyLoader(importFn, exportKey = 'default') {
  let cached = null
  return async () => {
    if (!cached) {
      const mod = await importFn()
      cached = mod[exportKey]
      gsap.registerPlugin(cached)
    }
    return cached
  }
}

export default defineNuxtPlugin(() => {
  if (process.client) {
    gsap.registerPlugin(ScrollTrigger, TextPlugin)

    gsap.defaults({ overwrite: 'auto' })
    gsap.ticker.lagSmoothing(500, 33)

    // ── Registered effects ──

    gsap.registerEffect({
      name: 'reveal',
      effect: (targets, config) =>
        gsap.fromTo(targets,
          { y: 20, autoAlpha: 0 },
          { y: 0, autoAlpha: 1, ...config },
        ),
      defaults: { duration: 0.8, ease: 'power2.out', stagger: 0 },
      extendTimeline: true,
    })

    gsap.registerEffect({
      name: 'textReveal',
      effect: (targets, config) =>
        gsap.to(targets, { y: '0%', ...config }),
      defaults: { duration: 0.6, ease: 'power3.out', stagger: 0.06, force3D: true },
      extendTimeline: true,
    })

    // Auto-sort ScrollTriggers by DOM position before every refresh
    ScrollTrigger.addEventListener('refreshInit', () => ScrollTrigger.sort())
  }

  // Lazy loading helpers for premium/heavy plugins
  const lazyLoadDrawSVG   = createLazyLoader(() => import('gsap/DrawSVGPlugin'))
  const lazyLoadMorphSVG  = createLazyLoader(() => import('gsap/MorphSVGPlugin'))
  const lazyLoadScramble  = createLazyLoader(() => import('gsap/ScrambleTextPlugin'))
  const lazyLoadSplitText = createLazyLoader(() => import('gsap/SplitText'), 'SplitText')

  return {
    provide: {
      gsap,
      ScrollTrigger,
      lazyLoadDrawSVG,
      lazyLoadMorphSVG,
      lazyLoadScramble,
      lazyLoadSplitText,
    },
  }
})
```

**Key patterns:** default imports (GSAP v3 uses default exports), `createLazyLoader` factory for premium plugins (dynamic import on first use, keeps initial bundle small), `ScrollTrigger.sort()` on `refreshInit` (pins refresh before downstream triggers). Provides: `$gsap`, `$ScrollTrigger`, plus four lazy loaders (`$lazyLoadSplitText`, etc.).
