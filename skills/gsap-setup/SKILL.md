---
name: gsap-setup
description: >
  GSAP setup and configuration for any framework — Vue 3, Nuxt 3, React, or Next.js. Detects
  framework from project files and provides the correct setup pattern.
  Triggers: GSAP setup, GSAP install, gsap plugin, GSAP Nuxt, GSAP Vue setup, GSAP React setup,
  GSAP Next.js setup, gsap config, GSAP accessibility setup, reduced motion setup, animation
  project setup, useGSAP, gsap registerPlugin.
  Non-triggers: Not for animation patterns (use gsap-animate), ScrollTrigger (use gsap-scroll),
  text effects (use gsap-text), or performance tuning (use gsap-optimise).
  Outcome: Produces a working GSAP plugin/provider with registered plugins, effects, global
  defaults, and accessibility setup.
---

# GSAP Setup

> **Flow**: **gsap-setup** → gsap-animate → gsap-optimise → gsap-test
> Detect the framework from project files (nuxt.config, next.config, package.json) and use the matching section below.

> **Companion**: For GSAP core API and defaults, invoke **gsap-core**. For framework lifecycle/cleanup, invoke **gsap-frameworks**. This skill covers setup recipes only. Requires: `greensock/gsap-skills`

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

```js
// plugins/gsap.js
import { gsap } from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

export default defineNuxtPlugin(() => {
  gsap.registerPlugin(ScrollTrigger)
  return { provide: { gsap, ScrollTrigger } }
})
```

Access: `const { $gsap: gsap, $ScrollTrigger: ScrollTrigger } = useNuxtApp()`

### Vue 3 (Vite)

```js
// src/main.js
import { gsap } from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'
gsap.registerPlugin(ScrollTrigger)
app.provide('gsap', gsap)
```

Access: `const gsap = inject('gsap')` or import directly (tree-shakes fine with Vite).

### React (Vite / CRA)

```jsx
import { gsap } from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'
import { useGSAP } from '@gsap/react'
gsap.registerPlugin(ScrollTrigger)

function Component() {
  const containerRef = useRef(null)
  useGSAP(() => {
    gsap.to('.box', { x: 200, duration: 1 })
  }, { scope: containerRef })
  return <div ref={containerRef}><div className="box">Animated</div></div>
}
```

### Next.js (App Router)

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

```js
gsap.defaults({ overwrite: 'auto' })
gsap.ticker.lagSmoothing(500, 33)
```

- `overwrite: 'auto'` — kills only conflicting properties on the same target.
- `lagSmoothing(500, 33)` — caps lag compensation so big frame drops don't cause a jarring jump.

---

## 4. registerEffect — Reusable Animations

```js
gsap.registerEffect({
  name: 'reveal',
  effect: (targets, config) =>
    gsap.fromTo(targets, { y: 20, autoAlpha: 0 }, { y: 0, autoAlpha: 1, ...config }),
  defaults: { duration: 0.8, ease: 'power2.out', stagger: 0 },
  extendTimeline: true,
})
```

Usage: `gsap.effects.reveal('.cards')` or on timelines: `tl.reveal('.cards', {}, '-=0.2')`

---

## 5. CSS Pre-hide

Elements animated by GSAP should be pre-hidden in CSS to prevent CLS. `autoAlpha` sets `visibility: visible` when it runs.

```css
.reveal, .text-reveal { visibility: hidden; }
```

---

## 6. Accessibility

### Vue / Nuxt composable

```js
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

## References

- `references/full-plugin-example.md` — Complete Nuxt plugin with lazy loaders, registered effects, and ScrollTrigger sort
