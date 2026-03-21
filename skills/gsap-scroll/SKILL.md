---
name: gsap-scroll
description: >
  ScrollTrigger-based GSAP animation patterns for Vue 3 / Nuxt 3. Covers: basic scroll reveal
  (autoAlpha + translate with toggleActions), ScrollTrigger.batch() for masonry/grid staggered
  reveals, multi-layer parallax with data-speed attributes, sticky stacking cards with scrubbed
  two-phase timelines and will-change lifecycle, scrubbed timeline progress mapping (progress bar +
  alternating cards), elastic type assembly with pin + scrub + scatter phases, ScrollTrigger.create()
  for service section switching with activate/deactivate callbacks, and refresh patterns (nextTick,
  lazy images, fonts, ignoreMobileResize). All examples use gsap.context() for cleanup and real
  component code from this codebase.
  Triggers: ScrollTrigger, scroll animation, scroll reveal, parallax, sticky cards, stacking cards,
  scrub, pin section, batch reveal, masonry reveal, scroll progress, timeline scrub, elastic type,
  section switching, ScrollTrigger.batch, ScrollTrigger.create, scroll refresh, scroll cleanup,
  GSAP scroll, Vue scroll animation, Nuxt scroll animation.
---

# GSAP Scroll -- ScrollTrigger Patterns

> **Flow**: gsap-setup -> gsap-animate -> **gsap-scroll** -> gsap-optimise -> gsap-test
> Cross-reference gsap-animate for context/cleanup, gsap-optimise for batch/scrub tuning.

---

## 1. Basic Scroll Reveal

Elements fade in + translate on scroll entry, reverse on exit.

```js
const { $gsap: gsap, $ScrollTrigger: ScrollTrigger } = useNuxtApp()
const sectionRef = ref(null)
let ctx

onMounted(() => {
  ctx = gsap.context(() => {
    const els = gsap.utils.toArray('.reveal', sectionRef.value)
    gsap.set(els, { y: 28, autoAlpha: 0 })

    els.forEach((el) => {
      gsap.to(el, {
        y: 0, autoAlpha: 1, duration: 0.8, ease: 'power2.out', force3D: true,
        scrollTrigger: {
          trigger: el, start: 'top 88%',
          toggleActions: 'play none none reverse',
          invalidateOnRefresh: true,
        },
      })
    })
  }, sectionRef.value)
})
onUnmounted(() => ctx?.revert())
```

- `autoAlpha` not `opacity` -- sets `visibility: hidden` at 0, prevents pointer events
- `toggleActions: 'play none none reverse'` -- plays on enter, reverses on leave-back
- `invalidateOnRefresh: true` -- recalculates start values on resize

---

## 2. ScrollTrigger.batch() -- Masonry/Grid Reveal

Batch elements entering viewport together. One ST per element, callbacks fire for groups within a time window.

```js
ctx = gsap.context(() => {
  const cards = gsap.utils.toArray('.masonry-card')
  gsap.set(cards, { y: 60, autoAlpha: 0, rotation: () => gsap.utils.random(-3, 3) })

  const ENTER = { y: 0, autoAlpha: 1, rotation: 0, stagger: 0.08, duration: 0.8,
                  ease: 'power3.out', force3D: true, overwrite: 'auto' }
  const EXIT_UP   = { y: -20, autoAlpha: 0, duration: 0.4, ease: 'power2.in', overwrite: 'auto' }
  const EXIT_DOWN = { y: 60,  autoAlpha: 0, duration: 0.4, ease: 'power2.in', overwrite: 'auto' }

  ScrollTrigger.batch(cards, {
    onEnter:     (batch) => gsap.to(batch, ENTER),
    onLeave:     (batch) => gsap.to(batch, EXIT_UP),
    onEnterBack: (batch) => gsap.to(batch, ENTER),
    onLeaveBack: (batch) => gsap.to(batch, EXIT_DOWN),
    start: 'top 85%',
    end: 'bottom 15%',
  })
}, sectionRef.value)
```

- `overwrite: 'auto'` prevents conflicting tweens on fast scroll
- `stagger: 0.08` within batch -- cards enter sequentially, not simultaneously
- Functional `rotation` re-evaluates per element for organic randomness

---

## 3. Parallax Layers

Multiple layers at different speeds via `data-speed`. Scrubbed -- purely scroll-driven.

```html
<div class="parallax-layer" data-speed="0.2"><!-- slow --></div>
<div class="parallax-layer" data-speed="0.6"><!-- fast --></div>
```

```js
ctx = gsap.context(() => {
  gsap.utils.toArray('.parallax-layer', sectionRef.value).forEach((layer) => {
    const speed = parseFloat(layer.dataset.speed) || 0.5
    gsap.to(layer, {
      yPercent: -50 * speed, ease: 'none', force3D: true,
      scrollTrigger: {
        trigger: sectionRef.value, start: 'top bottom', end: 'bottom top', scrub: 0.5, invalidateOnRefresh: true,
      },
    })
  })
}, sectionRef.value)
```

- `yPercent` not `y` -- percentage-based, scales with element size
- `ease: 'none'` -- linear mapping for natural parallax
- `scrub: 0.5` -- smoother than `scrub: true`, 0.5s catch-up
- Higher `data-speed` = more movement = appears closer

---

## 4. Stacking Cards (Sticky + Scrub)

CSS sticky + GSAP scrubbed two-phase timeline: fade/scale in, then stack/scale out.

```css
.stacking-card {
  position: relative;
  transform-origin: center top;
}
@media (min-width: 1024px) {
  .stacking-card {
    position: sticky;
    top: calc(5rem + var(--card-index, 0) * 0.5rem);
  }
}
```

```js
cards.forEach((card, index) => {
  gsap.set(card, { y: 100, autoAlpha: 0, scale: 0.9 })

  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: card, start: 'top 85%', end: 'top 25%', scrub: 0.5,
      invalidateOnRefresh: true,
      onEnter:     () => gsap.set(card, { willChange: 'transform, opacity' }),
      onLeave:     () => gsap.set(card, { willChange: 'auto' }),
      onEnterBack: () => gsap.set(card, { willChange: 'transform, opacity' }),
      onLeaveBack: () => gsap.set(card, { willChange: 'auto' }),
    },
  })

  // Phase 1: reveal
  tl.to(card, { y: 0, autoAlpha: 1, scale: 1, duration: 0.5, ease: 'power2.out', force3D: true })
  // Phase 2: stack with depth
  .to(card, { y: index * -15, scale: 1 - index * 0.015, duration: 0.5, force3D: true })
})
```

- `will-change` set in onEnter, released in onLeave -- avoids permanent GPU cost
- `--card-index` CSS variable offsets sticky `top` so cards stack with slight gaps
- Two-phase timeline: first tween reveals, second creates stacked depth

---

## 5. Scrubbed Timeline (Progress Mapping)

`scrub: 1` maps scroll to timeline progress. Progress bar, bouncing dots, alternating cards.

```js
// Alternating initial offsets
timelineCardRefs.value.forEach((card, i) => {
  if (card) gsap.set(card, { x: i % 2 === 0 ? -50 : 50, autoAlpha: 0, force3D: true })
})

const tl = gsap.timeline({
  scrollTrigger: { trigger: timelineRef.value, start: 'top 70%', end: 'bottom 30%', scrub: 1, invalidateOnRefresh: true },
})

// Progress bar: scaleY 0->1, linear
tl.to(timelineProgressRef.value, { scaleY: 1, ease: 'none', duration: 1 })

// Dots + cards in sequence
milestones.forEach((_, i) => {
  const offset = i * 0.14
  if (timelineDotRefs.value[i])
    tl.to(timelineDotRefs.value[i], { scale: 1, autoAlpha: 1, duration: 0.08, ease: 'back.out(3)', force3D: true }, offset)
  if (timelineCardRefs.value[i])
    tl.to(timelineCardRefs.value[i], { x: 0, autoAlpha: 1, duration: 0.14, ease: 'power2.out', force3D: true }, offset + 0.04)
})
```

- `scrub: 1` -- 1s smooth catch-up (good for complex timelines)
- `ease: 'none'` on progress bar, `back.out(3)` on dots for bounce
- Position offsets (`i * 0.14`) space milestones across the timeline

---

## 6. Elastic Type (Pin + Scrub Assembly)

Pin section, scrub three phases: assemble from random, hold, scatter outward.

```js
gsap.set(chars, {
  y: () => gsap.utils.random(-200, 200), x: () => gsap.utils.random(-100, 100),
  rotation: () => gsap.utils.random(-90, 90), scale: () => gsap.utils.random(0.4, 2.5),
  autoAlpha: 0,
})

const tl = gsap.timeline({
  scrollTrigger: { trigger: elasticPinRef.value, start: 'top top', end: '+=150%', pin: true, scrub: 1, invalidateOnRefresh: true },
})

// Phase 1: assemble
tl.to(chars, {
  y: 0, x: 0, rotation: 0, scale: 1, autoAlpha: 1,
  stagger: { each: 0.03, from: 'random' }, duration: 1, ease: 'elastic.out(1.2, 1)', force3D: true,
})
// Phase 2: hold
tl.to({}, { duration: 0.3 })
// Phase 3: scatter from edges
tl.to(chars, {
  y: () => gsap.utils.random(-300, 300), x: () => gsap.utils.random(-200, 200),
  rotation: () => gsap.utils.random(-120, 120), scale: 0, autoAlpha: 0,
  stagger: { each: 0.02, from: 'edges' }, duration: 0.8, ease: 'power2.in',
})
```

- `pin: true` -- element stays fixed while scroll drives timeline
- `end: '+=150%'` -- scroll distance = 150% of viewport height
- `from: 'random'` on assembly, `from: 'edges'` on scatter
- Empty tween `tl.to({}, { duration: 0.3 })` creates a hold phase

---

## 7. Refresh Patterns

ScrollTrigger caches positions. Refresh after anything that changes layout.

```js
await nextTick(); ScrollTrigger.refresh()                              // v-if toggles
img.addEventListener('load', () => ScrollTrigger.refresh())            // lazy images
document.fonts.ready.then(() => ScrollTrigger.refresh())               // font swap
ScrollTrigger.config({ ignoreMobileResize: true })                     // address bar
```

**useReveal handles fonts automatically** -- `init()` awaits `document.fonts.ready` before context creation.

---

## 8. Service Section Switching

`ScrollTrigger.create()` per section with callbacks -- no tween attached, purely event-driven switching.

```js
const activeIdx = ref(-1)
let revealed = false

// Gate: reveal sidebar + first service when section enters
ScrollTrigger.create({
  trigger: servicesCenter.value, start: 'top 70%', fastScrollEnd: true,
  onEnter: () => { if (!revealed) { revealed = true; switchTo(0) } },
  onLeaveBack: () => { revealed = false; deactivate(activeIdx.value); activeIdx.value = -1 },
})

// Per-category triggers
services.forEach((_, index) => {
  ScrollTrigger.create({
    trigger: categoryRefs.value[index], start: 'top 55%', end: 'bottom 45%', fastScrollEnd: true,
    onEnter:     () => { if (revealed && index > 0) switchTo(index) },
    onLeave:     () => { if (revealed && index < services.length - 1) switchTo(index + 1) },
    onEnterBack: () => { if (revealed) switchTo(index) },
    onLeaveBack: () => { if (revealed && index > 0) switchTo(index - 1) },
  })
})

function switchTo(index) {
  if (activeIdx.value === index) return
  if (activeIdx.value >= 0) deactivate(activeIdx.value)
  activeIdx.value = index
  activate(index)
}

function deactivate(i) {
  gsap.to(bodyRefs.value[i], { autoAlpha: 0, y: -10, duration: 0.3, overwrite: true })
  gsap.to(illustrationRefs.value[i], { autoAlpha: 0, scale: 0.95, duration: 0.3, overwrite: true })
}

function activate(i) {
  gsap.fromTo(bodyRefs.value[i], { autoAlpha: 0, y: 20 },
    { autoAlpha: 1, y: 0, duration: 0.5, ease: 'power2.out', overwrite: true })
  gsap.fromTo(illustrationRefs.value[i], { autoAlpha: 0, scale: 0.9 },
    { autoAlpha: 1, scale: 1, duration: 0.5, ease: 'back.out(1.4)', overwrite: true, force3D: true })
  // Staggered list items
  pointRefs.value[i]?.forEach((li, j) => {
    if (li) gsap.fromTo(li, { autoAlpha: 0, x: -30 },
      { autoAlpha: 1, x: 0, duration: 0.3, delay: 0.15 + j * 0.08, ease: 'power2.out', overwrite: true })
  })
}
```

- `ScrollTrigger.create()` (no tween) -- purely callback-driven
- `fastScrollEnd: true` -- snaps to correct state on fast scroll
- `overwrite: true` kills in-flight tweens; `fromTo` ensures correct start state
- `revealed` gate prevents activation before section enters viewport

---

## Quick Reference

| Property | Value | Use |
|----------|-------|-----|
| `toggleActions` | `'play none none reverse'` | Reveal on enter, undo on leave-back |
| `scrub: true` | 1:1 scroll | Direct control |
| `scrub: 0.5` | 0.5s catch-up | Smooth parallax |
| `scrub: 1` | 1s catch-up | Complex timelines |
| `pin: true` | Fixes element | Full-screen takeover |
| `once: true` | Fire once | One-shot reveals |
| `fastScrollEnd: true` | Snap on fast scroll | Callback sections |
| `invalidateOnRefresh` | Recalc on resize | Responsive values |

## Rules

1. Wrap all ScrollTrigger tweens in `gsap.context()` scoped to a ref
2. Use `autoAlpha` not `opacity` for reveals
3. Include `invalidateOnRefresh: true` when animating positional values
4. Use `overwrite: 'auto'` or `true` when elements can be re-triggered
5. Manage `will-change` lifecycle in ST callbacks (set on enter, release on leave)
6. Call `ScrollTrigger.refresh()` after DOM changes that affect document height
7. Set `ScrollTrigger.config({ ignoreMobileResize: true })` for mobile
8. Prefer `scrub: 0.5` over `scrub: true` for smoother motion
