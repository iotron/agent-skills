---
name: gsap-optimise
description: >
  GSAP performance optimisation for Vue 3 / Nuxt 3 and React. Apply after building animations
  (gsap-animate) to audit and improve performance.
  Triggers: GSAP performance, animation optimise, animation slow, jank, 60fps, memory leak,
  tween cleanup, force3D, will-change, quickTo, quickSetter, overwrite, layout thrashing,
  GPU compositing, animation audit, gsap optimise, animation performance review.
  Non-triggers: Not for building new animations (use gsap-animate and sub-skills). Not for
  testing (use gsap-test). Apply this skill AFTER animations are built.
  Outcome: Audits and improves existing GSAP animations for 60fps — GPU acceleration, high-frequency
  tools, anti-patterns, and a component-level performance checklist.
---

# GSAP Optimise — Performance Pass

> **Flow**: gsap-setup → gsap-animate → **gsap-optimise** → gsap-test
> **References**: `references/performance-deep-dive.md` for extended API details.

> **Companion**: For general GSAP performance guidance, invoke **gsap-performance**. This skill covers optimisation recipes and audit checklists only. Requires: `greensock/gsap-skills`

---

## 1. GPU Acceleration

```js
// GOOD — GPU-composited
gsap.to(el, { x: 100, y: 50, rotation: 45, scale: 1.2, opacity: 0.8 })
// BAD — triggers layout/reflow
gsap.to(el, { left: 100, top: 50, width: '120%', height: 200 })
```

| force3D | Behaviour | Use when |
|---------|-----------|----------|
| `"auto"` | 3D during tween, 2D after | Default — usually best |
| `true` | Stay 3D permanently | Elements animated repeatedly |
| `false` | Always 2D | Upscaled images (avoids blur) |

---

## 2. will-change

CSS `will-change: transform` is fine for always-animating elements (scroll-driven, looping). For interactive/repeated animations, manage in JS:

```js
self.add('applyEffect', (el) => {
  el.style.willChange = 'transform'
  gsap.to(el, { scale: 1.03, ...HOVER_IN })
})
self.add('resetEffect', (el) => {
  gsap.to(el, { scale: 1, ...HOVER_OUT,
    onComplete: () => { el.style.willChange = 'auto' }
  })
})
```

---

## 3. High-Frequency Animations

| Method | Animates? | Speed | Use case |
|--------|-----------|-------|----------|
| `gsap.to()` | Yes | Normal | General animation |
| `gsap.quickTo()` | Yes | Fast | Repeated cursor/scroll-driven |
| `gsap.quickSetter()` | No | Fastest | Per-frame direct value piping |

```js
// quickTo — pre-creates reusable tween
const xTo = gsap.quickTo('#cursor', 'x', { duration: 0.4, ease: 'power3' })
el.addEventListener('mousemove', (e) => xTo(e.clientX))

// quickSetter — bypasses tween creation
const setX = gsap.quickSetter('#el', 'x', 'px')
gsap.ticker.add(() => setX(mouseX))

// pipe — chain transformations
const xSet = gsap.utils.pipe(
  gsap.utils.clamp(0, 100), gsap.utils.snap(5),
  gsap.quickSetter('#el', 'x', 'px')
)
```

---

## 4. Lazy Rendering

GSAP defers first-render writes to end of tick — avoids layout thrashing. Default `lazy: true`.

When multiple `from()`/`fromTo()` target the same property, set `immediateRender: false` on later ones.

---

## 5. Anti-Patterns

```js
// BAD: bare gsap.to inside event handler — orphaned tween
el.addEventListener('mousemove', () => gsap.to(el, { x: 10 }))

// BAD: rapid fire without overwrite — tween pile-up
onMouseMove = () => gsap.to(el, { x: pos, duration: 0.3 })

// BAD: layout properties — triggers reflow
gsap.to(el, { width: 200, height: 100, left: 50 })

// BAD: transformPerspective on every frame (set once with gsap.set)
// BAD: permanent will-change on interactive elements
// BAD: new timeline every call (create once, play/reverse)
// BAD: no reduced motion check
// BAD: markers left in production
```

---

## 6. Performance Audit Checklist

**GPU & Rendering**
- [ ] Only transform/opacity animated (no left/top/width/height)
- [ ] `force3D: true` on repeatedly animated elements
- [ ] `will-change` managed: CSS for scroll-driven, JS for interactive
- [ ] `autoAlpha` used instead of `opacity`

**High-Frequency**
- [ ] `overwrite: 'auto'` on all rapid-fire tweens
- [ ] `quickTo()` for repeated animated updates
- [ ] `quickSetter()` for per-frame value piping

**ScrollTrigger**
- [ ] `ScrollTrigger.batch()` for large grids (50+ elements)
- [ ] `once: true` on one-shot triggers
- [ ] `invalidateOnRefresh: true` on responsive layouts
- [ ] `ScrollTrigger.refresh()` after dynamic content

**Cleanup**
- [ ] All tweens inside `gsap.context()` or `ctx.add()`
- [ ] `ctx.revert()` in `onUnmounted`
- [ ] Timelines created once and played/reversed

**Accessibility**
- [ ] `prefers-reduced-motion` respected via `gsap.matchMedia()`
- [ ] Content visible without JavaScript

---

## References

- `references/performance-deep-dive.md` — Extended API: quickTo, quickSetter, pipe, ScrollTrigger.batch, scrub tuning, matchMedia, registerEffect, ticker/lagSmoothing, autoAlpha, force3D, lazy rendering internals, context cleanup internals, lazy initialisation, timeline reuse
