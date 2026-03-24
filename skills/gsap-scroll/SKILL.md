---
name: gsap-scroll
description: >
  Production recipes for scroll-based GSAP animations in Vue 3 / Nuxt 3.
  Companion to official gsap-scrolltrigger skill (API reference).
  Triggers: ScrollTrigger, scroll animation, scroll reveal, parallax, sticky cards, stacking cards,
  scrub, pin section, batch reveal, masonry reveal, scroll progress, timeline scrub, elastic type,
  section switching, ScrollTrigger.batch, ScrollTrigger.create, scroll refresh, scroll cleanup,
  GSAP scroll, Vue scroll animation, Nuxt scroll animation.
  Non-triggers: Not for mouse-driven interactions (use gsap-interact), text effects (use gsap-text),
  SVG path drawing (use gsap-svg), or visual effects (use gsap-vfx).
  Outcome: Produces scroll-triggered animations — reveals, batch staggers, parallax layers, pinned
  sections, and scrubbed timelines.
---

# GSAP Scroll — ScrollTrigger Patterns

> **Flow**: gsap-setup → gsap-animate → **gsap-scroll** → gsap-optimise → gsap-test
> Cross-reference gsap-animate for context/cleanup, gsap-optimise for batch/scrub tuning.

> **Companion**: For ScrollTrigger API reference, invoke **gsap-scrolltrigger**. This skill covers scroll animation recipes only. Requires: `greensock/gsap-skills`

---

## 1. Basic Scroll Reveal

```js
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
```

- `autoAlpha` not `opacity` — sets `visibility: hidden` at 0
- `toggleActions: 'play none none reverse'` — plays on enter, reverses on leave-back

---

## 2. ScrollTrigger.batch() — Masonry/Grid Reveal

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
    start: 'top 85%', end: 'bottom 15%',
  })
}, sectionRef.value)
```

- `overwrite: 'auto'` prevents conflicting tweens on fast scroll
- Functional `rotation` re-evaluates per element for organic randomness

---

## 3. Parallax Layers

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

- `yPercent` not `y` — percentage-based, scales with element size
- `scrub: 0.5` — smoother than `scrub: true`, 0.5s catch-up

---

## 4. Refresh Patterns

```js
await nextTick(); ScrollTrigger.refresh()                              // v-if toggles
img.addEventListener('load', () => ScrollTrigger.refresh())            // lazy images
document.fonts.ready.then(() => ScrollTrigger.refresh())               // font swap
ScrollTrigger.config({ ignoreMobileResize: true })                     // address bar
```

---

## Advanced Patterns

See `references/scroll-patterns.md` for:
- **Stacking Cards** — CSS sticky + scrubbed two-phase timeline with will-change lifecycle
- **Scrubbed Timeline** — progress bar, bouncing dots, alternating card reveals
- **Elastic Type Assembly** — pin + scrub three-phase scatter/assemble/scatter
- **Service Section Switching** — ScrollTrigger.create() with activate/deactivate callbacks
- **Velocity Skew** — skewY transforms driven by ScrollTrigger.getVelocity() + gsap.quickSetter
- **Infinite Looped Panels** — pinSpacing: false stacking with seamless scroll boundary looping
- **Directionally Aware Header** — show/hide fixed header based on scroll direction
- **Pinned Panels with Overscroll** — slide-based pinning with fake-scroll for tall panels + scale/fade exit
- **Image Mask On Scroll** — before/after image reveal using counter-translating containers
- **Lateral Pin Indicator** — pinned section with side nav indicator + crossfading slide content
- **Horizontal Scrolling Gallery** — horizontal scroll via pin + scrub with xPercent translation

---

## Rules

1. Wrap all ScrollTrigger tweens in `gsap.context()` scoped to a ref
2. Use `autoAlpha` not `opacity` for reveals
3. Include `invalidateOnRefresh: true` when animating positional values
4. Use `overwrite: 'auto'` or `true` when elements can be re-triggered
5. Manage `will-change` lifecycle in ST callbacks (set on enter, release on leave)
6. Call `ScrollTrigger.refresh()` after DOM changes that affect document height
7. Prefer `scrub: 0.5` over `scrub: true` for smoother motion
