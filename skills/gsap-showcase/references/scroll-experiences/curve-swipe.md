# Curve Swipe

**Source:** [CodePen](https://codepen.io/GreenSock/pen/OJrJEaR) by GSAP
**Plugins:** MorphSVG
**Category:** scroll-experiences

## What Makes It Premium

The SVG curve wipe creates an elegant page transition effect where a gradient-filled SVG path morphs from a flat bottom edge through a curved bulge to a full-screen cover. The two-phase morph (flat to curve, curve to full) with different easings (`power2.in` then `power2.out`) creates a satisfying organic feel. The gradient fill adds visual richness that pure CSS clip-path transitions cannot achieve.

## Orchestration

A single SVG `<path>` element starts as a flat line at the bottom. The timeline morphs it through two states: first to a curved peak (`V 50 Q 50 0 100 50`) with `power2.in` easing, then to a full rectangle (`V 0 Q 50 0 100 0`) with `power2.out`. The timeline is reversed by default and toggled on click, creating a reversible transition. The SVG uses `preserveAspectRatio="xMidYMin slice"` to maintain full viewport coverage.

## Code

### HTML
```html
<p class="fixed braces">click me</p>
<div class="wrapper">
  <svg class="transition" viewBox="0 0 100 100" preserveAspectRatio="xMidYMin slice">
    <defs>
      <linearGradient id="grad" x1="0" y1="0" x2="99" y2="99" gradientUnits="userSpaceOnUse">
        <stop offset="0.2" stop-color="rgb(255, 135, 9)" />
        <stop offset="0.7" stop-color="rgb(247, 189, 248)" />
      </linearGradient>
    </defs>
    <!-- Initial state: flat line at bottom -->
    <path class="path" stroke="url(#grad)" fill="url(#grad)" stroke-width="2px"
          vector-effect="non-scaling-stroke"
          d="M 0 100 V 100 Q 50 100 100 100 V 100 z" />
  </svg>
</div>
```

### CSS
```css
* {
  padding: 0;
  margin: 0;
}

.fixed {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
  color: #fffce1;
  z-index: 999;
}

html {
  overflow: hidden;
  background: #0e100f;
  cursor: pointer;
}

.wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

.transition {
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  right: 0;
  width: 100%;
  height: 100%;
}
```

### JavaScript
```js
let path = document.querySelector(".path");

// Two morph targets: curved peak and full cover
const start = "M 0 100 V 50 Q 50 0 100 50 V 100 z";  // curve bulge
const end   = "M 0 100 V 0 Q 50 0 100 0 V 100 z";     // full rectangle

let tl = gsap.timeline();

// Phase 1: flat → curve (accelerating in)
tl.to(path, { morphSVG: start, ease: "power2.in" })
// Phase 2: curve → full cover (decelerating out)
  .to(path, { morphSVG: end, ease: "power2.out" })
  .reverse();  // start reversed — toggle on interaction

document.body.addEventListener("click", (e) => {
  tl.reversed(!tl.reversed());
});
```

## Adaptation Notes

- **Page transitions**: Use with a router — trigger `tl.play()` before navigation, load new content behind the SVG, then `tl.reverse()` to reveal.
- **Scroll-driven**: Replace click with ScrollTrigger scrub for a scroll-activated wipe effect.
- **Direction variants**: Modify the path `d` attribute to create wipes from top, left, or right. Change the `Q` control point to alter the curve shape.
- **Gradient customization**: Swap the `linearGradient` stops for brand colors. Add more stops for richer gradients.
- **Performance**: MorphSVG on a single path is extremely lightweight. The SVG overlay renders on the GPU compositor layer.
- **Framework integration**: Store the timeline ref and call `.reversed()` toggle from component methods. Clean up with `tl.kill()` on unmount.
