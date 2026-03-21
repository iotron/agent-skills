---
name: gsap-vfx
description: >
  GSAP visual effects for Vue 3 / Nuxt 3. Covers: cyberpunk glitch effect (RGB chromatic aberration,
  per-word ScrambleText, jitter keyframes with clip-path, noise bars, GPU promote/release lifecycle,
  badge hue-rotate sync, CLS-safe wrapper height lock), infinite marquee loop (duplicated track,
  calculated trackWidth, bidirectional scroll, force3D compositing), rolling number counters
  (proxy object pattern, snap rounding, ScrollTrigger once), floating decorations (yoyo stagger,
  random ranges for y/rotation/duration, sine easing), pulse ring (scale + autoAlpha fade loop
  after SVG draw), and repeating timeline patterns (repeat, repeatDelay, repeatRefresh, .add()
  absolute positioning). Triggers: glitch effect, cyberpunk glitch, RGB displacement, chromatic
  aberration, scramble text, jitter animation, noise bars, marquee, infinite scroll, rolling
  counter, animated number, count up, floating animation, yoyo, pulse ring, repeating timeline,
  GSAP VFX, visual effects, decorative animation, looping animation.
---

# GSAP VFX — Visual Effects

> **Flow**: gsap-setup → gsap-animate → **gsap-vfx** → gsap-optimise → gsap-test

---

## 1. Glitch Effect (RGB Displacement + Jitter)

Full cyberpunk glitch system: 1-second burst every 4 seconds with chromatic aberration, text scramble, jitter, and noise bars.

### Timeline structure

```js
const glitchTl = gsap.timeline({ repeat: -1, repeatDelay: 4, delay: 4, paused: true })
```

### RGB chromatic aberration

Two overlay layers clipped identically to the text, offset in opposite directions.

```js
glitchTl.to(redLayer, { x: -4, y: -2, autoAlpha: 0.8, duration: 0.06, ease: 'steps(1)', force3D: true }, 0)
glitchTl.to(blueLayer, { x: 4, y: 1, autoAlpha: 0.8, duration: 0.06, ease: 'steps(1)', force3D: true }, 0)
```

`ease: 'steps(1)'` creates instant frame-snapping — no interpolation between keyframes.

### Per-word scramble

```js
words.forEach((word, i) => {
  glitchTl.to(word, {
    scrambleText: { text: '{original}', chars: '!@#$%^&*', speed: 0.6 },
    duration: 0.6,
  }, i * 0.05) // staggered starts
})
```

### Jitter keyframes (applyJitter pattern)

12 keyframes at 0.06s intervals with random transforms and clip-path insets.

```js
function applyJitter(el) {
  gsap.set(el, {
    skewX: gsap.utils.random(-3, 3),
    x: gsap.utils.random(-6, 6),
    clipPath: `inset(${gsap.utils.random(0, 40)}% 0 ${gsap.utils.random(0, 40)}% 0)`,
  })
}

for (let i = 0; i < 12; i++) {
  glitchTl.call(applyJitter, [textEl], i * 0.06)
}
```

### Noise bars

4 horizontal bars at different vertical positions (scaleX 0 → 1 → 0).

```js
const barPositions = [10, 35, 60, 85] // % from top
barPositions.forEach((top, i) => {
  glitchTl.fromTo(bars[i],
    { scaleX: 0, autoAlpha: 0.6 },
    { scaleX: 1, autoAlpha: 0, duration: 0.3, ease: 'power2.in', force3D: true },
    i * 0.08
  )
})
```

### GPU lifecycle

Promote at burst start, release at burst end.

```js
function promoteGPU(els) {
  els.forEach(el => { el.style.willChange = 'transform, opacity, clip-path' })
}
function releaseGPU(els) {
  els.forEach(el => { el.style.willChange = 'auto' })
}

glitchTl.call(promoteGPU, [allGlitchEls], 0)
glitchTl.call(releaseGPU, [allGlitchEls], 0.9)
```

### Badge hue-rotate flash (synced with burst)

```js
glitchTl.to(badge, {
  filter: 'hue-rotate(90deg)', duration: 0.1, yoyo: true, repeat: 3, ease: 'steps(1)',
}, 0)
```

### CLS-safe wrapper height + cleanup at 0.9s

```js
// Lock height to prevent layout shift
onMounted(() => {
  wrapper.value.style.minHeight = `${wrapper.value.offsetHeight}px`
})

// All effects resolve with 0.1s fade to clean state
glitchTl.to([redLayer, blueLayer], { x: 0, y: 0, autoAlpha: 0, duration: 0.1 }, 0.9)
glitchTl.set(textEl, { skewX: 0, x: 0, clipPath: 'inset(0 0 0 0)' }, 0.9)
```

---

## 2. Marquee Loop (Infinite Scroll)

Continuous horizontal scroll using a duplicated track for seamless looping.

```html
<div ref="marqueeRef" class="overflow-hidden">
  <div ref="trackRef" class="flex whitespace-nowrap">
    <span v-for="item in [...items, ...items]" :key="item.id">{{ item.text }}</span>
  </div>
</div>
```

### Forward direction

```js
const track = trackRef.value
const trackWidth = track.scrollWidth / 2

gsap.to(track, { x: -trackWidth, duration: 20, repeat: -1, ease: 'none', force3D: true })
```

`ease: 'none'` is critical — any easing causes visible stuttering at the loop seam.

### Reverse direction

```js
gsap.fromTo(track,
  { x: -trackWidth },
  { x: 0, duration: 20, repeat: -1, ease: 'none', force3D: true }
)
```

### Speed based on content width

```js
const pixelsPerSecond = 60
const duration = trackWidth / pixelsPerSecond
```

---

## 3. Rolling Counters (Animated Numbers)

Proxy object pattern — GSAP animates a plain object, `onUpdate` writes to the DOM.

```js
const ctx = gsap.context(() => {
  function animateCounter(el, target) {
    const proxy = { value: 0 }
    gsap.to(proxy, {
      value: target,
      duration: 2.2,
      ease: 'power2.out',
      snap: { value: 1 },
      onUpdate() { el.textContent = String(Math.round(proxy.value)) },
      scrollTrigger: { trigger: el, start: 'top 85%', once: true },
    })
  }

  // Multiple counters
  counterRefs.value.forEach(el => {
    animateCounter(el, parseInt(el.dataset.target, 10))
  })
}, containerRef.value)
```

- `snap: { value: 1 }` rounds to integers during animation (no flickering decimals)
- `once: true` fires the counter only on first scroll into view
- For formatted numbers: `el.textContent = proxy.value.toLocaleString()`

---

## 4. Floating Decorations (Yoyo + Random)

Gentle infinite floating for decorative elements (icons, dots, shapes).

```js
const ctx = gsap.context(() => {
  gsap.to(decorations, {
    y: 'random(-18, 18)',
    rotation: 'random(-12, 12)',
    duration: 'random(3, 5)',
    ease: 'sine.inOut',
    force3D: true,
    repeat: -1,
    yoyo: true,
    repeatRefresh: true, // new random values each cycle
    stagger: { each: 0.4, repeat: -1, yoyo: true },
  })
}, containerRef.value)
```

- `yoyo: true` reverses each cycle, creating a pendulum feel
- `random()` generates unique values per element on first run
- `ease: 'sine.inOut'` — smooth acceleration/deceleration, no abrupt direction changes
- `repeatRefresh: true` regenerates random values each cycle (without it, same path forever)

---

## 5. Pulse Ring (Infinite Scale + Fade)

Expanding ring that fades out — used after SVG path drawing completes.

```js
const ctx = gsap.context(() => {
  // Standalone
  gsap.fromTo(ring,
    { scale: 1, autoAlpha: 0.7 },
    { scale: 2.2, autoAlpha: 0, duration: 1.5, repeat: -1, ease: 'power1.out', force3D: true }
  )

  // Chained after SVG draw
  const tl = gsap.timeline()
  tl.fromTo(path, { drawSVG: '0%' }, { drawSVG: '100%', duration: 1.2 })
  tl.fromTo(ring,
    { scale: 1, autoAlpha: 0.7 },
    { scale: 2.2, autoAlpha: 0, duration: 1.5, repeat: -1, force3D: true },
  )

  // Multiple staggered rings
  rings.forEach((ring, i) => {
    gsap.fromTo(ring,
      { scale: 1, autoAlpha: 0.7 },
      { scale: 2.2, autoAlpha: 0, duration: 1.5, repeat: -1, delay: i * 0.5, force3D: true }
    )
  })
}, containerRef.value)
```

---

## 6. Repeating Timeline Patterns

### Core configs

| Config | Behaviour |
|--------|-----------|
| `repeat: -1` | Infinite loop |
| `repeatDelay: N` | N seconds gap between cycles |
| `repeatRefresh: true` | Re-evaluate `random()` / functions each cycle |
| `yoyo: true` | Reverse on even cycles |
| `.add(tl, time)` | Place sub-timeline at absolute position |
| `.call(fn, args, time)` | Fire a callback at absolute position |

### Composing sub-timelines with .add()

```js
const ctx = gsap.context(() => {
  const master = gsap.timeline({ repeat: -1, repeatDelay: 4 })

  const glitchBurst = gsap.timeline()
  glitchBurst.to(el, { skewX: 5, duration: 0.06, force3D: true })
              .to(el, { skewX: -3, duration: 0.06, force3D: true })
              .to(el, { skewX: 0, duration: 0.06, force3D: true })

  const rgbFlash = gsap.timeline()
  rgbFlash.to(red, { x: -4, duration: 0.1, force3D: true }).to(red, { x: 0, duration: 0.1, force3D: true })

  master.add(glitchBurst, 0) // both start at 0s
  master.add(rgbFlash, 0)
  master.set([el, red], { clearProps: 'all' }, 0.9) // cleanup at 0.9s
}, containerRef.value)
```

Without `repeatRefresh`, `random()` resolves once and repeats the same value forever.

---

## Anti-Patterns

```js
// BAD: marquee with easing — visible stutter at loop seam
gsap.to(track, { x: -width, repeat: -1, ease: 'power1.inOut' })

// BAD: counter without snap — flickering decimals
gsap.to(proxy, { value: 100, onUpdate() { el.textContent = proxy.value } })

// BAD: no repeatRefresh — same "random" value every cycle
gsap.to(el, { x: 'random(-50, 50)', repeat: -1, yoyo: true })

// BAD: permanent will-change on glitch layers (GPU memory waste)
.glitch-layer { will-change: transform, clip-path; }

// BAD: glitch without height lock — CLS on every burst

// BAD: creating new timeline each glitch cycle instead of repeat: -1
setInterval(() => { gsap.timeline().to(el, { skewX: 5 }) }, 4000)
```

---

## References

- `references/vue-examples.md` — Full component implementations for glitch, marquee, counters, floating decorations
