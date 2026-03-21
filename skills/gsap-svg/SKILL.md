---
name: gsap-svg
description: >
  GSAP SVG animation patterns for Vue 3 / Nuxt 3. Covers: SVG path drawing with strokeDashoffset
  (manual and DrawSVG plugin), staggered path + node reveals, infinite pulse rings, SVG morphing
  with MorphSVGPlugin (attr d, scroll-driven scrub, lazy loading), Circuit Tree triple-layer
  animation pattern (fadeAnime breathing, waveAnime DrawSVG 3-phase, colorAnime elastic fill sweep
  with repeatRefresh), CTAHud 5-layer system (reveal, color cycling, wave, blink, glow with
  feGaussianBlur), Scallop Wave ellipse scale pattern, and critical SVG gotchas (fill transparent
  vs none, animation concern ownership, absolute timeline positioning, SVG DOM cleanup).
  Triggers: SVG animation, GSAP SVG, path drawing, strokeDashoffset, DrawSVG, drawSVG, SVG morph,
  MorphSVGPlugin, circuit tree, circuit board, CTAHud, HUD animation, pulse ring, scallop wave,
  SVG stagger, SVG fill, SVG stroke, SVG cleanup, feGaussianBlur, SVG glow, SVG blink.
---

# GSAP SVG — Vue / Nuxt Patterns

> **Flow**: gsap-setup → gsap-animate → **gsap-svg** → gsap-optimise → gsap-test
> **References**: CircuitTree.vue, CTAHud.vue for full component implementations.

---

## 1. SVG Path Drawing (strokeDashoffset)

### Manual approach (no DrawSVG plugin)

```js
const ctx = gsap.context(() => {
  const paths = containerRef.value.querySelectorAll('path')

  paths.forEach((path) => {
    const len = path.getTotalLength()
    gsap.set(path, { strokeDasharray: len, strokeDashoffset: len })
  })

  gsap.to(paths, {
    strokeDashoffset: 0,
    duration: 2,
    ease: 'power2.inOut',
    stagger: { each: 0.08, from: 'start' },
  })
}, containerRef.value)
```

### With DrawSVG plugin (lazy-loaded)

```js
const { $lazyLoadDrawSVG } = useNuxtApp()
await $lazyLoadDrawSVG()

const ctx = gsap.context(() => {
  gsap.from(paths, { drawSVG: '0%', duration: 2, stagger: 0.05 })
  gsap.to(paths, { drawSVG: '40% 80%', duration: 1.5 }) // partial range
}, containerRef.value)
```

### Staggered paths then nodes

```js
const ctx = gsap.context(() => {
  const tl = gsap.timeline()

  tl.to(paths, {
    strokeDashoffset: 0,
    duration: 1.5,
    ease: 'power2.inOut',
    stagger: { each: 0.05, from: 'start' },
  })
  .from(circles, {
    scale: 0, autoAlpha: 0,
    duration: 0.6,
    ease: 'back.out(2)',
    stagger: { each: 0.03, from: 'random' },
  }, '-=0.5')
}, containerRef.value)
```

### Infinite pulse ring

```js
const ctx = gsap.context(() => {
  gsap.fromTo(ringEl,
    { scale: 1, autoAlpha: 0.7 },
    { scale: 2.2, autoAlpha: 0, duration: 2, ease: 'power1.out', repeat: -1, force3D: true }
  )
}, containerRef.value)
```

---

## 2. SVG Morphing (MorphSVGPlugin)

### Animate path d attribute

```js
// <path d="M10 80 Q 95 10 180 80" data-path-to="M10 80 Q 52 150 180 80" />
const pathTo = pathEl.value.dataset.pathTo

const ctx = gsap.context(() => {
  gsap.to(pathEl.value, { attr: { d: pathTo }, duration: 1.5, ease: 'power2.inOut' })
}, containerRef.value)
```

### Scroll-driven morphing

```js
const ctx = gsap.context(() => {
  gsap.to(pathEl.value, {
    attr: { d: pathTo },
    ease: 'none',
    scrollTrigger: { trigger: sectionRef.value, start: 'top center', end: 'bottom center', scrub: 0.5 },
  })
}, containerRef.value)
```

### Lazy loading MorphSVGPlugin

```js
const { $lazyLoadMorphSVG } = useNuxtApp()
await $lazyLoadMorphSVG()

const ctx = gsap.context(() => {
  gsap.to(pathEl.value, { morphSVG: targetPathEl.value, duration: 1.5 })
}, containerRef.value)
```

---

## 3. Circuit Tree Pattern (Triple-Layer Animation)

Three synced 10s timelines via `.add()`. SVG: 209 elements (128 paths + 81 circles).

```js
const ctx = gsap.context(() => {
  const master = gsap.timeline({ repeat: -1 })
  master
    .add(createFadeAnime(allEls), 0)
    .add(createWaveAnime(openPaths), 0)
    .add(createColorAnime(allEls), 0)
}, containerRef.value)
```

### Layer 1: fadeAnime — breathing pulse

```js
function createFadeAnime(els) {
  return gsap.timeline()
    .set(els, { autoAlpha: 0 })
    .to(els, { autoAlpha: 0.3, duration: 3, stagger: { each: 0.03, from: 'random' } })
    .to(els, { autoAlpha: 0.6, duration: 4, yoyo: true, repeat: 3,
      stagger: { each: 0.03, from: 'random' } })
}
```

### Layer 2: waveAnime — DrawSVG 3-phase

```js
function createWaveAnime(openPaths) {
  return gsap.timeline()
    // Build 0-3s
    .fromTo(openPaths, { drawSVG: '0%' },
      { drawSVG: '40% 50%', duration: 3, stagger: { each: 0.03, from: 'random' } }, 0)
    // Peak 3-7s
    .to(openPaths,
      { drawSVG: '73% 80%', duration: 4, stagger: { each: 0.03, from: 'random' } }, 3)
    // Fade 7-10s
    .to(openPaths,
      { drawSVG: '100%', duration: 3, stagger: { each: 0.03, from: 'random' } }, 7)
}
```

### Layer 3: colorAnime — elastic fill sweep

```js
const COLORS = ['#02f4c8', '#41bbf6', '#2dd4bf', '#34d399', '#06b6d4', '#0ea5e9']

function createColorAnime(els) {
  return gsap.timeline({ repeat: -1, repeatRefresh: true })
    .to(els, {
      fill: () => gsap.utils.random(COLORS),
      duration: 7, ease: 'elastic.out(1, 0.3)',
      stagger: { each: 0.03, from: 'random' },
    }, 3)
}
```

`repeatRefresh: true` regenerates random colors each loop. Stagger of 0.03 across 209 elements = ~6.27s spread.

---

## 4. CTAHud 5-Layer System

Five concurrent layers on a HUD circuit board SVG. Reveal plays once; remaining four loop.

```js
const master = gsap.timeline()
master.add(revealTl, 0)       // once
master.add(colorTl, 1.5)      // looping layers start after reveal
master.add(waveTl, 1.5)
master.add(blinkTl, 1.5)
master.add(glowTl, 1.5)
```

**Layer 1 — Reveal**: DrawSVG from center, plays once.
```js
revealTl.from(allPaths, {
  drawSVG: '50% 50%', duration: 1.5, ease: 'power2.inOut',
  stagger: { each: 0.02, from: 'edges' },
})
```

**Layer 2 — Color cycling**: random teal/cyan fills with `repeatRefresh: true`, elastic easing, `from: 'edges'` stagger.

**Layer 3 — Wave**: sweeping DrawSVG 73-80% to 100%, 5s yoyo.

**Layer 4 — Blink**: circles `scale: 1.3`, `transformOrigin: '50% 50%'`, random stagger, yoyo.

**Layer 5 — Glow**: animate `feGaussianBlur` stdDeviation via `attr: { stdDeviation: 4 }`.

```html
<filter id="hud-glow"><feGaussianBlur class="blur-node" stdDeviation="10" /></filter>
```

```js
glowTl = gsap.timeline({ repeat: -1, yoyo: true })
glowTl.to('.blur-node', { attr: { stdDeviation: 4 }, duration: 1.5, ease: 'sine.inOut' })
```

---

## 5. Scallop Wave

SVG ellipses scale up from center outward with pronounced overshoot.

```js
const ctx = gsap.context(() => {
  gsap.from(ellipses, {
    scale: 0, autoAlpha: 0,
    transformOrigin: 'top center',
    duration: 0.8,
    ease: 'back.out(3)',
    force3D: true,
    stagger: { each: 0.04, from: 'center' },
    scrollTrigger: { trigger: containerRef.value, start: 'top 80%', toggleActions: 'play none none reverse' },
  })
}, containerRef.value)
```

- `transformOrigin: 'top center'` — grow downward from top edge
- `back.out(3)` — pronounced overshoot before settling
- `from: 'center'` — ripples outward from middle elements

---

## 6. Critical Gotchas

### fill: transparent NOT fill: none

`none` means "unpainted" in SVG spec -- not a color. GSAP cannot interpolate from it. Use `transparent` (`rgba(0,0,0,0)`) which is a real color value.

```js
gsap.set(paths, { fill: 'transparent' }) // GOOD — interpolates to any color
gsap.set(paths, { fill: 'none' })        // BAD — jumps instantly, no transition
```

### Separate animation concern ownership

Concurrent timelines on the same elements must own **different** properties. Wave owns stroke (dashoffset/dasharray). Color owns fill + autoAlpha. Never let two timelines fight over the same property.

### Absolute timeline positioning

Use `.add(animation, timeInSeconds)` not relative offsets for multi-layer compositions. Absolute positions don't shift when sibling animation durations change.

```js
master.add(fadeAnime, 0)    // 0s — deterministic
master.add(colorAnime, 3)   // 3s — won't shift if fadeAnime changes
// AVOID: master.add(colorAnime, '+=1') — shifts with fadeAnime
```

### SVG DOM cleanup

```js
onUnmounted(() => {
  ctx?.revert()
  circuitWrap.value?.replaceChildren() // clear SVG DOM — revert only kills tweens
})
```

`replaceChildren()` removes dynamically generated SVG nodes that `ctx.revert()` does not touch.

---

## Quick Reference

| Pattern | Plugin | Key Properties | Duration |
|---------|--------|----------------|----------|
| Path draw (manual) | None | strokeDasharray, strokeDashoffset | 1.5-2s |
| Path draw (plugin) | DrawSVG | drawSVG: '0%' to '100%' | 1.5-2s |
| Morph | MorphSVGPlugin | attr.d or morphSVG | 1-2s |
| Pulse ring | None | scale, autoAlpha, repeat: -1 | 2s/cycle |
| Color sweep | None | fill, repeatRefresh: true | 5-7s/cycle |
| Filter glow | None | attr.stdDeviation | 1.5s yoyo |
| Scallop wave | None | scale, back.out(3), from: 'center' | 0.8s + stagger |
