---
name: gsap-cursor
description: >
  Production recipes for cursor and pointer-driven GSAP animations in Vue 3 / Nuxt 3.
  Companion to official gsap-core and gsap-performance skills (API reference).
  Triggers: cursor follower, cursor effect, mouse follower, magnetic button, spotlight cursor,
  cursor trail, particle trail, spring cursor, elastic cursor, pointer animation, custom cursor,
  cursor glow, clipPath cursor, quickTo cursor, image preview hover, perspective tilt, 3D tilt cursor.
  Non-triggers: Not for hover/tilt card effects (use gsap-interact), not for scroll
  animations (use gsap-scroll), not for general mouse events without cursor effects.
  Outcome: Produces cursor-driven animation effects with proper cleanup, overwrite management,
  and performance optimization (quickTo, ticker, will-change lifecycle).
---

# GSAP Cursor — Pointer-Driven Effects

> **Flow**: gsap-setup → gsap-animate → **gsap-cursor** → gsap-optimise → gsap-test

> **Companion**: For GSAP core tween API, invoke **gsap-core**. For performance methods, invoke **gsap-performance**. This skill covers cursor effect recipes only. Requires: `greensock/gsap-skills`

All patterns assume `gsap.context()` cleanup is in place (see gsap-animate skill).

---

## 1. Magnetic Buttons (quickTo)

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

> See **gsap-performance** skill for quickTo vs quickSetter API comparison.

---

## 2. Flair Particle Trail

Distance-based particle spawning using `gsap.ticker` and pool recycling. Elements reappear at the cursor, animate out, then recycle via `gsap.utils.wrap`.

**Core loop** (runs every tick via `gsap.ticker.add`):

```js
gsap.ticker.add(() => {
  const dist = Math.hypot(lastPos.x - mousePos.x, lastPos.y - mousePos.y)
  if (dist > gap) { spawnParticle(); lastPos = mousePos }
})
```

**Spawn**: `gsap.set` positions element at cursor, `gsap.utils.wrap(0, pool.length)` cycles the index, timeline scales + fades + drops the particle.

> Full implementation with pool setup and animation timeline in `references/cursor-patterns.md`.

---

## 3. Spring Physics Elastic Followers

Multiple blobs chase the cursor with different `duration` values for a staggered, physics-like feel. Each blob uses `elastic.out` easing and `overwrite: 'auto'`.

```js
self.add('moveBlobs', (mx, my) => {
  blobs.forEach((blob, i) => {
    gsap.to(blob.el, {
      x: mx, y: my,
      duration: blob.dur, ease: 'elastic.out(1.2, 0.3)',
      overwrite: 'auto', force3D: true,
    })
  })
})
```

Rest positions are **percentage-based** (`containerWidth * pct / 100`), recalculated on reset.

> Full implementation with blob config, setup, and reset in `references/cursor-patterns.md`.

---

## 4. Spotlight Cursor (Clip-Path Circle)

Reveal a hidden layer by animating a `clipPath` circle to follow the cursor.

```js
self.add('moveSpotlight', (x, y) => {
  gsap.to(spotlight, {
    clipPath: `circle(130px at ${x}px ${y}px)`,
    duration: 0.25, ease: 'power2.out', overwrite: 'auto',
  })
})
```

Initial state: `gsap.set(spotlight, { clipPath: 'circle(0px at 50% 50%)' })` for clean revert.

> Full implementation with hide animation in `references/cursor-patterns.md`.

---

## 5. Cursor-Tracking Image Preview

Images follow the cursor when hovering over list items. Uses `gsap.quickTo` with a two-arg snap on first enter to avoid interpolation lag.

```js
const setX = gsap.quickTo(image, "x", { duration: 0.4, ease: "power3" })
// On first enter, snap: setX(e.clientX, e.clientX)
// On subsequent moves: setX(e.clientX)
```

Paused `autoAlpha` tween toggles visibility via `play()`/`reverse()` on mouseenter/mouseleave.

> Full implementation in `references/cursor-patterns.md`.

---

## 6. Cursor-Driven Perspective Tilt

3D perspective tilt driven by pointer position. Parent gets `perspective: 650`, children rotate/translate via `gsap.quickTo` + `gsap.utils.interpolate`.

```js
gsap.set("main", { perspective: 650 })
const outerRX = gsap.quickTo(".card", "rotationX", { ease: "power3" })
// pointermove: outerRX(gsap.utils.interpolate(15, -15, e.y / innerHeight))
// pointerleave: outerRX(0)
```

Outer element rotates (rotationX/Y), inner element translates (x/y) for parallax depth.

> Full implementation in `references/cursor-patterns.md`.

---

## 7. Best Practices

```js
// 1. overwrite: 'auto' on EVERY rapid-fire tween — prevents pile-up
gsap.to(el, { x: 100, overwrite: 'auto' })

// 2. ctx.add for cleanup — all event-handler tweens registered via ctx.add
self.add('onMove', (x, y) => { gsap.to(el, { x, y, overwrite: 'auto' }) })

// 3. will-change lifecycle — set on start, release onComplete
el.style.willChange = 'transform'
gsap.to(el, { x: 0, onComplete: () => { el.style.willChange = 'auto' } })

// 4. force3D: true on repeatedly animated elements
gsap.set(el, { force3D: true })

// 5. quickTo in onMounted, not in handlers — create once, call many
```

### Cleanup checklist

1. Every cursor tween registered via `ctx.add('name', fn)`
2. `ctx.revert()` called in `onUnmounted`
3. `quickTo` instances created in `onMounted`, not in handlers
4. `will-change` released in `onComplete`, never permanent
5. `gsap.ticker.remove(fn)` in cleanup for ticker-based effects

---

## References

- `references/cursor-patterns.md` — Flair particle trail, spring physics followers, spotlight cursor, circuit glow, cursor-tracking image preview, cursor-driven perspective tilt
