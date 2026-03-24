# iotron GSAP Cookbook

[![GitHub stars](https://img.shields.io/github/stars/iotron/agent-skills?style=social)](https://github.com/iotron/agent-skills)

Reusable Claude Code skills for premium GSAP animations — production recipes for any framework.

> If these skills save you time, [star the repo](https://github.com/iotron/agent-skills) — it helps others find it and motivates updates.

## Skills (12)

### Flow: setup → animate → optimise → test

| Skill | Purpose | Triggers when |
|-------|---------|---------------|
| **gsap-setup** | Project setup & config | Starting GSAP in Nuxt, Vue, React, or Next.js |
| **gsap-animate** | Core orchestrator | Any animation work — dispatches to sub-skills below |
| **gsap-scroll** | ScrollTrigger patterns | Scroll reveals, parallax, pin, stacking, batch |
| **gsap-interact** | Mouse-driven animation | Tilt cards, cursor followers, spotlight, magnetic buttons |
| **gsap-text** | Text animation | SplitText, ScrambleText, char split, elastic type |
| **gsap-svg** | SVG animation | DrawSVG, morph, circuit tree, path drawing |
| **gsap-vfx** | Visual effects | Glitch, marquee, counters, floating, pulse |
| **gsap-cursor** | Cursor-driven effects | Followers, trails, magnetic, spotlight, spring physics |
| **gsap-canvas** | Canvas + GSAP rendering | Particle systems, draw loops, canvas integration |
| **gsap-showcase** | Premium compositions | Complete multi-plugin animations for production |
| **gsap-optimise** | Performance audit | GPU, quickTo, force3D, anti-patterns, checklist |
| **gsap-test** | Testing & debug | Vitest, Playwright, markers, pre-launch checklist |

### How gsap-animate orchestrates sub-skills

| Building this? | Sub-skills loaded |
|---|---|
| Hero section | gsap-text + gsap-scroll + gsap-vfx |
| Services grid | gsap-scroll + gsap-interact |
| Circuit board | gsap-svg + gsap-interact |
| Cyber/terminal page | gsap-text + gsap-vfx + gsap-svg |
| Stats section | gsap-vfx + gsap-scroll |
| Premium hero | gsap-showcase + gsap-text + gsap-scroll + gsap-cursor |
| Interactive canvas | gsap-showcase + gsap-canvas + gsap-scroll |

### Progressive Disclosure

Skills follow a 3-tier loading pattern to minimise context window usage:

1. **YAML frontmatter** — always in context (~100 words per skill). Includes triggers, non-triggers, and expected outcome.
2. **SKILL.md body** — loaded when activated. Kept under 200 lines with concise patterns.
3. **`references/` files** — pulled in only when a specific pattern is needed. Contains full implementations, deep-dives, and production learnings.

## Prerequisites

Install the official GSAP skills first — this plugin provides production recipes that build on them:

```bash
/plugin marketplace add greensock/gsap-skills
/plugin install gsap-skills
/reload-plugins
```

| This plugin | Official GSAP skills |
|-------------|---------------------|
| **How** to build specific animations (recipes, patterns, gotchas) | **What** GSAP can do (API reference) |
| Any framework (Vue, Nuxt, React, Svelte, Astro, vanilla) | All frameworks |
| Production-tested multi-layer patterns | Complete plugin-by-plugin docs |
| Performance audit checklists | General performance guidance |

## Installation

```bash
/plugin marketplace add iotron/agent-skills
/plugin install iotron-gsap-cookbook
/reload-plugins
```

Skills auto-trigger based on context, or invoke directly:

```
/iotron-gsap-cookbook:gsap-animate
/iotron-gsap-cookbook:gsap-scroll
/iotron-gsap-cookbook:gsap-interact
/iotron-gsap-cookbook:gsap-text
/iotron-gsap-cookbook:gsap-svg
/iotron-gsap-cookbook:gsap-vfx
/iotron-gsap-cookbook:gsap-optimise
/iotron-gsap-cookbook:gsap-test
/iotron-gsap-cookbook:gsap-setup
/iotron-gsap-cookbook:gsap-cursor
/iotron-gsap-cookbook:gsap-canvas
/iotron-gsap-cookbook:gsap-showcase
```

## Updating

```bash
/plugin marketplace update iotron-gsap-cookbook
/reload-plugins
```

## Contributing

Found a bug or have a pattern to add? [Open an issue](https://github.com/iotron/agent-skills/issues) or PR.

## License

MIT
