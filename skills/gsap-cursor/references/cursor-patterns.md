# Cursor Patterns Reference

## Table of Contents
- [Flair Particle Trail](#flair-particle-trail)
- [Spring Physics (Elastic Cursor Followers)](#spring-physics-elastic-cursor-followers)
- [Spotlight Cursor (Clip-Path Circle)](#spotlight-cursor-clip-path-circle)
- [Circuit Glow (CSS Custom Properties)](#circuit-glow-css-custom-properties)
- [Cursor-Tracking Image Preview](#cursor-tracking-image-preview)
- [Cursor-Driven Perspective Tilt](#cursor-driven-perspective-tilt)

---

## Flair Particle Trail

Distance-based particle spawning using `gsap.ticker`, pool recycling with `gsap.utils.wrap`, and smooth interpolation. Particles appear at the cursor position, animate with scale/rotation/fade, then recycle.

### Pool setup

```html
<!-- Pre-render a pool of shape elements (e.g. 20-30) -->
<div v-for="i in 30" :key="i" ref="flairRefs" class="flair" />
```

```css
.flair {
  position: fixed;
  pointer-events: none;
  width: 24px;
  height: 24px;
  /* Style as desired: circles, stars, SVG shapes */
}
```

### Core logic

```js
const flair = gsap.utils.toArray('.flair')
const gap = 100            // minimum px between spawns
let index = 0
const wrapper = gsap.utils.wrap(0, flair.length)

let mousePos = { x: 0, y: 0 }
let lastMousePos = { ...mousePos }
let cachedMousePos = { ...mousePos }

gsap.defaults({ duration: 1 })

// Track mouse position (lightweight — no tweens here)
window.addEventListener('mousemove', (e) => {
  mousePos = { x: e.x, y: e.y }
})

// Ticker callback — runs every frame
function imageTrail() {
  const travelDistance = Math.hypot(
    lastMousePos.x - mousePos.x,
    lastMousePos.y - mousePos.y
  )

  // Smooth interpolation for cached position
  cachedMousePos.x = gsap.utils.interpolate(
    cachedMousePos.x || mousePos.x, mousePos.x, 0.1
  )
  cachedMousePos.y = gsap.utils.interpolate(
    cachedMousePos.y || mousePos.y, mousePos.y, 0.1
  )

  if (travelDistance > gap) {
    spawnParticle()
    lastMousePos = { ...mousePos }
  }
}

gsap.ticker.add(imageTrail)
```

### Spawn + animate

```js
function spawnParticle() {
  const wrappedIndex = wrapper(index)
  const shape = flair[wrappedIndex]

  // Kill any running tweens on this recycled element
  gsap.killTweensOf(shape)
  gsap.set(shape, { clearProps: 'all' })

  // Position at cursor
  gsap.set(shape, {
    opacity: 1,
    left: mousePos.x,
    top: mousePos.y,
    xPercent: -50,
    yPercent: -50,
  })

  // Animate: scale in with elastic, rotate, then fall away
  const tl = gsap.timeline()
  tl.from(shape, {
    opacity: 0,
    scale: 0,
    ease: 'elastic.out(1, 0.3)',
  })
  .to(shape, {
    rotation: 'random([-360, 360])',
  }, '<')
  .to(shape, {
    y: '120vh',
    ease: 'back.in(0.4)',
    duration: 1,
  }, 0)

  index++
}
```

### Cleanup (Vue/Nuxt)

```js
onUnmounted(() => {
  gsap.ticker.remove(imageTrail)
  window.removeEventListener('mousemove', onMouseMove)
})
```

**Key rules**: `gsap.utils.wrap` recycles pool elements | `gsap.killTweensOf` before reuse | `gsap.utils.interpolate` smooths cached position | distance threshold (`gap`) prevents over-spawning | `gsap.ticker.add` for frame-synced checks (not `requestAnimationFrame`)

---

## Spring Physics (Elastic Cursor Followers)

Multiple blobs chase the cursor with different durations for a staggered, physics-like feel. Rest positions are **percentage-based** relative to the container.

### Blob configuration

```js
const blobs = [
  { x: 15, y: 40, size: 90,  dur: 0.9, colorStop: 'rgba(20,184,166,0.4)' },
  { x: 30, y: 55, size: 55,  dur: 0.6, colorStop: 'rgba(var(--glow-secondary-rgb),0.35)' },
  { x: 55, y: 30, size: 120, dur: 1.1, colorStop: 'rgba(var(--glow-primary-rgb),0.3)' },
  { x: 68, y: 65, size: 60,  dur: 0.7, colorStop: 'rgba(6,182,212,0.35)' },
  { x: 80, y: 25, size: 75,  dur: 0.8, colorStop: 'rgba(20,184,166,0.35)' },
]
```

### Initial placement — percentage of container

```js
const setupVisuals = () => {
  const cw = container.offsetWidth
  const ch = container.offsetHeight
  blobRefs.value.forEach((blob, i) => {
    if (!blob) return
    const b = blobs[i]
    blob.style.width = `${b.size}px`
    blob.style.height = `${b.size}px`
    blob.style.background = `radial-gradient(circle, ${b.colorStop}, transparent)`
    blob.style.filter = `blur(${Math.round(b.size / 4)}px)`
    gsap.set(blob, { xPercent: -50, yPercent: -50, x: (cw * b.x) / 100, y: (ch * b.y) / 100 })
  })
}
```

### moveBlobs + resetBlobs

```js
onMounted(() => {
  ctx = gsap.context((self) => {
    setupVisuals()

    self.add('moveBlobs', (mx, my) => {
      blobRefs.value.forEach((blob, i) => {
        if (!blob) return
        gsap.to(blob, {
          x: mx, y: my,
          duration: blobs[i].dur,
          ease: 'elastic.out(1.2, 0.3)',
          overwrite: 'auto', force3D: true,
        })
      })
    })

    self.add('resetBlobs', () => {
      const cw = container.offsetWidth
      const ch = container.offsetHeight
      blobRefs.value.forEach((blob, i) => {
        if (!blob) return
        const b = blobs[i]
        gsap.to(blob, {
          x: (cw * b.x) / 100, y: (ch * b.y) / 100,
          duration: 1.6, ease: 'elastic.out(1, 0.4)', force3D: true,
        })
      })
    })
  }, containerRef.value?.closest('section'))
})
```

**Key rules**: rest positions are **percentage-based** (`cw * b.x / 100`) | on mousemove blobs go to **exact** cursor position | unique `duration` per blob creates stagger | `elastic.out(1.2, 0.3)` for overshoot on move, softer on reset

---

## Spotlight Cursor (Clip-Path Circle)

Reveal a hidden layer by moving a `clipPath` circle to follow the cursor.

```js
onMounted(() => {
  ctx = gsap.context((self) => {
    const spotlight = spotlightRef.value
    gsap.set(spotlight, { clipPath: 'circle(0px at 50% 50%)' })

    self.add('moveSpotlight', (x, y) => {
      gsap.to(spotlight, {
        clipPath: `circle(130px at ${x}px ${y}px)`,
        duration: 0.25, ease: 'power2.out', overwrite: 'auto',
      })
    })

    self.add('hideSpotlight', () => {
      gsap.to(spotlight, {
        clipPath: 'circle(0px at 50% 50%)',
        duration: 0.4, ease: 'power2.inOut', overwrite: 'auto',
      })
    })
  }, scopeRef.value)
})
```

**Key rules**: `overwrite: 'auto'` prevents pile-up | initial state via `gsap.set` for clean revert | short duration (0.25s) for responsive feel

---

## Circuit Glow (CSS Custom Properties)

Pipe mouse position directly to CSS custom properties without GSAP tweening. Uses a **real div element** with `mask-image` to reveal a glow layer.

### Template

```vue
<div class="relative w-full" @mousemove="onMouseMove">
  <div ref="circuitWrap" class="relative z-0" />
  <div ref="glowRef" class="circuit-glow absolute inset-0 z-[1]" style="--glow-x: 0.5; --glow-y: 0.5" />
</div>
```

### JS — unitless ratios (0-1), not pixels

```js
const onMouseMove = (e) => {
  if (!glowRef.value) return
  const rect = e.currentTarget.getBoundingClientRect()
  glowRef.value.style.setProperty('--glow-x', (e.clientX - rect.left) / rect.width)
  glowRef.value.style.setProperty('--glow-y', (e.clientY - rect.top) / rect.height)
}
```

### CSS

```css
.circuit-glow {
  background-image: url('~/assets/images/circuit-board.svg');
  @apply bg-cover bg-center bg-no-repeat opacity-0;
  filter: invert(1) sepia(1) saturate(5) hue-rotate(130deg) brightness(0.8);
  mask-image: radial-gradient(
    circle 350px at calc(var(--glow-x) * 100%) calc(var(--glow-y) * 100%),
    black 0%, transparent 100%
  );
  -webkit-mask-image: radial-gradient(
    circle 350px at calc(var(--glow-x) * 100%) calc(var(--glow-y) * 100%),
    black 0%, transparent 100%
  );
}
div:hover .circuit-glow {
  @apply opacity-35 transition-opacity duration-500;
}
```

**Key rules**: real `<div>` (not `::after`) | stores **unitless ratios** (0-1), CSS uses `calc(var * 100%)` | opacity is CSS-transition-based, no GSAP | direct `style.setProperty` for maximum speed

---

## Cursor-Tracking Image Preview

Source: [CodePen by GSAP](https://codepen.io/GreenSock/pen/PwqrzeG)

Images follow the cursor when hovering over list items. Uses `gsap.quickTo` for smooth position tracking with instant snap on first enter.

### HTML structure

```html
<ul role="list">
  <li class="container">
    <img class="swipeimage" src="portrait-image.jpg">
    <div class="text">
      <h3>Item label text</h3>
    </div>
  </li>
  <!-- Repeat for each item -->
</ul>
```

### CSS — fixed-position image at cursor

```css
.container img.swipeimage {
  position: fixed;
  top: 0;
  left: 0;
  width: 350px;
  height: 350px;
  object-fit: cover;
  transform: translateX(-50%) translateY(-50%);
  z-index: 9;
  opacity: 0;
  visibility: hidden;
  pointer-events: none;
}
```

### Core logic — quickTo with first-enter snap

```js
// Center the image origin at its midpoint
gsap.set(".container img.swipeimage", { yPercent: -50, xPercent: -50 });

let firstEnter;

gsap.utils.toArray(".container").forEach((el) => {
  const image = el.querySelector("img.swipeimage"),
    // Pre-compile quickTo tweens — much faster than gsap.to on every mousemove
    setX = gsap.quickTo(image, "x", { duration: 0.4, ease: "power3" }),
    setY = gsap.quickTo(image, "y", { duration: 0.4, ease: "power3" }),
    align = (e) => {
      if (firstEnter) {
        // On first enter, snap immediately (pass start = end to skip interpolation)
        // See: https://gsap.com/docs/v3/GSAP/gsap.quickTo()/#optionally-define-a-start-value
        setX(e.clientX, e.clientX);
        setY(e.clientY, e.clientY);
        firstEnter = false;
      } else {
        setX(e.clientX);
        setY(e.clientY);
      }
    },
    startFollow = () => document.addEventListener("mousemove", align),
    stopFollow = () => document.removeEventListener("mousemove", align),
    // Fade tween — paused, played/reversed on enter/leave
    fade = gsap.to(image, {
      autoAlpha: 1,          // opacity + visibility in one property
      ease: "none",
      paused: true,
      duration: 0.1,
      onReverseComplete: stopFollow  // Only remove listener after fade-out completes
    });

  el.addEventListener("mouseenter", (e) => {
    firstEnter = true;
    fade.play();
    startFollow();
    align(e);               // Immediately position on enter (no lag)
  });

  el.addEventListener("mouseleave", () => fade.reverse());
});
```

**Key patterns**: `gsap.quickTo` pre-compiles position tweens for 60fps tracking | `quickTo(value, startValue)` two-arg form snaps instantly on first enter (no interpolation lag) | `autoAlpha` handles both opacity and visibility | paused tween with `play()`/`reverse()` for enter/leave toggling | `onReverseComplete` defers listener removal until fade-out finishes | `gsap.set` with `yPercent: -50, xPercent: -50` centers image at cursor

---

## Cursor-Driven Perspective Tilt

Source: [CodePen by GSAP](https://codepen.io/GreenSock/pen/qBzaNQy)

Official GSAP 3D perspective tilt — an element tilts based on pointer position using `gsap.quickTo` for rotationX, rotationY, x, y. Resets smoothly on pointer leave.

### HTML structure

```html
<main>
  <div class="logo-outer">
    <!-- Any content: SVG, image, card, etc. -->
    <svg class="logo" viewBox="0 0 623 231">...</svg>
  </div>
</main>
```

### CSS — perspective container

```css
main {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100vh;
}

.logo-outer {
  width: 50%;
  height: 50%;
  border-radius: 3rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

svg.logo {
  width: 66%;
  max-height: 66%;
}
```

### Core logic — quickTo for rotation + translation

```js
const main = document.querySelector("main");

// Set perspective on the parent — required for 3D transforms on children
gsap.set("main", { perspective: 650 });

// Pre-compile 4 quickTo instances: outer container rotates, inner element translates
const outerRX = gsap.quickTo(".logo-outer", "rotationX", { ease: "power3" });
const outerRY = gsap.quickTo(".logo-outer", "rotationY", { ease: "power3" });
const innerX = gsap.quickTo(".logo", "x", { ease: "power3" });
const innerY = gsap.quickTo(".logo", "y", { ease: "power3" });

main.addEventListener("pointermove", (e) => {
  // Map pointer position to rotation/translation ranges using gsap.utils.interpolate
  // Vertical pointer position → rotationX (15 at top, -15 at bottom)
  outerRX(gsap.utils.interpolate(15, -15, e.y / window.innerHeight));
  // Horizontal pointer position → rotationY (-15 at left, 15 at right)
  outerRY(gsap.utils.interpolate(-15, 15, e.x / window.innerWidth));
  // Inner element shifts slightly for parallax depth effect
  innerX(gsap.utils.interpolate(-30, 30, e.x / window.innerWidth));
  innerY(gsap.utils.interpolate(-30, 30, e.y / window.innerHeight));
});

// Reset all transforms smoothly on pointer leave
main.addEventListener("pointerleave", (e) => {
  outerRX(0);
  outerRY(0);
  innerX(0);
  innerY(0);
});
```

**Key patterns**: `gsap.set(parent, { perspective: 650 })` enables 3D space for children | `gsap.quickTo` pre-compiles 4 independent tweens (rotation + translation) | `gsap.utils.interpolate(min, max, progress)` maps normalized pointer position (0-1) to value ranges | outer container rotates (rotationX/Y) while inner element translates (x/y) for parallax depth | `pointerleave` resets all quickTo instances to 0 for smooth return | uses `pointermove`/`pointerleave` (not mouse events) for touch+mouse compatibility
