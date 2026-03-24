# Canvas Patterns — Full Implementations

> Source: [GSAP Canvas Particles CodePen](https://codepen.io/GreenSock/pen/NWZRRNb)

---

## Particle Orbit System

A spiral particle system where GSAP animates plain objects and an `onUpdate` callback renders sprite images to canvas. Particles orbit inward with staggered timing, creating a continuous vortex effect.

### HTML

```html
<main>
  <canvas></canvas>
</main>
```

### CSS

```css
html, body {
  margin: 0;
  padding: 0;
}

main {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100vh;
  background: rgb(14, 16, 15);
}
```

### JavaScript

```js
// ---------------------------------------------------------------------------
// 1. Canvas setup
// ---------------------------------------------------------------------------
// Get the canvas element and its 2D rendering context.
// Set canvas dimensions to fill the viewport.
const c = document.querySelector('canvas')
const ctx = c.getContext('2d')
let cw = (c.width = window.innerWidth)
let ch = (c.height = window.innerHeight)

// radius determines the starting distance of particles from center.
// Using the larger viewport dimension ensures particles start offscreen.
let radius = Math.max(cw, ch)

// ---------------------------------------------------------------------------
// 2. Particle array — plain JS objects, not DOM elements
// ---------------------------------------------------------------------------
// Each particle holds animated properties (x, y, scale, rotate) plus an
// Image object for its sprite. GSAP will tween the numeric properties;
// the draw function reads them every frame.
const particles = Array(99)

for (let i = 0; i < particles.length; i++) {
  particles[i] = {
    x: 0,
    y: 0,
    scale: 0,
    rotate: 0,
    img: new Image(),
  }
  // Cycle through 21 different flair sprite images (indices 2–22)
  particles[i].img.src =
    'https://assets.codepen.io/16327/flair-' + (2 + (i % 21)) + '.png'
}

// ---------------------------------------------------------------------------
// 3. Timeline with onUpdate draw loop
// ---------------------------------------------------------------------------
// The timeline animates the particle objects. The onUpdate callback fires
// every time GSAP updates any value — this is our "render loop."
//
// Key patterns:
//   - fromTo with function-based values: each particle gets a unique starting
//     position based on its index, using trigonometric functions to create
//     a spiral distribution around the center.
//   - The "from" x/y use cos/sin with angle*10 to create a multi-loop spiral
//     pattern (10 full rotations spread across all particles).
//   - Negative stagger (each: -0.05) means the last particle starts first,
//     creating an inward spiral motion.
//   - repeat: -1 on the stagger makes each particle's tween loop forever.
const tl = gsap
  .timeline({ onUpdate: draw })
  .fromTo(
    particles,
    {
      // Function-based "from" values — evaluated per particle
      x: (i) => {
        // Distribute particles in a spiral pattern using their index
        const angle = (i / particles.length) * Math.PI * 2 - Math.PI / 2
        // angle*10 creates 10 full spiral loops
        return Math.cos(angle * 10) * radius
      },
      y: (i) => {
        const angle = (i / particles.length) * Math.PI * 2 - Math.PI / 2
        return Math.sin(angle * 10) * radius
      },
      scale: 1.1,
      rotate: 0,
    },
    {
      duration: 5,
      ease: 'sine',
      x: 0,            // All particles converge to center
      y: 0,
      scale: 0,         // Shrink to nothing at center
      rotate: -3,       // ~170 degrees rotation during flight
      stagger: {
        each: -0.05,    // Negative = last particle starts first (inward spiral)
        repeat: -1,     // Each particle's individual tween repeats forever
      },
    },
    0
  )
  .seek(99) // Jump far ahead so the repeating staggers are mid-flight
            // on first render (avoids a blank start)

// ---------------------------------------------------------------------------
// 4. Draw function — the canvas rendering pipeline
// ---------------------------------------------------------------------------
// Called by GSAP on every frame via the timeline's onUpdate callback.
//
// Pattern:
//   1. Sort particles by scale for z-ordering (smaller = further away = drawn first)
//   2. Clear the entire canvas
//   3. For each particle: translate to center, apply rotation, draw the sprite
//   4. Reset transform after each particle (cheaper than save/restore)
function draw() {
  // Sort by scale so smaller (more distant) particles render behind larger ones
  particles.sort((a, b) => a.scale - b.scale)

  // Clear previous frame
  ctx.clearRect(0, 0, cw, ch)

  particles.forEach((p, i) => {
    // Move origin to canvas center — all particle positions are relative to center
    ctx.translate(cw / 2, ch / 2)

    // Apply particle rotation
    ctx.rotate(p.rotate)

    // Draw the sprite image at the particle's animated position and size.
    // p.x/p.y are offsets from center; width/height are scaled by p.scale.
    ctx.drawImage(
      p.img,
      p.x,
      p.y,
      p.img.width * p.scale,
      p.img.height * p.scale
    )

    // Reset the transform matrix — faster than ctx.save()/ctx.restore()
    ctx.resetTransform()
  })
}

// ---------------------------------------------------------------------------
// 5. Resize handler — recalculate dimensions and invalidate
// ---------------------------------------------------------------------------
// When the viewport resizes:
//   1. Update canvas dimensions (this also clears the canvas)
//   2. Recalculate the radius so particles start from the correct offscreen distance
//   3. Call tl.invalidate() — this forces GSAP to re-evaluate the function-based
//      "from" values (the trig calculations) using the new radius on the next cycle
window.addEventListener('resize', () => {
  cw = c.width = innerWidth
  ch = c.height = innerHeight
  radius = Math.max(cw, ch)
  tl.invalidate()
})

// ---------------------------------------------------------------------------
// 6. Click-to-toggle play/pause via timeScale
// ---------------------------------------------------------------------------
// Instead of tl.pause()/tl.play(), we tween the timeline's timeScale property.
// This creates a smooth ease into pause (timeScale → 0) or play (timeScale → 1)
// rather than an abrupt stop/start.
//
// tl.isActive() returns true when the timeline is playing — we toggle to 0.
// When paused (timeScale === 0, isActive() is false) — we toggle back to 1.
c.addEventListener('pointerup', () => {
  gsap.to(tl, {
    timeScale: tl.isActive() ? 0 : 1,
  })
})
```

---

## Key Patterns Summary

| Pattern | Detail |
|---|---|
| **Plain object animation** | GSAP tweens `{x, y, scale, rotate}` — never touches DOM/canvas directly |
| **onUpdate as render loop** | Timeline's `onUpdate` callback replaces manual `requestAnimationFrame` |
| **Function-based values** | `x: (i) => ...` gives each particle a unique starting position |
| **Trigonometric spiral** | `cos(angle*10)` / `sin(angle*10)` distributes particles in 10 spiral loops |
| **Negative stagger** | `each: -0.05` — last index starts first, creating inward motion |
| **seek(99)** | Jump ahead so infinite-repeating staggers are mid-animation on load |
| **Z-sort by scale** | `sort((a,b) => a.scale - b.scale)` — smaller particles drawn first (behind) |
| **resetTransform** | Faster than `save()`/`restore()` for per-particle transform resets |
| **invalidate on resize** | Re-evaluates function-based from-values with updated dimensions |
| **timeScale toggle** | `gsap.to(tl, { timeScale: 0 })` for smooth pause; `1` for smooth resume |

---

## Canvas Morphs — MorphSVG Rendered to Canvas

> Source: [GSAP Canvas Morphing CodePen](https://codepen.io/GreenSock/pen/WYWZab)

MorphSVG plugin can render morph animations directly to canvas instead of DOM SVG elements. SVG paths are defined in a hidden SVG for shape data, but the actual rendering happens on a `<canvas>` element via a custom `render` callback. This avoids DOM reflows entirely while keeping the expressive power of SVG path morphing.

### HTML

```html
<div class="container">
  <canvas></canvas>
</div>

<!-- Hidden SVG holds shape definitions only — never rendered to DOM -->
<svg id="svg-stage" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" fill="none">
  <defs>
    <linearGradient id="grad" x1="0" y1="0" x2="99" y2="99" gradientUnits="userSpaceOnUse">
      <stop offset="0.2" stop-color="rgb(255, 135, 9)" />
      <stop offset="0.7" stop-color="rgb(247, 189, 248)" />
    </linearGradient>
    <path id="shape2" d="M20,1 85,1 85,66 51,98 51,66 20,66z" />
    <path id="shape3" d="M47.1,0.8 73.3,0.8 61.9,37.2 77.1,37.2 30.7,99.4 45.8,51.9 29,51.9z" />
    <path id="shape4" d="M50,1l49,49L50,99L1,50L50,1z" />
  </defs>

  <path id="shape1" fill="url(#grad)"
    d="M74.6 50.2h-.2v-.4h.2a24.4 24.4 0 1 0-24.4-24.4v.2h-.4v-.2a24.4 24.4 0 1 0-24.4 24.4h.2v.4h-.2a24.4 24.4 0 1 0 24.4 24.4v-.2h.4v.2a24.4 24.4 0 1 0 24.4-24.4z" />
</svg>
```

### CSS

```css
body {
  width: 100%;
  height: 100vh;
  margin: 0;
  padding: 0;
  background: #0e100f;
  display: flex;
  align-items: center;
  justify-content: center;
}

.container {
  display: inline-block;
  text-align: center;
  visibility: hidden; /* revealed by gsap.set autoAlpha */
}

.container canvas {
  width: 200px;
  height: 200px;
}
```

### JavaScript

```js
// ---------------------------------------------------------------------------
// 1. Canvas setup & viewBox mapping
// ---------------------------------------------------------------------------
// The SVG viewBox is 100x100. We render to a 200x200 canvas, so sizeRatio
// maps SVG coordinate space to canvas pixel space.
let shape    = document.querySelector("#shape1");
let canvas   = document.querySelector("canvas");
let ctx      = canvas.getContext("2d");
let size     = 200;
let viewBox  = { x: 0, y: 0, width: 100, height: 100 };
let sizeRatio = size / viewBox.width;

// Gradient in SVG coordinate space — matches the SVG linearGradient
const gradient = ctx.createLinearGradient(0, 0, viewBox.width, viewBox.height);
gradient.addColorStop(0.2, "rgb(255, 135, 9)");
gradient.addColorStop(0.7, "rgb(247, 189, 248)");

// Apply HiDPI scaling before any drawing
setDisplay();

gsap.defaults({ ease: "power2.inOut" });
gsap.set(".container", { autoAlpha: 1 });

// ---------------------------------------------------------------------------
// 2. MorphSVG timeline with custom render callback
// ---------------------------------------------------------------------------
// Key insight: morphSVG accepts a `render` function that receives the
// interpolated rawPath data. Instead of updating a DOM <path> element,
// we draw it to canvas. The SVG element (#shape1) is just a data source.
//
// The rawPath is an array of segments, each segment is a flat array of
// cubic bezier coordinates: [x0, y0, cp1x, cp1y, cp2x, cp2y, x1, y1, ...]
var tl = gsap.timeline({
  repeat: -1,
  id: "morphing",
  paused: false,
  defaults: { duration: 2, ease: "expo.inOut" },
})
  .to(shape, { morphSVG: { shape: "#shape2", render: draw } })
  .to(shape, { morphSVG: { shape: "#shape3", render: draw } }, "+=0.5")
  .to(shape, { morphSVG: { shape: "#shape4", render: draw } }, "+=0.5")
  .to(shape, { morphSVG: { shape: "#shape1", render: draw } }, "+=0.5");

// Force first render so canvas isn't blank on load
tl.progress(0.0001);

// Click to toggle pause/play
document.body.addEventListener("click", function () {
  tl.paused(!tl.paused());
});

// ---------------------------------------------------------------------------
// 3. Custom render function — draws rawPath to canvas
// ---------------------------------------------------------------------------
// MorphSVG calls this on every frame with the interpolated path data.
// rawPath is an array of segments (sub-paths). Each segment is a flat
// Float32Array of cubic bezier control points.
//
// Pattern:
//   1. Clear canvas
//   2. Set fill style
//   3. Build a canvas path from rawPath bezier data
//   4. Fill with even-odd rule (handles holes in complex shapes)
function draw(rawPath, target) {
  var l, segment, j, i;
  ctx.clearRect(0, 0, size, size);
  ctx.fillStyle = gradient;
  ctx.beginPath();
  for (j = 0; j < rawPath.length; j++) {
    segment = rawPath[j];
    l = segment.length;
    // Move to the start of each sub-path
    ctx.moveTo(segment[0], segment[1]);
    // Draw cubic bezier curves (6 values per curve: cp1x, cp1y, cp2x, cp2y, x, y)
    for (i = 2; i < l; i += 6) {
      ctx.bezierCurveTo(
        segment[i], segment[i + 1],
        segment[i + 2], segment[i + 3],
        segment[i + 4], segment[i + 5]
      );
    }
    if (segment.closed) {
      ctx.closePath();
    }
  }
  // "evenodd" fill rule correctly handles the clover shape's interior holes
  ctx.fill("evenodd");
}

// ---------------------------------------------------------------------------
// 4. HiDPI display setup
// ---------------------------------------------------------------------------
// Scales the canvas backing store by devicePixelRatio for crisp rendering
// on Retina/HiDPI displays. Then applies the viewBox-to-canvas coordinate
// mapping so SVG path coordinates draw at the correct canvas positions.
function setDisplay() {
  var ratio = window.devicePixelRatio || 1;
  canvas.width  = size * ratio;
  canvas.height = size * ratio;
  // CSS size stays at `size` — backing store is ratio× larger
  gsap.set(canvas, { width: size, height: size });
  // Combine HiDPI ratio with viewBox→canvas mapping
  ratio *= sizeRatio;
  ctx.translate(-viewBox.x * ratio, -viewBox.y * ratio);
  ctx.scale(ratio, ratio);
}
```

---

### Key Patterns Summary

| Pattern | Detail |
|---|---|
| **morphSVG render callback** | `morphSVG: { shape: "#id", render: draw }` — redirects morph output to a custom function instead of DOM |
| **rawPath structure** | Array of segments, each a flat array of cubic bezier coords — `[x0, y0, cp1x, cp1y, cp2x, cp2y, x1, y1, ...]` |
| **SVG as data source** | Hidden SVG `<defs>` hold shape paths — never rendered to DOM, only used by MorphSVG for interpolation |
| **Canvas bezierCurveTo** | Maps directly to rawPath's cubic bezier data — 6 values per curve segment |
| **evenodd fill rule** | `ctx.fill("evenodd")` handles complex shapes with interior holes (like the clover) |
| **HiDPI canvas** | Backing store scaled by `devicePixelRatio`, CSS size stays fixed — crisp on Retina |
| **viewBox mapping** | `ctx.translate` + `ctx.scale` maps SVG coordinate space to canvas pixel space |
| **Gradient in SVG space** | `createLinearGradient` uses viewBox coords (0→100), not canvas pixels — works because of the scale transform |
