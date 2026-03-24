---
name: gsap-svg
description: >
  Production recipes for GSAP SVG animations.
  Companion to official gsap-plugins skill (API reference).
  Triggers: SVG animation, GSAP SVG, path drawing, strokeDashoffset, DrawSVG, drawSVG, SVG morph,
  MorphSVGPlugin, circuit tree, circuit board, CTAHud, HUD animation, pulse ring, scallop wave,
  SVG stagger, SVG fill, SVG stroke, SVG cleanup, feGaussianBlur, SVG glow, SVG blink,
  shape overlay, page transition, section transition, smooth morph, morph interpolation,
  dynamic morphing, SVG curtain, shape transition.
  Non-triggers: Not for text animation (use gsap-text), scroll reveals without SVG (use gsap-scroll),
  mouse interactions (use gsap-interact), or non-SVG visual effects (use gsap-vfx).
  Outcome: Produces SVG animations — path drawing, morphing, circuit board patterns, HUD systems,
  pulse rings, and scallop waves.
---

# GSAP SVG — Vue / Nuxt Patterns

> **Flow**: gsap-setup → gsap-animate → **gsap-svg** → gsap-optimise → gsap-test

> **Companion**: For DrawSVG/MorphSVG API reference, invoke **gsap-plugins**. This skill covers SVG animation recipes only. Requires: `greensock/gsap-skills`

---

## 1. SVG Path Drawing (strokeDashoffset)

### Manual approach

```js
const ctx = gsap.context(() => {
  const paths = containerRef.value.querySelectorAll('path')
  paths.forEach((path) => {
    const len = path.getTotalLength()
    gsap.set(path, { strokeDasharray: len, strokeDashoffset: len })
  })
  gsap.to(paths, {
    strokeDashoffset: 0, duration: 2, ease: 'power2.inOut',
    stagger: { each: 0.08, from: 'start' },
  })
}, containerRef.value)
```

### With DrawSVG plugin (lazy-loaded)

```js
await $lazyLoadDrawSVG()
gsap.from(paths, { drawSVG: '0%', duration: 2, stagger: 0.05 })
gsap.to(paths, { drawSVG: '40% 80%', duration: 1.5 }) // partial range
```

### Staggered paths then nodes

```js
const tl = gsap.timeline()
tl.to(paths, { strokeDashoffset: 0, duration: 1.5, ease: 'power2.inOut', stagger: { each: 0.05 } })
  .from(circles, { scale: 0, autoAlpha: 0, duration: 0.6, ease: 'back.out(2)',
    stagger: { each: 0.03, from: 'random' } }, '-=0.5')
```

### Infinite pulse ring

```js
gsap.fromTo(ringEl,
  { scale: 1, autoAlpha: 0.7 },
  { scale: 2.2, autoAlpha: 0, duration: 2, ease: 'power1.out', repeat: -1, force3D: true }
)
```

---

## 2. SVG Morphing (MorphSVGPlugin)

```js
const pathTo = pathEl.value.dataset.pathTo
gsap.to(pathEl.value, { attr: { d: pathTo }, duration: 1.5, ease: 'power2.inOut' })

// Scroll-driven
gsap.to(pathEl.value, {
  attr: { d: pathTo }, ease: 'none',
  scrollTrigger: { trigger: sectionRef.value, start: 'top center', end: 'bottom center', scrub: 0.5 },
})

// With MorphSVGPlugin
await $lazyLoadMorphSVG()
gsap.to(pathEl.value, { morphSVG: targetPathEl.value, duration: 1.5 })
```

---

## 3. Scallop Wave

```js
gsap.from(ellipses, {
  scale: 0, autoAlpha: 0, transformOrigin: 'top center',
  duration: 0.8, ease: 'back.out(3)', force3D: true,
  stagger: { each: 0.04, from: 'center' },
  scrollTrigger: { trigger: containerRef.value, start: 'top 80%', toggleActions: 'play none none reverse' },
})
```

---

## 4. Critical Gotchas

### fill: transparent NOT fill: none
`none` means "unpainted" — GSAP cannot interpolate from it. Use `transparent`.

### Separate animation concern ownership
Concurrent timelines must own **different** properties. Wave owns stroke. Color owns fill + autoAlpha.

### Absolute timeline positioning
Use `.add(animation, timeInSeconds)` not relative offsets for multi-layer compositions.

### SVG DOM cleanup
```js
onUnmounted(() => {
  ctx?.revert()
  circuitWrap.value?.replaceChildren() // clear SVG DOM
})
```

---

## References

- `references/svg-patterns.md` — Circuit Tree triple-layer animation, CTAHud 5-layer system, Dynamic Morphing shape overlays, Smooth Morph with interpolation
