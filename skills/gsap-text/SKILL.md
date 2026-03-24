---
name: gsap-text
description: >
  Production recipes for GSAP text animations using SplitText and ScrambleTextPlugin.
  Companion to official gsap-plugins skill (API reference).
  Triggers: SplitText, split text, text animation, word animation, scramble text, ScrambleTextPlugin,
  text reveal, masked text, clip reveal, word stagger, text stagger, character animation, char split,
  kinetic text, elastic type, text decode, terminal text, cyber text, glitch text, chainTextReveal,
  useReveal text, GSAP text, Vue text animation, Nuxt text animation.
  Non-triggers: Not for scroll-driven animation without text (use gsap-scroll), mouse interactions
  (use gsap-interact), SVG animation (use gsap-svg), or general visual effects (use gsap-vfx).
  Outcome: Produces text animations — masked word reveals, scramble decodes, kinetic splits, and
  the useReveal composable API.
---

# GSAP Text — SplitText + ScrambleText

> **Flow**: gsap-setup → gsap-animate → **gsap-text** → gsap-optimise → gsap-test
> **Key files**: `composables/useReveal.js`, `plugins/gsap.js`, `assets/css/tailwind.css`

> **Companion**: For SplitText/ScrambleText API reference, invoke **gsap-plugins**. This skill covers text animation recipes only. Requires: `greensock/gsap-skills`

---

## 1. SplitText Setup

```js
// Preferred — useReveal handles everything:
const { init, hero, scroll, split } = useReveal(sectionRef)
onMounted(() => init(() => scroll()))

// Manual setup:
const { $gsap: gsap, $lazyLoadSplitText, $lazyLoadScramble } = useNuxtApp()
let SplitText, ctx
const splits = []

onMounted(async () => {
  await document.fonts.ready               // CRITICAL: fonts before split
  SplitText = await $lazyLoadSplitText()
  ctx = gsap.context(() => { /* ... */ }, scopeRef.value)
})

onUnmounted(() => {
  splits.forEach(s => s.revert())
  splits.length = 0
  ctx?.revert()
})
```

### Creating a split

```js
const s = SplitText.create(el, { type: 'words', mask: 'words' })
splits.push(s)
gsap.set(s.words, { y: '100%' })
gsap.set(el, { visibility: 'visible' })
```

---

## 2. Masked Word Slide-Up (Production Pattern)

The standard heading reveal used across 25+ components.

```js
ctx = gsap.context(() => {
  const s = SplitText.create(el, { type: 'words', mask: 'words' })
  splits.push(s)
  gsap.set(s.words, { y: '100%' })
  gsap.set(el, { visibility: 'visible' })

  gsap.to(s.words, {
    y: '0%', duration: 0.8, ease: 'power4.out', stagger: 0.06, force3D: true,
    scrollTrigger: {
      trigger: el, start: 'top 85%',
      toggleActions: 'play none none reverse', invalidateOnRefresh: true,
    },
  })
}, scopeRef.value)
```

### Via useReveal (preferred)

```html
<div ref="sectionRef">
  <div class="reveal">
    <h2 class="text-reveal">HEADING TEXT</h2>
  </div>
</div>
```

```js
const { init, scroll } = useReveal(sectionRef)
onMounted(() => init(() => scroll()))
```

---

## 3. ScrambleText Decode

See **gsap-plugins** skill for ScrambleText API reference. Recipe:

```js
gsap.to(el, {
  duration: 0.8 + el.textContent.length * 0.005,
  scrambleText: { text: 'FINAL TEXT', chars: '▓░▒█▀▄╬╠╣▐▌■□', revealDelay: 0.3, speed: 0.4 },
})
```

---

## 4. Gotchas Summary

See `references/learnings.md` for full details and code examples.

- Await `document.fonts.ready` before `SplitText.create()`
- `autoSplit: true` is for **lines** only (responsive reflow) — words/chars don't need it. Always pair with `onSplit()`
- Parent autoAlpha + child mask: use `visibility`, not `autoAlpha`
- Never blank `textContent` before scramble
- Set `overwrite: false` on colocated per-word tweens
- SplitText preserves `<span>` — use `:deep(div)` for gradient styles
- CSS `.reveal` + `.text-reveal` pre-hide classes prevent FOUC

---

## 5. useReveal Composable Reference

| Method | Description |
|--------|-------------|
| `init(fn)` | Async: waits for fonts, lazy-loads SplitText + Scramble, creates gsap.context |
| `hero(selector?, overrides?)` | Immediate reveal for above-fold. Auto-detects `.text-reveal` children |
| `scroll(selector?, overrides?)` | Per-element ScrollTrigger reveal with `.reveal` + `.text-reveal` |
| `split(target)` | SplitText wrapper: creates masked word split, tracks for cleanup |

```js
// Hero (above-fold, immediate)
const { init, hero } = useReveal(sectionRef)
onMounted(() => init(() => hero('.reveal', { textDelay: 0.2 })))

// Scroll (below-fold, per-element ScrollTrigger)
const { init, scroll } = useReveal(sectionRef)
onMounted(() => init(() => scroll('.reveal', { start: 'top 75%', once: true })))
```

Cleanup is automatic — `onUnmounted` reverts gsap.context and all SplitText instances.

---

## References

- `references/text-patterns.md` — Combined clip+scramble, kinetic character split, elastic type assembly, rolling text (3D slot machine), horizontal text (containerAnimation), masked lines reveal, autoSplit with ScrollTrigger
- `references/learnings.md` — 7 critical gotchas from production debugging
