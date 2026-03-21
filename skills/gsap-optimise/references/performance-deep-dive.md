# GSAP Performance Deep Dive

Extended reference for each optimisation technique. Read specific sections as needed.

## Table of Contents

- [quickTo API](#quickto-api)
- [quickSetter API](#quicksetter-api)
- [Piping with gsap.utils](#piping-with-gsaputils)
- [ScrollTrigger.batch API](#scrolltriggerbatch-api)
- [ScrollTrigger scrub tuning](#scrolltrigger-scrub-tuning)
- [matchMedia responsive animations](#matchmedia-responsive-animations)
- [registerEffect API](#registereffect-api)
- [Ticker and lagSmoothing](#ticker-and-lagsmoothing)
- [autoAlpha details](#autoalpha-details)
- [force3D compositor layer management](#force3d-compositor-layer-management)
- [Lazy rendering internals](#lazy-rendering-internals)
- [gsap.config and gsap.defaults](#gsapconfig-and-gsapdefaults)
- [Context cleanup internals](#context-cleanup-internals)

---

## quickTo API

```js
gsap.quickTo(target, property, vars) → Function
```

Returns a **reusable function** that animates a single numeric property. Internally reuses one tween — no object creation overhead per call.

### Parameters

| Param | Type | Description |
|-------|------|-------------|
| `target` | String/Element | Selector or element |
| `property` | String | Numeric CSS property (`"x"`, `"y"`, `"opacity"`, `"rotation"`) |
| `vars` | Object | Tween config: `duration`, `ease`, `delay` |

### Returned function

```js
const xTo = gsap.quickTo('#el', 'x', { duration: 0.4, ease: 'power3' })

xTo(100)       // animate from current value to 100
xTo(100, 500)  // animate from 500 to 100 (optional start value)
xTo.tween      // access underlying Tween instance
```

### Performance note

50-250% faster than calling `gsap.to()` repeatedly because it **reuses** the tween instance. Skips: unit conversion, relative values, function-based values, `"random()"` parsing, plugin parsing, property alias conversion.

### Mouse follower pattern

```js
const xTo = gsap.quickTo('#cursor', 'x', { duration: 0.6, ease: 'power3' })
const yTo = gsap.quickTo('#cursor', 'y', { duration: 0.6, ease: 'power3' })

document.addEventListener('mousemove', (e) => {
  xTo(e.clientX)
  yTo(e.clientY)
})
```

### Limitations

- One property per quickTo (create multiple for x + y)
- No advanced plugin features (morphSVG, attr)
- Property aliases restricted (`"x"` works, `"translateX"` doesn't)

---

## quickSetter API

```js
gsap.quickSetter(target, property, unit?) → Function
```

Returns a function that **immediately** (no animation) sets a property value. Fastest possible way to pipe data to a target.

### Parameters

| Param | Type | Description |
|-------|------|-------------|
| `target` | String/Element/Array | Selector, element, or array of elements |
| `property` | String | CSS property or `"css"` / `"attr"` for object-based |
| `unit` | String (optional) | Auto-appended unit, e.g. `"px"`, `"%"`, `"deg"` |

### Usage

```js
const setX = gsap.quickSetter('#el', 'x', 'px')
const setRot = gsap.quickSetter('#el', 'rotation', 'deg')

gsap.ticker.add(() => {
  setX(mouseX)
  setRot(scrollY * 0.5)
})
```

### Object-based setter

```js
const boxSet = gsap.quickSetter('#box', 'css')
boxSet({ x: 100, y: 50, rotation: 45 })
```

---

## Piping with gsap.utils

Chain quickSetter/quickTo with utility functions for clamping, snapping, mapping.

```js
const xSet = gsap.utils.pipe(
  gsap.utils.clamp(0, 100),           // clamp value 0-100
  gsap.utils.snap(5),                  // snap to nearest 5
  gsap.quickSetter('#el', 'x', 'px')  // set immediately
)

// Now xSet(137) → clamps to 100 → snaps to 100 → sets x: 100px
```

```js
const mapper = gsap.utils.pipe(
  gsap.utils.mapRange(0, window.innerWidth, -50, 50),  // map viewport → range
  gsap.quickTo('#el', 'x', { duration: 0.3 })
)

document.addEventListener('mousemove', (e) => mapper(e.clientX))
```

---

## ScrollTrigger.batch API

Creates one ScrollTrigger per target but batches callbacks into a single function call within an interval window.

```js
ScrollTrigger.batch('.cards', {
  interval: 0.1,       // callback batching window (seconds)
  batchMax: 3,          // max elements per batch
  onEnter: (batch) => {
    gsap.to(batch, { autoAlpha: 1, y: 0, stagger: 0.08, overwrite: true })
  },
  onLeave: (batch) => {
    gsap.to(batch, { autoAlpha: 0, y: -20, overwrite: true })
  },
  onEnterBack: (batch) => {
    gsap.to(batch, { autoAlpha: 1, y: 0, stagger: 0.08, overwrite: true })
  },
  onLeaveBack: (batch) => {
    gsap.to(batch, { autoAlpha: 0, y: 20, overwrite: true })
  },
  start: 'top 85%',
  end: 'bottom 15%',
})
```

**Why batch > individual**: Creating one ScrollTrigger per element (via `.forEach`) is wasteful for lists of similar items. `batch()` creates the same triggers internally but staggers the callbacks for a natural cascade effect.

---

## ScrollTrigger scrub tuning

| Value | Behaviour | Performance impact |
|-------|-----------|-------------------|
| `true` | 1:1 scroll-to-progress mapping | Highest — continuous main-thread updates |
| `0.5` | Smooth catch-up with 0.5s lag | Better — interpolated, fewer sharp changes |
| `1` | Smooth catch-up with 1s lag | Smoothest, lowest overhead |

**Rule**: Use `scrub: true` only for simple transform/opacity. Use a number (0.5–1) for complex timelines.

### Additional scrub properties

```js
ScrollTrigger.create({
  trigger: el,
  scrub: 0.5,
  snap: {
    snapTo: 'labels',        // snap to timeline labels
    duration: { min: 0.2, max: 0.5 },
    ease: 'power1.inOut',
  },
  fastScrollEnd: true,       // force completion on fast scroll exit
})
```

---

## matchMedia responsive animations

Animations auto-revert when the media query stops matching — no manual cleanup needed.

```js
const mm = gsap.matchMedia()

mm.add({
  isDesktop: '(min-width: 1024px)',
  isMobile:  '(max-width: 1023px)',
  reduceMotion: '(prefers-reduced-motion: reduce)',
}, (context) => {
  const { isDesktop, isMobile, reduceMotion } = context.conditions

  if (reduceMotion) {
    gsap.set('.animate', { opacity: 1 }) // skip animation
    return
  }

  if (isDesktop) {
    gsap.to('.hero', { x: 200, scrollTrigger: { trigger: '.hero', scrub: 1 } })
  }

  if (isMobile) {
    gsap.to('.hero', { y: 100, scrollTrigger: { trigger: '.hero', scrub: 1 } })
  }
})
```

**Accessibility**: Always include `(prefers-reduced-motion: reduce)` condition.

---

## registerEffect API

```js
gsap.registerEffect({
  name: 'slideUp',
  effect: (targets, config) => {
    return gsap.from(targets, {
      y: config.distance,
      autoAlpha: 0,
      duration: config.duration,
      stagger: config.stagger,
      ease: config.ease,
    })
  },
  defaults: { distance: 30, duration: 0.6, stagger: 0.08, ease: 'power2.out' },
  extendTimeline: true,  // enables tl.slideUp('.el')
})
```

**Key rules**:
- `extendTimeline: true` requires the effect to return a Tween or Timeline
- Registered effects are global — register once in a plugin, not per-component
- Use for DRY reveal animations, standard transitions

---

## Ticker and lagSmoothing

### gsap.ticker

Heartbeat of the GSAP engine. Fires once per `requestAnimationFrame`. Use it for custom per-frame logic.

```js
gsap.ticker.add((time, deltaTime, frame) => {
  // runs every frame — use for custom physics, cursor lerp, etc.
  setX(lerp(currentX, targetX, 0.1))
})

// Remove when done
gsap.ticker.remove(myCallback)
```

### lagSmoothing

Prevents animations from jumping forward after CPU spikes.

```js
gsap.ticker.lagSmoothing(500, 33)  // default: if >500ms gap, act as 33ms
gsap.ticker.lagSmoothing(1000, 16) // more tolerant: 1s threshold, 16ms catchup
gsap.ticker.lagSmoothing(0)        // disable: frame-perfect (video/audio sync)
```

### autoSleep

Ticker auto-sleeps after `autoSleep` idle frames to save battery. Default: 120 (~2s).

```js
gsap.config({ autoSleep: 60 }) // sleep after 1 second of inactivity
```

---

## autoAlpha details

```js
// Identical to opacity, plus:
// - Sets visibility: hidden at 0 (removes from hit-testing, improves rendering)
// - Sets visibility: inherit at any non-zero value
// - Better than opacity alone for hidden elements
gsap.set(el, { autoAlpha: 0 })
gsap.to(el, { autoAlpha: 1, duration: 0.5 })
```

**Progressive enhancement**: Set elements to `visibility: hidden` in CSS, then animate `autoAlpha` to 1. If JS fails, elements stay hidden rather than showing unstyled flicker.

---

## force3D compositor layer management

Browser creates a separate GPU layer for 3D-transformed elements. Benefits: animations don't repaint parent layers. Cost: GPU memory per layer.

**`"auto"` (default)**: Promotes during animation → demotes after. Ideal for one-shot animations.

**`true`**: Stays promoted. Use for:
- Elements animated repeatedly (mousemove tilt cards)
- Elements in long scrub timelines
- Avoids constant promote/demote thrashing

**`false`**: Never promotes. Use for:
- Upscaled images (avoids sub-pixel blur in Safari/Chrome)
- Static elements that don't need GPU

---

## Lazy rendering internals

When a tween renders for the first time, GSAP defers writing values to the end of the current tick. This avoids read/write/read/write layout thrashing.

```
Tick start
  ├─ Tween A reads startValues
  ├─ Tween B reads startValues   ← no interleaved writes yet
  ├─ Tween A writes endValues
  └─ Tween B writes endValues    ← all writes batched at end
```

- Default: `lazy: true` for all tweens
- Exception: `duration: 0` tweens have `lazy: false` (need instant apply)
- Override: `lazy: false` only when you need the write to be immediately visible to subsequent code in the same tick

---

## gsap.config and gsap.defaults

### gsap.config() — engine-level, not inherited by tweens

| Option | Default | Notes |
|--------|---------|-------|
| `force3D` | `"auto"` | Global default for all tweens |
| `autoSleep` | `120` | Frames before ticker sleeps |
| `nullTargetWarn` | `true` | Set `false` in production |
| `units` | `{ rotation: "deg" }` | Default units per property |

### gsap.defaults() — inherited by every new tween

```js
gsap.defaults({
  ease: 'power2.out',
  duration: 0.5,
  overwrite: 'auto',  // global overwrite for all tweens
})
```

**Per-timeline defaults** override global:

```js
const tl = gsap.timeline({ defaults: { ease: 'power3.out', duration: 0.3 } })
tl.to(a, { x: 100 })  // inherits ease + duration from timeline
tl.to(b, { y: 50 })
```

---

## Context cleanup internals

### What ctx.revert() actually does

1. Kills all tweens created within the context (including mid-flight)
2. Kills all ScrollTriggers created within the context
3. Restores all inline styles that GSAP applied to their pre-animation state
4. Removes all event listeners registered via `ctx.add()`

### ctx.add() named handlers

```js
ctx = gsap.context((self) => {
  self.add('move', (x, y) => {
    gsap.to(el, { x, y, overwrite: 'auto' })
  })
})

// Call via: ctx.move(100, 200)
// Tweens created inside 'move' are collected by the context
```

### Scope parameter

The optional second argument scopes all selector text within the context:

```js
ctx = gsap.context(() => {
  gsap.to('.box', { x: 100 }) // only matches .box INSIDE sectionRef
}, sectionRef.value)
```
