---
name: gsap-canvas
description: >
  Production recipes for GSAP animations rendered to HTML5 Canvas.
  Companion to official gsap-core and gsap-timeline skills (API reference).
  Triggers: canvas animation, GSAP canvas, particle system, canvas particles, canvas rendering,
  onUpdate canvas, draw loop, sprite animation, canvas timeline, canvas GSAP, canvas morph,
  MorphSVG canvas, shape morphing canvas.
  Non-triggers: Not for DOM-based animation (use gsap-animate), not for SVG DOM morphing (use gsap-svg),
  not for WebGL/Three.js.
  Outcome: Produces canvas-based animations using GSAP timelines with custom onUpdate
  rendering pipelines, particle systems, and resize handling.
---

# GSAP Canvas — Rendering Patterns

> **Flow**: gsap-setup → **gsap-canvas** → gsap-optimise → gsap-test

> **Companion**: For GSAP core API reference, invoke **gsap-core**. For timeline sequencing, invoke **gsap-timeline**. This skill covers canvas rendering recipes only. Requires: `greensock/gsap-skills`

---

## 1. The Pattern

Canvas animation with GSAP is fundamentally different from DOM animation. GSAP never touches a DOM element directly. Instead:

1. **Create plain JS objects** with numeric properties (x, y, scale, rotation, alpha).
2. **Animate those objects** with a GSAP timeline or tween — GSAP interpolates the numbers.
3. **Attach an `onUpdate` callback** to the timeline that reads the current object values and draws them to canvas using the Canvas 2D API.

```
Plain objects  →  GSAP tweens numbers  →  onUpdate draws to canvas
     ↑                                          ↓
  { x, y, scale }                      ctx.drawImage(img, x, y, w*scale, h*scale)
```

This means GSAP handles easing, stagger, repeat, yoyo, and timeline sequencing — while you own the rendering pipeline entirely.

```js
// Minimal example: animate a circle across canvas
const dot = { x: 0, y: 250, radius: 20 }

gsap.to(dot, {
  x: 800, duration: 2, ease: 'power2.inOut',
  onUpdate() {
    ctx.clearRect(0, 0, cw, ch)
    ctx.beginPath()
    ctx.arc(dot.x, dot.y, dot.radius, 0, Math.PI * 2)
    ctx.fill()
  },
})
```

---

## 2. Particle System Recipe

Animate an array of particles with sprite images orbiting inward. GSAP handles all motion; `onUpdate` redraws every frame.

### Setup particle array

```js
const particles = Array.from({ length: 99 }, (_, i) => ({
  x: 0, y: 0, scale: 0, rotate: 0,
  img: Object.assign(new Image(), {
    src: `https://assets.codepen.io/16327/flair-${2 + (i % 21)}.png`,
  }),
}))
```

### Build the timeline

```js
const radius = Math.max(cw, ch)

const tl = gsap.timeline({ onUpdate: draw })
  .fromTo(particles, {
    x: (i) => {
      const angle = (i / particles.length) * Math.PI * 2 - Math.PI / 2
      return Math.cos(angle * 10) * radius
    },
    y: (i) => {
      const angle = (i / particles.length) * Math.PI * 2 - Math.PI / 2
      return Math.sin(angle * 10) * radius
    },
    scale: 1.1,
    rotate: 0,
  }, {
    duration: 5, ease: 'sine',
    x: 0, y: 0, scale: 0, rotate: -3,
    stagger: { each: -0.05, repeat: -1 },
  }, 0)
  .seek(99) // jump ahead so repeating stagger is mid-flow
```

### The draw function

```js
function draw() {
  particles.sort((a, b) => a.scale - b.scale) // z-sort by scale
  ctx.clearRect(0, 0, cw, ch)
  particles.forEach((p) => {
    ctx.translate(cw / 2, ch / 2)
    ctx.rotate(p.rotate)
    ctx.drawImage(p.img, p.x, p.y, p.img.width * p.scale, p.img.height * p.scale)
    ctx.resetTransform()
  })
}
```

---

## 3. Controls & Resize

### Play/pause toggle via timeScale

```js
canvas.addEventListener('pointerup', () => {
  gsap.to(tl, {
    timeScale: tl.isActive() ? 0 : 1, // smooth ease to pause/play
  })
})
```

### Resize handler with invalidate

```js
window.addEventListener('resize', () => {
  cw = canvas.width = innerWidth
  ch = canvas.height = innerHeight
  radius = Math.max(cw, ch)
  tl.invalidate() // recalculates functional from-values on next render
})
```

`invalidate()` forces GSAP to re-evaluate function-based values (the trig calculations) using updated dimensions.

---

## 4. Performance Tips

| Tip | Why |
|---|---|
| Use `gsap.ticker.add(fn)` for render loops | Syncs with GSAP's internal rAF — one frame budget, no double-paints |
| Minimize state changes | Batch `translate`/`rotate` calls; call `resetTransform()` once per particle |
| Sort sparingly | `particles.sort()` every frame is O(n log n) — skip if z-order is fixed |
| Use `will-change: transform` on the `<canvas>` | Promotes to GPU layer, reduces compositing cost |
| Prefer `ctx.resetTransform()` over save/restore | Faster — avoids stack push/pop overhead |
| Pre-render to offscreen canvas | For static sprites, draw once to `OffscreenCanvas`, then `drawImage` from it |

---

## References

- `references/canvas-patterns.md` — Full implementations: particle orbit system, canvas morphs (MorphSVG rendered to canvas)
