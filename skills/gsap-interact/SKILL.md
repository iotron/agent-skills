---
name: gsap-interact
description: >
  Mouse-driven interactive GSAP animations for Vue 3 / Nuxt 3. Covers: 3D tilt cards with
  rotationX/Y from cursor position and inner parallax layers, spring physics elastic cursor
  followers with per-blob durations and homeX/homeY rest positions, spotlight cursor effect
  via animated clipPath circle, magnetic buttons using gsap.quickTo (50-250% faster than
  gsap.to), circuit glow via CSS custom properties with radial gradient masks, and shared
  best practices for event-handler tweens (ctx.add for cleanup, overwrite: 'auto' on every
  rapid-fire tween, will-change lifecycle management, force3D: true on repeatedly animated
  elements, quickTo vs quickSetter selection, transformPerspective set once via gsap.set).
  Triggers: GSAP mouse, mousemove animation, tilt card, 3D tilt, cursor follower, elastic
  blob, spring physics, spotlight effect, clipPath animation, magnetic button, quickTo,
  quickSetter, hover animation, interactive GSAP, cursor effect, parallax hover, circuit
  glow, CSS custom property animation, gsap-interact, mouse-driven animation.
---

# GSAP Interact — Mouse-Driven Animations

> **Flow**: gsap-setup → gsap-animate → **gsap-interact** → gsap-optimise → gsap-test
> **References**: `references/vue-examples.md` for full component code.

All patterns below assume GSAP is registered via the project's `gsap.js` plugin and that `gsap.context()` cleanup is in place (see gsap-animate skill).

---

## 1. 3D Tilt Cards

### Cursor-to-rotation calculation

```js
// Normalise cursor position to -1..+1 range
const rect = card.getBoundingClientRect()
const xPct = ((e.clientX - rect.left) / rect.width - 0.5) * 2   // -1 to +1
const yPct = ((e.clientY - rect.top) / rect.height - 0.5) * 2   // -1 to +1
```

### Shared tween defaults

```js
const TILT_IN  = { duration: 0.35, ease: 'power2.out', overwrite: 'auto' }
const TILT_OUT = { duration: 0.7,  ease: 'elastic.out(1, 0.4)', overwrite: 'auto' }
```

### Full setup with ctx.add

```js
let ctx

onMounted(() => {
  ctx = gsap.context((self) => {
    const card  = cardRef.value
    const inner = innerRef.value

    // Set perspective ONCE — never per-frame
    gsap.set(card,  { transformPerspective: 900, force3D: true })
    gsap.set(inner, { force3D: true })

    self.add('applyTilt', (xPct, yPct) => {
      card.style.willChange  = 'transform'
      inner.style.willChange = 'transform'
      gsap.to(card, {
        rotationY: xPct * 14,
        rotationX: -yPct * 14,
        ...TILT_IN,
      })
      // Inner content parallax shift
      gsap.to(inner, {
        x: xPct * 10,
        y: yPct * 10,
        force3D: true,
        ...TILT_IN,
      })
    })

    self.add('resetTilt', () => {
      gsap.to(card, {
        rotationX: 0,
        rotationY: 0,
        ...TILT_OUT,
        onComplete: () => { card.style.willChange = 'auto' },
      })
      gsap.to(inner, { x: 0, y: 0, force3D: true, ...TILT_OUT,
        onComplete: () => { inner.style.willChange = 'auto' },
      })
    })
  }, scopeRef.value)
})

onUnmounted(() => ctx?.revert())
```

### Template and event handlers

```vue
<div ref="scopeRef" class="tilt-wrapper">
  <div ref="cardRef" class="tilt-shell" @mousemove="onTilt" @mouseleave="ctx.resetTilt()">
    <div ref="innerRef"><slot /></div>
  </div>
</div>
```

```js
function onTilt(e) {
  const rect = cardRef.value.getBoundingClientRect()
  ctx.applyTilt(
    ((e.clientX - rect.left) / rect.width - 0.5) * 2,
    ((e.clientY - rect.top) / rect.height - 0.5) * 2,
  )
}
```

### Required CSS

```css
.tilt-wrapper { perspective: 1000px; }
.tilt-shell   { transform-style: preserve-3d; }
```

**Key rules**: `transformPerspective: 900` via `gsap.set` once (not per-frame) | `force3D: true` on 3D targets | `overwrite: 'auto'` in TILT_IN kills stale tweens | `will-change` set on start, released in `onComplete` of leave

---

## 2. Spring Physics (Elastic Cursor Followers)

Multiple blobs chase the cursor with different durations for a staggered, physics-like feel.

```js
const BLOB_CONFIG = [
  { el: null, duration: 0.6,  homeX: -40, homeY: -20 },
  { el: null, duration: 0.8,  homeX:  30, homeY: -35 },
  { el: null, duration: 1.1,  homeX:   0, homeY:  25 },
]

onMounted(() => {
  ctx = gsap.context((self) => {
    BLOB_CONFIG.forEach((b, i) => {
      b.el = blobRefs.value[i]
      gsap.set(b.el, { x: b.homeX, y: b.homeY, force3D: true })
    })

    self.add('moveBlobs', (mouseX, mouseY) => {
      BLOB_CONFIG.forEach((b) => {
        b.el.style.willChange = 'transform'
        gsap.to(b.el, {
          x: mouseX + b.homeX,
          y: mouseY + b.homeY,
          duration: b.duration,
          ease: 'elastic.out(1.2, 0.3)',
          overwrite: 'auto',
        })
      })
    })

    self.add('resetBlobs', () => {
      BLOB_CONFIG.forEach((b) => {
        gsap.to(b.el, {
          x: b.homeX,
          y: b.homeY,
          duration: 0.8,
          ease: 'elastic.out(1, 0.4)',
          overwrite: 'auto',
          onComplete: () => { b.el.style.willChange = 'auto' },
        })
      })
    })
  }, scopeRef.value)
})
```

**Key rules**: unique `duration` per blob (0.6-1.1s) creates stagger | `elastic.out(1.2, 0.3)` for overshoot | `overwrite: 'auto'` kills stale tweens | `homeX`/`homeY` define rest positions

---

## 3. Spotlight Cursor (Clip-Path Circle)

Reveal a hidden layer by moving a `clipPath` circle to follow the cursor.

```js
onMounted(() => {
  ctx = gsap.context((self) => {
    const spotlight = spotlightRef.value

    gsap.set(spotlight, { clipPath: 'circle(0px at 50% 50%)' })

    self.add('moveSpotlight', (x, y) => {
      gsap.to(spotlight, {
        clipPath: `circle(130px at ${x}px ${y}px)`,
        duration: 0.25,
        ease: 'power2.out',
        overwrite: 'auto',
      })
    })

    self.add('hideSpotlight', () => {
      gsap.to(spotlight, {
        clipPath: 'circle(0px at 50% 50%)',
        duration: 0.4,
        ease: 'power2.inOut',
        overwrite: 'auto',
      })
    })
  }, scopeRef.value)
})
```

Call `ctx.moveSpotlight(localX, localY)` from mousemove, `ctx.hideSpotlight()` from mouseleave.

**Key rules**: `overwrite: 'auto'` prevents pile-up | initial state via `gsap.set` for clean revert | short duration (0.25s) for responsive feel

---

## 4. Magnetic Buttons (quickTo)

`gsap.quickTo` pre-compiles a tween and re-targets it each call -- 50-250% faster than `gsap.to` for frequently updated properties.

```js
let xTo, yTo

onMounted(() => {
  ctx = gsap.context(() => {
    const btn = btnRef.value

    // Pre-create quickTo instances (one per property)
    xTo = gsap.quickTo(btn, 'x', { duration: 0.4, ease: 'power3' })
    yTo = gsap.quickTo(btn, 'y', { duration: 0.4, ease: 'power3' })
  }, scopeRef.value)
})
```

```js
function onMagnetMove(e) {
  const rect = btnRef.value.getBoundingClientRect()
  const offsetX = (e.clientX - rect.left - rect.width / 2) * 0.35
  const offsetY = (e.clientY - rect.top - rect.height / 2) * 0.35
  xTo(offsetX)
  yTo(offsetY)
}

function onMagnetLeave() {
  xTo(0)
  yTo(0)
}
```

**Key rules**: create `quickTo` once in `onMounted`, call in mousemove | one instance per property | no `overwrite` needed (quickTo re-targets internally) | multiplier (0.35) controls pull strength

### quickTo vs quickSetter

| Tool | Use case | Speed |
|------|----------|-------|
| `gsap.quickTo` | Tweened property updates (mousemove with easing) | Fast |
| `gsap.quickSetter` | Instant per-frame value piping (ticker/RAF, no easing) | Fastest |

```js
// quickSetter — for gsap.ticker or requestAnimationFrame
const setX = gsap.quickSetter(el, 'x', 'px')
const setY = gsap.quickSetter(el, 'y', 'px')

gsap.ticker.add(() => {
  setX(currentMouseX)
  setY(currentMouseY)
})
```

---

## 5. Circuit Glow (CSS Custom Properties)

For maximum speed, pipe mouse position directly to CSS custom properties without GSAP tweening. The GPU handles the radial gradient via the shader pipeline.

```js
function onGlowMove(e) {
  const rect = glowRef.value.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  glowRef.value.style.setProperty('--glow-x', `${x}px`)
  glowRef.value.style.setProperty('--glow-y', `${y}px`)
}

function onGlowLeave() {
  glowRef.value.style.setProperty('--glow-x', '50%')
  glowRef.value.style.setProperty('--glow-y', '50%')
}
```

### CSS

```css
.circuit-glow {
  --glow-x: 50%;
  --glow-y: 50%;
  position: relative;
}
.circuit-glow::after {
  content: '';
  position: absolute;
  inset: 0;
  background: radial-gradient(
    350px circle at var(--glow-x) var(--glow-y),
    rgba(0, 255, 170, 0.15),
    transparent 70%
  );
  pointer-events: none;
}
```

**Key rules**: direct `style.setProperty` (no GSAP overhead) | CSS custom properties + `radial-gradient` = GPU-composited | 350px default radius | `pointer-events: none` on pseudo-element

---

## 6. Patterns & Best Practices

### Core rules

```js
// 1. ALL event-handler tweens inside ctx.add for cleanup
self.add('onHover', (el) => { gsap.to(el, { scale: 1.05, overwrite: 'auto', force3D: true }) })  // GOOD
el.addEventListener('mouseenter', () => { gsap.to(el, { scale: 1.05 }) })  // BAD — orphaned

// 2. overwrite: 'auto' on EVERY rapid-fire tween (mousemove = 60+ fires/sec)
gsap.to(el, { x: 100, overwrite: 'auto' })  // GOOD — kills stale tweens
gsap.to(el, { x: 100 })                      // BAD — hundreds pile up

// 3. will-change: set on start, release onComplete (never permanent in CSS)
el.style.willChange = 'transform'
gsap.to(el, { x: 0, onComplete: () => { el.style.willChange = 'auto' } })

// 4. force3D: true — set once, keeps element on compositor layer
gsap.set(el, { force3D: true })
```

### Tool selection guide

| Scenario | Tool | Why |
|----------|------|-----|
| Mousemove with easing | `gsap.quickTo` | Pre-compiled, re-targets instantly |
| Per-frame value piping (ticker/RAF) | `gsap.quickSetter` | Zero tween overhead |
| One-shot hover in/out | `gsap.to` + `overwrite: 'auto'` | Simple, auto-cleans |
| CSS custom properties | Direct `style.setProperty` | No GSAP needed, fastest |

### Cleanup checklist

1. Every interactive tween registered via `ctx.add('name', fn)`
2. `ctx.revert()` called in `onUnmounted`
3. `transformPerspective` set via `gsap.set` (not per-frame)
4. `quickTo` instances created in `onMounted`, not in handlers
5. `will-change` released in `onComplete`, never permanent
