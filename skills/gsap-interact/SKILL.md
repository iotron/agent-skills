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

Multiple blobs chase the cursor with different durations for a staggered, physics-like feel. Rest positions are **percentage-based** relative to the container, so they adapt to any container size.

```js
// Blob config — x/y are PERCENTAGES (0-100) of container dimensions
const blobs = [
  { x: 15, y: 40, size: 90,  dur: 0.9, colorStop: 'rgba(20,184,166,0.4)' },
  { x: 30, y: 55, size: 55,  dur: 0.6, colorStop: 'rgba(var(--glow-secondary-rgb),0.35)' },
  { x: 55, y: 30, size: 120, dur: 1.1, colorStop: 'rgba(var(--glow-primary-rgb),0.3)' },
  { x: 68, y: 65, size: 60,  dur: 0.7, colorStop: 'rgba(6,182,212,0.35)' },
  { x: 80, y: 25, size: 75,  dur: 0.8, colorStop: 'rgba(20,184,166,0.35)' },
]

const containerRef = ref(null)
const blobRefs = ref([])
let ctx
```

### Initial placement — percentage of container

```js
const setupVisuals = () => {
  const container = containerRef.value
  if (!container) return
  const cw = container.offsetWidth
  const ch = container.offsetHeight

  blobRefs.value.forEach((blob, i) => {
    if (!blob) return
    const b = blobs[i]
    blob.style.width = `${b.size}px`
    blob.style.height = `${b.size}px`
    blob.style.background = `radial-gradient(circle, ${b.colorStop}, transparent)`
    blob.style.filter = `blur(${Math.round(b.size / 4)}px)`
    // Position at percentage of container; xPercent/yPercent: -50 centres the blob on the point
    gsap.set(blob, { xPercent: -50, yPercent: -50, x: (cw * b.x) / 100, y: (ch * b.y) / 100 })
  })
}
```

### moveBlobs — blobs go to exact cursor position

```js
onMounted(() => {
  ctx = gsap.context((self) => {
    setupVisuals()

    // mx, my are local coords (e.clientX - rect.left, e.clientY - rect.top)
    self.add('moveBlobs', (mx, my) => {
      blobRefs.value.forEach((blob, i) => {
        if (!blob) return
        gsap.to(blob, {
          x: mx,                        // exact cursor position, NOT mx + offset
          y: my,
          duration: blobs[i].dur,       // unique per blob → staggered elastic feel
          ease: 'elastic.out(1.2, 0.3)',
          overwrite: 'auto',
          force3D: true,
        })
      })
    })

    // resetBlobs recalculates container dimensions for fresh percentage positions
    self.add('resetBlobs', () => {
      const container = containerRef.value
      if (!container) return
      const cw = container.offsetWidth
      const ch = container.offsetHeight
      blobRefs.value.forEach((blob, i) => {
        if (!blob) return
        const b = blobs[i]
        gsap.to(blob, {
          x: (cw * b.x) / 100,
          y: (ch * b.y) / 100,
          duration: 1.6,                // slower return
          ease: 'elastic.out(1, 0.4)',  // softer spring on reset
          force3D: true,                // no overwrite needed — single reset call
        })
      })
    })
  }, containerRef.value?.closest('section'))
})
```

### Event wiring

```js
const onMouseMove = (e) => {
  const container = containerRef.value
  if (!container || !ctx) return
  const rect = container.getBoundingClientRect()
  ctx.moveBlobs(e.clientX - rect.left, e.clientY - rect.top)
}

const onMouseLeave = () => { ctx?.resetBlobs() }
```

**Key rules**: rest positions are **percentage-based** (`cw * b.x / 100`) so layout is responsive | on mousemove blobs go to the **exact** cursor position (no offset added) | `resetBlobs` re-reads container dimensions for accurate return | unique `duration` per blob (0.6-1.1s) creates stagger | `elastic.out(1.2, 0.3)` for overshoot on move, softer `elastic.out(1, 0.4)` with longer 1.6s duration on reset | `overwrite: 'auto'` on move tweens kills stale tweens

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

## 5. Circuit Glow (CSS Custom Properties + mask-image)

For maximum speed, pipe mouse position directly to CSS custom properties without GSAP tweening. Uses a **real div element** (not a pseudo-element) with `mask-image` to reveal a glow layer. Opacity is CSS-transition-based, triggered by parent `:hover`.

### Template — real div, not ::after

```vue
<div class="relative w-full" @mousemove="onMouseMove">
  <!-- Main content (z-0) -->
  <div ref="circuitWrap" class="relative z-0" />

  <!-- Cursor glow — real div, absolute overlay -->
  <div ref="glowRef" class="circuit-glow absolute inset-0 z-[1]" style="--glow-x: 0.5; --glow-y: 0.5" />
</div>
```

### JS — unitless ratios (0-1), not pixels

```js
const glowRef = ref(null)

const onMouseMove = (e) => {
  if (!glowRef.value) return
  const rect = e.currentTarget.getBoundingClientRect()
  // Store UNITLESS RATIOS (0 to 1), not pixel values
  glowRef.value.style.setProperty('--glow-x', (e.clientX - rect.left) / rect.width)
  glowRef.value.style.setProperty('--glow-y', (e.clientY - rect.top) / rect.height)
}
```

No `onMouseLeave` handler needed — opacity fades out via CSS transition when cursor leaves the parent.

### CSS — mask-image with calc(), parent :hover for opacity

```css
.circuit-glow {
  /* Use a background-image of the circuit SVG with color filters */
  background-image: url('~/assets/images/circuit-board.svg');
  @apply bg-cover bg-center bg-no-repeat opacity-0;
  filter: invert(1) sepia(1) saturate(5) hue-rotate(130deg) brightness(0.8);

  /* mask-image uses calc(var * 100%) to convert unitless ratio → percentage position */
  mask-image: radial-gradient(
    circle 350px at calc(var(--glow-x) * 100%) calc(var(--glow-y) * 100%),
    black 0%, transparent 100%
  );
  -webkit-mask-image: radial-gradient(
    circle 350px at calc(var(--glow-x) * 100%) calc(var(--glow-y) * 100%),
    black 0%, transparent 100%
  );
}

/* Opacity controlled by CSS transition on parent hover — NOT GSAP */
div:hover .circuit-glow {
  @apply opacity-35 transition-opacity duration-500;
}
```

**Key rules**: real `<div>` element (not `::after` pseudo) | stores **unitless ratios** (0-1) via `style.setProperty`, CSS uses `calc(var(--glow-x) * 100%)` to position | `mask-image` with `radial-gradient` reveals the glow area | opacity is **CSS-transition-based** (`opacity-0` default, `opacity-35` on parent `:hover` with `transition-opacity duration-500`) — no GSAP involved | direct `style.setProperty` (no GSAP overhead) for cursor tracking

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
