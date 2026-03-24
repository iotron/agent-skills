---
name: gsap-interact
description: >
  Mouse-driven interactive GSAP animations for Vue 3 / Nuxt 3.
  Triggers: GSAP mouse, mousemove animation, tilt card, 3D tilt, cursor follower, elastic blob,
  spring physics, spotlight effect, clipPath animation, magnetic button, quickTo, quickSetter,
  hover animation, interactive GSAP, cursor effect, parallax hover, circuit glow, CSS custom
  property animation, mouse-driven animation.
  Non-triggers: Not for scroll-driven animation (use gsap-scroll), text effects (use gsap-text),
  SVG drawing (use gsap-svg), or visual effects like glitch/marquee (use gsap-vfx).
  Outcome: Produces mouse-driven interactive animations — tilt cards, cursor followers, spotlights,
  magnetic buttons, and circuit glow effects.
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

## 2. Magnetic Buttons (quickTo)

`gsap.quickTo` pre-compiles a tween — 50-250% faster than `gsap.to` for frequent updates.

```js
let xTo, yTo
ctx = gsap.context(() => {
  xTo = gsap.quickTo(btn, 'x', { duration: 0.4, ease: 'power3' })
  yTo = gsap.quickTo(btn, 'y', { duration: 0.4, ease: 'power3' })
}, scopeRef.value)

function onMagnetMove(e) {
  const rect = btn.getBoundingClientRect()
  xTo((e.clientX - rect.left - rect.width / 2) * 0.35)
  yTo((e.clientY - rect.top - rect.height / 2) * 0.35)
}
function onMagnetLeave() { xTo(0); yTo(0) }
```

### quickTo vs quickSetter

| Tool | Use case | Speed |
|------|----------|-------|
| `gsap.quickTo` | Tweened updates (mousemove with easing) | Fast |
| `gsap.quickSetter` | Instant per-frame value piping (ticker/RAF) | Fastest |

---

## 3. Patterns & Best Practices

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

### Tool selection guide

| Scenario | Tool | Why |
|----------|------|-----|
| Mousemove with easing | `gsap.quickTo` | Pre-compiled, re-targets instantly |
| Per-frame piping (ticker/RAF) | `gsap.quickSetter` | Zero tween overhead |
| One-shot hover in/out | `gsap.to` + `overwrite: 'auto'` | Simple, auto-cleans |
| CSS custom properties | Direct `style.setProperty` | No GSAP needed, fastest |

### Cleanup checklist

1. Every interactive tween registered via `ctx.add('name', fn)`
2. `ctx.revert()` called in `onUnmounted`
3. `transformPerspective` set via `gsap.set` (not per-frame)
4. `quickTo` instances created in `onMounted`, not in handlers
5. `will-change` released in `onComplete`, never permanent

---

## References

- `references/interaction-patterns.md` — Spring physics elastic followers, spotlight cursor, circuit glow with CSS custom properties
