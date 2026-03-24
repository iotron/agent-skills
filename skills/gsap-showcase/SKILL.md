---
name: gsap-showcase
description: >
  Production-grade premium animation compositions from official GSAP showcases.
  Complete multi-plugin references for building beautiful, polished animations.
  Companion to official gsap-core, gsap-plugins, gsap-timeline skills (API reference).
  Triggers: premium animation, showcase, production animation, beautiful animation,
  hero animation, page transition, loading animation, immersive scroll, complete animation,
  polished animation, animation composition, multi-plugin animation.
  Non-triggers: Not for single-technique recipes (use gsap-scroll, gsap-text, etc.),
  not for API reference (use official gsap-skills), not for setup (use gsap-setup).
  Outcome: Provides a complete premium animation reference with orchestration notes,
  which gsap-animate then adapts using individual technique skills.
---

# GSAP Showcase — Premium Animation References

> **Companion**: For individual GSAP plugin APIs, invoke the relevant official skill
> (gsap-core, gsap-plugins, gsap-timeline, gsap-scrolltrigger, etc.).
> This skill covers **complete compositions** only.

---

## How to Use

This skill is the **taste layer** in a two-layer recipe system:

- **Layer 1 — Building blocks**: Individual technique skills (`gsap-scroll`, `gsap-text`,
  `gsap-cursor`, `gsap-vfx`, etc.) provide isolated, copy-paste patterns.
- **Layer 2 — Showcase** (this skill): Complete premium compositions that combine
  multiple plugins into polished, production-grade animations.

**Workflow for premium animations:**

1. **Check showcase** for a composition similar to the request.
2. **Study its orchestration** — plugin combination, timing, layering, responsive handling.
3. **Implement using technique skills** for each component (scroll triggers via `gsap-scroll`,
   text reveals via `gsap-text`, interactions via `gsap-interact`, etc.).

The showcase provides the *what* and *why*; technique skills provide the *how*.

---

## Categories

| Category | What | Example |
|---|---|---|
| `hero-sections` | Above-fold hero compositions | Text reveals + scroll + responsive |
| `scroll-experiences` | Immersive scroll-driven scenes | 3D carousels, pinned sequences |
| `interactive-scenes` | Complex interactive compositions | Multi-plugin showcases |
| `page-transitions` | Full-page transitions | Between-page animations |
| `loading-sequences` | Premium loading/reveal | Staged loading animations |

---

## Showcase Entry Format

Each reference file in `references/<category>/` contains:

| Section | Contents |
|---|---|
| **Source** | Original CodePen link and author credit |
| **Plugins Used** | GSAP plugins required (ScrollTrigger, SplitText, etc.) |
| **What Makes It Premium** | Key design qualities — timing, layering, polish |
| **Orchestration Notes** | How plugins combine, sequencing strategy, responsive approach |
| **Complete Code** | Full HTML + CSS + JS (production-ready reference) |
| **Adaptation Notes** | How to modify for different content, frameworks, breakpoints |

---

## Available Showcases

| Showcase | Category | Plugins | Reference |
|---|---|---|---|
| Responsive SplitText Animator | `hero-sections` | SplitText, GSDevTools | [responsive-splittext-animator](references/hero-sections/responsive-splittext-animator.md) |
| Responsive matchMedia | `hero-sections` | gsap.matchMedia() | [responsive-matchmedia](references/hero-sections/responsive-matchmedia.md) |
| Scrubbed Vertical Rolodex | `scroll-experiences` | ScrollTrigger | [scrubbed-rolodex](references/scroll-experiences/scrubbed-rolodex.md) |
| Free For All (Flagship) | `interactive-scenes` | SplitText, Physics2D, MotionPath, DrawSVG, CustomBounce, CustomWiggle | [free-for-all](references/interactive-scenes/free-for-all.md) |
| Complex SVG Timeline | `interactive-scenes` | Timeline labels, SVG attr, GSDevTools | [complex-svg-timeline](references/interactive-scenes/complex-svg-timeline.md) |
| Scrubbed Bento Gallery | `scroll-experiences` | ScrollTrigger, Flip | [scrubbed-bento-gallery](references/scroll-experiences/scrubbed-bento-gallery.md) |
| Card Stack | `scroll-experiences` | Flip | [card-stack](references/scroll-experiences/card-stack.md) |
| Curve Swipe | `scroll-experiences` | MorphSVG | [curve-swipe](references/scroll-experiences/curve-swipe.md) |
| Image Sequence | `scroll-experiences` | ScrollTrigger | [image-sequence](references/scroll-experiences/image-sequence.md) |
| Infinite Card Slider | `scroll-experiences` | ScrollTrigger, Draggable | [infinite-card-slider](references/scroll-experiences/infinite-card-slider.md) |

---

## Category References

- [`references/hero-sections/`](references/hero-sections/) — Above-fold hero compositions (2 showcases)
- [`references/scroll-experiences/`](references/scroll-experiences/) — Immersive scroll scenes (6 showcases)
- [`references/interactive-scenes/`](references/interactive-scenes/) — Multi-plugin interactive compositions (2 showcases)
- [`references/page-transitions/`](references/page-transitions/) — Full-page transitions *(planned)*
- [`references/loading-sequences/`](references/loading-sequences/) — Premium loading animations *(planned)*
