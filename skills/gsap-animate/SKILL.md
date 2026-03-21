---
name: gsap-animate
description: >
  Core GSAP animation orchestrator for Vue 3 / Nuxt 3 and React. Provides foundational patterns
  (gsap.context, ctx.add, cleanup, composable coexistence, autoAlpha, shared tween defaults,
  matchMedia accessibility) and dispatches to specialised sub-skills based on the animation type
  needed. Sub-skills: gsap-scroll (ScrollTrigger, parallax, pin, batch), gsap-interact (tilt,
  spring, cursor, quickTo), gsap-text (SplitText, scramble, char split), gsap-svg (DrawSVG,
  morph, circuit), gsap-vfx (glitch, marquee, counters, floating).
  Triggers: GSAP animation, gsap.to, gsap.from, gsap.context, ctx.add, Vue GSAP, Nuxt GSAP,
  animation pattern, component animation, section animation, layout animation.
---

# GSAP Animate — Core Orchestrator

> **Flow**: gsap-setup → **gsap-animate** → gsap-optimise → gsap-test

This skill provides the **foundational patterns** every GSAP animation needs, then directs to the right sub-skill based on what you're building.

---

## Sub-Skill Dispatch

| Building this? | Use this sub-skill |
|---|---|
| Scroll reveals, parallax, pinned sections, stacking cards | **gsap-scroll** |
| Tilt cards, cursor followers, spotlight, magnetic buttons | **gsap-interact** |
| Text reveals (SplitText), scramble decode, kinetic type | **gsap-text** |
| SVG path drawing, morphing, circuit boards | **gsap-svg** |
| Glitch effects, marquee, counters, floating elements | **gsap-vfx** |

### Common Layout Combinations

| Layout | Sub-skills to combine |
|---|---|
| **Hero section** | gsap-text (heading reveal) + gsap-scroll (content fade) + gsap-vfx (background) |
| **Services grid** | gsap-scroll (batch reveal) + gsap-interact (tilt hover) |
| **Circuit board section** | gsap-svg (circuit animation) + gsap-interact (cursor glow) |
| **Case study page** | gsap-scroll (parallax + stacking) + gsap-text (headings) |
| **Cyber/terminal page** | gsap-text (scramble decode) + gsap-vfx (glitch) + gsap-svg (circuit) |
| **Landing page hero** | gsap-text (elastic type) + gsap-scroll (pin + scrub) |
| **Stats section** | gsap-vfx (counters) + gsap-scroll (reveal on scroll) |
| **Gallery** | gsap-scroll (horizontal scroll + pin) + gsap-interact (magnetic hover) |

---

## 1. Context & Cleanup (Foundation)

**Rule**: Every tween must live inside a `gsap.context()` — directly or via `ctx.add()`.

```js
let ctx

onMounted(() => {
  ctx = gsap.context((self) => {
    // Scroll-driven tweens (auto-collected)
    gsap.to('.item', { y: 0, scrollTrigger: { ... } })

    // One-time setup (auto-collected)
    gsap.set(el, { transformPerspective: 900 })

    // Event-handler tweens (collected via ctx.add)
    self.add('onHover', (el, x, y) => {
      gsap.to(el, { rotationX: x, rotationY: y, overwrite: 'auto' })
    })
  }, scopeRef.value) // scope: selectors only match inside this element
})

onUnmounted(() => ctx?.revert()) // kills ALL tweens + ScrollTriggers + restores styles
```

### What ctx.revert() does

1. Kills all tweens created within the context (including mid-flight)
2. Kills all ScrollTriggers created within the context
3. Restores all inline styles to pre-animation state

**Does NOT remove DOM event listeners.** You must still clean up `addEventListener` calls yourself (in `onUnmounted` or via Vue template `@` handlers which auto-clean). `ctx.add()` named methods organize event-handler code and track the GSAP tweens they create, but they don't manage the DOM listeners that call them.

No manual `scrollTriggers[]` array needed — context collects everything.

### ctx.add() Named Handlers

Register event-driven tweens so context tracks them for cleanup:

```js
self.add('applyTilt', (card, xPct, yPct) => {
  card.style.willChange = 'transform'
  gsap.to(card, { rotationY: xPct * 14, rotationX: -yPct * 14, ...TILT_IN })
})

self.add('resetTilt', (card) => {
  gsap.to(card, { rotationX: 0, rotationY: 0, ...TILT_OUT,
    onComplete: () => { card.style.willChange = 'auto' }
  })
})

// Call from event handlers:
ctx.applyTilt(card, xPct, yPct)
```

---

## 2. Composable Coexistence

When `useReveal()` (owns its own context) coexists with interactive animations, create **separate** contexts:

```js
// useReveal manages its own cleanup internally
const { init, scroll } = useReveal(sectionRef)

// Separate context for interactive animations
let tiltCtx

onMounted(() => {
  init(() => scroll()) // reveal context

  tiltCtx = gsap.context((self) => {
    self.add('applyTilt', (shell, xPct, yPct) => { ... })
    self.add('resetTilt', (shell) => { ... })
  }, sectionRef.value) // interactive context
})

onUnmounted(() => tiltCtx?.revert()) // useReveal handles its own
```

Two contexts, two lifecycles — never mix them.

---

## 3. autoAlpha vs opacity

`autoAlpha` = `opacity` + `visibility: hidden` at 0. Prevents pointer events, improves rendering.

```js
gsap.set(els, { autoAlpha: 0 })               // hidden + invisible
gsap.to(els, { autoAlpha: 1, duration: 0.8 }) // fades in, visibility: inherit
```

**Always** use `autoAlpha` instead of `opacity` for reveal animations.

---

## 4. Shared Tween Defaults

Extract repeated configs to constants — spread into each tween:

```js
const HOVER_IN  = { duration: 0.35, ease: 'power2.out', overwrite: 'auto' }
const HOVER_OUT = { duration: 0.7,  ease: 'elastic.out(1, 0.4)' }

gsap.to(el, { scale: 1.03, force3D: true, ...HOVER_IN })
gsap.to(el, { scale: 1, force3D: true, ...HOVER_OUT })
```

---

## 5. Accessibility via matchMedia

**Rule**: Always respect `prefers-reduced-motion`.

```js
const mm = gsap.matchMedia()

mm.add({
  isDesktop: '(min-width: 1024px)',
  isMobile: '(max-width: 1023px)',
  reduceMotion: '(prefers-reduced-motion: reduce)',
}, (context) => {
  const { isDesktop, isMobile, reduceMotion } = context.conditions

  if (reduceMotion) {
    gsap.set('.animate', { autoAlpha: 1 }) // show immediately
    return
  }

  if (isDesktop) {
    gsap.to('.hero', { x: 200, scrollTrigger: { trigger: '.hero', scrub: 1 } })
  }
  if (isMobile) {
    gsap.to('.hero', { y: 100, scrollTrigger: { trigger: '.hero', scrub: 1 } })
  }
})

onUnmounted(() => mm?.revert())
```

---

## 6. Timeline Position Parameters

```js
tl.to(el, { x: 100 })              // after previous
  .to(el, { y: 100 }, '+=0.5')     // 0.5s gap
  .to(el, { z: 100 }, '-=0.25')    // 0.25s overlap
  .to(el, { rotation: 90 }, '<')   // same start as previous
  .to(el, { scale: 2 }, '<0.5')    // 0.5s after previous starts
  .to(el, { autoAlpha: 0 }, 2)     // absolute: 2s from timeline start
```

Use **absolute positioning** (`tl.add(anim, 3)`) for deterministic timing independent of other durations. Use **relative** (`<`, `+=`, `-=`) for choreography coupled to siblings.

---

## 7. registerEffect — Consuming Effects

Registered once in the plugin (see gsap-setup), use anywhere:

```js
// Direct usage
gsap.effects.reveal('.cards')
gsap.effects.reveal('.section', { duration: 1.2 })

// On timelines (requires extendTimeline: true in registration)
tl.reveal('.cards', { duration: 0.4 }, '-=0.2')
tl.reveal('.items', {}, '<0.1')
```

---

## 8. SPA Route Cleanup

Context handles everything — no manual ScrollTrigger cleanup:

```js
onMounted(() => {
  ctx = gsap.context(() => { /* all animations */ }, containerRef.value)
})

onUnmounted(() => {
  ctx?.revert() // kills tweens + ScrollTriggers for this page
})
```

---

## References

- `references/vue-examples.md` — Full component implementations:
  - 3D tilt card with composable coexistence
  - Multi-concept page with single context and 6 named handlers
  - Accessible hero with matchMedia and reduced motion
