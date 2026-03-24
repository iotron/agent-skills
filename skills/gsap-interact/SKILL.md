---
name: gsap-interact
description: >
  Production recipes for mouse-driven interactive GSAP animations in Vue 3 / Nuxt 3.
  Companion to official gsap-core and gsap-performance skills (API reference).
  Triggers: GSAP mouse, mousemove animation, tilt card, 3D tilt, quickTo, quickSetter,
  hover animation, interactive GSAP, parallax hover, drag interaction, Draggable, liveSnap,
  mouse-driven animation, dock effect, macOS dock, magnification, proximity scale, dock magnify.
  Non-triggers: Not for scroll-driven animation (use gsap-scroll), text effects (use gsap-text),
  SVG drawing (use gsap-svg), visual effects like glitch/marquee (use gsap-vfx),
  or cursor/pointer effects (use gsap-cursor).
  Outcome: Produces mouse-driven interactive animations — tilt cards, hover effects,
  and drag interactions.
---

# GSAP Interact — Mouse-Driven Animations

> **Flow**: gsap-setup → gsap-animate → **gsap-interact** → gsap-optimise → gsap-test

> **Companion**: For GSAP core tween API, invoke **gsap-core**. For performance methods, invoke **gsap-performance**. This skill covers interaction recipes only. Requires: `greensock/gsap-skills`

All patterns assume `gsap.context()` cleanup is in place (see gsap-animate skill).

---

## 1. 3D Tilt Cards

### Cursor-to-rotation calculation

```js
const rect = card.getBoundingClientRect()
const xPct = ((e.clientX - rect.left) / rect.width - 0.5) * 2
const yPct = ((e.clientY - rect.top) / rect.height - 0.5) * 2
```

### Full setup with ctx.add

```js
const TILT_IN  = { duration: 0.35, ease: 'power2.out', overwrite: 'auto' }
const TILT_OUT = { duration: 0.7,  ease: 'elastic.out(1, 0.4)', overwrite: 'auto' }

ctx = gsap.context((self) => {
  gsap.set(card, { transformPerspective: 900, force3D: true })

  self.add('applyTilt', (xPct, yPct) => {
    card.style.willChange = 'transform'
    gsap.to(card, { rotationY: xPct * 14, rotationX: -yPct * 14, ...TILT_IN })
    gsap.to(inner, { x: xPct * 10, y: yPct * 10, force3D: true, ...TILT_IN })
  })

  self.add('resetTilt', () => {
    gsap.to(card, { rotationX: 0, rotationY: 0, ...TILT_OUT,
      onComplete: () => { card.style.willChange = 'auto' } })
    gsap.to(inner, { x: 0, y: 0, force3D: true, ...TILT_OUT })
  })
}, scopeRef.value)
```

**Required CSS**: `.tilt-wrapper { perspective: 1000px; }` `.tilt-shell { transform-style: preserve-3d; }`

---

## 2. Patterns & Best Practices

```js
// 1. ALL event-handler tweens inside ctx.add for cleanup
self.add('onHover', (el) => { gsap.to(el, { scale: 1.05, overwrite: 'auto' }) })

// 2. overwrite: 'auto' on EVERY rapid-fire tween
gsap.to(el, { x: 100, overwrite: 'auto' })

// 3. will-change: set on start, release onComplete
el.style.willChange = 'transform'
gsap.to(el, { x: 0, onComplete: () => { el.style.willChange = 'auto' } })

// 4. force3D: true on repeatedly animated elements
gsap.set(el, { force3D: true })
```

> See **gsap-performance** skill for tool selection guidance.

### Cleanup checklist

1. Every interactive tween registered via `ctx.add('name', fn)`
2. `ctx.revert()` called in `onUnmounted`
3. `transformPerspective` set via `gsap.set` (not per-frame)
4. `quickTo` instances created in `onMounted`, not in handlers
5. `will-change` released in `onComplete`, never permanent

---

## References

- `references/interaction-patterns.md` — Draggable with liveSnap for SVG point editing, macOS Dock magnification effect
