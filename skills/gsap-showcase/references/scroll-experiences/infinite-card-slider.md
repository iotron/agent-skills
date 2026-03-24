# Infinite Card Slider

**Source:** [CodePen](https://codepen.io/GreenSock/pen/gOBqEOa) by GSAP
**Plugins:** ScrollTrigger, Draggable
**Category:** scroll-experiences

## What Makes It Premium

This seamless infinite-scrolling card carousel combines scroll and drag interaction with snap-to-card behavior. The "seamless loop" technique builds a raw animation sequence with extra overlapping items at both ends, then scrubs a window through it — creating the illusion of infinite content from a finite set. The scroll wrapping detects when the user hits the boundaries and silently repositions, making the loop completely invisible.

## Orchestration

A `buildSeamlessLoop()` function creates a master timeline where each card animates through: scale up from 0 + slide from right, then scale back down + slide to left. Extra iterations at both ends create overlap for seamless wrapping. ScrollTrigger drives a `scrub` tween that converts scroll progress to a wrapped time on the seamless loop. When scroll hits the very top or bottom, `wrap()` increments/decrements the iteration counter and repositions the scroll. Draggable provides touch/mouse drag as an alternative input, feeding the same scrub mechanism. `scrollEnd` event snaps to the nearest card.

## Code

### HTML
```html
<div class="gallery">
  <ul class="cards">
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-01.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-02.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-03.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-04.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-05.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-06.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-07.png)"></li>
    <!-- Duplicate set for seamless visual continuity -->
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-01.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-02.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-03.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-04.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-05.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-06.png)"></li>
    <li style="background-image: url(https://assets.codepen.io/16327/portrait-number-07.png)"></li>
  </ul>
  <div class="actions">
    <button class="prev">Prev</button>
    <button class="next">Next</button>
  </div>
</div>
<div class="drag-proxy"></div>
```

### CSS
```css
* {
  box-sizing: border-box;
}

body {
  background: #111;
  min-height: 100vh;
  padding: 0;
  margin: 0;
}

.gallery {
  position: absolute;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.cards {
  position: absolute;
  width: 14rem;
  height: 18rem;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.cards li {
  list-style: none;
  padding: 0;
  margin: 0;
  width: 14rem;
  aspect-ratio: 9/16;
  text-align: center;
  line-height: 18rem;
  font-size: 2rem;
  position: absolute;
  background-size: contain;
  background-repeat: no-repeat;
  top: 0;
  left: 0;
  border-radius: 0.8rem;
}

.actions {
  position: absolute;
  bottom: 25px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
}

.drag-proxy {
  visibility: hidden;
  position: absolute;
}
```

### JavaScript
```js
gsap.registerPlugin(ScrollTrigger, Draggable);

let iteration = 0;

// All cards start hidden and off-screen
gsap.set('.cards li', { xPercent: 400, opacity: 0, scale: 0 });

const spacing = 0.1,  // stagger spacing between cards
  snapTime = gsap.utils.snap(spacing),
  cards = gsap.utils.toArray('.cards li'),

  // Animation for each card: scale in from right, scale out to left
  animateFunc = element => {
    const tl = gsap.timeline();
    tl.fromTo(element,
      { scale: 0, opacity: 0 },
      { scale: 1, opacity: 1, zIndex: 100, duration: 0.5,
        yoyo: true, repeat: 1, ease: "power1.in", immediateRender: false })
      .fromTo(element,
        { xPercent: 400 },
        { xPercent: -400, duration: 1, ease: "none", immediateRender: false }, 0);
    return tl;
  },

  seamlessLoop = buildSeamlessLoop(cards, spacing, animateFunc),
  playhead = { offset: 0 },
  wrapTime = gsap.utils.wrap(0, seamlessLoop.duration()),

  // Reusable scrub tween — invalidated and restarted on each update
  scrub = gsap.to(playhead, {
    offset: 0,
    onUpdate() {
      seamlessLoop.time(wrapTime(playhead.offset));
    },
    duration: 0.5,
    ease: "power3",
    paused: true
  }),

  trigger = ScrollTrigger.create({
    start: 0,
    onUpdate(self) {
      let scroll = self.scroll();
      if (scroll > self.end - 1) {
        wrap(1, 2);  // scrolled to very end — wrap forward
      } else if (scroll < 1 && self.direction < 0) {
        wrap(-1, self.end - 2);  // scrolled to very start — wrap backward
      } else {
        scrub.vars.offset = (iteration + self.progress) * seamlessLoop.duration();
        scrub.invalidate().restart();
      }
    },
    end: "+=3000",
    pin: ".gallery"
  }),

  progressToScroll = progress =>
    gsap.utils.clamp(1, trigger.end - 1, gsap.utils.wrap(0, 1, progress) * trigger.end),

  wrap = (iterationDelta, scrollTo) => {
    iteration += iterationDelta;
    trigger.scroll(scrollTo);
    trigger.update();
  };

// Snap to nearest card when scrolling stops
ScrollTrigger.addEventListener("scrollEnd", () => scrollToOffset(scrub.vars.offset));

function scrollToOffset(offset) {
  let snappedTime = snapTime(offset),
      progress = (snappedTime - seamlessLoop.duration() * iteration) / seamlessLoop.duration(),
      scroll = progressToScroll(progress);
  if (progress >= 1 || progress < 0) {
    return wrap(Math.floor(progress), scroll);
  }
  trigger.scroll(scroll);
}

// Prev/Next buttons
document.querySelector(".next").addEventListener("click", () => scrollToOffset(scrub.vars.offset + spacing));
document.querySelector(".prev").addEventListener("click", () => scrollToOffset(scrub.vars.offset - spacing));

// Drag support — hidden proxy element receives drag events
Draggable.create(".drag-proxy", {
  type: "x",
  trigger: ".cards",
  onPress() {
    this.startOffset = scrub.vars.offset;
  },
  onDrag() {
    scrub.vars.offset = this.startOffset + (this.startX - this.x) * 0.001;
    scrub.invalidate().restart();
  },
  onDragEnd() {
    scrollToOffset(scrub.vars.offset);
  }
});

/**
 * Builds a seamless looping timeline from an array of items.
 * Creates extra animations at both ends so scrubbing wraps cleanly.
 */
function buildSeamlessLoop(items, spacing, animateFunc) {
  let overlap = Math.ceil(1 / spacing),
      startTime = items.length * spacing + 0.5,
      loopTime = (items.length + overlap) * spacing + 1,
      rawSequence = gsap.timeline({ paused: true }),
      seamlessLoop = gsap.timeline({
        paused: true,
        repeat: -1,
        onRepeat() {
          // Edge case fix: prevent stuck playhead at duration boundary
          this._time === this._dur && (this._tTime += this._dur - 0.01);
        }
      }),
      l = items.length + overlap * 2,
      time, i, index;

  for (i = 0; i < l; i++) {
    index = i % items.length;
    time = i * spacing;
    rawSequence.add(animateFunc(items[index]), time);
    i <= items.length && seamlessLoop.add("label" + i, time);
  }

  rawSequence.time(startTime);
  seamlessLoop.to(rawSequence, {
    time: loopTime,
    duration: loopTime - startTime,
    ease: "none"
  }).fromTo(rawSequence,
    { time: overlap * spacing + 1 },
    { time: startTime,
      duration: startTime - (overlap * spacing + 1),
      immediateRender: false,
      ease: "none"
    });

  return seamlessLoop;
}
```

## Adaptation Notes

- **Card content**: Replace `<li>` background images with any card component — the animation function only cares about scale, opacity, and xPercent.
- **Vertical variant**: Change `xPercent` to `yPercent` in `animateFunc` for a vertical carousel.
- **Spacing control**: Adjust `spacing` constant (0.1 = 10 cards visible in the loop window). Smaller values = more cards visible simultaneously.
- **Disable drag on desktop**: Conditionally create the Draggable instance based on touch detection if scroll-only is preferred on desktop.
- **Framework integration**: Store `trigger`, `scrub`, and `seamlessLoop` refs for cleanup. Call `.kill()` on all three in unmount lifecycle.
- **Accessibility**: Add `aria-roledescription="carousel"` to `.gallery` and `aria-label` to each card. Respect `prefers-reduced-motion` by disabling scroll-driven animation and showing a static grid instead.
