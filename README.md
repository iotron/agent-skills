# iotron Agent Skills

[![GitHub stars](https://img.shields.io/github/stars/iotron/agent-skills?style=social)](https://github.com/iotron/agent-skills)

Reusable Claude Code skills for GSAP animation in Vue/Nuxt and React projects.

> If these skills save you time, [star the repo](https://github.com/iotron/agent-skills) — it helps others find it and motivates updates.

## Skills (9)

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

## Installation

```bash
# Install this plugin
claude /plugin install iotron/agent-skills

# Also install the official GSAP skills (API reference)
claude /plugin install greensock/gsap-skills
```

## Companion: Official GSAP Skills

Designed to work alongside [greensock/gsap-skills](https://github.com/greensock/gsap-skills):

| This plugin | Official GSAP skills |
|-------------|---------------------|
| **How** to use GSAP well (patterns, optimisation) | **What** GSAP can do (API reference) |
| Vue/Nuxt + React focused | All frameworks |
| Real codebase patterns & gotchas | Complete API coverage |
| Anti-patterns & audit checklists | Plugin-by-plugin docs |

## Contributing

Found a bug or have a pattern to add? [Open an issue](https://github.com/iotron/agent-skills/issues) or PR.

If you're using these skills in your projects, drop a star — it takes 2 seconds and helps the community find quality GSAP tooling.

## License

MIT
