---
name: gsap-optimise
description: >
  GSAP performance optimisation for Vue 3 / Nuxt 3 and React. Apply after building animations
  (gsap-animate) to audit and improve performance. Covers: GPU acceleration (force3D, transforms
  vs layout properties), will-change lifecycle (set on start, release onComplete), high-frequency
  tools (gsap.quickTo significantly faster for repeated updates, gsap.quickSetter for per-frame piping, overwrite: 'auto'),
  lazy rendering internals, lazy initialisation pattern, timeline reuse, ticker/lagSmoothing tuning,
  gsap.utils.pipe, anti-patterns compilation, and component audit checklist.
  Triggers: GSAP performance, animation optimise, animation slow, jank, 60fps, memory leak,
  tween cleanup, force3D, will-change, quickTo, quickSetter, overwrite, layout thrashing,
  GPU compositing, animation audit, gsap optimise, animation performance review.
---

# GSAP Optimise — Performance Pass

> **Flow**: gsap-vue-setup → gsap-animate → **gsap-optimise** → gsap-test
> **References**: `references/performance-deep-dive.md` for extended API details.

Apply this skill to audit and improve existing GSAP animations for 60fps performance.

---

## 1. GPU Acceleration

### Transforms only — never layout properties

```js
// GOOD — GPU-composited, no layout recalc
gsap.to(el, { x: 100, y: 50, rotation: 45, scale: 1.2, opacity: 0.8 })

// BAD — triggers layout/reflow on every frame
gsap.to(el, { left: 100, top: 50, width: '120%', height: 200, margin: 10 })
```

### force3D

| Value | Behaviour | Use when |
|-------|-----------|----------|
| `"auto"` | 3D during tween, 2D after | **Default — usually best** |
| `true` | Stay 3D permanently | Elements animated repeatedly (mousemove tilt) |
| `false` | Always 2D | Upscaled images (avoids sub-pixel blur) |

```js
gsap.to(el, { x: 100, force3D: true })   // repeated interaction target
gsap.to(img, { scale: 1, force3D: false }) // crisp image
```

---

## 2. will-change

For **one-shot animations** (scroll reveals, page load), `will-change: transform` in CSS is fine — the official GSAP performance skill recommends it.

For **interactive/repeated animations** (mousemove, hover), manage the lifecycle in JS — set on interaction start, release when done. This avoids permanent GPU memory cost for elements that aren't always animating.

```js
// OK — CSS will-change for elements that always animate (scroll-driven, looping)
.parallax-layer { will-change: transform; }

// BETTER for interactive — JS lifecycle
self.add('applyEffect', (el) => {
  el.style.willChange = 'transform'
  gsap.to(el, { scale: 1.03, ...HOVER_IN })
})
self.add('resetEffect', (el) => {
  gsap.to(el, { scale: 1, ...HOVER_OUT,
    onComplete: () => { el.style.willChange = 'auto' }
  })
})

// BAD — permanent in CSS (wastes GPU memory)
.card { will-change: transform; }
```

---

## 3. High-Frequency Animations

### overwrite: 'auto'

Kills in-progress tweens on the same target/properties. **Required** on all rapid-fire tweens (mousemove, scroll callbacks).

```js
gsap.to(el, { x: pos, overwrite: 'auto', duration: 0.3 })
```

### gsap.quickTo() — animated updates (significantly faster for repeated updates)

Pre-creates a reusable tween function. Ideal for cursor followers, parallax.

```js
const xTo = gsap.quickTo('#cursor', 'x', { duration: 0.4, ease: 'power3' })
const yTo = gsap.quickTo('#cursor', 'y', { duration: 0.4, ease: 'power3' })

el.addEventListener('mousemove', (e) => { xTo(e.clientX); yTo(e.clientY) })
```

### gsap.quickSetter() — immediate updates (no animation)

Bypasses full tween creation overhead — sets values directly for maximum per-frame speed.

```js
const setX = gsap.quickSetter('#el', 'x', 'px')
const setY = gsap.quickSetter('#el', 'y', 'px')
// inside ticker or mousemove
setX(mouseX); setY(mouseY)
```

### When to use which

| Method | Animates? | Speed | Use case |
|--------|-----------|-------|----------|
| `gsap.to()` | Yes | Normal | General animation |
| `gsap.quickTo()` | Yes | Fast (reuses single tween) | Repeated cursor/scroll-driven |
| `gsap.quickSetter()` | No (instant) | Fastest | Per-frame direct value piping |
| `gsap.set()` | No (instant) | Fast | General one-off sets |

### gsap.utils.pipe() — chain transformations

```js
const xSet = gsap.utils.pipe(
  gsap.utils.clamp(0, 100),
  gsap.utils.snap(5),
  gsap.quickSetter('#el', 'x', 'px')
)
// xSet(137) → clamps to 100 → snaps to 100 → sets x: 100px
```

---

## 4. Lazy Rendering

GSAP defers first-render writes to the end of the current tick — avoids read/write/read/write layout thrashing. Default `lazy: true`.

- Do **not** set `lazy: false` unless you need the write immediately visible in the same tick.
- Zero-duration tweens (`duration: 0`) have `lazy: false` by default.

### immediateRender on stacked from()/fromTo()

When multiple `from()` or `fromTo()` tweens target the same property of the same element, set `immediateRender: false` on the later ones. Otherwise the second tween's start state overwrites the first tween's end state before it runs.

```js
// BAD — second from() immediately overrides the first
gsap.from(el, { x: -100 })
gsap.from(el, { y: -100 }) // x snaps to current, first tween invisible

// GOOD — later tween defers initial render
gsap.from(el, { x: -100 })
gsap.from(el, { y: -100, immediateRender: false })
```

---

## 5. Lazy Initialisation

Create timelines only when needed — not all at mount.

```js
// GOOD — created on first use
let panelTl = null
const getPanelTimeline = () => {
  if (!panelTl) {
    panelTl = gsap.timeline({ paused: true })
      .from('.panel', { autoAlpha: 0, y: 20 })
      .from('.panel-content', { autoAlpha: 0, stagger: 0.1 })
  }
  return panelTl
}
const showPanel = () => getPanelTimeline().play()
const hidePanel = () => getPanelTimeline().reverse()

// BAD — created at mount even if never used
onMounted(() => {
  const tl1 = gsap.timeline().to('.panel1', { x: 100 })
  const tl2 = gsap.timeline().to('.panel2', { y: 100 })
})
```

---

## 6. Timeline Reuse

```js
// GOOD — create once, play/reverse
const tl = gsap.timeline({ paused: true })
tl.to(el, { autoAlpha: 1 }).to(el, { y: -20 })
const show = () => tl.play()
const hide = () => tl.reverse()

// BAD — new timeline every call
const show = () => gsap.timeline().to(el, { autoAlpha: 1 }).to(el, { y: -20 })
```

---

## 7. Ticker & lagSmoothing

```js
gsap.ticker.lagSmoothing(500, 33) // example: if >500ms gap, act as 33ms
gsap.ticker.lagSmoothing(0)       // disable for frame-perfect (video sync)

gsap.config({ autoSleep: 120 })   // custom value — GSAP default is 60. Frames before ticker sleeps (saves battery)
```

---

## 8. Anti-Patterns

```js
// BAD: bare gsap.to inside event handler — orphaned tween
el.addEventListener('mousemove', () => gsap.to(el, { x: 10 }))

// BAD: rapid fire without overwrite — tween pile-up
onMouseMove = () => gsap.to(el, { x: pos, duration: 0.3 })

// BAD: layout properties — triggers reflow
gsap.to(el, { width: 200, height: 100, left: 50 })

// BAD: transformPerspective on every frame (set once with gsap.set)
gsap.to(el, { transformPerspective: 900, rotationX: x })

// BAD: manual scrollTriggers[] array (context collects them)
const scrollTriggers = []

// BAD: permanent will-change on interactive elements (use JS lifecycle instead)
.hover-card { will-change: transform; } // OK for scroll-driven, BAD for hover/mousemove

// BAD: new timeline every call
const show = () => gsap.timeline().to(el, { ... })

// CONSIDER: batch() instead of per-element ScrollTrigger for large grids (50+ items)
// Per-element ScrollTrigger is fine for small sets — batch() is an optimization, not a requirement
ScrollTrigger.batch('.grid-item', { onEnter: (batch) => gsap.to(batch, { ... }) })

// BAD: no reduced motion check
onMounted(() => gsap.from(el, { y: 100, duration: 1 }))

// BAD: tweens before DOM is ready (setup runs before mount)
const tween = gsap.to('.box', { x: 100 })

// BAD: no ScrollTrigger refresh after dynamic content
fetch('/api/data').then(render) // positions are stale

// BAD: markers left in production
scrollTrigger: { markers: true }
```

---

## 9. Performance Audit Checklist

Apply to every GSAP-using component:

**GPU & Rendering**
- [ ] Only transform/opacity properties animated (no left/top/width/height)
- [ ] `force3D: true` on repeatedly animated elements; `"auto"` elsewhere
- [ ] `will-change` managed: CSS for scroll-driven, JS lifecycle for interactive
- [ ] `autoAlpha` used instead of `opacity` for show/hide
- [ ] `transformPerspective` set once via `gsap.set()`, not per-frame

**High-Frequency**
- [ ] `overwrite: 'auto'` on all rapid-fire tweens
- [ ] `quickTo()` for repeated animated cursor/scroll updates
- [ ] `quickSetter()` for per-frame direct value piping

**ScrollTrigger**
- [ ] `ScrollTrigger.batch()` for large grids (50+ similar elements)
- [ ] `once: true` on one-shot triggers
- [ ] `invalidateOnRefresh: true` on responsive layouts
- [ ] `scrub` uses smoothing number (e.g. `0.5`) not bare `true` when possible
- [ ] `ScrollTrigger.refresh()` called after dynamic content / images / fonts

**Cleanup**
- [ ] All tweens inside `gsap.context()` or `ctx.add()` handlers
- [ ] `ctx.revert()` called in `onUnmounted`
- [ ] No manual `scrollTriggers[]` array
- [ ] SPA route change cleans up via context

**Organisation**
- [ ] Shared tween defaults extracted to constants
- [ ] Reusable animations via `gsap.registerEffect()`
- [ ] Timelines created once and played/reversed, not recreated
- [ ] Complex timelines lazily initialised

**Accessibility**
- [ ] `prefers-reduced-motion` respected via `gsap.matchMedia()` or composable
- [ ] Content visible without JavaScript
- [ ] CSS pre-hide only when JS + motion preference allows

---

## References

- `references/performance-deep-dive.md` — Extended API for quickTo, quickSetter, pipe, ScrollTrigger.batch, scrub tuning, matchMedia, registerEffect, ticker/lagSmoothing, autoAlpha, force3D, lazy rendering internals, context cleanup internals
