---
name: gsap-test
disable-model-invocation: true
description: >
  Testing, debugging, and verification for GSAP animations in Vue 3 / Nuxt 3 and React.
  Triggers: test animation, animation cleanup test, Playwright animation, animation debug,
  animation markers, memory leak animation, animation screenshot, visual QA animation,
  animation verification, pre-launch animation check.
  Non-triggers: Not for building animations (use gsap-animate). Not for performance optimisation
  (use gsap-optimise). Use this skill for verification and QA.
  Outcome: Produces test infrastructure — Vitest cleanup tests, Playwright hooks, debugging
  patterns, and pre-launch checklists.
---

# GSAP Test & Debug

> **Flow**: gsap-setup → gsap-animate → gsap-optimise → **gsap-test**

> **Companion**: This skill covers testing and debugging recipes. No API overlap — unique to this plugin.

---

## 1. Vitest — Animation Cleanup

```js
import { describe, it, expect, vi, afterEach } from 'vitest'
import { mount } from '@vue/test-utils'
import { gsap } from 'gsap'

afterEach(() => gsap.globalTimeline.clear())

describe('MyComponent animations', () => {
  it('cleans up all animations on unmount', async () => {
    const wrapper = mount(MyComponent)
    await new Promise(r => setTimeout(r, 100))
    const before = gsap.globalTimeline.getChildren().length
    await wrapper.unmount()
    expect(gsap.globalTimeline.getChildren().length).toBeLessThan(before)
  })

  it('cleans up ScrollTriggers on unmount', async () => {
    const { ScrollTrigger } = await import('gsap/ScrollTrigger')
    gsap.registerPlugin(ScrollTrigger)
    const wrapper = mount(MyComponent)
    await new Promise(r => setTimeout(r, 100))
    await wrapper.unmount()
    expect(ScrollTrigger.getAll().length).toBe(0)
  })

  it('respects reduced motion', async () => {
    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: vi.fn().mockImplementation(query => ({
        matches: query === '(prefers-reduced-motion: reduce)',
        media: query, addEventListener: vi.fn(),
      })),
    })
    const wrapper = mount(MyComponent)
    await new Promise(r => setTimeout(r, 100))
    expect(getComputedStyle(wrapper.find('.animated').element).visibility).not.toBe('hidden')
  })
})
```

---

## 2. Playwright — Animation Hooks

### Expose completeAllAnimations

```js
// plugins/gsap-test-hooks.client.js (dev/test only)
if (process.env.NODE_ENV !== 'production') {
  window.completeAllAnimations = () => {
    gsap.globalTimeline.progress(1)
    ScrollTrigger.getAll().forEach(st => st.scroll(st.end))
  }
  window.skipAnimations = () => gsap.globalTimeline.timeScale(100)
}
```

### Playwright test

```js
test('page renders correctly after animations', async ({ page }) => {
  await page.goto('/')
  await page.evaluate(() => window.completeAllAnimations?.())
  await page.waitForTimeout(500)
  await expect(page).toHaveScreenshot('homepage.png', { fullPage: true })
})

test('page works with reduced motion', async ({ page }) => {
  await page.emulateMedia({ reducedMotion: 'reduce' })
  await page.goto('/')
  await expect(page.locator('.hero')).toBeVisible()
})
```

### Skip via URL param

```js
if (window.location.search.includes('skip-animations')) {
  gsap.globalTimeline.timeScale(100)
}
```

---

## 3. Debugging

```js
// Markers (dev only)
ScrollTrigger.defaults({ markers: true })

// Console patterns
console.log('ScrollTriggers:', ScrollTrigger.getAll())
console.log('Active tweens:', gsap.globalTimeline.getChildren().length)

// Memory leak check — after navigating away:
// Both should be 0 after leaving the page
console.log('Remaining tweens:', gsap.globalTimeline.getChildren().length)
console.log('Remaining ScrollTriggers:', ScrollTrigger.getAll().length)
```

---

## 4. Pre-Launch Checklist

**Markers & Debug**
- [ ] All `markers: true` removed
- [ ] No `console.log` in animation callbacks
- [ ] `nullTargetWarn: false` in gsap.config for production

**Performance**
- [ ] Test on real mobile devices
- [ ] Verify 60fps in DevTools Performance tab
- [ ] No layout thrashing (only transform/opacity)
- [ ] `will-change` released after animations complete

**Accessibility**
- [ ] Test with `prefers-reduced-motion: reduce`
- [ ] Content visible with JavaScript disabled
- [ ] No critical content hidden behind animations

**Cleanup**
- [ ] Navigate all pages — no leaked tweens/ScrollTriggers
- [ ] `gsap.globalTimeline.getChildren().length` is 0 after leaving
- [ ] `ScrollTrigger.getAll().length` is 0 after leaving

**ScrollTrigger**
- [ ] `ScrollTrigger.refresh()` after lazy images and fonts
- [ ] Triggers fire correctly at all viewport sizes
- [ ] Pin sections work on mobile (real device)

**Visual QA**
- [ ] Screenshots capture final animated state
- [ ] Full-page scroll triggers all reveals
- [ ] Test at all breakpoints defined in matchMedia
