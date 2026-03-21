# Vue/Nuxt Full Examples

## Table of Contents

- [Example 1: 3D Tilt Card with Composable Coexistence](#example-1-3d-tilt-card-with-composable-coexistence)
- [Example 2: Multi-Concept Page with Single Context](#example-2-multi-concept-page-with-single-context)
- [Example 3: Accessible Hero with matchMedia](#example-3-accessible-hero-with-matchmedia)

---

## Example 1: 3D Tilt Card with Composable Coexistence

**Pattern**: `useReveal()` owns its own context + a **separate** `tiltCtx` for interactive animations.

```vue
<template>
  <section id="services" ref="sectionRef" class="py-20 min-h-screen">
    <div class="grid grid-cols-1 sm:grid-cols-3 gap-6">
      <div
        v-for="(service, i) in services"
        :key="service.id"
        class="tilt-wrapper reveal"
        :ref="el => tiltWrappers[i] = el"
        @mousemove="onTiltMove($event, i)"
        @mouseleave="onTiltLeave(i)"
      >
        <div class="tilt-shell h-full" :ref="el => tiltShells[i] = el">
          <div class="p-6">
            <div :ref="el => tiltImages[i] = el">
              <img :src="service.img" :alt="service.title" />
            </div>
            <div :ref="el => tiltContents[i] = el">
              <h3>{{ service.title }}</h3>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
const { $gsap: gsap } = useNuxtApp()

const sectionRef = ref(null)
// useReveal has its own internal gsap.context — do NOT mix
const { init, scroll } = useReveal(sectionRef)

// Tilt refs — one per card
const tiltWrappers = ref([])
const tiltShells   = ref([])
const tiltImages   = ref([])
const tiltContents = ref([])

// ── Separate context for interactive tilt ──
let tiltCtx

// Shared tween defaults — defined once, spread into each gsap.to()
const TILT_IN  = { duration: 0.35, ease: 'power2.out', overwrite: 'auto' }
const TILT_OUT = { duration: 0.7,  ease: 'elastic.out(1, 0.4)' }

const onTiltMove = (e, i) => {
  const wrapper = tiltWrappers.value[i]
  const shell   = tiltShells.value[i]
  if (!wrapper || !shell || !tiltCtx) return

  const rect = wrapper.getBoundingClientRect()
  const xPct = ((e.clientX - rect.left) / rect.width  - 0.5) * 2
  const yPct = ((e.clientY - rect.top)  / rect.height - 0.5) * 2

  tiltCtx.applyTilt(shell, tiltImages.value[i], tiltContents.value[i], xPct, yPct)
}

const onTiltLeave = (i) => {
  if (!tiltCtx) return
  tiltCtx.resetTilt(tiltShells.value[i], tiltImages.value[i], tiltContents.value[i])
}

onMounted(() => {
  // useReveal manages its own cleanup
  init(() => scroll())

  // Tilt context — separate from useReveal
  tiltCtx = gsap.context((self) => {
    // Set transformPerspective ONCE (not on every mousemove)
    tiltShells.value.forEach(shell => {
      if (shell) gsap.set(shell, { transformPerspective: 900 })
    })

    self.add('applyTilt', (shell, img, content, xPct, yPct) => {
      shell.style.willChange = 'transform'
      if (img)     img.style.willChange = 'transform'
      if (content) content.style.willChange = 'transform'
      gsap.to(shell, { rotationY: xPct * 12, rotationX: -yPct * 12, scale: 1.03, force3D: true, ...TILT_IN })
      if (img)     gsap.to(img,     { x: xPct * 12, y: yPct * 12, force3D: true, ...TILT_IN })
      if (content) gsap.to(content, { x: xPct * 5,  y: yPct * 5,  force3D: true, ...TILT_IN })
    })

    self.add('resetTilt', (shell, img, content) => {
      if (!shell) return
      gsap.to(shell, {
        rotationX: 0, rotationY: 0, scale: 1, force3D: true, ...TILT_OUT,
        onComplete: () => { shell.style.willChange = 'auto' },
      })
      if (img)     gsap.to(img,     { x: 0, y: 0, force3D: true, ...TILT_OUT,
        onComplete: () => { img.style.willChange = 'auto' },
      })
      if (content) gsap.to(content, { x: 0, y: 0, force3D: true, ...TILT_OUT,
        onComplete: () => { content.style.willChange = 'auto' },
      })
    })
  }, sectionRef.value)
})

onUnmounted(() => {
  // useReveal handles its own cleanup internally
  tiltCtx?.revert()
})
</script>

<style scoped>
.tilt-wrapper { perspective: 1000px; }
.tilt-shell   { transform-style: preserve-3d; }
</style>
```

### Key takeaways

1. **Two contexts, two lifecycles** — `useReveal` owns one, `tiltCtx` owns the other.
2. **ctx.add()** registers `applyTilt` / `resetTilt` so every tween from mousemove is collected.
3. **transformPerspective** set once via `gsap.set()` at mount — not per-frame.
4. **will-change** set on hover start, released in `onComplete` of the leave tween.

---

## Example 2: Multi-Concept Page with Single Context

**Pattern**: One `gsap.context()` for scroll animations + `self.add()` for all event handlers.

```vue
<script setup>
const { $gsap: gsap, $ScrollTrigger: ScrollTrigger } = useNuxtApp()

// Template refs (abbreviated — see full component for all refs)
const containerRef     = ref(null) // bound to root element for context scope
const tiltCards        = ref([])
const tiltContents     = ref([])
const spotlightRef     = ref(null)
const blobRefs         = ref([])

let ctx
const TILT_IN  = { duration: 0.35, ease: 'power2.out', overwrite: 'auto' }
const TILT_OUT = { duration: 0.7,  ease: 'elastic.out(1, 0.4)' }

// ── Setup helpers (called inside context) ──
const setupCounters = () => {
  stats.forEach((stat, i) => {
    const el = counterEls.value[i]
    if (!el) return
    const proxy = { value: 0 }
    ScrollTrigger.create({
      trigger: el.closest('.stat-item') ?? el,
      start: 'top 85%', once: true,
      onEnter() {
        gsap.to(proxy, {
          value: stat.target, duration: 2.2, ease: 'power2.out',
          snap: { value: 1 },
          onUpdate() { el.textContent = String(Math.round(proxy.value)) },
        })
      },
    })
  })
}

const setupReveal = () => {
  const els = gsap.utils.toArray('.reveal')
  gsap.set(els, { y: 28, autoAlpha: 0 })
  els.forEach((el) => {
    gsap.to(el, {
      y: 0, autoAlpha: 1, duration: 0.8, ease: 'power2.out', force3D: true,
      scrollTrigger: { trigger: el, start: 'top 88%', toggleActions: 'play none none reverse' },
    })
  })
}

// ── Event handlers call named ctx methods ──
const onTiltMove = (e, i) => {
  const card = tiltCards.value[i]
  if (!card || !ctx) return
  const rect = card.parentElement.getBoundingClientRect()
  const xPct = ((e.clientX - rect.left) / rect.width  - 0.5) * 2
  const yPct = ((e.clientY - rect.top)  / rect.height - 0.5) * 2
  ctx.applyTilt(card, tiltContents.value[i], null, xPct, yPct)
}

const onTiltLeave = (i) => ctx?.resetTilt(tiltCards.value[i], tiltContents.value[i], null)

// ── Lifecycle ──
onMounted(() => {
  ctx = gsap.context((self) => {
    // 1. Scroll-driven (auto-collected) — selectors scoped to containerRef
    setupCounters()
    setupReveal()

    // 2. One-time setup (auto-collected)
    tiltCards.value.forEach(card => {
      if (card) gsap.set(card, { transformPerspective: 900 })
    })

    // 3. Named handlers for event-driven tweens (collected via ctx.add)
    self.add('applyTilt', (card, content, glow, xPct, yPct) => {
      card.style.willChange = 'transform'
      if (content) content.style.willChange = 'transform'
      gsap.to(card, { rotationY: xPct * 14, rotationX: -yPct * 14, scale: 1.04, force3D: true, ...TILT_IN })
      if (content) gsap.to(content, { x: xPct * 10, y: yPct * 10, force3D: true, ...TILT_IN })
    })

    self.add('resetTilt', (card, content, glow) => {
      if (!card) return
      gsap.to(card, {
        rotationX: 0, rotationY: 0, scale: 1, force3D: true, ...TILT_OUT,
        onComplete: () => { card.style.willChange = 'auto' },
      })
      if (content) gsap.to(content, { x: 0, y: 0, force3D: true, ...TILT_OUT,
        onComplete: () => { content.style.willChange = 'auto' },
      })
    })

    self.add('moveSpotlight', (x, y) => {
      gsap.to(spotlightRef.value, {
        clipPath: `circle(130px at ${x}px ${y}px)`,
        duration: 0.25, ease: 'power2.out', overwrite: 'auto',
      })
    })

    self.add('hideSpotlight', () => {
      gsap.to(spotlightRef.value, {
        clipPath: 'circle(0px at 50% 50%)', duration: 0.5, ease: 'power2.inOut',
        overwrite: 'auto',
      })
    })

    self.add('moveBlobs', (mx, my) => {
      blobRefs.value.forEach((blob) => {
        if (!blob) return
        gsap.to(blob, { x: mx, y: my, duration: 0.9, ease: 'elastic.out(1.2, 0.3)', overwrite: 'auto' })
      })
    })

    self.add('resetBlobs', () => {
      blobRefs.value.forEach((blob) => {
        if (!blob) return
        gsap.to(blob, { x: homeX, y: homeY, duration: 1.6, ease: 'elastic.out(1, 0.4)', overwrite: 'auto' })
      })
    })
  }, containerRef.value) // scope: selectors only match inside this element
})

onUnmounted(() => {
  ctx?.revert()
})
</script>
```

### Key takeaways

1. **One context, many setup helpers** — all functions called inside `gsap.context()` callback get collected.
2. **Six named handlers** via `self.add()` — tilt, spotlight, spring, each pair (apply + reset).
3. **No `scrollTriggers[]` array** — `ctx.revert()` kills ScrollTriggers automatically.
4. **`overwrite: 'auto'`** on every rapid-fire tween to prevent animation buildup.
5. **`force3D: true`** on transform-heavy tweens for GPU compositing.

---

## Example 3: Accessible Hero with matchMedia

**Pattern**: `gsap.matchMedia()` with responsive breakpoints + `prefers-reduced-motion` in a single context.

```vue
<template>
  <section ref="heroRef" class="hero min-h-dvh flex items-center">
    <div class="hero-content max-w-7xl mx-auto px-6">
      <h1 ref="titleRef" class="reveal text-5xl lg:text-7xl font-bold">
        Build the Future
      </h1>
      <p ref="subtitleRef" class="reveal mt-6 text-xl text-gray-400">
        Next-generation digital experiences
      </p>
      <div ref="ctaRef" class="reveal mt-10">
        <button class="btn-primary">Get Started</button>
      </div>
    </div>
    <div ref="bgRef" class="hero-bg absolute inset-0 -z-10" />
  </section>
</template>

<script setup>
const { $gsap: gsap, $ScrollTrigger: ScrollTrigger } = useNuxtApp()

const heroRef     = ref(null)
const titleRef    = ref(null)
const subtitleRef = ref(null)
const ctaRef      = ref(null)
const bgRef       = ref(null)

let mm

onMounted(() => {
  mm = gsap.matchMedia()

  // ── Reduced motion: show everything immediately ──
  mm.add('(prefers-reduced-motion: reduce)', () => {
    gsap.set([titleRef.value, subtitleRef.value, ctaRef.value], { autoAlpha: 1, y: 0 })
  })

  // ── Desktop: full animation suite ──
  mm.add('(min-width: 1024px) and (prefers-reduced-motion: no-preference)', () => {
    const tl = gsap.timeline({ defaults: { ease: 'power3.out' } })

    tl.from(bgRef.value, { scale: 1.15, autoAlpha: 0, duration: 1.5 })
      .from(titleRef.value, { autoAlpha: 0, y: 60, duration: 1 }, '-=1')
      .from(subtitleRef.value, { autoAlpha: 0, y: 40, duration: 0.8 }, '-=0.5')
      .from(ctaRef.value, { autoAlpha: 0, y: 30, duration: 0.6 }, '-=0.3')

    // Parallax on scroll
    gsap.to(bgRef.value, {
      yPercent: -20,
      ease: 'none',
      scrollTrigger: {
        trigger: heroRef.value,
        start: 'top top',
        end: 'bottom top',
        scrub: 0.5,
      },
    })
  })

  // ── Mobile: simpler, faster animations ──
  mm.add('(max-width: 1023px) and (prefers-reduced-motion: no-preference)', () => {
    gsap.from([titleRef.value, subtitleRef.value, ctaRef.value], {
      autoAlpha: 0,
      y: 30,
      duration: 0.6,
      stagger: 0.15,
      ease: 'power2.out',
    })
    // No parallax on mobile — saves battery + avoids scroll jank
  })
})

onUnmounted(() => {
  mm?.revert() // reverts ALL matchMedia contexts
})
</script>

<style scoped>
.hero-bg {
  background: linear-gradient(135deg, #0a0a0a, #1a1a2e);
}
</style>
```

### Key takeaways

1. **Three matchMedia conditions** — reduced-motion, desktop, mobile. Each auto-reverts on change.
2. **Reduced motion first** — shows content immediately with `gsap.set()`, no animation.
3. **Desktop gets full treatment** — timeline sequence + parallax on scroll.
4. **Mobile gets lighter animations** — simpler fade, no parallax (saves battery).
5. **`mm.revert()`** cleans up all three contexts in `onUnmounted`.
6. **No need for `useReducedMotion` composable** — matchMedia handles it declaratively.
