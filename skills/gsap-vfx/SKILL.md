---
name: gsap-vfx
description: >
  Production recipes for GSAP visual effects.
  Companion to official gsap-core and gsap-timeline skills (API reference).
  Triggers: glitch effect, cyberpunk glitch, RGB displacement, chromatic aberration, scramble text,
  jitter animation, noise bars, marquee, infinite scroll, directional marquee, rolling counter,
  animated number, count up, floating animation, yoyo, pulse ring, repeating timeline, GSAP VFX,
  visual effects, decorative animation, looping animation, footer bounce, morph bounce, elastic footer.
  Non-triggers: Not for scroll-driven reveals (use gsap-scroll), text-only animation (use gsap-text),
  SVG path/morph (use gsap-svg), or mouse-driven interaction (use gsap-interact).
  Outcome: Produces visual effects — glitch/chromatic aberration, infinite marquees, directional marquees,
  rolling counters, floating decorations, pulse rings, footer bounce reveals, and repeating timeline patterns.
---

# GSAP VFX — Visual Effects

> **Flow**: gsap-setup → gsap-animate → **gsap-vfx** → gsap-optimise → gsap-test

> **Companion**: For core tween/timeline API, invoke **gsap-core** and **gsap-timeline**. This skill covers visual effects recipes only. Requires: `greensock/gsap-skills`

---

## 1. Glitch Effect

Full cyberpunk glitch system: 1-second burst every 4 seconds with chromatic aberration, scramble, jitter, noise bars. See `references/glitch-effect.md` for the complete implementation.

---

## 2. Marquee Loop (Infinite Scroll)

```html
<div ref="marqueeRef" class="overflow-hidden">
  <div ref="trackRef" class="flex whitespace-nowrap">
    <span v-for="item in [...items, ...items]" :key="item.id">{{ item.text }}</span>
  </div>
</div>
```

```js
const trackWidth = trackRef.value.scrollWidth / 2
gsap.to(trackRef.value, { x: -trackWidth, duration: 20, repeat: -1, ease: 'none', force3D: true })

// Reverse: gsap.fromTo(track, { x: -trackWidth }, { x: 0, ... })
// Speed-based: duration = trackWidth / pixelsPerSecond
```

`ease: 'none'` is critical — any easing causes visible stuttering at the loop seam.

---

## 3. Rolling Counters

Proxy object pattern — GSAP animates a plain object, `onUpdate` writes to DOM.

```js
function animateCounter(el, target) {
  const proxy = { value: 0 }
  gsap.to(proxy, {
    value: target, duration: 2.2, ease: 'power2.out',
    snap: { value: 1 },
    onUpdate() { el.textContent = String(Math.round(proxy.value)) },
    scrollTrigger: { trigger: el, start: 'top 85%', once: true },
  })
}
counterRefs.value.forEach(el => animateCounter(el, parseInt(el.dataset.target, 10)))
```

- `snap: { value: 1 }` rounds to integers (no flickering decimals)
- `once: true` fires only on first scroll into view

---

## 4. Floating Decorations

```js
gsap.to(decorations, {
  y: 'random(-18, 18)', rotation: 'random(-12, 12)', duration: 'random(3, 5)',
  ease: 'sine.inOut', force3D: true,
  repeat: -1, yoyo: true, repeatRefresh: true,
  stagger: { each: 0.4, repeat: -1, yoyo: true },
})
```

- `repeatRefresh: true` regenerates random values each cycle

---

## 5. Pulse Ring

```js
gsap.fromTo(ring,
  { scale: 1, autoAlpha: 0.7 },
  { scale: 2.2, autoAlpha: 0, duration: 1.5, repeat: -1, ease: 'power1.out', force3D: true }
)

// Multiple staggered rings
rings.forEach((ring, i) => {
  gsap.fromTo(ring,
    { scale: 1, autoAlpha: 0.7 },
    { scale: 2.2, autoAlpha: 0, duration: 1.5, repeat: -1, delay: i * 0.5, force3D: true }
  )
})
```

---

## 6. Directional Marquee (Observer)

Advanced marquee using the official GSAP `horizontalLoop` helper + Observer plugin. Changes direction and speed based on scroll/swipe — unlike the basic marquee above which only scrolls one direction.

```js
gsap.registerPlugin(Observer)
const tl = horizontalLoop(gsap.utils.toArray('.rail h4'), { repeat: -1, paddingRight: 30 })

Observer.create({
  onChangeY(self) {
    let factor = self.deltaY < 0 ? -2.5 : 2.5
    gsap.timeline({ defaults: { ease: 'none' } })
      .to(tl, { timeScale: factor * 2.5, duration: 0.2, overwrite: true })
      .to(tl, { timeScale: factor / 2.5, duration: 1 }, '+=0.3')
  }
})
```

Full `horizontalLoop` helper and Vue 3 adaptation: see `references/vfx-patterns.md`

---

## 7. Footer Bounce (MorphSVG + ScrollTrigger Velocity)

Footer reveal where an SVG path morphs from a curved bulge to flat with elastic bounce scaled by scroll speed.

```js
gsap.registerPlugin(ScrollTrigger, MorphSVGPlugin)
const down   = 'M0-0.3C0-0.3,464,156,1139,156S2278-0.3,2278-0.3V683H0V-0.3z'
const center = 'M0-0.3C0-0.3,464,0,1139,0s1139-0.3,1139-0.3V683H0V-0.3z'

ScrollTrigger.create({
  trigger: '.footer', start: 'top bottom',
  onEnter: self => {
    const variation = self.getVelocity() / 10000
    gsap.fromTo('#bouncy-path',
      { morphSVG: down },
      { duration: 2, morphSVG: center, ease: `elastic.out(${1 + variation}, ${1 - variation})`, overwrite: true }
    )
  }
})
```

Full markup, CSS, and Vue 3 adaptation: see `references/vfx-patterns.md`

---

## 8. Repeating Timeline Patterns

See **gsap-timeline** skill for repeat, repeatDelay, yoyo, and position parameter reference.

```js
const master = gsap.timeline({ repeat: -1, repeatDelay: 4 })
master.add(glitchBurst, 0)
master.add(rgbFlash, 0)
master.set([el, red], { clearProps: 'all' }, 0.9)
```

---

## Anti-Patterns

```js
// BAD: marquee with easing — stutter at loop seam
gsap.to(track, { x: -width, repeat: -1, ease: 'power1.inOut' })

// BAD: counter without snap — flickering decimals
gsap.to(proxy, { value: 100, onUpdate() { el.textContent = proxy.value } })

// BAD: no repeatRefresh — same "random" value every cycle
gsap.to(el, { x: 'random(-50, 50)', repeat: -1, yoyo: true })

// BAD: permanent will-change on glitch layers
// BAD: glitch without height lock — CLS on every burst
// BAD: setInterval instead of repeat: -1
```

---

## References

- `references/glitch-effect.md` — Full glitch implementation: RGB displacement, scramble, jitter, noise bars, GPU lifecycle
- `references/vfx-patterns.md` — Directional marquee (Observer + horizontalLoop helper) and footer bounce (MorphSVG + ScrollTrigger velocity)
