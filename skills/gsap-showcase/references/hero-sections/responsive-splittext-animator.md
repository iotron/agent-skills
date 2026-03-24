# Responsive SplitText Animator Helper

**Source:** [CodePen](https://codepen.io/GreenSock/pen/dPpRrqa) by GSAP
**Plugins:** SplitText, GSDevTools
**Category:** hero-sections

## What Makes It Premium

The SplitTextAnimator helper function creates a subscription pattern where multiple animations share one SplitText instance and automatically reconnect on re-split. This means text animations stay perfectly synchronized during viewport resizes without manual cleanup. The combination of random stagger with word-level animation creates organic, premium reveal timing that feels intentional rather than mechanical.

## Orchestration

A single master timeline sequences the entire hero: title fade-in, decorative dots staggering in, then SplitText word reveals with random stagger. The SplitTextAnimator helper wraps `autoSplit` + `onSplit` so each animation subscribes to the SplitText instance. On resize, a debounced `delayedCall` hides content, then restarts the timeline after the SplitText re-splits. GSDevTools provides scrubbing during development.

## Code

### HTML
```html
<div class="panel">
  <div class="content">
    <h3 class="title text-center">Responsive SplitText Animator Helper Function</h3>
    <div class="dot-container">
      <div class="dot gradient-green"></div>
      <div class="dot gradient-green"></div>
      <div class="dot gradient-green"></div>
    </div>
    <div class="content-text ">Quas dolor ullam alias repellendus odit eum fugit aspernatur sit aperiam assumenda recusandae deleniti labore atque dolores cupiditate, nam impedit nulla similique fuga in sapiente.</div>
    <div class="dot-container">
      <div class="dot gradient-green"></div>
      <div class="dot gradient-green"></div>
      <div class="dot gradient-green"></div>
    </div>
  </div>
</div>
```

### CSS
```css
.content {
  width: 100%;
  max-width: 1200px;
  font-size: 24px;
  opacity: 0;
  visibility: hidden;

  .content-text {
    text-align: center;
  }

  .dot-container {
    display: flex;
    width: 100%;
    justify-content: center;
    padding: 1rem 0;
    gap: 10px;

    .dot {
      width: 15px;
      height: 15px;
      border-radius: 15px;
    }
  }
}
```

### JavaScript
```js
console.clear();

gsap.registerPlugin(SplitText);

const mainTitle = document.querySelector(".title");
const contentTexts = gsap.utils.toArray(".content-text");
const dotContainers = gsap.utils.toArray(".dot-container");
const contentTextsAmount = contentTexts.length;

const tl = gsap.timeline();

// Phase 1: Title reveal
tl.from(mainTitle, {
  autoAlpha: 0,
  y: 100
});

// Phase 2: Interleaved dots + text reveals
dotContainers.forEach((e, i) => {
  const dots = gsap.utils.toArray(".dot", e);
  tl.from(dots, {
    x: 50,
    autoAlpha: 0,
    stagger: 0.05
  });
  if (i < contentTextsAmount) {
    // SplitTextAnimator wraps autoSplit + onSplit for responsive re-animation
    const split = SplitTextAnimator(contentTexts[i], {
      type: "lines, words"
    });
    tl.add(
      split.animate((self) => {
        return gsap.from(self.words, {
          y: -100,
          autoAlpha: 0,
          stagger: {
            amount: 0.5,
            from: "random" // Random stagger creates organic feel
          }
        });
      })
    );
  }
});

// Debounced resize: hide content, wait for re-split, restart timeline
const resizeTimer = gsap
  .delayedCall(0.2, () => {
    gsap.set(".content", { autoAlpha: 1 });
    tl.restart();
  })
  .pause();
window.addEventListener("resize", () => {
  gsap.set(".content", { autoAlpha: 0 });
  resizeTimer.restart(true);
});

gsap.set(".content", { autoAlpha: 1 });

GSDevTools.create({ animation: tl });

/**
 * SplitTextAnimator — reusable helper that creates a single SplitText instance
 * and allows multiple animations to subscribe. All animations auto-update on re-split.
 *
 * Usage:
 *   let split = SplitTextAnimator(target, {type: "words,lines"});
 *   tl.add(split.animate((self) => gsap.to(self.words, {...})));
 *   tl.add(split.animate((self) => gsap.to(self.lines, {...})));
 */
function SplitTextAnimator(target, config) {
  let originalOnSplit = config.onSplit,
    subscribers = [],
    self = SplitText.create(target, {
      ...config,
      autoSplit: true,
      onSplit(self) {
        // Notify all subscribed animations to rebuild
        subscribers.forEach((f) => f(self));
        originalOnSplit && originalOnSplit(self);
      }
    });
  self.animate = (create) => {
    let animation,
      onSplit = (self) => {
        let parent, startTime;
        if (animation) {
          // Preserve position in parent timeline before killing
          parent = animation.parent;
          startTime = animation.startTime();
          animation.kill();
        }
        animation = create && create(self);
        // Re-insert at same position in parent timeline
        parent && parent.add(animation, startTime || 0);
      };
    subscribers.push(onSplit);
    onSplit(self);
    return animation;
  };
  return self;
}
```

## Adaptation Notes

- The `SplitTextAnimator` helper is the key reusable pattern here — extract it as a utility for any project that needs responsive text animations.
- Replace the dot decorations with any interstitial elements (icons, dividers, etc.) while keeping the interleaved timeline structure.
- The debounced resize pattern (hide -> wait -> restart) prevents flash-of-unstyled-text during resize. Adjust the 0.2s delay based on your layout complexity.
- In frameworks (Vue/React/Nuxt), call `SplitTextAnimator` inside `onMounted`/`useEffect` and kill the timeline on unmount.
- Remove `GSDevTools.create()` for production — it is a development-only debugging tool.
