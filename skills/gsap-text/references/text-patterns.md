# Text Animation Patterns Reference

## Table of Contents
- [Combined Clip + Scramble (chainTextReveal)](#combined-clip--scramble-chaintextreveal)
- [Kinetic Character Split](#kinetic-character-split)
- [Elastic Type Assembly (Pin + Scrub)](#elastic-type-assembly-pin--scrub)
- [Rolling Text (Slot Machine Effect)](#rolling-text-slot-machine-effect)
- [Horizontal Text (ContainerAnimation + SplitText)](#horizontal-text-containeranimation--splittext)
- [Masked Lines with SplitText](#masked-lines-with-splittext)
- [AutoSplit with ScrollTrigger](#autosplit-with-scrolltrigger)

---

## Combined Clip + Scramble (chainTextReveal)

The signature production pattern: word slides up from mask AND scramble-decodes simultaneously.

Per word: slide-up tween + scramble tween at the same timeline position. Both MUST have `overwrite: false` (see learnings.md). The `chainTextReveal` function in `useReveal.js` handles this automatically.

```js
ctx = gsap.context(() => {
  const s = SplitText.create(el, { type: 'words', mask: 'words' })
  splits.push(s)
  gsap.set(s.words, { y: '100%' })
  gsap.set(el, { visibility: 'visible' })

  const CLIP  = { duration: 0.8, ease: 'power4.out', force3D: true }
  const SCRAM = { duration: 0.5, ease: 'none' }

  const tl = gsap.timeline({
    scrollTrigger: { trigger: el, start: 'top 80%', toggleActions: 'play none none reverse' },
  })

  s.words.forEach((w, i) => {
    const pos = i * 0.08
    tl.to(w, { y: '0%', overwrite: false, ...CLIP }, pos)
    tl.to(w, { overwrite: false, ...SCRAM,
      scrambleText: { text: w.textContent, chars: '01', speed: 0.5 },
    }, pos)
  })
}, scopeRef.value)
```

---

## Kinetic Character Split

Characters start at random positions and assemble into the final word.

```js
ctx = gsap.context(() => {
  const split = SplitText.create(el, { type: 'chars' })
  splits.push(split)

  gsap.set(split.chars, {
    y: () => gsap.utils.random(-200, 200), x: () => gsap.utils.random(-100, 100),
    rotation: () => gsap.utils.random(-90, 90), scale: () => gsap.utils.random(0.3, 2),
    autoAlpha: 0,
  })
  gsap.to(split.chars, {
    y: 0, x: 0, rotation: 0, scale: 1, autoAlpha: 1,
    duration: 1.4, ease: 'power4.out', force3D: true,
    stagger: { each: 0.04, from: 'random' },
    scrollTrigger: { trigger: el, start: 'top 80%', toggleActions: 'play none none reverse' },
  })
}, scopeRef.value)
```

---

## Elastic Type Assembly (Pin + Scrub)

Characters start scattered, assemble on scroll, hold in place, then scatter again.

```js
ctx = gsap.context(() => {
  const split = SplitText.create(el, { type: 'chars' })
  splits.push(split)

  gsap.set(split.chars, {
    y: () => gsap.utils.random(-300, 300), x: () => gsap.utils.random(-200, 200),
    rotation: () => gsap.utils.random(-180, 180), scale: 0, autoAlpha: 0,
  })

  const tl = gsap.timeline({
    scrollTrigger: { trigger: el, start: 'top center', end: '+=150%', pin: true, scrub: 1 },
  })

  // Phase 1: Assemble
  tl.to(split.chars, {
    y: 0, x: 0, rotation: 0, scale: 1, autoAlpha: 1, duration: 1,
    ease: 'elastic.out(1.2, 1)', stagger: { each: 0.03, from: 'random' },
  })
  // Phase 2: Hold
  tl.to({}, { duration: 0.5 })
  // Phase 3: Scatter
  tl.to(split.chars, {
    y: () => gsap.utils.random(-400, 400), x: () => gsap.utils.random(-300, 300),
    rotation: () => gsap.utils.random(-180, 180), scale: 0, autoAlpha: 0, duration: 1,
    ease: 'power2.in', stagger: { each: 0.02, from: 'edges' },
  })
}, scopeRef.value)
```

---

## Rolling Text (Slot Machine Effect)

Source: [CodePen — Rolling Text](https://codepen.io/GreenSock/pen/dPMjJWv)

3D character rotation creating a slot-machine / rolling text effect. Each line's characters rotate on the X-axis through a perspective container.

### HTML

```html
<div class="container">
  <div class="tube">
    <h1 class="line line1">SplitText</h1>
    <h1 class="line line2">SplitText</h1>
    <h1 class="line line3">SplitText</h1>
    <h1 class="line line3">SplitText</h1>
  </div>
</div>
```

### CSS

```css
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  visibility: hidden;
}

.tube {
  position: relative;
  width: 100%;
  height: 24vw;
}

.line {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  line-height: 1;
  margin: 0;
  letter-spacing: -0.6vw;
  font-size: 18vw;
  white-space: nowrap;
  text-align: center;
}

/* Hide back faces for clean 3D rotation */
.line div {
  backface-visibility: hidden;
}
```

### JS

```js
gsap.registerPlugin(SplitText);

const container = document.querySelector(".container");
gsap.set(container, { visibility: "visible" });

const lines = document.querySelectorAll(".line");

// Split each line into characters
const splitLines = Array.from(lines).map(line =>
  new SplitText(line, { type: "chars", charsClass: "char" })
);

// 3D perspective setup — depth pushes transform origin behind the text
const width = window.innerWidth;
const depth = -width / 8;
const transformOrigin = `50% 50% ${depth}`;

gsap.set(lines, { perspective: 700, transformStyle: "preserve-3d" });

// Infinite rolling loop
const animTime = 0.9;
const tl = gsap.timeline({ repeat: -1 });

splitLines.forEach((split, index) => {
  tl.fromTo(
    split.chars,
    { rotationX: -90 },
    {
      rotationX: 90,
      stagger: 0.08,
      duration: animTime,
      ease: "none",
      transformOrigin
    },
    index * 0.45 // offset each line for cascading effect
  );
});
```

**Key patterns:**
- `perspective` + `transformStyle: "preserve-3d"` on parent lines
- `backface-visibility: hidden` on split char divs
- `fromTo` with rotationX -90 → 90 for continuous 3D roll
- Staggered line offsets create cascading slot-machine feel

---

## Horizontal Text (ContainerAnimation + SplitText)

Source: [CodePen — ContainerAnimation SplitText](https://codepen.io/GreenSock/pen/MYyBrZw)

Horizontal scrolling text where each character animates in from random positions using ScrollTrigger's `containerAnimation` for nested scroll-driven animations.

### HTML

```html
<section class="Horizontal">
  <div class="container">
    <h3 class="Horizontal__text heading-xl">
      The containerAnimation property allows us to create ScrollTriggered
      animations within a container that's animated horizontally.
    </h3>
  </div>
</section>
```

### CSS

```scss
.Horizontal {
  overflow: hidden;
  height: 100vh;
  display: flex;
  align-items: center;
}

.Horizontal__text {
  display: flex;
  width: max-content;
  white-space: nowrap;
  gap: 4vw;
  padding-left: 100vw;
}

.heading-xl {
  font-size: clamp(2rem, 10vw, 12rem);
  font-weight: 600;
  line-height: 1.1;
}
```

### JS

```js
gsap.registerPlugin(SplitText, ScrollTrigger);

let wrapper = document.querySelector(".Horizontal");
let text = document.querySelector(".Horizontal__text");
let split = SplitText.create(".Horizontal__text", { type: "chars, words" });

// Main horizontal scroll tween — this becomes the containerAnimation
const scrollTween = gsap.to(text, {
  xPercent: -100,
  ease: "none",
  scrollTrigger: {
    trigger: wrapper,
    pin: true,
    end: "+=5000px",
    scrub: true
  }
});

// Each character has its own ScrollTrigger nested inside the horizontal scroll
split.chars.forEach((char) => {
  gsap.from(char, {
    yPercent: "random(-200, 200)",
    rotation: "random(-20, 20)",
    ease: "back.out(1.2)",
    scrollTrigger: {
      trigger: char,
      containerAnimation: scrollTween, // key: nests inside horizontal scroll
      start: "left 100%",
      end: "left 30%",
      scrub: 1
    }
  });
});
```

**Key patterns:**
- `containerAnimation` links per-char ScrollTriggers to the horizontal scroll tween
- Horizontal start/end values (`left 100%`, `left 30%`) instead of vertical
- `random()` for organic character entry positions and rotations
- `pin: true` + `scrub` on the outer wrapper for horizontal scroll

---

## Masked Lines with SplitText

Source: [CodePen — Masked Lines with SplitText](https://codepen.io/GreenSock/pen/LEYqezo)

Line-by-line masked reveal using `mask: "lines"` — each line slides up from behind its mask container for a clean clip-path-free reveal.

### HTML

```html
<div class="container">
  <h1 class="split">The text in this paragraph is split by words and lines.
  We have enabled masking on the lines so that we can animate the lines
  to create a fun 'reveal' animation.</h1>
</div>

<button>Replay Slowly</button>
```

### CSS

```css
.container {
  max-width: 80vw;
}

.split {
  opacity: 0;
  text-align: center;
  font-size: clamp(2rem, 5rem, 3vw);
  letter-spacing: 0.05rem;
  will-change: transform;
}

.split * {
  will-change: transform;
}
```

### JS

```js
gsap.registerPlugin(SplitText);

document.fonts.ready.then(() => {
  gsap.set(".split", { opacity: 1 });

  let split;
  SplitText.create(".split", {
    type: "words,lines",
    linesClass: "line",
    autoSplit: true,
    mask: "lines",   // key: creates overflow-hidden mask per line
    onSplit: (self) => {
      split = gsap.from(self.lines, {
        duration: 0.6,
        yPercent: 100,  // slide up from below mask
        opacity: 0,
        stagger: 0.1,
        ease: "expo.out",
      });
      return split;  // return so autoSplit can kill/recreate on resize
    }
  });

  // Replay at slow speed
  document.querySelector("button").addEventListener("click", (e) => {
    split.timeScale(0.2).play(0);
  });
});
```

**Key patterns:**
- `mask: "lines"` creates overflow-hidden wrappers — no CSS clip-path needed
- `autoSplit: true` + `onSplit` callback for responsive re-splitting on resize
- Return the animation from `onSplit` so SplitText can manage lifecycle
- `document.fonts.ready` before splitting to prevent measurement errors
- `yPercent: 100` slides each line up from its mask

---

## AutoSplit with ScrollTrigger

Source: [CodePen — AutoSplit with ScrollTrigger](https://codepen.io/GreenSock/pen/GggpRoB)

The canonical pattern for responsive line-split scroll animations. Uses `autoSplit: true` with `onSplit` to automatically re-split and recreate ScrollTrigger animations on browser resize.

### HTML

```html
<div class="spacer"></div>

<div class="container">
  <h2 class="split">This demo shows the correct way to set up your SplitText
  line animations with ScrollTrigger. It's important to return your animation
  inside the onSplit callback and set autoSplit to true.</h2>
</div>

<!-- Repeat .container blocks as needed -->
<div class="spacer"></div>
```

### CSS

```css
.container {
  width: 90vw;
  margin-top: 40vh;
}

.split {
  opacity: 0;
  text-align: center;
  font-size: 2rem;
  will-change: transform;
}

.split * {
  will-change: transform;
}

.spacer {
  height: 40vh;
}
```

### JS

```js
gsap.registerPlugin(SplitText, ScrollTrigger);

gsap.set(".split", { opacity: 1 });

document.fonts.ready.then(() => {
  let containers = gsap.utils.toArray(".container");

  containers.forEach((container) => {
    let text = container.querySelector(".split");

    SplitText.create(text, {
      type: "words,lines",
      mask: "lines",
      linesClass: "line",
      autoSplit: true,        // re-split on resize
      onSplit: (instance) => {
        // Return the animation so SplitText can kill/recreate it
        return gsap.from(instance.lines, {
          yPercent: 120,
          stagger: 0.1,
          scrollTrigger: {
            trigger: container,  // use parent container, not the text itself
            scrub: true,
            start: "clamp(top center)",
            end: "clamp(bottom center)"
          }
        });
      }
    });
  });
});
```

**Key patterns:**
- `autoSplit: true` watches for resize and re-splits lines automatically
- `onSplit` callback must **return** the animation — SplitText kills the old one before re-creating
- `clamp()` in ScrollTrigger start/end prevents trigger from extending beyond viewport
- Trigger on parent `container`, not the text element — more reliable with pinning
- `mask: "lines"` for clean overflow-hidden reveal per line
- `document.fonts.ready` ensures correct line measurements before splitting
