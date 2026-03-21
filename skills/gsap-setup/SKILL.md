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

Register as a client-only plugin (`.client.js` suffix = runs only in browser).

```js
// plugins/gsap.client.js
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

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
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

// Optional: provide globally via app
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
import { ScrollTrigger } from 'gsap/ScrollTrigger'
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

Manual cleanup (without `@gsap/react`):

```jsx
useEffect(() => {
  const ctx = gsap.context(() => {
    gsap.to(ref.current, { x: 100 })
  })
  return () => ctx.revert()
}, [])
```

### Next.js (App Router)

Register in a client component — mark with `'use client'`.

```jsx
// lib/gsap.js — shared registration
'use client'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { useGSAP } from '@gsap/react'

gsap.registerPlugin(ScrollTrigger)

export { gsap, ScrollTrigger, useGSAP }
```

```jsx
// components/AnimatedSection.jsx
'use client'
import { useRef } from 'react'
import { gsap, useGSAP } from '@/lib/gsap'

export function AnimatedSection() {
  const containerRef = useRef(null)

  useGSAP(() => {
    gsap.from('.item', { autoAlpha: 0, y: 30, stagger: 0.1 })
  }, { scope: containerRef })

  return <section ref={containerRef}>...</section>
}
```

**Important**: Never import GSAP in Server Components — it needs the DOM.

---

## 3. Global Defaults

Set once at app initialisation — inherited by every tween.

```js
gsap.defaults({ ease: 'power2.out', duration: 0.5 })

gsap.config({
  force3D: 'auto',       // GPU compositing during animation
  autoSleep: 120,        // frames before ticker sleeps (saves battery)
  nullTargetWarn: false,  // silence null-target warnings in production
})
```

---

## 4. registerEffect — Reusable Animations

Register effects once at app level so they're available everywhere.

```js
gsap.registerEffect({
  name: 'fadeIn',
  effect: (targets, config) => gsap.to(targets, {
    autoAlpha: 1, y: 0, duration: config.duration, stagger: config.stagger,
  }),
  defaults: { duration: 0.6, stagger: 0.08 },
  extendTimeline: true, // enables tl.fadeIn('.cards')
})
```

Usage: `gsap.effects.fadeIn('.cards')` or on timelines: `tl.fadeIn('.cards', {}, '-=0.2')`

---

## 5. CSS Progressive Enhancement

Content **must** be visible without JS. Pre-hide only when animations will run.

```css
/* Default: content visible */
.reveal { opacity: 1; transform: translateY(0); }

/* Pre-hide ONLY when JS is active AND motion is OK */
html.js-animations .reveal {
  opacity: 0;
  transform: translateY(30px);
}

@media (prefers-reduced-motion: reduce) {
  html.js-animations .reveal {
    opacity: 1;
    transform: none;
  }
}
```

Add the class at app startup:

```js
// In plugin/entry — runs once
if (!window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
  document.documentElement.classList.add('js-animations')
}
```

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
// plugins/gsap.client.js
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { SplitText } from 'gsap/SplitText'

export default defineNuxtPlugin(() => {
  gsap.registerPlugin(ScrollTrigger, SplitText)

  gsap.defaults({ ease: 'power2.out', duration: 0.5 })
  gsap.config({ nullTargetWarn: false })

  gsap.registerEffect({
    name: 'fadeIn',
    effect: (targets, config) => gsap.to(targets, {
      autoAlpha: 1, y: 0, duration: config.duration, stagger: config.stagger,
    }),
    defaults: { duration: 0.6, stagger: 0.08 },
    extendTimeline: true,
  })

  if (!window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    document.documentElement.classList.add('js-animations')
  }

  return { provide: { gsap, ScrollTrigger, SplitText } }
})
```
