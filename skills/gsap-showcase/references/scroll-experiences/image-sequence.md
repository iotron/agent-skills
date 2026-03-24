# Image Sequence

**Source:** [CodePen](https://codepen.io/GreenSock/pen/jOxeEaV) by GSAP
**Plugins:** ScrollTrigger
**Category:** scroll-experiences

## What Makes It Premium

This is the canonical Apple-style scroll-driven image sequence pattern — frame-by-frame playback controlled by scroll position. The canvas-based approach renders 147 frames smoothly by only drawing when the frame index changes, avoiding unnecessary repaints. The reusable `imageSequence()` helper function abstracts all the complexity into a clean config object, making it trivial to apply to any image sequence.

## Orchestration

GSAP tweens a `playhead.frame` property from 0 to the last frame index with `ease: "none"` for linear frame distribution across the scroll distance. The `onUpdate` callback checks if the rounded frame has changed before drawing to canvas, preventing redundant `drawImage` calls. ScrollTrigger scrubs from `start: 0` to `end: "max"` (entire page), with the canvas fixed-positioned in the viewport. Images are preloaded into an array on initialization.

## Code

### HTML
```html
<canvas id="image-sequence" width="1158" height="770" />
```

### CSS
```css
body {
  height: 300vh;
  background: #000;
}

canvas {
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  max-width: 80vw;
  max-height: 80vh;
}
```

### JavaScript
```js
// Apple-style scroll-driven image sequence
let frameCount = 147,
    urls = new Array(frameCount).fill().map((o, i) =>
      `https://www.apple.com/105/media/us/airpods-pro/2019/1299e2f5_9206_4470_b28e_08307a42f19b/anim/sequence/large/${(i+1).toString().padStart(4, '0')}.jpg`
    );

imageSequence({
  urls,
  canvas: "#image-sequence",
  scrollTrigger: {
    start: 0,
    end: "max",    // entire page scroll distance
    scrub: true,
  }
});

/**
 * Reusable image sequence helper.
 * Config:
 * - urls [Array]: image URL array
 * - canvas [Canvas]: target <canvas> element or selector
 * - scrollTrigger [Object]: ScrollTrigger config
 * - clear [Boolean]: clear canvas before each frame (for transparent images)
 * - paused [Boolean]: start paused (useful for non-scroll playback)
 * - fps [Number]: frames per second for duration calc (default 30)
 * - onUpdate [Function]: callback(frameIndex, imageElement)
 *
 * Returns: GSAP Tween instance
 */
function imageSequence(config) {
  let playhead = { frame: 0 },
      canvas = gsap.utils.toArray(config.canvas)[0],
      ctx = canvas.getContext("2d"),
      curFrame = -1,
      onUpdate = config.onUpdate,
      images,
      updateImage = function() {
        let frame = Math.round(playhead.frame);
        if (frame !== curFrame) {  // only draw when frame actually changes
          config.clear && ctx.clearRect(0, 0, canvas.width, canvas.height);
          ctx.drawImage(images[frame], 0, 0);
          curFrame = frame;
          onUpdate && onUpdate.call(this, frame, images[frame]);
        }
      };

  // Preload all images into array
  images = config.urls.map((url, i) => {
    let img = new Image();
    img.src = url;
    i || (img.onload = updateImage);  // draw first frame once loaded
    return img;
  });

  return gsap.to(playhead, {
    frame: images.length - 1,
    ease: "none",             // linear distribution across scroll
    onUpdate: updateImage,
    duration: images.length / (config.fps || 30),
    paused: !!config.paused,
    scrollTrigger: config.scrollTrigger
  });
}
```

## Adaptation Notes

- **Image optimization**: Use WebP format and appropriate resolution for the viewport. Consider generating multiple sizes and selecting based on `window.devicePixelRatio`.
- **Preloading strategy**: For large sequences, implement progressive loading — show a poster frame while remaining images load in the background.
- **Canvas sizing**: Set `width`/`height` attributes to match source image dimensions. Use CSS `max-width`/`max-height` for responsive display.
- **Transparent sequences**: Set `clear: true` in config if frames have transparency (e.g., product reveals on custom backgrounds).
- **Framework integration**: The `imageSequence` helper is framework-agnostic. In Vue/React, call it in `onMounted`/`useEffect` and store the returned tween for cleanup via `tween.kill()`.
- **Scroll distance**: Adjust `body` height or use `end: "+=3000"` with `pin: true` to control how much scrolling maps to the full sequence.
