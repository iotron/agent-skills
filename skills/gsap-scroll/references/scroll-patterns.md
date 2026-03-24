# ScrollTrigger Advanced Patterns

## Table of Contents
- [Stacking Cards (Sticky + Scrub)](#stacking-cards-sticky--scrub)
- [Scrubbed Timeline (Progress Mapping)](#scrubbed-timeline-progress-mapping)
- [Elastic Type (Pin + Scrub Assembly)](#elastic-type-pin--scrub-assembly)
- [Service Section Switching](#service-section-switching)

---

## Stacking Cards (Sticky + Scrub)

CSS sticky + GSAP scrubbed two-phase timeline: fade/scale in, then stack/scale out.

```css
.stacking-card {
  position: relative;
  transform-origin: center top;
}
@media (min-width: 1024px) {
  .stacking-card {
    position: sticky;
    top: calc(5rem + var(--card-index, 0) * 0.5rem);
  }
}
```

```js
cards.forEach((card, index) => {
  gsap.set(card, { y: 100, autoAlpha: 0, scale: 0.9 })

  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: card, start: 'top 85%', end: 'top 25%', scrub: 0.5,
      invalidateOnRefresh: true,
      onEnter:     () => gsap.set(card, { willChange: 'transform, opacity' }),
      onLeave:     () => gsap.set(card, { willChange: 'auto' }),
      onEnterBack: () => gsap.set(card, { willChange: 'transform, opacity' }),
      onLeaveBack: () => gsap.set(card, { willChange: 'auto' }),
    },
  })

  // Phase 1: reveal
  tl.to(card, { y: 0, autoAlpha: 1, scale: 1, duration: 0.5, ease: 'power2.out', force3D: true })
  // Phase 2: stack with depth
  .to(card, { y: index * -15, scale: 1 - index * 0.015, duration: 0.5, force3D: true })
})
```

- `will-change` set in onEnter, released in onLeave — avoids permanent GPU cost
- `--card-index` CSS variable offsets sticky `top` so cards stack with slight gaps
- Two-phase timeline: first tween reveals, second creates stacked depth

---

## Scrubbed Timeline (Progress Mapping)

`scrub: 1` maps scroll to timeline progress. Progress bar, bouncing dots, alternating cards.

```js
// Alternating initial offsets
timelineCardRefs.value.forEach((card, i) => {
  if (card) gsap.set(card, { x: i % 2 === 0 ? -50 : 50, autoAlpha: 0, force3D: true })
})

const tl = gsap.timeline({
  scrollTrigger: { trigger: timelineRef.value, start: 'top 70%', end: 'bottom 30%', scrub: 1, invalidateOnRefresh: true },
})

// Progress bar: scaleY 0->1, linear
tl.to(timelineProgressRef.value, { scaleY: 1, ease: 'none', duration: 1 })

// Dots + cards in sequence
milestones.forEach((_, i) => {
  const offset = i * 0.14
  if (timelineDotRefs.value[i])
    tl.to(timelineDotRefs.value[i], { scale: 1, autoAlpha: 1, duration: 0.08, ease: 'back.out(3)', force3D: true }, offset)
  if (timelineCardRefs.value[i])
    tl.to(timelineCardRefs.value[i], { x: 0, autoAlpha: 1, duration: 0.14, ease: 'power2.out', force3D: true }, offset + 0.04)
})
```

- `scrub: 1` — 1s smooth catch-up (good for complex timelines)
- Position offsets (`i * 0.14`) space milestones across the timeline

---

## Elastic Type (Pin + Scrub Assembly)

Pin section, scrub three phases: assemble from random, hold, scatter outward.

```js
gsap.set(chars, {
  y: () => gsap.utils.random(-200, 200), x: () => gsap.utils.random(-100, 100),
  rotation: () => gsap.utils.random(-90, 90), scale: () => gsap.utils.random(0.4, 2.5),
  autoAlpha: 0,
})

const tl = gsap.timeline({
  scrollTrigger: { trigger: elasticPinRef.value, start: 'top top', end: '+=150%', pin: true, scrub: 1, invalidateOnRefresh: true },
})

// Phase 1: assemble
tl.to(chars, {
  y: 0, x: 0, rotation: 0, scale: 1, autoAlpha: 1,
  stagger: { each: 0.03, from: 'random' }, duration: 1, ease: 'elastic.out(1.2, 1)', force3D: true,
})
// Phase 2: hold
tl.to({}, { duration: 0.3 })
// Phase 3: scatter from edges
tl.to(chars, {
  y: () => gsap.utils.random(-300, 300), x: () => gsap.utils.random(-200, 200),
  rotation: () => gsap.utils.random(-120, 120), scale: 0, autoAlpha: 0,
  stagger: { each: 0.02, from: 'edges' }, duration: 0.8, ease: 'power2.in',
})
```

- `pin: true` — element stays fixed while scroll drives timeline
- `end: '+=150%'` — scroll distance = 150% of viewport height
- Empty tween `tl.to({}, { duration: 0.3 })` creates a hold phase

---

## Service Section Switching

`ScrollTrigger.create()` per section with callbacks — no tween attached, purely event-driven switching.

```js
const activeIdx = ref(-1)
let revealed = false

// Gate: reveal sidebar + first service when section enters
ScrollTrigger.create({
  trigger: servicesCenter.value, start: 'top 70%', fastScrollEnd: true,
  onEnter: () => { if (!revealed) { revealed = true; switchTo(0) } },
  onLeaveBack: () => { revealed = false; deactivate(activeIdx.value); activeIdx.value = -1 },
})

// Per-category triggers
services.forEach((_, index) => {
  ScrollTrigger.create({
    trigger: categoryRefs.value[index], start: 'top 55%', end: 'bottom 45%', fastScrollEnd: true,
    onEnter:     () => { if (revealed && index > 0) switchTo(index) },
    onLeave:     () => { if (revealed && index < services.length - 1) switchTo(index + 1) },
    onEnterBack: () => { if (revealed) switchTo(index) },
    onLeaveBack: () => { if (revealed && index > 0) switchTo(index - 1) },
  })
})

function switchTo(index) {
  if (activeIdx.value === index) return
  if (activeIdx.value >= 0) deactivate(activeIdx.value)
  activeIdx.value = index
  activate(index)
}

function deactivate(i) {
  gsap.to(bodyRefs.value[i], { autoAlpha: 0, y: -10, duration: 0.3, overwrite: true })
  gsap.to(illustrationRefs.value[i], { autoAlpha: 0, scale: 0.95, duration: 0.3, overwrite: true })
}

function activate(i) {
  gsap.fromTo(bodyRefs.value[i], { autoAlpha: 0, y: 20 },
    { autoAlpha: 1, y: 0, duration: 0.5, ease: 'power2.out', overwrite: true })
  gsap.fromTo(illustrationRefs.value[i], { autoAlpha: 0, scale: 0.9 },
    { autoAlpha: 1, scale: 1, duration: 0.5, ease: 'back.out(1.4)', overwrite: true, force3D: true })
  pointRefs.value[i]?.forEach((li, j) => {
    if (li) gsap.fromTo(li, { autoAlpha: 0, x: -30 },
      { autoAlpha: 1, x: 0, duration: 0.3, delay: 0.15 + j * 0.08, ease: 'power2.out', overwrite: true })
  })
}
```

- `ScrollTrigger.create()` (no tween) — purely callback-driven
- `fastScrollEnd: true` — snaps to correct state on fast scroll
- `overwrite: true` kills in-flight tweens; `fromTo` ensures correct start state
- `revealed` gate prevents activation before section enters viewport

---

## Velocity Skew on Scroll

Applies dynamic skewY transforms to elements based on scroll velocity. Uses `gsap.quickSetter` for per-frame performance.

**Source**: [CodePen eYpGLYL](https://codepen.io/GreenSock/pen/eYpGLYL)

```html
<img src="image-1.jpg" alt="" class="skewElem">
<img src="image-2.jpg" alt="" class="skewElem">
<img src="image-3.jpg" alt="" class="skewElem">
<img src="image-4.jpg" alt="" class="skewElem">
<img src="image-5.jpg" alt="" class="skewElem">
```

```css
body {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  padding-top: 20vh;
}
body > img {
  margin-bottom: 20vh;
}
img {
  object-fit: cover;
  width: 300px;
  max-width: 80vw;
  aspect-ratio: 1/1;
}
```

```js
let proxy = { skew: 0 },
    skewSetter = gsap.quickSetter(".skewElem", "skewY", "deg"), // fast — bypasses .set() overhead
    clamp = gsap.utils.clamp(-20, 20); // cap skew at ±20 degrees

ScrollTrigger.create({
  onUpdate: (self) => {
    let skew = clamp(self.getVelocity() / -300);
    // Only update if new skew is MORE severe — avoids interrupting smooth tween-back-to-zero
    if (Math.abs(skew) > Math.abs(proxy.skew)) {
      proxy.skew = skew;
      gsap.to(proxy, {
        skew: 0, duration: 0.8, ease: "power3",
        overwrite: true,
        onUpdate: () => skewSetter(proxy.skew)
      });
    }
  }
});

// Right edge anchored to scroll bar side; force3D for GPU compositing
gsap.set(".skewElem", { transformOrigin: "right center", force3D: true });
```

**Key patterns**:
- `gsap.quickSetter()` — fastest way to set a single property per frame (no tween overhead)
- `ScrollTrigger.getVelocity()` — returns scroll speed in px/s, sign indicates direction
- `gsap.utils.clamp()` — prevents extreme values on fast flicks
- Proxy object pattern — tween a plain object, apply via quickSetter in `onUpdate`
- `overwrite: true` — kills previous tween when velocity changes direction

---

## Infinite Looped Panels

Layered pinning with infinite scroll looping — panels stack on top of each other and loop seamlessly.

**Source**: [CodePen VwbywPd](https://codepen.io/GreenSock/pen/VwbywPd)

```html
<div class="description panel">
  <h1>Layered pinning with infinite looping</h1>
  <p>Use pinning to layer panels on top of each other as you scroll.</p>
</div>

<section class="panel green">
  <h2 class="panel__number">1</h2>
</section>
<section class="panel">
  <h2 class="panel__number">2</h2>
</section>
<section class="panel purple">
  <h2 class="panel__number">3</h2>
</section>
<section class="panel">
  <h2 class="panel__number">4</h2>
</section>
<section class="panel blue">
  <h2 class="panel__number">5</h2>
</section>
```

```js
gsap.registerPlugin(ScrollTrigger);

let panels = gsap.utils.toArray(".panel"),
    copy = panels[0].cloneNode(true);
panels[0].parentNode.appendChild(copy); // Clone first panel to end for seamless looping

// Pin each panel with no spacing — they stack visually
panels.forEach((panel, i) => {
  ScrollTrigger.create({
    trigger: panel,
    start: "top top",
    pin: true,
    pinSpacing: false  // Panels overlap instead of pushing content down
  });
});

let maxScroll;
let pageScrollTrigger = ScrollTrigger.create({
  // Custom snap function handles edge cases at scroll boundaries
  snap(value) {
    let snappedValue = gsap.utils.snap(1 / panels.length, value);
    if (snappedValue <= 0) {
      // Don't snap to exactly 0 — keep 1px+ from top to prevent wrap
      return 1.05 / maxScroll;
    } else if (snappedValue >= 1) {
      // Don't snap to exactly end — keep 1px+ from bottom to prevent wrap
      return maxScroll / (maxScroll + 1.05);
    }
    return snappedValue;
  }
});

function onResize() {
  maxScroll = ScrollTrigger.maxScroll(window) - 1;
}
onResize();
window.addEventListener("resize", onResize);

// Non-passive listener to preventDefault at scroll boundaries for looping
window.addEventListener("scroll", e => {
  let scroll = pageScrollTrigger.scroll();
  if (scroll > maxScroll) {
    pageScrollTrigger.scroll(1);
    e.preventDefault();
  } else if (scroll < 1) {
    pageScrollTrigger.scroll(maxScroll - 1);
    e.preventDefault();
  }
}, { passive: false });
```

**Key patterns**:
- `pinSpacing: false` — panels overlap/stack instead of creating scroll gaps
- `cloneNode(true)` on first panel appended to end — creates seamless loop illusion
- Custom `snap()` function — prevents snapping to exact 0 or 1 which would break the loop
- `pageScrollTrigger.scroll()` — programmatically jump scroll position at boundaries
- `{ passive: false }` — required for `preventDefault()` on scroll events
- `ScrollTrigger.maxScroll()` — utility to get max scrollable distance

---

## Directionally Aware Header (Show/Hide on Scroll Direction)

Header slides in when scrolling up, hides when scrolling down. Extremely common UI pattern.

**Source**: [CodePen qBawMGb](https://codepen.io/GreenSock/pen/qBawMGb)

```html
<div class="main-tool-bar">Header</div>
<div class="scrollable-area"></div>
```

```css
.main-tool-bar {
  height: 80px;
  background: linear-gradient(144deg, #00bae2 4.56%, #fec5fb 72.98%);
  color: #0e100f;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  width: 100%;
  left: 0;
  top: 0;
}
.scrollable-area {
  height: 200vh;
}
```

```js
// Create the hide animation paused, then set progress to 1 (visible state)
const showAnim = gsap.from('.main-tool-bar', {
  yPercent: -100,
  paused: true,
  duration: 0.2
}).progress(1);

ScrollTrigger.create({
  start: "top top",
  end: "max",
  onUpdate: (self) => {
    // direction: -1 = scrolling up, 1 = scrolling down
    self.direction === -1 ? showAnim.play() : showAnim.reverse();
  }
});
```

**Key patterns**:
- `gsap.from().progress(1)` — creates animation in hidden state but immediately shows it (progress=1 means "at the end" = visible)
- `paused: true` — animation only plays/reverses when explicitly called
- `self.direction` — `-1` for scrolling up, `1` for scrolling down
- `play()` / `reverse()` — smooth toggle without creating new tweens each frame
- `end: "max"` — ScrollTrigger stays active for entire page scroll
- No `trigger` element needed — uses the whole page as the scroll context

---

## Pinned Panels with Overscroll

Slide-based pinning where panels pin and unpin cleanly, with overscroll handling for panels taller than the viewport. Fake-scrolls inner content before scaling/fading out.

**Source**: [Gist 4ba7a53](https://gist.github.com/4ba7a53cace6e1a679a6c9a4bf41018d)

```html
<div class="slides-wrapper">
  <section class="section section-1">
    <div class="section-content">
      <div class="section-inner">
        <h1>Section 1</h1>
        <img class="image" src="image.jpg" alt="" />
      </div>
    </div>
  </section>
  <section class="section section-2">
    <div class="section-content">
      <div class="section-inner">
        <h1>Section 2</h1>
        <p>Long scrollable content inside a pinned panel...</p>
      </div>
    </div>
  </section>
  <!-- more sections... -->
</div>
```

```css
.section {
  width: 100%;
  height: calc(100vh - 64px);
  display: flex;
  justify-content: center;
  position: relative;
  box-sizing: border-box;
  overflow: hidden;
  border-radius: 10px;
}
.section-inner {
  height: 100%;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
}
/* For overscroll panels — height: auto + bottom padding */
.section-2 .section-inner {
  height: auto;
  padding-bottom: 20vh;
}
```

```js
gsap.registerPlugin(ScrollTrigger);

var panels = gsap.utils.toArray(".section");
panels.pop(); // last panel doesn't need exit animation

panels.forEach((panel, i) => {
  let innerpanel = panel.querySelector(".section-inner");
  let panelHeight = innerpanel.offsetHeight;
  let windowHeight = window.innerHeight;
  let difference = panelHeight - windowHeight;

  // Ratio of animation dedicated to fake-scrolling (for overscroll panels)
  let fakeScrollRatio = difference > 0 ? (difference / (difference + windowHeight)) : 0;

  // Add margin to compensate for fake-scroll duration
  if (fakeScrollRatio) {
    panel.style.marginBottom = panelHeight * fakeScrollRatio + "px";
  }

  let tl = gsap.timeline({
    scrollTrigger: {
      trigger: panel,
      start: "bottom bottom",
      end: () => fakeScrollRatio ? `+=${innerpanel.offsetHeight}` : "bottom top",
      pinSpacing: false,
      pin: true,
      scrub: true
    }
  });

  // Phase 1: fake scroll inner content (only for overscroll panels)
  if (fakeScrollRatio) {
    tl.to(innerpanel, {
      yPercent: -100, y: window.innerHeight,
      duration: 1 / (1 - fakeScrollRatio) - 1, ease: "none"
    });
  }
  // Phase 2: scale down + fade out
  tl.fromTo(panel, { scale: 1, opacity: 1 }, { scale: 0.7, opacity: 0.5, duration: 0.9 })
    .to(panel, { opacity: 0, duration: 0.1 });
});
```

**Key patterns**:
- `fakeScrollRatio` — calculates how much of the animation is inner-content scrolling vs exit
- `pinSpacing: false` — panels overlap, creating a slide-over-slide effect
- `marginBottom` compensation — ensures next panel arrives at the right time
- Two-phase exit: scale down to 0.7 (90% of exit), then fade to 0 (last 10%)
- `panels.pop()` — last panel doesn't need an exit animation

---

## Image Mask On Scroll

Before/after image comparison revealed on scroll using counter-translating containers. The outer container slides in while the inner image slides the opposite direction, creating a wipe/mask effect.

**Source**: [Gist 6fefd07](https://gist.github.com/6fefd07444b22399f12ad4237eabddbc)

```html
<section class="comparisonSection">
  <div class="comparisonImage beforeImage">
    <img src="before.jpg" alt="before">
  </div>
  <div class="comparisonImage afterImage">
    <img src="after.jpg" alt="after">
  </div>
</section>
```

```css
.comparisonSection {
  position: relative;
  padding-bottom: 56.25%; /* 16:9 aspect ratio — responsive */
}
.comparisonImage {
  width: 100%;
  height: 100%;
}
.afterImage {
  position: absolute;
  overflow: hidden;
  top: 0;
  transform: translate(100%, 0px);
}
.afterImage img {
  transform: translate(-100%, 0px);
}
.comparisonImage img {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
}
```

```js
gsap.utils.toArray(".comparisonSection").forEach(section => {
  let tl = gsap.timeline({
    scrollTrigger: {
      trigger: section,
      start: "center center",
      // Match scroll distance to section width — keeps reveal speed constant
      end: () => "+=" + section.offsetWidth,
      scrub: true,
      pin: true,
      anticipatePin: 1
    },
    defaults: { ease: "none" }
  });
  // Animate container one way...
  tl.fromTo(section.querySelector(".afterImage"),
    { xPercent: 100, x: 0 }, { xPercent: 0 })
    // ...and image the opposite way (simultaneously)
    .fromTo(section.querySelector(".afterImage img"),
      { xPercent: -100, x: 0 }, { xPercent: 0 }, 0);
});
```

**Key patterns**:
- Counter-translation trick — container moves right-to-left, image inside moves left-to-right, creating a mask/wipe
- `end: () => "+=" + section.offsetWidth` — scroll distance matches width for constant-speed reveal
- `anticipatePin: 1` — prevents visual jump when pin starts
- `padding-bottom: 56.25%` — maintains 16:9 responsive aspect ratio without fixed height
- Works for any number of comparison sections (uses `toArray` + `forEach`)

---

## Lateral Pin Indicator

Pinned section with a side navigation indicator that highlights the current item and crossfades slide content as you scroll through list items.

**Source**: [Gist 699eed9](https://gist.github.com/699eed93235106234be0e65f17746c30)

```html
<section class="section pin-section">
  <div class="content">
    <ul class="list">
      <li>Animate</li>
      <li>Anything</li>
      <li>With</li>
      <li>GSAP</li>
    </ul>
    <div class="fill"></div>
    <div class="right">
      <div class="slide center"><img src="slide-1.png" alt="" /></div>
      <div class="slide center"><img src="slide-2.png" alt="" /></div>
      <div class="slide center"><img src="slide-3.png" alt="" /></div>
      <div class="slide center"><img src="slide-4.png" alt="" /></div>
    </div>
  </div>
</section>
```

```css
.content {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  padding: 0 10px;
  position: relative;
}
.content ul {
  font-size: 30px;
  margin: 0;
  padding: 0;
  padding-right: 10px;
  list-style: none;
  flex-grow: 0;
}
.content .fill {
  position: absolute;
  top: 0;
  left: 0;
  width: 2px;
  height: 100%;
  background-color: var(--color-shockingly-green);
}
.content .right {
  flex-grow: 1;
  position: relative;
}
.right .slide {
  position: absolute;
  width: 50%;
  top: 50%;
  transform: translateY(-50%);
  right: 1rem;
  opacity: 0;
  visibility: hidden;
}
```

```js
gsap.registerPlugin(ScrollTrigger);

const list = document.querySelector(".list");
const fill = document.querySelector(".fill");
const listItems = gsap.utils.toArray("li", list);
const slides = gsap.utils.toArray(".slide");

const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".pin-section",
    start: "top top",
    end: "+=" + listItems.length * 50 + "%",
    pin: true,
    scrub: true
  }
});

// Scale fill bar to 1/N initially (first item visible)
fill && gsap.set(fill, { scaleY: 1 / listItems.length, transformOrigin: "top left" });

listItems.forEach((item, i) => {
  const previousItem = listItems[i - 1];
  if (previousItem) {
    tl.set(item, { color: "#0ae448" }, 0.5 * i)                    // highlight current
      .to(slides[i], { autoAlpha: 1, duration: 0.2 }, "<")         // show current slide
      .set(previousItem, { color: "#fffce1" }, "<")                 // unhighlight previous
      .to(slides[i - 1], { autoAlpha: 0, duration: 0.2 }, "<");    // hide previous slide
  } else {
    gsap.set(item, { color: "#0ae448" });
    gsap.set(slides[i], { autoAlpha: 1 });
  }
});

// Fill bar grows across full timeline duration
tl.to(fill, {
  scaleY: 1, transformOrigin: "top left", ease: "none", duration: tl.duration()
}, 0)
  .to({}, {}); // small pause at end before unpin
```

**Key patterns**:
- `end: "+=" + listItems.length * 50 + "%"` — scroll distance scales with number of items
- `autoAlpha` crossfade — previous slide fades out as next fades in at the same position (`"<"`)
- `scaleY` fill bar — grows from `1/N` to `1` across the full timeline, acting as progress indicator
- `.set()` for color changes — instant state switch (no tween duration)
- `.to({}, {})` at end — adds a brief hold before the section unpins

---

## Horizontal Scrolling Gallery

Horizontal scroll gallery using ScrollTrigger pin + scrub. Pins a section and translates a strip of images horizontally as the user scrolls vertically. Includes ScrollSmoother integration.

**Source**: [Gist 9ac9e14](https://gist.github.com/9ac9e14a23b0ff5e2be5367b00777af4)

```html
<div id="smooth-wrapper">
  <div id="smooth-content">
    <section id="portfolio">
      <div class="container-fluid">
        <div class="horiz-gallery-wrapper">
          <div class="horiz-gallery-strip">
            <div class="project-wrap"><img src="image-1.jpg" alt="" /></div>
            <div class="project-wrap"><img src="image-2.jpg" alt="" /></div>
            <div class="project-wrap"><img src="image-3.jpg" alt="" /></div>
            <!-- more items... -->
          </div>
        </div>
      </div>
    </section>
  </div>
</div>
```

```css
.horiz-gallery-strip,
.horiz-gallery-wrapper {
  display: flex;
  flex-wrap: nowrap;
  will-change: transform;
  position: relative;
}
.project-wrap {
  width: 33vw;
  padding: 2rem;
  box-sizing: content-box;
}
.project-wrap img {
  width: 100%;
  aspect-ratio: 1/1;
  object-fit: cover;
}
```

```js
gsap.registerPlugin(ScrollTrigger, ScrollSmoother);

const smoother = ScrollSmoother.create({
  wrapper: "#smooth-wrapper",
  content: "#smooth-content",
  smooth: 2,
  normalizeScroll: true,
  ignoreMobileResize: true,
  preventDefault: true
});

const horizontalSections = gsap.utils.toArray(".horiz-gallery-wrapper");

horizontalSections.forEach(function (sec) {
  const pinWrap = sec.querySelector(".horiz-gallery-strip");
  let pinWrapWidth, horizontalScrollLength;

  function refresh() {
    pinWrapWidth = pinWrap.scrollWidth;
    horizontalScrollLength = pinWrapWidth - window.innerWidth;
  }
  refresh();

  gsap.to(pinWrap, {
    scrollTrigger: {
      scrub: true,
      trigger: sec,
      pin: sec,
      start: "center center",
      end: () => `+=${pinWrapWidth}`,
      invalidateOnRefresh: true
    },
    x: () => -horizontalScrollLength,
    ease: "none"
  });

  ScrollTrigger.addEventListener("refreshInit", refresh);
});
```

**Key patterns**:
- `x: () => -horizontalScrollLength` — functional value recalculates on refresh for responsiveness
- `end: () => "+=" + pinWrapWidth` — scroll distance matches total strip width for natural speed
- `ScrollTrigger.addEventListener("refreshInit", refresh)` — recalculates dimensions before ST recalculates positions
- `invalidateOnRefresh: true` — forces tween to re-evaluate functional values on resize
- `start: "center center"` — pins when section is centered in viewport
- ScrollSmoother optional — pattern works without it (remove wrapper/content and ScrollSmoother.create)
- `will-change: transform` on strip — pre-promotes to GPU layer for smooth horizontal translation
