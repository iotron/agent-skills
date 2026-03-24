# Scrubbed Bento Gallery

**Source:** [CodePen](https://codepen.io/GreenSock/pen/YPKbMOa) by GSAP
**Plugins:** ScrollTrigger, Flip
**Category:** scroll-experiences

## What Makes It Premium

The bento grid morphs between two CSS Grid states using Flip plugin, creating a satisfying layout transformation as the user scrolls. The `expoScale(1, 5)` easing gives the transition a cinematic quality where items accelerate into their final positions. The combination of ScrollTrigger scrub with Flip's automatic interpolation between grid states eliminates manual position calculations entirely.

## Orchestration

Flip captures the "final" grid state (full-bleed columns), then reverts to the initial bento layout. ScrollTrigger scrubs through the Flip animation on scroll with the gallery parent pinned. On resize, the entire context is reverted and rebuilt to handle responsive grid recalculations. The `simple: true` flag on Flip optimizes performance by only animating position/size rather than computing complex transforms.

## Code

### HTML
```html
<div class="gallery-wrap">
  <div class="gallery gallery--bento gallery--switch" id="gallery-8">
    <div class="gallery__item">
      <img src="https://assets.codepen.io/16327/portrait-pattern-1.jpg" alt="" />
    </div>
    <div class="gallery__item">
      <img src="https://assets.codepen.io/16327/portrait-image-12.jpg" alt="" />
    </div>
    <div class="gallery__item">
      <img src="https://assets.codepen.io/16327/portrait-image-8.jpg" alt="" />
    </div>
    <div class="gallery__item">
      <img src="https://assets.codepen.io/16327/portrait-pattern-2.jpg" alt="" />
    </div>
    <div class="gallery__item">
      <img src="https://assets.codepen.io/16327/portrait-image-4.jpg" alt="" />
    </div>
    <div class="gallery__item">
      <img src="https://assets.codepen.io/16327/portrait-image-3.jpg" alt="" />
    </div>
    <div class="gallery__item">
      <img src="https://assets.codepen.io/16327/portrait-pattern-3.jpg" alt="" />
    </div>
    <div class="gallery__item">
      <img src="https://assets.codepen.io/16327/portrait-image-1.jpg" alt="" />
    </div>
  </div>
</div>
<div class="section">
  <h2>Here is some content</h2>
  <p>Lorem ipsum dolor sit amet...</p>
</div>
```

### CSS
```scss
p {
  font-size: 1.2rem;
  margin-bottom: 1rem;
}

.gallery-wrap {
  position: relative;
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.gallery {
  position: relative;
  width: 100%;
  height: 100%;
  flex: none;
}

.gallery__item {
  background-position: 50% 50%;
  background-size: cover;
  flex: none;
  position: relative;
  img {
    object-fit: cover;
    width: 100%;
    height: 100%;
  }
}

/* Initial bento layout — compact grid */
.gallery--bento {
  display: grid;
  gap: 1vh;
  grid-template-columns: repeat(3, 32.5vw);
  grid-template-rows: repeat(4, 23vh);
  justify-content: center;
  align-content: center;
}

/* Final state — full-bleed columns (Flip target) */
.gallery--final.gallery--bento {
  grid-template-columns: repeat(3, 100vw);
  grid-template-rows: repeat(4, 49.5vh);
  gap: 1vh;
}

/* Grid area assignments for asymmetric bento layout */
.gallery--bento .gallery__item:nth-child(1) { grid-area: 1 / 1 / 3 / 2; }
.gallery--bento .gallery__item:nth-child(2) { grid-area: 1 / 2 / 2 / 3; }
.gallery--bento .gallery__item:nth-child(3) { grid-area: 2 / 2 / 4 / 3; }
.gallery--bento .gallery__item:nth-child(4) { grid-area: 1 / 3 / 3 / 3; }
.gallery--bento .gallery__item:nth-child(5) { grid-area: 3 / 1 / 3 / 2; }
.gallery--bento .gallery__item:nth-child(6) { grid-area: 3 / 3 / 5 / 4; }
.gallery--bento .gallery__item:nth-child(7) { grid-area: 4 / 1 / 5 / 2; }
.gallery--bento .gallery__item:nth-child(8) { grid-area: 4 / 2 / 5 / 3; }

.section {
  padding: 2rem 5rem;
}
```

### JavaScript
```js
gsap.registerPlugin(ScrollTrigger);
gsap.registerPlugin(Flip);

let flipCtx;

const createTween = () => {
  let galleryElement = document.querySelector("#gallery-8");
  let galleryItems = galleryElement.querySelectorAll(".gallery__item");

  // Revert previous context on resize to avoid stale ScrollTriggers
  flipCtx && flipCtx.revert();
  galleryElement.classList.remove("gallery--final");

  flipCtx = gsap.context(() => {
    // Capture final state by temporarily adding the class
    galleryElement.classList.add("gallery--final");
    const flipState = Flip.getState(galleryItems);
    galleryElement.classList.remove("gallery--final");

    // Flip.to animates FROM current state TO captured final state
    const flip = Flip.to(flipState, {
      simple: true,                  // optimize: only animate position/size
      ease: "expoScale(1, 5)"       // cinematic acceleration curve
    });

    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: galleryElement,
        start: "center center",
        end: "+=100%",
        scrub: true,
        pin: galleryElement.parentNode  // pin the wrapper, not the grid
      }
    });
    tl.add(flip);

    return () => gsap.set(galleryItems, { clearProps: "all" });
  });
};

createTween();

// Rebuild on resize — grid dimensions change so Flip state must be recaptured
window.addEventListener("resize", createTween);
```

## Adaptation Notes

- **Framework integration**: Wrap `createTween` in `onMounted` with `gsap.context()` for Vue/React cleanup. Use `onUnmounted`/`useEffect` cleanup to call `flipCtx.revert()`.
- **Responsive**: The bento grid uses `vw`/`vh` units so it scales naturally. Adjust `grid-template-columns` and `grid-template-rows` for different breakpoints.
- **Content flexibility**: Add/remove `.gallery__item` elements and update the `nth-child` grid-area rules. The JS is content-agnostic.
- **Performance**: `simple: true` on Flip is critical for grids with many items — it skips expensive transform calculations.
- **Debounce resize**: In production, debounce the resize handler to avoid excessive rebuilds during window dragging.
