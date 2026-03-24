# Free For All — Flagship Multi-Plugin Showcase

**Source:** [CodePen](https://codepen.io/GreenSock/pen/OPPjMQV) by GSAP
**Plugins:** SplitText, Physics2DPlugin, MotionPathPlugin, DrawSVGPlugin, CustomBounce, CustomWiggle, GSDevTools
**Category:** interactive-scenes

## What Makes It Premium

This is the GSAP flagship showcase — a single coordinated timeline that composes seven different plugins into one seamless animation. The timing layers are masterful: text characters reveal with random offsets while a 3D poly drops with custom bounce physics, confetti explodes with Physics2D, a paper plane flies along a motion path, and SVG paths draw on simultaneously. The use of CustomBounce with squash deformation and CustomWiggle for the hand wave adds physicality that standard easing cannot achieve.

## Orchestration

The master timeline uses **labels** (`"explode"`, `"flight"`) as sync points for parallel animation groups:

1. **Phase 0 (0s):** SplitText chars reveal with random y-offset and rotation
2. **Phase 0.5s:** 3D poly drops with CustomBounce + squash deformation (scaleX/scaleY)
3. **Label "explode" (1s):** SVG bang/spin/sprinkles scale in with `back.out`, confetti images explode via Physics2D (velocity, angle, gravity), DrawSVG draws the first path, text labels slide apart with elastic ease
4. **Label "flight" (1.3s):** Plane appears, second SVG path draws, plane follows MotionPath with autoRotate, then fades
5. **Hand (1.3-1.5s):** Hand slides up then wiggles with CustomWiggle

Click replays the entire sequence from the start via `tl.play(0)`.

## Code

### HTML
```html
<div class="container" role="image" aria-label="a big colourful confetti explosion and text saying free for all">
  <span id="free">free </span>
  <svg id="explode" viewBox="0 0 2058 871" fill="none" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    <g id="GSAP_GTM_blog_hero_2400x1260">
      <g id="bang">
        <g id="Vector">
          <path d="M833.956 626.101L747.149 801.943C745.505 805.265 749.467 808.518 752.373 806.239L932.942 664.473C934.689 663.099 937.25 663.708 938.201 665.725L981.674 758.526C983.006 761.361 987.106 761.117 988.092 758.143L1032.59 623.926C1033.4 621.474 1036.49 620.743 1038.31 622.569L1166.26 751.22C1169.02 754.003 1173.56 750.733 1171.79 747.202L1087.08 576.839C1085.69 574.073 1088.34 570.995 1091.26 571.986L1216.42 613.942C1219.88 615.107 1222.55 610.776 1219.97 608.184L1087.3 474.785C1085.49 472.959 1086.21 469.845 1088.65 469.027L1210.09 428.324C1213.81 427.072 1212.93 421.54 1209 421.54H1089.81C1087.04 421.54 1085.4 418.444 1086.94 416.13L1174.11 284.645C1176.08 281.67 1172.86 278 1169.68 279.6L988.922 370.47C987.036 371.409 984.753 370.47 984.096 368.452L939.654 234.374C938.599 231.209 934.136 231.209 933.098 234.374L889.158 366.921C888.345 369.374 885.248 370.104 883.432 368.278L798.528 282.905C796.019 280.383 791.833 282.818 792.732 286.262L825.998 411.382C826.777 414.304 823.75 416.757 821.068 415.382L662.071 333.176C658.56 331.367 655.256 335.924 658.041 338.725L785.432 466.818C787.248 468.645 786.522 471.758 784.083 472.576L662.642 513.279C658.923 514.532 659.805 520.063 663.732 520.063H787.629C790.431 520.063 792.075 523.229 790.466 525.542L697.414 659.22C695.493 661.968 698.175 665.586 701.34 664.508L829.752 621.265C832.676 620.273 835.323 623.335 833.956 626.118V626.101Z" fill="url(#paint0_linear_6028_4729)" />
          <path d="M833.956 626.101L747.149 801.943C745.505 805.265 749.467 808.518 752.373 806.239L932.942 664.473C934.689 663.099 937.25 663.708 938.201 665.725L981.674 758.526C983.006 761.361 987.106 761.117 988.092 758.143L1032.59 623.926C1033.4 621.474 1036.49 620.743 1038.31 622.569L1166.26 751.22C1169.02 754.003 1173.56 750.733 1171.79 747.202L1087.08 576.839C1085.69 574.073 1088.34 570.995 1091.26 571.986L1216.42 613.942C1219.88 615.107 1222.55 610.776 1219.97 608.184L1087.3 474.785C1085.49 472.959 1086.21 469.845 1088.65 469.027L1210.09 428.324C1213.81 427.072 1212.93 421.54 1209 421.54H1089.81C1087.04 421.54 1085.4 418.444 1086.94 416.13L1174.11 284.645C1176.08 281.67 1172.86 278 1169.68 279.6L988.922 370.47C987.036 371.409 984.753 370.47 984.096 368.452L939.654 234.374C938.599 231.209 934.136 231.209 933.098 234.374L889.158 366.921C888.345 369.374 885.248 370.104 883.432 368.278L798.528 282.905C796.019 280.383 791.833 282.818 792.732 286.262L825.998 411.382C826.777 414.304 823.75 416.757 821.068 415.382L662.071 333.176C658.56 331.367 655.256 335.924 658.041 338.725L785.432 466.818C787.248 468.645 786.522 471.758 784.083 472.576L662.642 513.279C658.923 514.532 659.805 520.063 663.732 520.063H787.629C790.431 520.063 792.075 523.229 790.466 525.542L697.414 659.22C695.493 661.968 698.175 665.586 701.34 664.508L829.752 621.265C832.676 620.273 835.323 623.335 833.956 626.118V626.101Z" fill="url(#pattern)" fill-opacity="0.6" />
        </g>
      </g>

      <g id="ffd">
        <path d="M906.652 763.429C906.652 760.441 910.072 758.744 912.451 760.552L947.419 787.123C949.322 788.569 949.322 791.431 947.419 792.877L912.451 819.447C910.072 821.255 906.653 819.559 906.652 816.571V797.522L877.799 819.447C875.42 821.255 872 819.559 872 816.571V763.429C872 760.441 875.42 758.744 877.799 760.552L906.652 782.476V763.429Z" fill="#05F34A" />
        <path fill="url(#pattern)" fill-opacity="0.6" style="mix-blend-mode:multiply" d="M906.652 763.429C906.652 760.441 910.072 758.744 912.451 760.552L947.419 787.123C949.322 788.569 949.322 791.431 947.419 792.877L912.451 819.447C910.072 821.255 906.653 819.559 906.652 816.571V797.522L877.799 819.447C875.42 821.255 872 819.559 872 816.571V763.429C872 760.441 875.42 758.744 877.799 760.552L906.652 782.476V763.429Z" />
      </g>
      <g id="wiggle">
        <g>
          <path d="M890.757 732.409C873.509 732.409 860.512 731.11 849.85 728.297C819.605 720.346 810.296 700.231 807.443 688.947C803.899 674.95 804.562 653.594 827.861 631.243C835.873 623.566 846.55 615.514 861.463 605.933C885.771 590.32 917.861 572.903 951.823 554.448C971.001 544.03 991.981 532.645 1011.78 521.346C1004.27 522.226 996.592 523.15 988.797 524.088C935.252 530.509 874.576 537.782 822.66 537.782C798.308 537.782 779.49 535.891 765.124 532.01C734.288 523.669 722.689 505.488 718.41 491.708C714.894 480.438 712.17 457.913 733.381 433.671C740.254 425.807 749.36 417.986 761.219 409.79C762.631 408.809 765.095 407.15 768.438 404.899C773.928 401.19 782.069 395.693 791.824 388.983L760.556 352.649L793.899 323.847C795.037 322.866 821.997 299.649 853.855 278.74C873.826 265.638 891.91 255.97 907.616 250.011C936.592 239.001 961.347 239.347 981.189 251.006C996.866 260.227 1006.56 276.489 1007.13 294.497C1008.09 325.319 984.892 354.496 918.509 406.01C899.1 421.074 878.812 435.764 861.017 448.274C898.956 445.994 940.497 441.002 978.307 436.471C1016.04 431.94 1051.68 427.669 1079.4 426.558C1096.55 425.879 1109.59 426.399 1120.44 428.231C1149.42 433.094 1162.38 448.318 1168.16 460.236C1175.44 475.243 1175.11 492.415 1167.24 508.576C1163.1 517.061 1156.75 525.545 1147.83 534.477C1120.03 562.312 1065.06 593.235 1009.37 623.566C1051.52 609.324 1078.59 592.86 1079.07 592.571L1078.97 592.643L1125.61 667.519C1121.37 670.174 1020.25 732.395 890.771 732.395L890.757 732.409Z" fill="url(#paint1_linear_6028_4729)" />
          <path d="M890.757 732.409C873.509 732.409 860.512 731.11 849.85 728.297C819.605 720.346 810.296 700.231 807.443 688.947C803.899 674.95 804.562 653.594 827.861 631.243C835.873 623.566 846.55 615.514 861.463 605.933C885.771 590.32 917.861 572.903 951.823 554.448C971.001 544.03 991.981 532.645 1011.78 521.346C1004.27 522.226 996.592 523.15 988.797 524.088C935.252 530.509 874.576 537.782 822.66 537.782C798.308 537.782 779.49 535.891 765.124 532.01C734.288 523.669 722.689 505.488 718.41 491.708C714.894 480.438 712.17 457.913 733.381 433.671C740.254 425.807 749.36 417.986 761.219 409.79C762.631 408.809 765.095 407.15 768.438 404.899C773.928 401.19 782.069 395.693 791.824 388.983L760.556 352.649L793.899 323.847C795.037 322.866 821.997 299.649 853.855 278.74C873.826 265.638 891.91 255.97 907.616 250.011C936.592 239.001 961.347 239.347 981.189 251.006C996.866 260.227 1006.56 276.489 1007.13 294.497C1008.09 325.319 984.892 354.496 918.509 406.01C899.1 421.074 878.812 435.764 861.017 448.274C898.956 445.994 940.497 441.002 978.307 436.471C1016.04 431.94 1051.68 427.669 1079.4 426.558C1096.55 425.879 1109.59 426.399 1120.44 428.231C1149.42 433.094 1162.38 448.318 1168.16 460.236C1175.44 475.243 1175.11 492.415 1167.24 508.576C1163.1 517.061 1156.75 525.545 1147.83 534.477C1120.03 562.312 1065.06 593.235 1009.37 623.566C1051.52 609.324 1078.59 592.86 1079.07 592.571L1078.97 592.643L1125.61 667.519C1121.37 670.174 1020.25 732.395 890.771 732.395L890.757 732.409Z" fill="url(#pattern1_6028_4729)" fill-opacity="0.6" />
        </g>
        <g fill="url(#pattern)" style="mix-blend-mode:multiply">
          <path d="M890.757 732.409C873.509 732.409 860.512 731.11 849.85 728.297..." />
          <!-- Duplicate wiggle paths with noise pattern overlay -->
        </g>
      </g>
      <rect id="box" x="539" y="469.42" width="87.0018" height="80.7347" transform="rotate(-83.3705 539 469.42)" fill="url(#pattern)" />
      <path class="sprinkle" d="M733.022 222.569L709.927 203.163..." fill="#397DFF" />
      <path id="path" d="M538.5 556.481C565.165 497.803..." stroke="#FEC5FB" stroke-width="12" stroke-miterlimit="10" stroke-linecap="round" />
      <g id="spin">
        <path d="M1122.34 375.405..." fill="url(#paint2_linear_6028_4729)" />
      </g>
      <path class="sprinkle" d="M1172.43 119.19..." stroke="#FF783E" stroke-width="12" stroke-linecap="round" />
      <path class="sprinkle" d="M1178.57 43.2882..." fill="#BAA5F5" stroke="black" stroke-width="3" />
      <circle class="sprinkle" cx="783.75" cy="239.751" r="17.25" fill="#FAF005" />
      <circle class="sprinkle" cx="1207.5" cy="336.501" r="28.5" fill="#FAF005" />
      <g id="plane">
        <g class="innerplane">
          <!-- Paper plane SVG paths -->
        </g>
      </g>
      <path id="path_2" d="M973.861 226.794..." stroke="#0AE448" stroke-width="3" />
      <path class="sprinkle" d="..." fill="#BAA5F5" stroke="#1B1E1A" />
      <!-- Additional sprinkle decorations -->
      <g id="hand">
        <!-- Waving hand SVG group -->
      </g>
      <g id="main">
        <image href="https://assets.codepen.io/16327/3D-poly.png" x="758" y="320" width="344" height="370" />
      </g>
    </g>
    <defs>
      <linearGradient id="paint0_linear_6028_4729" x1="341.328" y1="165.489" x2="1348.16" y2="553.463" gradientUnits="userSpaceOnUse">
        <stop offset="0.427083" stop-color="#FF8709" />
        <stop offset="0.791667" stop-color="#F7BDF8" />
      </linearGradient>
      <!-- Additional gradients and patterns -->
      <pattern id="pattern" patternContentUnits="objectBoundingBox" width="0.625" height="0.625">
        <use xlink:href="#svg-noise" transform="scale(0.00125)"></use>
      </pattern>
      <image id="svg-noise" width="500" height="500" xlink:href="https://assets.codepen.io/16327/noise-e82662fe.png"></image>
    </defs>
  </svg>
  <span id="all">for all</span>
</div>

<div class="confetti">
  <img src="https://assets.codepen.io/16327/2D-circles.png" />
  <img src="https://assets.codepen.io/16327/2D-keyframe.png" />
  <img src="https://assets.codepen.io/16327/2D-lightning.png" />
  <img src="https://assets.codepen.io/16327/2D-star.png" />
  <img src="https://assets.codepen.io/16327/2D-flower.png" />
  <img src="https://assets.codepen.io/16327/3D-cone.png" />
  <img src="https://assets.codepen.io/16327/3D-spiral.png" />
  <img src="https://assets.codepen.io/16327/3D-spiral.png" />
  <img src="https://assets.codepen.io/16327/3D-tunnel.png" />
  <img src="https://assets.codepen.io/16327/3D-hoop.png" />
  <img src="https://assets.codepen.io/16327/3D-semi.png" />
  <img src="https://assets.codepen.io/16327/2D-circles.png" />
  <img src="https://assets.codepen.io/16327/2D-keyframe.png" />
  <img src="https://assets.codepen.io/16327/2D-lightning.png" />
  <img src="https://assets.codepen.io/16327/2D-star.png" />
  <img src="https://assets.codepen.io/16327/2D-flower.png" />
  <img src="https://assets.codepen.io/16327/3D-cone.png" />
  <img src="https://assets.codepen.io/16327/3D-spiral.png" />
  <img src="https://assets.codepen.io/16327/3D-spiral.png" />
  <img src="https://assets.codepen.io/16327/3D-tunnel.png" />
  <img src="https://assets.codepen.io/16327/3D-hoop.png" />
  <img src="https://assets.codepen.io/16327/3D-semi.png" />
</div>
```

### CSS
```css
body {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background-color: #0e100f;
}

#explode {
  width: 60vw;
  overflow: visible;
}

.container {
  position: relative;
  opacity: 0;
}

#free, #all {
  position: absolute;
  font-size: 8vw;
  top: 45%;
}

#free {
  left: 10%;
}

#all {
  left: 41%;
}

.confetti {
  position: fixed;
  left: 50%;
  transform: translateX(-50%);
  top: 50;
  height: 30px;
  width: 30px;
  z-index: -1;
}

.confetti img {
  width: 100%;
  max-width: 30px;
  position: absolute;
  opacity: 0;
}
```

### JavaScript
```js
gsap.registerPlugin(
  SplitText,
  DrawSVGPlugin,
  GSDevTools,
  MotionPathPlugin,
  MotionPathHelper
);

gsap.set("#plane", { opacity: 0 });
gsap.set(".container", { opacity: 1 });
gsap.set(".confetti img", { scale: "random(0.1, 1)", opacity: 0 });

// Custom eases — CustomBounce adds squash deformation, CustomWiggle creates organic shake
CustomBounce.create("myBounce", { strength: 0.6, squash: 3 });
CustomWiggle.create("myWiggle", { wiggles: 6 });

// SplitText with mask:"words" for per-character reveal within word boundaries
let free = SplitText.create("#free", { type: "words, chars", mask: "words" });
let all = SplitText.create("#all", { type: "words, chars", mask: "words" });

let tl = gsap.timeline({ delay: 1 });

// Labels as sync points for parallel animation groups
tl.addLabel("explode", 1);
tl.addLabel("flight", 1.3);

// Phase 1: Text character reveal with random direction
tl.from([free.chars, all.chars], {
  duration: 0.7,
  y: "random([-500, 500])",
  rotation: "random([-30, 30])",
  ease: "expo.out",
  stagger: {
    from: "random",
    amount: 0.3
  }
})
  // Phase 2: 3D poly drops with custom bounce + squash deformation
  .from("#main", { duration: 2, opacity: 0, y: -2000, ease: "myBounce" }, 0.5)
  .to(
    "#main",
    {
      duration: 2,
      scaleX: 1.4,
      scaleY: 0.6,
      ease: "myBounce-squash", // Paired squash ease from CustomBounce
      transformOrigin: "center bottom"
    },
    0.5
  )
  // Phase 3: Text labels slide apart with elastic ease at "explode" label
  .to("#free", { duration: 2, xPercent: -20, ease: "elastic.out(1,0.3)" }, "explode")
  .to("#all", { duration: 2, xPercent: 50, ease: "elastic.out(1,0.3)" }, "explode")
  // Phase 4: Plane + DrawSVG paths
  .set("#plane", { opacity: 1 }, "flight")
  .from("#path", { duration: 0.5, drawSVG: 0 }, "explode")
  .from("#path_2", { duration: 0.8, drawSVG: 0 }, "flight")
  // Phase 5: Plane flies along motion path with autoRotate
  .from(
    "#plane",
    {
      duration: 1,
      ease: "sine.inOut",
      scale: 0.2,
      transformOrigin: "center center",
      motionPath: {
        path: "M973.861,226.794 C1015.92,240.459 1041.39,136.212 1005.93,135.899...",
        align: "#path_2",
        alignOrigin: [0.5, 0.5],
        autoRotate: 180,
        start: 1,
        end: 0
      }
    },
    "flight"
  )
  .to(".innerplane", { duration: 0.2, opacity: 0 }, 2)
  // Phase 6: Confetti explosion with Physics2D
  .set(".confetti img", { opacity: 1 }, "explode+=.2")
  .to(
    ".confetti img",
    {
      duration: 2,
      rotation: "random(-360, 360)",
      scale: "random(0.5, 1)",
      physics2D: {
        velocity: "random(800, 2000)",
        angle: "random(150, 360)",
        gravity: 3000,
        acceleration: 100
      }
    },
    "explode+=.2"
  )
  // Phase 7: SVG decorative elements scale in
  .from("#wiggle", { duration: 0.7, transformOrigin: "center center", scale: 0, rotation: 60, ease: "back.out(4)" }, "explode+=.4")
  .from("#bang, #spin", { duration: 0.7, transformOrigin: "center center", scale: 0, rotation: -60, ease: "back.out(4)" }, "explode+=.1")
  .from(".sprinkle", { scale: 0, rotation: 360, transformOrigin: "center center", ease: "back.out" }, "explode")
  .from("#ffd", { xPercent: -800, opacity: 0, ease: "back.out" }, "explode")
  // Phase 8: Hand wiggle with CustomWiggle ease
  .from("#hand", { duration: 0.4, rotation: "+=30", ease: "myWiggle", transformOrigin: "center center" }, 1.5)
  .from("#hand", { opacity: 0, duration: 0.2, yPercent: 100 }, 1.3);

// Click to replay
document.body.addEventListener("click", (e) => {
  tl.play(0);
});
```

## Adaptation Notes

- This showcase demonstrates the **label-based orchestration pattern**: define labels for major phases, then attach animations to labels with offsets. This scales to any number of animation groups without absolute time calculations.
- The `CustomBounce.create()` + `-squash` paired eases are the key to natural drop physics. Without squash, bouncing objects look weightless.
- Physics2D confetti pattern: randomize velocity, angle, and scale on each particle; gravity pulls them down naturally. Works with any number of elements.
- MotionPath with `autoRotate` and `start/end` reversal makes the plane face its direction of travel. The path is an SVG `d` attribute value.
- For production, remove GSDevTools and MotionPathHelper. Consider splitting the timeline into named functions for maintainability.
- In frameworks, the click-to-replay pattern can be replaced with route transition hooks or intersection observers.
- CustomBounce, CustomWiggle, Physics2DPlugin, DrawSVGPlugin, and MotionPathHelper require GSAP Club membership or the "free for all" plugins now included with GSAP 3.12+.
