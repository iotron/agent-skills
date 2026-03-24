# SVG Patterns Reference

## Table of Contents
- [Circuit Tree Pattern (Triple-Layer Animation)](#circuit-tree-pattern-triple-layer-animation)
- [CTAHud 5-Layer System](#ctahud-5-layer-system)

---

## Circuit Tree Pattern (Triple-Layer Animation)

Three synced 10s timelines via `.add()` on a master with `repeatRefresh: true`. All layers target the same `allEls` array. No stagger on fade or wave layers; only colorAnime uses stagger.

```js
const COLORS = '#02f4c8|#41bbf6|#2dd4bf|#34d399|#06b6d4|#0ea5e9'
const randomColorStr = `random([${COLORS.split('|').map(c => `"${c}"`).join(',')}])`

const allEls = [...circuitWrap.value.querySelectorAll('polygon, path, rect, line, circle, polyline, ellipse')]
allEls.forEach(el => { el.style.cssText = 'fill:transparent;stroke-linecap:round;stroke-linejoin:round' })

gsap.set(allEls, { stroke: randomColorStr, strokeWidth: '0.3rem', autoAlpha: 0 })

gsap.timeline({ repeat: -1, repeatRefresh: true })
  .add(fadeAnime(), 0)
  .add(waveAnime(), 0)
  .add(colorAnime(), 3)
```

`repeatRefresh: true` lives on the **master** timeline so `randomColorStr` re-rolls every cycle.

### Layer 1: fadeAnime — breathing envelope (10s)

1s fade-in, 8s breathing glow (0.3 to 0.6, yoyo x3), 1s fade-out.

```js
function fadeAnime() {
  const tl = gsap.timeline()
  tl.fromTo(allEls, { autoAlpha: 0 }, { autoAlpha: 0.3, duration: 1, ease: 'sine.in' })
    .to(allEls, { autoAlpha: 0.6, duration: 2, ease: 'sine.inOut', yoyo: true, repeat: 3 })
    .to(allEls, { autoAlpha: 0, duration: 1, ease: 'sine.out' })
  return tl
}
```

### Layer 2: waveAnime — DrawSVG 3-phase (10s)

Three sequential tweens: build (3s), peak (4s), collapse (3s). First tween also sets `stroke: randomColorStr`.

```js
function waveAnime() {
  const tl = gsap.timeline()
  tl.fromTo(allEls, { drawSVG: 0 }, { drawSVG: '40% 50%', stroke: randomColorStr, duration: 3, ease: 'sine.in' })
    .to(allEls, { drawSVG: '73% 80%', duration: 4, ease: 'power1.inOut' })
    .to(allEls, { drawSVG: '100% 100%', strokeWidth: '0.2rem', duration: 3, ease: 'sine.out' })
  return tl
}
```

### Layer 3: colorAnime — elastic fill sweep

Positioned at `3` on master. Uses `yoyo: true, repeat: 1` — fill sweeps in then back out.

```js
function colorAnime() {
  const tl = gsap.timeline({ yoyo: true, repeat: 1 })
  tl.to(allEls, {
    fill: randomColorStr, duration: 2,
    stagger: { each: 0.005, from: 'edges' }, ease: 'elastic',
  })
  return tl
}
```

---

## CTAHud 5-Layer System

Five concurrent layers on a HUD circuit board SVG. Reveal plays once; remaining four loop.

```js
const master = gsap.timeline()
master.add(revealTl, 0)       // once
master.add(colorTl, 1.5)      // looping layers start after reveal
master.add(waveTl, 1.5)
master.add(blinkTl, 1.5)
master.add(glowTl, 1.5)
```

**Layer 1 — Reveal**: DrawSVG from center, plays once.
```js
revealTl.from(allPaths, {
  drawSVG: '50% 50%', duration: 1.5, ease: 'power2.inOut',
  stagger: { each: 0.02, from: 'edges' },
})
```

**Layer 2 — Color cycling**: random teal/cyan fills with `repeatRefresh: true`, elastic easing, `from: 'edges'` stagger.

**Layer 3 — Wave**: sweeping DrawSVG 73-80% to 100%, 5s yoyo.

**Layer 4 — Blink**: circles `scale: 1.3`, `transformOrigin: '50% 50%'`, random stagger, yoyo.

**Layer 5 — Glow**: animate `feGaussianBlur` stdDeviation via `attr: { stdDeviation: 4 }`.

```html
<filter id="hud-glow"><feGaussianBlur class="blur-node" stdDeviation="10" /></filter>
```

```js
glowTl = gsap.timeline({ repeat: -1, yoyo: true })
glowTl.to('.blur-node', { attr: { stdDeviation: 4 }, duration: 1.5, ease: 'sine.inOut' })
```

---

## Dynamic Morphing — SVG Shape Overlays

Elegant page/section transition using MorphSVG-style shape overlays. Multiple gradient-filled paths animate via procedurally generated bezier curves, creating a fluid curtain effect. Click to toggle open/close.

> Source: Forked from [Blake Bowen's CodePen](https://codepen.io/osublake/pen/BYwgBg), enhanced with GSAP gradients.

### HTML — SVG overlay with gradient fills

```html
<svg class="shape-overlays" viewBox="0 0 100 100" preserveAspectRatio="none">
  <defs>
    <linearGradient id="gradient1" x1="0%" y1="0%" x2="0%" y2="100%">
      <stop offset="0%"   stop-color="#ff8709"/>
      <stop offset="100%" stop-color="#f7bdf8"/>
    </linearGradient>
    <linearGradient id="gradient2" x1="0%" y1="0%" x2="0%" y2="100%">
      <stop offset="0%"   stop-color="#ffd9b0"/>
      <stop offset="100%" stop-color="#ff8709"/>
    </linearGradient>
  </defs>
  <path class="shape-overlays__path" fill="url(#gradient2)"></path>
  <path class="shape-overlays__path" fill="url(#gradient1)"></path>
</svg>
```

### JS — Procedural path morphing with randomized delay

```js
let overlay = document.querySelector(".shape-overlays");
let paths = document.querySelectorAll(".shape-overlays__path");

let numPoints = 10;
let numPaths = paths.length;
let delayPointsMax = 0.3;
let delayPerPath = 0.25;
let duration = 0.9;
let isOpened = false;
let pointsDelay = [];
let allPoints = [];

let tl = gsap.timeline({
  onUpdate: render,
  defaults: { ease: "power2.inOut", duration: 0.9 }
});

for (let i = 0; i < numPaths; i++) {
  let points = [];
  allPoints.push(points);
  for (let j = 0; j < numPoints; j++) {
    points.push(100);
  }
}

overlay.addEventListener("click", onClick);

function onClick() {
  if (!tl.isActive()) {
    isOpened = !isOpened;
    toggle();
  }
}

function toggle() {
  tl.progress(0).clear();

  for (let i = 0; i < numPoints; i++) {
    pointsDelay[i] = Math.random() * delayPointsMax;
  }

  for (let i = 0; i < numPaths; i++) {
    let points = allPoints[i];
    let pathDelay = delayPerPath * (isOpened ? i : (numPaths - i - 1));

    for (let j = 0; j < numPoints; j++) {
      let delay = pointsDelay[j];
      tl.to(points, { [j]: 0 }, delay + pathDelay);
    }
  }
}

function render() {
  for (let i = 0; i < numPaths; i++) {
    let path = paths[i];
    let points = allPoints[i];

    let d = "";
    d += isOpened ? `M 0 0 V ${points[0]} C` : `M 0 ${points[0]} C`;

    for (let j = 0; j < numPoints - 1; j++) {
      let p = (j + 1) / (numPoints - 1) * 100;
      let cp = p - (1 / (numPoints - 1) * 100) / 2;
      d += ` ${cp} ${points[j]} ${cp} ${points[j+1]} ${p} ${points[j+1]}`;
    }

    d += isOpened ? ` V 100 H 0` : ` V 0 H 0`;
    path.setAttribute("d", d);
  }
}
```

### Key patterns

- **Procedural bezier path generation** — builds SVG `d` attribute from a points array, creating smooth curves via cubic bezier control points
- **`onUpdate: render`** — GSAP timeline drives the render loop; no `requestAnimationFrame` needed
- **Randomized per-point delay** — each of the 10 control points gets a random delay (0–0.3s), creating an organic wave effect
- **Staggered path layers** — `delayPerPath` offsets each gradient layer, producing a layered curtain reveal
- **Toggle pattern** — `tl.progress(0).clear()` resets timeline before rebuilding, enabling bidirectional animation

---

## Smooth Morph — MorphSVG with Smooth Interpolation

Clean SVG morphing between complex shapes using MorphSVG's `smooth` option for better anchor point distribution. Includes interactive controls for tuning morph quality.

### HTML — SVG with source and destination paths

```html
<svg id="svg-stage" viewBox="0 0 100 100" fill="none" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <linearGradient id="grad" x1="0" y1="0" x2="99" y2="99" gradientUnits="userSpaceOnUse">
      <stop offset="0.2" stop-color="rgb(255, 135, 9)" />
      <stop offset="0.7" stop-color="rgb(247, 189, 248)" />
    </linearGradient>
    <path class="shape" id="destinationPath"
      d="M92.5088 9C92.7799 9.00016 93.0022 9.2209 93 9.49023C92.7532 30.5644 76.5914 47.8354 55.8965 50C76.5911 52.1646 92.7509 69.4356 93 90.5098C93.0022 90.7791 92.7799 90.9998 92.5088 91H10.4912C10.2201 90.9998 9.99781 90.7791 10 90.5098C10.2468 69.4359 26.4081 52.165 47.1025 50C26.4084 47.835 10.2491 30.564 10 9.49023C9.99781 9.2209 10.2201 9.00016 10.4912 9H92.5088Z" />
  </defs>

  <path class="shape" id="gsapPath" fill="url(#grad)"
    d="M90.5126 8.18339C90.7804 8.18098 91 8.40055 91 8.66837V49.1821C90.9011 71.771 72.5376 90.0458 49.9131 89.9999C27.2405 89.9541 9 71.2836 9 48.6079V8.485C9 8.21717 9.21956 7.99761 9.48737 8.00002C31.9067 8.2606 50 26.516 50 48.9988C50.0989 26.6005 68.1536 8.44398 90.5126 8.18339Z" />
</svg>
```

### CSS

```css
.content {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

svg {
  width: 50vmin;
  height: 50vmin;
  overflow: visible;
}
```

### JS — MorphSVG with smooth option and MotionPathHelper

```js
let destinationClone = document.querySelector("#destinationPath").cloneNode();
let pathEditorGSAP, anim;

function setup() {
  if (anim) {
    pathEditorGSAP && pathEditorGSAP.kill();
    anim.revert();
    document.querySelector("#destinationPath").replaceWith(destinationClone.cloneNode());
  }

  let smoothEnabled = true;
  let smoothPoints = 80;
  let redrawEnabled = true;
  let showAnchors = false;
  let zoomEnabled = false;

  gsap.set("#gsapPath", {
    transformOrigin: "left center",
    scale: zoomEnabled ? 4 : 1
  });

  pathEditorGSAP = showAnchors &&
    MotionPathHelper.editPath("#gsapPath", {
      handleSize: zoomEnabled ? 1 : 4
    });

  anim = gsap.to("#gsapPath", {
    duration: 2,
    ease: "power1.inOut",
    repeatDelay: 0.1,
    repeat: 50,
    yoyo: true,
    onUpdate: showAnchors ? () => pathEditorGSAP.update(true) : null,
    delay: 1,
    morphSVG: {
      shape: "#destinationPath",
      smooth: smoothEnabled && {
        points: smoothPoints,  // Number of anchor points for interpolation
        redraw: redrawEnabled  // Reconfigure anchors for even distribution
      }
    }
  });
}

setup();
```

### Key patterns

- **`morphSVG.smooth`** — adds extra anchor points along the path for smoother interpolation between complex shapes
- **`smooth.points`** — controls number of interpolation points (higher = smoother but costlier); default 80 works well for most shapes
- **`smooth.redraw`** — reconfigures anchor distribution for more even spacing, which can improve morph quality
- **`anim.revert()`** — cleanly kills and reverts the animation before reconfiguring; essential for dynamic parameter changes
- **`yoyo: true`** — morphs back and forth between source and destination shapes continuously
- **`MotionPathHelper.editPath()`** — debug tool for visualizing anchor points during development (remove in production)
