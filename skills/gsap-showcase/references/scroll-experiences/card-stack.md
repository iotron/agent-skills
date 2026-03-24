# Card Stack

**Source:** [CodePen](https://codepen.io/GreenSock/pen/KKYezKx) by GSAP
**Plugins:** Flip
**Category:** scroll-experiences

## What Makes It Premium

The card stack creates a tactile "deck of cards" interaction where clicking shuffles the top card to the bottom with smooth enter/leave transitions. Flip handles the DOM reorder animation automatically, computing position deltas for each card. The staggered offset positioning (each card shifted 20px) gives natural depth perception, and the `onEnter`/`onLeave` callbacks create polished opacity and position transitions for appearing/disappearing cards.

## Orchestration

On each click, the last child is cloned, prepended to the slider, and the original removed. Flip captures state before the DOM change, then `Flip.from()` animates all cards from their old positions to new ones. `onEnter` fades in the new front card from below, while `onLeave` slides the departing card off to the left with rotation, then removes it from the DOM on complete. The `absolute: true` flag prevents layout jumps during the transition.

## Code

### HTML
```html
<div class="slider">
  <img class="item item-5" src="https://assets.codepen.io/16327/portrait-number-5.png" alt="" />
  <img class="item item-4" src="https://assets.codepen.io/16327/portrait-number-4.png" alt="" />
  <img class="item item-3" src="https://assets.codepen.io/16327/portrait-number-3.png" alt="" />
  <img class="item item-2" src="https://assets.codepen.io/16327/portrait-number-2.png" alt="" />
  <img class="item item-1" src="https://assets.codepen.io/16327/portrait-number-1.png" alt="" />
</div>
```

### CSS
```css
body {
  min-height: 100vh;
  cursor: pointer;
}

.slider {
  position: absolute;
  width: 300px;
  height: 200px;
  perspective: 100px;
  top: 30vh;
  left: 50%;
  transform: translateX(-50%);
}

.item {
  position: absolute;
  width: 300px;
  aspect-ratio: 2/3;
  background: transparent;
}

/* Staggered offset creates depth illusion */
.item:nth-child(5) { left: 0;    top: 0; }
.item:nth-child(4) { left: 20px; top: -20px; }
.item:nth-child(3) { left: 40px; top: -40px; }
.item:nth-child(2) { left: 60px; top: -60px; }
.item:nth-child(1) { left: 80px; top: -80px; }
```

### JavaScript
```js
const slider = document.querySelector(".slider");
const items = gsap.utils.toArray(".item");

function moveCard() {
  const lastItem = slider.querySelector(".item:last-child");

  if (slider && lastItem) {
    lastItem.style.display = "none";
    const newItem = document.createElement("img");
    newItem.className = lastItem.className;
    newItem.src = lastItem.src;
    // Insert clone at beginning — this becomes the new "front" card
    slider.insertBefore(newItem, slider.firstChild);
  }
}

document.body.addEventListener("click", (e) => {
  // Capture positions BEFORE DOM change
  let state = Flip.getState(".item");

  moveCard();

  Flip.from(state, {
    targets: ".item",
    ease: "sine.inOut",
    absolute: true,         // prevent layout reflow during animation
    onEnter: (elements) => {
      // New card enters from below with fade
      return gsap.from(elements, {
        duration: 0.3,
        yPercent: 20,
        opacity: 0,
        ease: "expo.out"
      });
    },
    onLeave: (element) => {
      // Departing card slides off to the left
      return gsap.to(element, {
        duration: 0.3,
        yPercent: 5,
        xPercent: -5,
        transformOrigin: "bottom left",
        opacity: 0,
        ease: "expo.out",
        onComplete() {
          slider.removeChild(element[0]);
        }
      });
    }
  });
});
```

## Adaptation Notes

- **Auto-play**: Replace the click listener with a `gsap.delayedCall()` or `setInterval` for automatic card cycling.
- **Scroll-driven**: Wrap in a ScrollTrigger with `snap` to advance one card per scroll section.
- **Card content**: Replace `<img>` elements with `<div>` cards containing any content — Flip handles position animation regardless of content.
- **Stack depth**: Adjust the `nth-child` CSS offsets and add more items for deeper stacks. The offset pattern (20px increments) is easy to generate programmatically.
- **Framework integration**: In Vue/React, use a reactive array for card data and let Flip handle the DOM transition via `Flip.getState()` before state update and `Flip.from()` after re-render.
