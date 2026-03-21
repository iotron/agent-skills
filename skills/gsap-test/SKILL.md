---
name: gsap-test
description: >
  Testing, debugging, and verification for GSAP animations in Vue 3 / Nuxt 3 and React.
  Covers: Vitest animation cleanup verification, Playwright testing hooks (completeAllAnimations,
  skip-animations), debugging with markers, console patterns, memory leak detection, reduced
  motion testing, pre-launch checklist, and visual QA integration.
  Triggers: test animation, animation cleanup test, Playwright animation, animation debug,
  animation markers, memory leak animation, animation screenshot, visual QA animation,
  animation verification, pre-launch animation check.
---

# GSAP Test & Debug

> **Flow**: gsap-vue-setup → gsap-animate → gsap-optimise → **gsap-test**

---

## 1. Vitest — Animation Cleanup

Verify no orphaned tweens or ScrollTriggers after unmount.

```js
import { describe, it, expect, vi, afterEach } from 'vitest'
import { mount } from '@vue/test-utils'
import { gsap } from 'gsap'
import MyComponent from '~/components/MyComponent.vue'

afterEach(() => {
  // Safety net: clear any leaked animations
  gsap.globalTimeline.clear()
})

describe('MyComponent animations', () => {
  it('cleans up all animations on unmount', async () => {
    const wrapper = mount(MyComponent)
    await new Promise(r => setTimeout(r, 100)) // let onMounted run

    const beforeCount = gsap.globalTimeline.getChildren().length

    await wrapper.unmount()

    // All animations should be killed by ctx.revert()
    expect(gsap.globalTimeline.getChildren().length).toBeLessThan(beforeCount)
  })

  it('cleans up ScrollTriggers on unmount', async () => {
    const { ScrollTrigger } = await import('gsap/ScrollTrigger')
    gsap.registerPlugin(ScrollTrigger)

    const wrapper = mount(MyComponent)
    await new Promise(r => setTimeout(r, 100))

    await wrapper.unmount()

    // No ScrollTriggers should survive
    expect(ScrollTrigger.getAll().length).toBe(0)
  })

  it('respects reduced motion preference', async () => {
    // Mock reduced motion
    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: vi.fn().mockImplementation(query => ({
        matches: query === '(prefers-reduced-motion: reduce)',
        media: query,
        addEventListener: vi.fn(),
      })),
    })

    const wrapper = mount(MyComponent)
    await new Promise(r => setTimeout(r, 100))

    // With reduced motion, elements should be in final state with no animation
    const el = wrapper.find('.reveal').element
    expect(gsap.getProperty(el, 'opacity')).toBe(1)
  })
})
```

---

## 2. Playwright — Animation Hooks

### Expose completeAllAnimations

Add to your app for testing purposes:

```js
// plugins/gsap-test-hooks.client.js (only in dev/test)
export default defineNuxtPlugin(() => {
  if (process.env.NODE_ENV !== 'production') {
    window.completeAllAnimations = () => {
      gsap.globalTimeline.progress(1)
      ScrollTrigger.getAll().forEach(st => st.scroll(st.end))
    }

    window.skipAnimations = () => {
      gsap.globalTimeline.timeScale(100)
    }
  }
})
```

### Playwright test

```js
import { test, expect } from '@playwright/test'

test('page renders correctly after animations', async ({ page }) => {
  await page.goto('/')

  // Complete all animations instantly
  await page.evaluate(() => {
    if (window.completeAllAnimations) window.completeAllAnimations()
  })
  await page.waitForTimeout(500)

  // Screenshot with animations in final state
  await expect(page).toHaveScreenshot('homepage.png', { fullPage: true })
})

test('page works with reduced motion', async ({ page }) => {
  // Emulate reduced motion preference
  await page.emulateMedia({ reducedMotion: 'reduce' })
  await page.goto('/')

  // Content should be visible immediately
  const hero = page.locator('.hero')
  await expect(hero).toBeVisible()
})
```

### Skip animations via URL param

```js
// In your Nuxt plugin
if (window.location.search.includes('skip-animations')) {
  gsap.globalTimeline.timeScale(100)
}
```

---

## 3. Debugging

### Markers (dev only)

```js
// Enable globally during development
ScrollTrigger.defaults({ markers: true })

// Or per-trigger with custom colours
scrollTrigger: {
  markers: { startColor: 'green', endColor: 'red', fontSize: '12px' }
}
```

### Console patterns

```js
// List all active ScrollTriggers
console.log('ScrollTriggers:', ScrollTrigger.getAll())
console.log('Active tweens:', gsap.globalTimeline.getChildren().length)

// Log animation lifecycle
gsap.to(el, {
  x: 100,
  onStart: () => console.log('Animation started'),
  onComplete: () => console.log('Animation completed'),
  onUpdate: function () { console.log('Progress:', this.progress().toFixed(2)) },
})

// Check target exists
const target = document.querySelector('.animated-element')
if (!target) console.warn('Animation target ".animated-element" not found!')
```

### Memory leak detection

```js
// In browser DevTools console:
// 1. Take heap snapshot
// 2. Navigate/interact with animations
// 3. Take another heap snapshot
// 4. Compare — look for detached DOM nodes or growing tween counts

// Quick check: run after navigating away from animated page
console.log('Remaining tweens:', gsap.globalTimeline.getChildren().length)
console.log('Remaining ScrollTriggers:', ScrollTrigger.getAll().length)
// Both should be 0 after leaving the page
```

---

## 4. Pre-Launch Checklist

**Markers & Debug**
- [ ] All `markers: true` removed
- [ ] No `console.log` in animation callbacks
- [ ] `nullTargetWarn: false` set in gsap.config for production

**Performance**
- [ ] Test on real mobile devices (not just DevTools emulation)
- [ ] Check DevTools Performance tab — verify 60fps
- [ ] No layout thrashing (only transform/opacity animated)
- [ ] `will-change` released after animations complete

**Accessibility**
- [ ] Test with `prefers-reduced-motion: reduce` enabled
  - macOS: System Settings → Accessibility → Display → Reduce Motion
  - Chrome DevTools: Rendering tab → Emulate prefers-reduced-motion
- [ ] Content visible with JavaScript disabled
- [ ] No critical content hidden behind animations

**Cleanup**
- [ ] Navigate between all pages — check no leaked tweens/ScrollTriggers
- [ ] `gsap.globalTimeline.getChildren().length` is 0 after leaving animated pages
- [ ] `ScrollTrigger.getAll().length` is 0 after leaving animated pages

**ScrollTrigger**
- [ ] `ScrollTrigger.refresh()` called after lazy images and fonts load
- [ ] Triggers fire correctly at all viewport sizes
- [ ] Pin sections work on mobile (test with real device)
- [ ] `ignoreMobileResize: true` if address-bar resize causes jank

**Visual QA**
- [ ] Screenshots capture final animated state (use completeAllAnimations)
- [ ] Full-page scroll triggers all reveal animations
- [ ] Test at all breakpoints defined in matchMedia
