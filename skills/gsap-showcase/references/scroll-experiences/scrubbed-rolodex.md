# Scrubbed Vertical Rolodex

**Source:** [CodePen](https://codepen.io/GreenSock/pen/QwbrQjy) by GSAP
**Plugins:** ScrollTrigger
**Category:** scroll-experiences

## What Makes It Premium

The 3D card flip effect uses `rotationX` with a negative-Z `transformOrigin` to create convincing depth as cards rotate in and out. The scroll-scrubbed pin keeps the carousel fixed while the user scrolls, creating the illusion of a physical rolodex. The timeline is dynamically built from the actual element count, making it trivially extensible — add more slides and the animation automatically adjusts scroll distance.

## Orchestration

Each slide pair (current + next) gets a synchronized tween: the current slide rotates to +90 degrees (flipping "up and away") while the next slide rotates from -90 to 0 (flipping "in from below"). A configurable delay between transitions creates a brief pause at each card. The `onComplete` callback resets exited slides to -90 degrees for clean state. ScrollTrigger pins the wrapper and calculates total scroll distance from slide count: `(slides.length - 1) * 50%` of viewport.

## Code

### HTML
```html
<div class="spacer"></div>
<div class="wrapper center">
  <div class="slider">
    <div class="slide center gradient-green">Slide 1</div>
    <div class="slide center gradient-purple">Slide 2</div>
    <div class="slide center gradient-blue">Slide 3</div>
    <div class="slide center gradient-orange">Slide 4</div>
    <div class="slide center gradient-blue-dark">Slide 5</div>
    <div class="slide center gradient-red">Slide 6</div>
  </div>
</div>
<div class="spacer"></div>
```

### CSS
```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.wrapper {
  width: 100%;
  height: 100vh;
}

.slider {
  width: 300px;
  height: 300px;
  position: relative;
  perspective: 500px; /* Controls depth of 3D effect */
}

.slider:after {
  position: absolute;
  content: "";
  top: -1rem;
  bottom: -1rem;
  right: -1rem;
  left: -1rem;
  outline: 1.5px dashed var(--light);
  border-radius: 10px;
}

.slide {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  font-size: 1.5rem;
  backface-visibility: hidden; /* Prevents seeing reverse side during flip */
}

.spacer {
  height: 50vh;
}
```

### JavaScript
```js
console.clear();
gsap.registerPlugin(ScrollTrigger);

const slides = gsap.utils.toArray(".slide");
const delay = 0.5; // Pause duration between card flips

const tl = gsap.timeline({
  defaults: {
    ease: "power1.inOut",
    // Negative Z offset creates depth — card rotates around an axis behind its face
    transformOrigin: "center center -150px"
  },
  scrollTrigger: {
    trigger: ".wrapper",
    start: "top top",
    // Dynamic end: scroll distance scales with number of slides
    end: "+=" + (slides.length - 1) * 50 + "%",
    pin: true,
    scrub: true,
    markers: true // Remove for production
  }
});

// Initial state: first slide at 0deg, all others at -90deg (hidden below)
gsap.set(slides, {
  rotationX: (i) => (i ? -90 : 0),
  transformOrigin: "center center -150px"
});

// Build flip pairs: each current slide flips up (+90) while next flips in (0)
slides.forEach((slide, i) => {
  const nextSlide = slides[i + 1];
  if (!nextSlide) return;
  tl.to(
    slide,
    {
      rotationX: 90,
      // Reset to -90 after exit so it could re-enter if scrubbed back
      onComplete: () => gsap.set(slide, { rotationX: -90 })
    },
    "+=" + delay // Gap between flips
  ).to(
    nextSlide,
    {
      rotationX: 0
    },
    "<" // Sync with current slide's exit
  );
});

// Keep final slide visible with an empty tween
tl.to({}, { duration: delay });
```

## Adaptation Notes

- Add or remove `.slide` elements — the timeline auto-adapts since it iterates `gsap.utils.toArray()`.
- Adjust `transformOrigin` Z-offset (`-150px`) to control flip depth. Larger negative values = wider arc. Match with the `.slider` `perspective` value.
- The `delay` variable controls the scroll-pause between cards. Increase for a more deliberate feel.
- Replace `markers: true` with `false` for production. Consider adding `snap` to ScrollTrigger config to snap to each card position.
- For horizontal flips, change `rotationX` to `rotationY` and adjust `transformOrigin` accordingly.
- In frameworks, kill ScrollTrigger instances on unmount: `ScrollTrigger.getAll().forEach(t => t.kill())`.
