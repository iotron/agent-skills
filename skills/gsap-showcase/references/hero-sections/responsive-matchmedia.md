# Responsive matchMedia Animation Branching

**Source:** [CodePen](https://codepen.io/GreenSock/pen/wvLPVGz) by GSAP
**Plugins:** gsap.matchMedia()
**Category:** hero-sections

## What Makes It Premium

This demonstrates the correct GSAP approach to responsive animations: `gsap.matchMedia()` creates completely separate animation branches for different breakpoints, with automatic cleanup when conditions change. Rather than hiding/showing with CSS media queries (which leaves zombie animations running), matchMedia kills and rebuilds animations cleanly. The `back.out` entrance ease and continuous rotation create satisfying visual feedback.

## Orchestration

A single `gsap.matchMedia()` call defines two named conditions (`isSmall` at max-width 800px, `isLarge` at min-width 801px). Each condition branch independently sets display, creates entrance animations, and runs continuous rotations. When the viewport crosses the breakpoint, GSAP automatically reverts all animations and DOM changes from the previous branch, then runs the new branch. No manual cleanup needed.

## Code

### HTML
```html
<div class="box small">
  <img src="https://assets.codepen.io/16327/scroll-flair-2.png" alt="" />
</div>

<div class="box large">
  <img src="https://assets.codepen.io/16327/scroll-flair-2.png" alt="" />
</div>

<div class="text">
  <h4 class="braces">Resize the window</h4>
</div>
```

### CSS
```css
@font-face {
  font-display: block;
  font-family: Mori;
  font-style: normal;
  font-weight: 600;
  src: url(https://assets.codepen.io/16327/PPMori-Regular.woff) format("woff");
}

html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100vh;
}

body {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-family: "Mori";
  background: #0e100f;
  text-align: center;
  color: rgb(255, 252, 225);
  font-size: clamp(1rem, 1.5rem, 3vw);
}

.box {
  background: #fffce1;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 2rem;
  opacity: 0;
}

.small {
  aspect-ratio: 9/16;
  width: 20%;
}

.large {
  aspect-ratio: 16/9;
  width: 50%;
}

.box img {
  width: 33%;
}

.text {
  position: fixed;
  bottom: 2rem;
}
```

### JavaScript
```js
// Initial state — boxes hidden until matchMedia activates
gsap.set(".box", { opacity: 1 });

const mm = gsap.matchMedia();

// Named conditions — cleaner than separate mm.add() calls
mm.add({
  isSmall: "(max-width: 800px)",
  isLarge: "(min-width: 801px)"
}, (c) => {
  // Each branch runs independently; previous branch auto-reverts on breakpoint change
  if (c.conditions.isSmall) {
    gsap.set(".large", { display: "none" });
    gsap.from(".small", { scale: 0, ease: "back.out" });
    gsap.to(".small img", { rotation: 360, repeat: -1, ease: "none", duration: 1 });
  }
  if (c.conditions.isLarge) {
    gsap.set(".small", { display: "none" });
    gsap.from(".large", { scale: 0, ease: "back.out" });
    gsap.to(".large img", { rotation: -360, repeat: -1, ease: "none", duration: 1 });
  }
});

// Alternative syntax — separate mm.add() calls for each breakpoint:
// mm.add("(max-width: 800px)", () => { ... });
// mm.add("(min-width: 801px)", () => { ... });
```

## Adaptation Notes

- Use named conditions (`isSmall`, `isLarge`, `isTablet`) when you need multiple breakpoints in one callback — it reads more clearly than nested ifs.
- Use separate `mm.add()` calls when breakpoint animations are completely independent with no shared logic.
- All animations created inside the callback are automatically killed and reverted when the condition no longer matches. This includes `gsap.set()` calls — they get reverted too.
- In frameworks, call `mm.revert()` on component unmount to clean up all matchMedia listeners.
- This pattern replaces CSS-only responsive approaches that leave GSAP animations running invisibly on hidden elements, wasting resources.
- For scroll-driven responsive animations, nest `ScrollTrigger.create()` inside `matchMedia` callbacks — they will be killed and recreated on breakpoint changes automatically.
