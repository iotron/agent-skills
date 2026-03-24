# Complex SVG Timeline with Mask Animations

**Source:** [CodePen](https://codepen.io/GreenSock/pen/PorGpww) by GSAP
**Plugins:** Timeline labels, SVG attr manipulation, GSDevTools
**Category:** interactive-scenes

## What Makes It Premium

This showcase demonstrates elaborate SVG mask and transform choreography using only core GSAP (no extra plugins). Five distinct image groups reveal through different mask techniques — scale burst, circle wipe, quadrant rotation, bounce entrance, and triangle/circle mask expansion. The negative stagger on group 3's mask rectangles creates an elegant "closing iris" effect. Each group transitions seamlessly into the next with precise overlap timing, creating a continuous visual flow from a single looping timeline.

## Orchestration

A single `gsap.timeline({ repeat: -1 })` drives all five groups in sequence with overlapping transitions:

1. **Group 1 (0s):** Scale from 0.1 to 1 with `expo.inOut`, then rotate 15deg while images scatter to offset positions with `back` ease. Fade out with staggered opacity.
2. **Group 2 (1.3s):** Two SVG mask circles expand via `attr: { r: 124 }` to reveal the image, then slide apart horizontally to wipe away.
3. **Group 3 (2.6s):** Four mask rectangles scale in from their respective corners with **negative stagger** (`stagger: -0.03`) while the group rotates from -90 to 0 with `expo` ease.
4. **Group 4 (3.8s):** Single image bounces in with `bounce` ease at small scale, rotates from 15 to 0 degrees.
5. **Group 5 (4.7s):** SVG triangle path and circle mask elements scale up with `expo` ease to reveal final image.

All mask animations use SVG `<mask>` elements with `attr` tweens on geometric properties (r, cx, scale).

## Code

### HTML
```html
<main>
  <svg id="svg-stage" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 248 248">

    <g class="group1">
      <image href="https://assets.codepen.io/16327/flair-2.png" />
      <image href="https://assets.codepen.io/16327/flair-2.png" />
      <image href="https://assets.codepen.io/16327/flair-2.png" />
    </g>

    <g class="group2">
      <image href="https://assets.codepen.io/16327/flair-3.png" mask="url(#g2_mask)"/>
    </g>
    <mask id="g2_mask" fill="#fff">
      <circle cx="124" cy="0" r="0"/>
      <circle cx="124" cy="248" r="0"/>
    </mask>

    <g class="group3">
      <image href="https://assets.codepen.io/16327/flair-4.png" mask="url(#g3_mask)"/>
    </g>
    <mask id="g3_mask" fill="#fff">
      <rect x="0" y="0" width="124" height="124"/>
      <rect x="124" y="0" width="124" height="124"/>
      <rect x="0" y="124" width="124" height="124"/>
      <rect x="124" y="124" width="124" height="124"/>
    </mask>

    <g class="group4">
      <image href="https://assets.codepen.io/16327/flair-5.png" x="70" />
    </g>

    <g class="group5">
      <image href="https://assets.codepen.io/16327/flair-7.png" mask="url(#g5_mask)"/>
    </g>
    <mask id="g5_mask" fill="#fff">
      <path d="M0 248h248L124 0 0 247z"/>
      <circle cx="124" cy="83" r="83"/>
    </mask>

  </svg>
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
  background: #0e100f;
}

#svg-stage {
  opacity: 0;
  width: 57.14%;
  max-height: 248px;
}
```

### JavaScript
```js
gsap.set('#svg-stage', { opacity: 1 });

const tl = gsap.timeline({ repeat: -1, repeatDelay: 0.5 })

// === GROUP 1: Scale burst + scatter ===
// Scale entire group from center, then distribute images to offset positions
.fromTo('.group1',
  { scale: 0.1, transformOrigin: '124 124' },
  { duration: 0.35, scale: 1, ease: 'expo.inOut' }
)
.to('.group1', { duration: 1.2, rotate: 15, ease: 'none' }, 0.1)
// Scatter: each image gets unique x/y/scale via function-based values
.to('.group1 image', {
  scale: (i) => [0.4, 0.2, 0.3][i],
  x: (i) => [0, 135, 100][i],
  y: (i) => [90, 24, 124][i],
  ease: 'back'
}, 0.4)
// Staggered fadeout
.to('.group1 image', { duration: 0.01, opacity: 0, stagger: 0.06 }, 1.1)

// === GROUP 2: Circle mask wipe reveal ===
// Two circles (top and bottom) expand to fill, then slide apart
.to('#g2_mask circle', { duration: 0.4, attr: { r: "124" }, ease: 'circ' }, 1.3)
.fromTo('.group2',
  { scale: 1, transformOrigin: '124 124' },
  { duration: 1.5, scale: 0.9, ease: 'none' }, 1.3
)
// Wipe: circles slide in opposite directions to clear the mask
.to('#g2_mask circle', {
  duration: 0.3,
  attr: { cx: (i) => ["+=248", "-=248"][i] },
  ease: 'sine.in'
}, 2.45)

// === GROUP 3: Quadrant rotation with negative stagger ===
// Group rotates from -90 while four mask rects scale from their corners
.fromTo('.group3',
  { transformOrigin: '124 124', rotate: -90 },
  { duration: 0.9, rotate: 0, ease: 'expo' }, 2.6
)
.fromTo('#g3_mask rect',
  // Each rect scales from its own corner via function-based transformOrigin
  { transformOrigin: (i) => ['0 124', '124 0', '124 124', '248 124'][i], scale: 0 },
  { duration: 0.4, scale: 1, ease: 'expo', stagger: -0.03 }, // Negative stagger = reverse order
  2.6
)
.to('.group3', { duration: 0.01, scale: 0 }, 3.7)

// === GROUP 4: Bounce entrance ===
.from('.group4 image', { duration: 0.01, opacity: 0 }, 3.8)
.fromTo('.group4',
  { transformOrigin: '83 124', rotate: 15, scale: 0.2 },
  { duration: 0.5, rotate: 0, scale: 0.85, ease: 'bounce' }, 3.8
)
.to('.group4 image', { duration: 0.01, opacity: 0 }, 4.7)

// === GROUP 5: Triangle + circle mask expansion ===
.fromTo('#g5_mask path',
  { transformOrigin: '124 124', scale: 0 },
  { duration: 0.8, scale: 1, ease: 'expo' }, 4.7
)
.fromTo('#g5_mask circle',
  { transformOrigin: '83 0', scale: 0 },
  { scale: 1, ease: 'expo' }, 4.7
);

GSDevTools.create({ animation: tl });
```

## Adaptation Notes

- This is a masterclass in **SVG mask animation** — the masks are the animation targets, not the images. Animate mask geometry (r, cx, scale, x, y) to create reveals and wipes.
- Function-based values `(i) => [...]` are the key to per-element customization without separate tweens. Use this pattern whenever elements in a group need different destinations.
- Negative stagger (`stagger: -0.03`) reverses the animation order — last element animates first. This creates a "closing" effect rather than "opening."
- The `transformOrigin` uses SVG coordinate values (e.g., `'124 124'`) rather than CSS percentages because this is SVG content.
- Replace image URLs with your own assets. The mask shapes (circles, rects, paths) define the reveal geometry and can be swapped for custom SVG shapes.
- Remove `GSDevTools.create()` for production.
- In frameworks, this pattern works well as a loading animation or section transition. The looping timeline can be paused/played based on visibility.
