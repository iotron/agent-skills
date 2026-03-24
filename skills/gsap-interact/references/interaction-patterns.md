# Interaction Patterns Reference

## Table of Contents
- [Draggable with liveSnap](#draggable-with-livesnap)

---

## Draggable with liveSnap

SVG point-editing interaction using GSAP Draggable with `liveSnap` — drag handles snap to nearby points with inertia.

> Source: [Blake Bowen's CodePen](https://codepen.io/osublake/) (forked)

### HTML — SVG structure

```html
<svg id="svg" viewBox="0 0 400 400">
  <defs>
    <circle class="handle" r="10" />
    <circle class="marker" r="4" />

    <linearGradient id="grad-1" x1="0" y1="0" x2="400" y2="400" gradientUnits="userSpaceOnUse">
       <stop offset="0.2" stop-color="#00bae2"></stop>
       <stop offset="0.8" stop-color="#fec5fb"></stop>
    </linearGradient>
  </defs>

  <polygon id="star" stroke="url(#grad-1)"
    points="261,220 298,335 200,264 102,335 139,220 42,149 162,148 200,34 238,148 358,149" />
  <g id="marker-layer"></g>
  <g id="handle-layer"></g>
</svg>
```

### JS — Draggable with liveSnap and inertia

```js
var star = document.querySelector("#star");
var markerDef = document.querySelector("defs .marker");
var handleDef = document.querySelector("defs .handle");
var markerLayer = document.querySelector("#marker-layer");
var handleLayer = document.querySelector("#handle-layer");

// Build a points array from the SVG polygon's point list
var points = [];
var numPoints = star.points.numberOfItems;

for (var i = 0; i < numPoints; i++) {
  var point = star.points.getItem(i);
  points[i] = { x: point.x, y: point.y };
  createHandle(point);
}

function createHandle(point) {
  var marker = createClone(markerDef, markerLayer, point);
  var handle = createClone(handleDef, handleLayer, point);

  // Update the live SVG point on every drag frame
  var update = function () { point.x = this.x; point.y = this.y; };

  var draggable = new Draggable(handle, {
    onDrag: update,
    onThrowUpdate: update,   // Also fires during inertia throw
    inertia: true,            // Enable throw/flick behavior
    bounds: window,
    liveSnap: {
      points: points,         // Snap to any polygon vertex
      radius: 15              // Only snap within 15px proximity
    }
  });
}

// Clone an SVG template element and position it
function createClone(node, parent, point) {
  var element = node.cloneNode(true);
  parent.appendChild(element);
  gsap.set(element, { x: point.x, y: point.y });
  return element;
}
```

### Key patterns

- **`liveSnap.points`** — accepts an array of `{x, y}` objects; Draggable snaps to the nearest point within `radius` during drag
- **`liveSnap.radius`** — distance threshold (in pixels) for snapping; outside this radius, no snap occurs
- **`inertia: true`** — enables throw/flick physics; `onThrowUpdate` fires each frame during the inertia phase
- **`onDrag` + `onThrowUpdate`** — both update the SVG polygon point in real time, keeping the shape in sync with handles
- **SVG `points.getItem(i)`** — returns a live `SVGPoint` reference; mutating `.x`/`.y` immediately updates the rendered polygon

---

## macOS Dock Effect

macOS-style dock magnification on hover — items scale up based on proximity to cursor using cosine-based distance falloff.

> Source: Forked from [Blake Bowen's CodePen](https://codepen.io/osublake/pen/GYzqjL)

### HTML — Dock toolbar

```html
<div class="wrapper">
  <ul class="toolbar">
    <li class="toolbarItem">
      <a class="toolbarLink" href="#!">
        <img class="toolbarImg" src="icon-1.png" alt="">
      </a>
    </li>
    <li class="toolbarItem">
      <a class="toolbarLink" href="#!">
        <img class="toolbarImg" src="icon-2.png" alt="">
      </a>
    </li>
    <!-- repeat for each dock icon -->
  </ul>
</div>
```

### CSS — Fixed bottom dock

```css
.wrapper {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translate(-50%);
  display: flex;
  justify-content: center;
}

.toolbar {
  display: inline-flex;
  justify-content: center;
  align-items: flex-end;
  height: 40px;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
  margin: 0;
  padding: 10px;
  background-color: rgba(55, 66, 77, 0.25);
  list-style: none;
}

.toolbarItem {
  width: 40px;
  height: 40px;
  margin: 0 4px;
}

.toolbarLink {
  display: block;
  height: 100%;
}

.toolbarImg {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 50%;
}
```

### JS — Proximity-based magnification

```js
let icons = document.querySelectorAll(".toolbarItem");
let dock = document.querySelector(".toolbar");
let firstIcon = icons[0];

let min = 48;   // icon width + margin
let max = 120;  // max scaled size
let bound = min * Math.PI;

gsap.set(icons, {
  transformOrigin: "50% 120%",
  height: 40
});

gsap.set(dock, {
  position: "relative",
  height: 60
});

dock.addEventListener("mousemove", (event) => {
  let offset = dock.getBoundingClientRect().left + firstIcon.offsetLeft;
  updateIcons(event.clientX - offset);
});

dock.addEventListener("mouseleave", (event) => {
  gsap.to(icons, {
    duration: 0.3,
    scale: 1,
    x: 0
  });
});

function updateIcons(pointer) {
  for (let i = 0; i < icons.length; i++) {
    let icon = icons[i];
    let distance = (i * min + min / 2) - pointer;
    let x = 0;
    let scale = 1;

    if (-bound < distance && distance < bound) {
      let rad = distance / min * 0.5;
      scale = 1 + (max / min - 1) * Math.cos(rad);
      x = 2 * (max - min) * Math.sin(rad);
    } else {
      x = (-bound < distance ? 2 : -2) * (max - min);
    }

    gsap.to(icon, {
      duration: 0.3,
      x: x,
      scale: scale
    });
  }
}
```

### Key patterns

- **Cosine-based scale falloff** — `Math.cos(rad)` produces a smooth bell curve; items nearest the cursor scale most, with graceful falloff to neighbors
- **Sine-based displacement** — `Math.sin(rad)` pushes neighboring items outward, preventing overlap during magnification
- **`bound = min * Math.PI`** — defines the influence radius; items beyond this distance remain at base scale
- **`transformOrigin: "50% 120%"`** — scales icons upward from below their center, mimicking the macOS dock's bottom-anchored growth
- **`mouseleave` reset** — smoothly animates all icons back to `scale: 1, x: 0` with 0.3s duration
- **Per-icon `gsap.to`** — each icon gets its own tween for independent smooth animation; GSAP handles overwrite automatically
