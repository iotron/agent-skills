---
name: gsap-text
description: >
  GSAP text animation patterns for Vue 3 / Nuxt 3 using SplitText and ScrambleTextPlugin. Covers:
  SplitText setup with lazy-loading and font-ready awaiting, masked word slide-up reveal (production
  pattern used in 25+ component headings), ScrambleText decode with terminal/cyber/glitch character
  sets, combined clip + scramble chainTextReveal pattern, kinetic character split with random scatter,
  elastic type assembly with pin + scrub scroll, and CRITICAL gotchas from real production debugging
  (autoSplit resize bugs, font CLS, overwrite conflicts, blank textContent collapse). Also covers the
  useReveal composable API (hero, scroll, split, chainTextReveal) and CSS pre-hide classes.
  Triggers: SplitText, split text, text animation, word animation, scramble text, ScrambleTextPlugin,
  text reveal, masked text, clip reveal, word stagger, text stagger, character animation, char split,
  kinetic text, elastic type, text decode, terminal text, cyber text, glitch text, chainTextReveal,
  useReveal text, GSAP text, Vue text animation, Nuxt text animation.
---

# GSAP Text Animations — SplitText + ScrambleText

> **Flow**: gsap-setup → gsap-animate → **gsap-text** → gsap-optimise → gsap-test
> **Key files**: `client/app/composables/useReveal.js`, `client/app/plugins/gsap.js`, `client/app/assets/css/tailwind.css`

---

## 1. SplitText Setup

SplitText is a premium GSAP plugin — lazy-load it to keep the initial bundle small.

```js
// In a component using useReveal (preferred — handles everything):
const { init, hero, scroll, split } = useReveal(sectionRef)
onMounted(() => init(() => scroll()))

// Manual setup (concept pages, one-off animations):
const { $gsap: gsap, $lazyLoadSplitText, $lazyLoadScramble } = useNuxtApp()
let SplitText
let ctx
const splits = []

onMounted(async () => {
  // CRITICAL: Wait for fonts BEFORE splitting (prevents CLS)
  ;[SplitText] = await Promise.all([
    $lazyLoadSplitText(),
    $lazyLoadScramble(),    // load if using scramble
    document.fonts.ready,   // prevents incorrect width measurements
  ])

  ctx = gsap.context(() => {
    // animations here
  }, scopeRef.value)
})

onUnmounted(() => {
  splits.forEach(s => s.revert())  // restore original DOM
  splits.length = 0
  ctx?.revert()
})
```

### Creating a split

```js
// Called inside gsap.context() — see setup above
const s = SplitText.create(el, {
  type: 'words',        // 'words' | 'chars' | 'lines' | 'words,chars'
  mask: 'words',        // adds overflow:clip wrapper per word (for slide-up)
})
splits.push(s)          // track for cleanup

gsap.set(s.words, { y: '100%' })       // pre-hide below mask
gsap.set(el, { visibility: 'visible' }) // un-hide container (was .text-reveal)
```

---

## 2. Masked Word Slide-Up (Production Pattern)

The standard heading reveal used across 25+ components. Words slide up from behind a clip mask.

**Timing constants** (single source of truth in `useReveal.js`):
```js
const SLIDE_DURATION = 0.8
const SLIDE_EASE     = 'power4.out'
const SLIDE_STAGGER  = 0.06
```

### Standalone (scroll-triggered)

```js
ctx = gsap.context(() => {
  const s = SplitText.create(el, { type: 'words', mask: 'words' })
  splits.push(s)

  gsap.set(s.words, { y: '100%' })
  gsap.set(el, { visibility: 'visible' })

  gsap.to(s.words, {
    y: '0%',
    duration: 0.8,
    ease: 'power4.out',
    stagger: 0.06,
    force3D: true,
    scrollTrigger: {
      trigger: el,
      start: 'top 85%',
      toggleActions: 'play none none reverse',
      invalidateOnRefresh: true,
    },
  })
}, scopeRef.value)
```

### Via useReveal (preferred for production)

```html
<div ref="sectionRef">
  <div class="reveal">
    <h2 class="text-reveal">HEADING TEXT</h2>
  </div>
</div>
```

```js
const sectionRef = ref(null)
const { init, scroll } = useReveal(sectionRef)
onMounted(() => init(() => scroll()))
// useReveal auto-detects .text-reveal inside .reveal, splits, and chains
```

---

## 3. ScrambleText Decode

ScrambleTextPlugin replaces text character-by-character with random chars, then reveals final text.

### Character sets

| Style | Chars | Feel |
|-------|-------|------|
| Terminal | `'▓░▒█▀▄╬╠╣▐▌■□●○'` | Cyber / terminal readout |
| Glitch | `'!@#$%^&*'` | Cyberpunk / hacker |
| Binary | `'01'` | Data stream / minimal |
| Katakana | `'アイウエオカキクケコ'` | Matrix / cyberpunk |
| Hex | `'0123456789ABCDEF'` | Memory addresses |
| Blocks | `'▓░▒█'` | Retro terminal |

### Basic scramble

```js
ctx = gsap.context(() => {
  gsap.to(el, {
    duration: 0.8 + el.textContent.length * 0.005, // scale with content
    scrambleText: { text: 'FINAL TEXT', chars: '▓░▒█▀▄╬╠╣▐▌■□', revealDelay: 0.3, speed: 0.4 },
  })
}, scopeRef.value)
```

### Staggered per-line (hero pattern)

```js
ctx = gsap.context(() => {
  const tl = gsap.timeline({ delay: 0.3 })
  lines.forEach((line, i) => {
    tl.to(line.ref, {
      duration: 1.2,
      scrambleText: { text: line.text, chars: '▓░▒█▀▄╬╠╣▐▌■□', revealDelay: 0.3, speed: 0.4 },
    }, i * 0.3)  // stagger: 0.15 + index * 0.12 for per-word variant
  })
}, scopeRef.value)
```

### Hover scramble (interactive, via ctx.add)

```js
ctx = gsap.context((self) => {
  self.add('cardScramble', (i) => {
    gsap.to(titleRefs.value[i], {
      duration: 0.8,
      overwrite: 'auto',
      scrambleText: { text: cards[i].title, chars: cards[i].chars, revealDelay: 0.2, speed: 0.5 },
    })
  })
}, scopeRef.value)
const onHover = (i) => ctx?.cardScramble(i) // from @mouseenter
```

---

## 4. Combined Clip + Scramble (chainTextReveal)

The signature production pattern: word slides up from mask AND scramble-decodes simultaneously.

Per word: slide-up tween + scramble tween at the same timeline position. Both MUST have `overwrite: false` (see Gotchas). The `chainTextReveal` function in `useReveal.js` handles this automatically.

### Implementation (manual or concept pages)

```js
ctx = gsap.context(() => {
  const s = SplitText.create(el, { type: 'words', mask: 'words' })
  splits.push(s)
  gsap.set(s.words, { y: '100%' })
  gsap.set(el, { visibility: 'visible' })

  const CLIP  = { duration: 0.8, ease: 'power4.out', force3D: true }
  const SCRAM = { duration: 0.5, ease: 'none' }

  const tl = gsap.timeline({
    scrollTrigger: { trigger: el, start: 'top 80%', toggleActions: 'play none none reverse' },
  })

  s.words.forEach((w, i) => {
    const pos = i * 0.08
    tl.to(w, { y: '0%', overwrite: false, ...CLIP }, pos)
    tl.to(w, { overwrite: false, ...SCRAM,
      scrambleText: { text: w.textContent, chars: '01', speed: 0.5 },
    }, pos)
  })
}, scopeRef.value)
```

---

## 5. Kinetic Character Split

Characters start at random positions and assemble into the final word.

```js
ctx = gsap.context(() => {
  const split = SplitText.create(el, { type: 'chars' })
  splits.push(split)

  gsap.set(split.chars, {
    y: () => gsap.utils.random(-200, 200), x: () => gsap.utils.random(-100, 100),
    rotation: () => gsap.utils.random(-90, 90), scale: () => gsap.utils.random(0.3, 2),
    autoAlpha: 0,
  })
  gsap.to(split.chars, {
    y: 0, x: 0, rotation: 0, scale: 1, autoAlpha: 1,
    duration: 1.4, ease: 'power4.out', force3D: true,
    stagger: { each: 0.04, from: 'random' },
    scrollTrigger: { trigger: el, start: 'top 80%', toggleActions: 'play none none reverse' },
  })
}, scopeRef.value)
```

---

## 6. Elastic Type Assembly (Pin + Scrub)

Characters start scattered, assemble on scroll, hold in place, then scatter again. Three-phase timeline with pin + scrub.

```js
ctx = gsap.context(() => {
  const split = SplitText.create(el, { type: 'chars' })
  splits.push(split)

  gsap.set(split.chars, {
    y: () => gsap.utils.random(-300, 300), x: () => gsap.utils.random(-200, 200),
    rotation: () => gsap.utils.random(-180, 180), scale: 0, autoAlpha: 0,
  })

  const tl = gsap.timeline({
    scrollTrigger: { trigger: el, start: 'top center', end: '+=150%', pin: true, scrub: 1 },
  })

  // Phase 1: Assemble — elastic ease gives bouncy landing
  tl.to(split.chars, {
    y: 0, x: 0, rotation: 0, scale: 1, autoAlpha: 1, duration: 1,
    ease: 'elastic.out(1.2, 1)', stagger: { each: 0.03, from: 'random' },
  })
  // Phase 2: Hold — empty tween creates scroll pause
  tl.to({}, { duration: 0.5 })
  // Phase 3: Scatter — edges stagger sends center chars last
  tl.to(split.chars, {
    y: () => gsap.utils.random(-400, 400), x: () => gsap.utils.random(-300, 300),
    rotation: () => gsap.utils.random(-180, 180), scale: 0, autoAlpha: 0, duration: 1,
    ease: 'power2.in', stagger: { each: 0.02, from: 'edges' },
  })
}, scopeRef.value)
```

---

## 7. CRITICAL GOTCHAS

These are hard-won lessons from real production debugging. Read carefully before implementing any text animation.

### NEVER use autoSplit: true with timelines

`autoSplit: true` attaches a ResizeObserver that re-splits the DOM on resize (including font load events). This **invalidates all element references** stored in the timeline. Words array points to removed DOM nodes.

```js
// BAD — timeline targets become stale after resize/font load
const s = SplitText.create(el, { type: 'words', autoSplit: true })
gsap.to(s.words, { y: 0 }) // these elements may be destroyed on resize

// GOOD — one-time split, stable references
const s = SplitText.create(el, { type: 'words' })
gsap.to(s.words, { y: 0 }) // these elements persist until s.revert()
```

### Await document.fonts.ready BEFORE SplitText.create()

SplitText measures element dimensions at creation time. If fallback fonts are active, measurements will be wrong, causing CLS when the web font loads and text reflows.

```js
// GOOD — always in useReveal.init() and manual setups
await document.fonts.ready
SplitText = await $lazyLoadSplitText()
// NOW safe to split
```

### visibility: 'visible', NOT autoAlpha: 1, in split()

Split words are hidden by the mask (`y: '100%'`). The container just needs `visibility: visible` to un-hide. Using `autoAlpha: 1` would set `opacity: 1` which overrides the parent `.reveal` wrapper's fade-in animation.

```js
// BAD — overrides parent .reveal opacity tween
gsap.set(el, { autoAlpha: 1 })

// GOOD — container visible, words still hidden by mask
gsap.set(el, { visibility: 'visible' })
```

### Never blank textContent before scramble

ScrambleTextPlugin reads the element's current dimensions. If you clear `textContent` first, the word div collapses to 0x0 pixels and the scramble animates inside an invisible box.

```js
// BAD — element collapses
el.textContent = ''
gsap.to(el, { scrambleText: { text: 'HELLO' } })

// GOOD — scramble handles the transition from existing text
gsap.to(el, { scrambleText: { text: 'HELLO', chars: '01' } })
```

### overwrite: false on colocated per-word tweens

The plugin sets `gsap.defaults({ overwrite: 'auto' })`. When two tweens target the same element at the same timeline position, `overwrite: 'auto'` kills the first tween (scramble) when the second (y slide) starts. **Explicitly set `overwrite: false`** on both.

```js
// BAD — scramble gets killed by overwrite: 'auto'
tl.to(w, { y: '0%', duration: 0.8 }, pos)
tl.to(w, { scrambleText: { text: w.textContent } }, pos)

// GOOD — both tweens coexist on the same element at the same position
tl.to(w, { y: '0%', overwrite: false, duration: 0.8 }, pos)
tl.to(w, { overwrite: false, scrambleText: { text: w.textContent } }, pos)
```

### SplitText preserves `<span>` elements

SplitText creates word `<div>` elements inside existing `<span>` tags. This means `.text-gradient` spans get a nested div. Use `:deep(div)` in scoped CSS to target word divs for gradient styles.

```css
/* In tailwind.css (global) — already handles this: */
.text-gradient,
.text-gradient div {
  @apply bg-gradient-to-r from-teal-500 to-cyan-400 bg-clip-text text-transparent;
}
```

### .text-reveal CSS class pre-hides elements

Both `.reveal` and `.text-reveal` have `visibility: hidden` in `tailwind.css` to prevent FOUC (flash of unstyled content) between SSR render and GSAP initialization. The animation then sets `visibility: visible`.

```css
/* client/app/assets/css/tailwind.css */
.reveal,
.text-reveal {
  visibility: hidden;
}
```

---

## 8. useReveal Composable Reference

The `useReveal` composable (`client/app/composables/useReveal.js`) is the production API for all reveal + text animations.

### API

| Method | Description |
|--------|-------------|
| `init(fn)` | Async setup: waits for fonts, lazy-loads SplitText + Scramble, creates gsap.context |
| `hero(selector?, overrides?)` | Immediate reveal for above-fold. Auto-detects `.text-reveal` children and chains word stagger + scramble |
| `scroll(selector?, overrides?)` | Per-element ScrollTrigger reveal. Combines `.reveal` + `.text-reveal` into one timeline per element |
| `split(target)` | SplitText wrapper: creates masked word split, pre-hides words, makes container visible, tracks for cleanup |
| `chainTextReveal(tl, words, position)` | Adds per-word slide-up + scramble decode to any timeline at given position |

### Usage examples

```js
// Hero (above-fold, immediate)
const { init, hero } = useReveal(sectionRef)
onMounted(() => init(() => hero('.reveal', { textDelay: 0.2 })))

// Scroll (below-fold, per-element ScrollTrigger)
const { init, scroll } = useReveal(sectionRef)
onMounted(() => init(() => scroll('.reveal', { trigger: sectionRef.value, start: 'top 75%', once: true })))

// Manual split + chainTextReveal (custom timelines)
const { init, split, chainTextReveal, gsap } = useReveal(sectionRef)
onMounted(() => init(() => {
  const s = split('.custom-heading')
  const tl = gsap.timeline({ scrollTrigger: { trigger: '.custom-heading', start: 'top 80%' } })
  tl.reveal('.wrapper')
  chainTextReveal(tl, s.words, '-=0.4')
}))
```

Cleanup is automatic -- `onUnmounted` reverts gsap.context and all SplitText instances.
