# Text Animation Learnings

Hard-won lessons from real production debugging. Read before implementing any text animation.

## Table of Contents
- [autoSplit: true — for responsive LINE splitting only](#autosplit-true--for-responsive-line-splitting-only)
- [Await document.fonts.ready BEFORE SplitText](#await-documentfontsready-before-splittext)
- [Parent autoAlpha + child mask conflict](#parent-autoalpha--child-mask-conflict)
- [Never blank textContent before scramble](#never-blank-textcontent-before-scramble)
- [overwrite: false on colocated per-word tweens](#overwrite-false-on-colocated-per-word-tweens)
- [SplitText preserves span elements](#splittext-preserves-span-elements)
- [CSS pre-hide classes prevent FOUC](#css-pre-hide-classes-prevent-fouc)

---

## autoSplit: true — for responsive LINE splitting only

`autoSplit: true` attaches a ResizeObserver that re-splits when fonts finish loading or when the **element width changes AND lines are split**. It exists because line-wrapped `<div>` elements become stale when text reflows to different lines on resize.

**When to use `autoSplit`:** Only when splitting `type: "lines"` (or a combination that includes lines). Words and characters reflow naturally without re-splitting — they're individual elements regardless of which line they land on.

**When using `autoSplit`:** Always create animations inside `onSplit()` and return the tween/timeline so SplitText can clean up and sync progress on re-split.

```js
// BAD — autoSplit with words-only (unnecessary, words reflow naturally)
SplitText.create(el, {
  type: 'words', mask: 'words', autoSplit: true,
  onSplit(self) {
    return gsap.from(self.words, { y: '100%', duration: 0.8, stagger: 0.06 })
  },
})

// GOOD — autoSplit with lines (lines need re-splitting on resize)
SplitText.create(el, {
  type: 'lines, words', mask: 'lines', autoSplit: true,
  onSplit(self) {
    return gsap.from(self.words, { y: 100, autoAlpha: 0, duration: 1, stagger: 0.05 })
  },
})

// GOOD — words-only split after fonts.ready (no autoSplit needed)
await document.fonts.ready
const s = SplitText.create(el, { type: 'words', mask: 'words' })
gsap.from(s.words, { y: '100%', duration: 0.8, stagger: 0.06 })
```

---

## Await document.fonts.ready BEFORE SplitText

SplitText measures element dimensions at creation time. If fallback fonts are active, measurements will be wrong, causing CLS when the web font loads.

```js
// GOOD — always in useReveal.init() and manual setups
await document.fonts.ready
SplitText = await $lazyLoadSplitText()
// NOW safe to split
```

---

## Parent autoAlpha + child mask conflict

When a **parent** uses `autoAlpha` (fade-in) and a **child** uses SplitText with `mask: 'words'` (words hidden via `y: '100%'`), don't set `autoAlpha: 1` on the child container. It overrides the parent's opacity control.

The child only needs `visibility: 'visible'` to un-hide. Words remain invisible because they're below the mask, not because of opacity.

```js
// BAD (mask: 'words' + parent has autoAlpha tween)
gsap.set(textEl, { autoAlpha: 1 })

// GOOD (mask: 'words') — just un-hide
gsap.set(textEl, { visibility: 'visible' })

// OK (no mask, no parent autoAlpha conflict)
gsap.set(textEl, { autoAlpha: 1 })
```

---

## Never blank textContent before scramble

ScrambleTextPlugin reads the element's current dimensions. If you clear `textContent` first, the word div collapses to 0x0 pixels.

```js
// BAD — element collapses
el.textContent = ''
gsap.to(el, { scrambleText: { text: 'HELLO' } })

// GOOD — scramble handles the transition
gsap.to(el, { scrambleText: { text: 'HELLO', chars: '01' } })
```

---

## overwrite: false on colocated per-word tweens

This project's plugin sets `gsap.defaults({ overwrite: 'auto' })` globally. With `overwrite: 'auto'`, two tweens targeting the same element at the same timeline position will conflict. **Explicitly set `overwrite: false`**.

```js
// BAD — scramble gets killed by overwrite: 'auto'
tl.to(w, { y: '0%', duration: 0.8 }, pos)
tl.to(w, { scrambleText: { text: w.textContent } }, pos)

// GOOD — both tweens coexist
tl.to(w, { y: '0%', overwrite: false, duration: 0.8 }, pos)
tl.to(w, { overwrite: false, scrambleText: { text: w.textContent } }, pos)
```

---

## SplitText preserves span elements

SplitText creates word `<div>` elements inside existing `<span>` tags. Use `:deep(div)` for gradient styles.

```css
.text-gradient,
.text-gradient div {
  @apply bg-gradient-to-r from-teal-500 to-cyan-400 bg-clip-text text-transparent;
}
```

---

## CSS pre-hide classes prevent FOUC

Use `visibility: hidden` (not `display: none` — which breaks ScrollTrigger measurements). GSAP's `autoAlpha` sets `visibility: inherit` when animating.

```css
.reveal, .text-reveal { visibility: hidden; }
```
