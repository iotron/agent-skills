---
name: gsap-animate
description: >
  Production recipes for core GSAP animation orchestration. Provides
  foundational patterns and dispatches to specialised sub-skills based on animation type.
  Companion to official gsap-frameworks and gsap-timeline skills (API reference).
  Triggers: GSAP animation, gsap.to, gsap.from, gsap.context, ctx.add, Vue GSAP, Nuxt GSAP,
  animation pattern, component animation, section animation, layout animation.
  Non-triggers: Not for initial GSAP installation/setup (use gsap-setup). Not for specific pattern
  implementation — dispatches to sub-skills instead.
  Outcome: Provides foundational animation patterns (context, cleanup, composables, autoAlpha,
  matchMedia) and dispatches to the correct sub-skill based on animation type.
---

# GSAP Animate — Core Orchestrator

> **Flow**: gsap-setup → **gsap-animate** → gsap-optimise → gsap-test

> **Companion**: For framework lifecycle/cleanup, invoke **gsap-frameworks**. For timeline API, invoke **gsap-timeline**. This skill covers animation orchestration recipes only. Requires: `greensock/gsap-skills`

---

## Sub-Skill Dispatch

| Building this? | Use this sub-skill |
|---|---|
| Scroll reveals, parallax, pinned sections, stacking cards | **gsap-scroll** |
| Tilt cards, hover effects, drag interactions | **gsap-interact** |
| Text reveals (SplitText), scramble decode, kinetic type | **gsap-text** |
| SVG path drawing, morphing, circuit boards | **gsap-svg** |
| Glitch effects, marquee, counters, floating elements | **gsap-vfx** |
| Cursor followers, trails, magnetic, spotlight | **gsap-cursor** |
| Canvas particle systems, canvas rendering | **gsap-canvas** |
| Premium section or full-page composition | **gsap-showcase** (reference) + technique skills |

### Common Layout Combinations

| Layout | Sub-skills to combine |
|---|---|
| **Hero section** | gsap-text + gsap-scroll + gsap-vfx |
| **Services grid** | gsap-scroll + gsap-interact |
| **Circuit board** | gsap-svg + gsap-interact |
| **Cyber/terminal page** | gsap-text + gsap-vfx + gsap-svg |
| **Stats section** | gsap-vfx + gsap-scroll |
| **Premium hero** | gsap-showcase + gsap-text + gsap-scroll + gsap-cursor |
| **Interactive canvas scene** | gsap-showcase + gsap-canvas + gsap-scroll |

---

## 1. Context & Cleanup (Foundation)

**Rule**: Every tween must live inside a `gsap.context()`.

```js
let ctx
onMounted(() => {
  ctx = gsap.context((self) => {
    gsap.to('.item', { y: 0, scrollTrigger: { ... } })
    gsap.set(el, { transformPerspective: 900 })

    self.add('onHover', (el, x, y) => {
      gsap.to(el, { rotationX: x, rotationY: y, overwrite: 'auto' })
    })
  }, scopeRef.value)
})
onUnmounted(() => ctx?.revert())
```

`ctx.revert()` kills all tweens + ScrollTriggers + restores inline styles. Does NOT remove DOM event listeners.

---

## 2. Composable Coexistence

When `useReveal()` coexists with interactive animations, create **separate** contexts:

```js
const { init, scroll } = useReveal(sectionRef)
let tiltCtx
onMounted(() => {
  init(() => scroll())
  tiltCtx = gsap.context((self) => {
    self.add('applyTilt', (shell, xPct, yPct) => { ... })
  }, sectionRef.value)
})
onUnmounted(() => tiltCtx?.revert())
```

---

## 3. autoAlpha

Always use `autoAlpha` over `opacity` for reveals — see **gsap-core** skill for details.

---

## 4. Shared Tween Defaults

```js
const HOVER_IN  = { duration: 0.35, ease: 'power2.out', overwrite: 'auto' }
const HOVER_OUT = { duration: 0.7,  ease: 'elastic.out(1, 0.4)' }
gsap.to(el, { scale: 1.03, force3D: true, ...HOVER_IN })
```

---

## 5. Accessibility via matchMedia

```js
const mm = gsap.matchMedia()
mm.add({
  isDesktop: '(min-width: 1024px)',
  isMobile: '(max-width: 1023px)',
  reduceMotion: '(prefers-reduced-motion: reduce)',
}, (context) => {
  const { isDesktop, reduceMotion } = context.conditions
  if (reduceMotion) { gsap.set('.animate', { autoAlpha: 1 }); return }
  if (isDesktop) { gsap.to('.hero', { x: 200, scrollTrigger: { ... } }) }
})
onUnmounted(() => mm?.revert())
```

---

## 6. Timeline Position Parameters

See **gsap-timeline** skill for the full position parameter reference (`+=`, `-=`, `<`, absolute).

---

## 7. registerEffect — Consuming Effects

```js
gsap.effects.reveal('.cards')
gsap.effects.reveal('.section', { duration: 1.2 })
tl.reveal('.cards', { duration: 0.4 }, '-=0.2')
```

---

## References

- `references/vue-examples.md` — Full component implementations:
  - 3D tilt card with composable coexistence
  - Multi-concept page with single context and 6 named handlers
  - Accessible hero with matchMedia and reduced motion
