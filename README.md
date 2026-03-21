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

### Step 1: Add the marketplace and install

```bash
/plugin marketplace add iotron/agent-skills
/plugin install iotron
/reload-plugins
```

Skills are now available as `iotron:gsap-animate`, `iotron:gsap-scroll`, etc.

### Step 2 (optional): Enable short names

By default, plugin skills are namespaced (`iotron:gsap-animate`). To use short names like `/gsap-animate`, symlink from the plugin cache into your project's skills directory:

```bash
# Run from your project root
CACHE=$(ls -d ~/.claude/plugins/marketplaces/iotron*/skills 2>/dev/null | head -1)
mkdir -p .claude/skills
for skill in "$CACHE"/gsap-*; do
  ln -sf "$skill" ".claude/skills/$(basename $skill)"
done
```

Now `/gsap-animate`, `/gsap-scroll`, etc. work directly. Updates via `/plugin marketplace update iotron` auto-propagate through the symlinks.

### Companion: Official GSAP Skills

Also install the official GSAP API reference skills:

```bash
/plugin marketplace add greensock/gsap-skills
/plugin install gsap-skills
```

| This plugin | Official GSAP skills |
|-------------|---------------------|
| **How** to use GSAP well (patterns, optimisation) | **What** GSAP can do (API reference) |
| Vue/Nuxt + React focused | All frameworks |
| Real codebase patterns & gotchas | Complete API coverage |
| Anti-patterns & audit checklists | Plugin-by-plugin docs |

## Updating

```bash
/plugin marketplace update iotron
/reload-plugins
```

If you used symlinks (Step 2), they auto-update — no action needed.

## Contributing

Found a bug or have a pattern to add? [Open an issue](https://github.com/iotron/agent-skills/issues) or PR.

If you're using these skills in your projects, drop a star — it takes 2 seconds and helps the community find quality GSAP tooling.

## License

MIT
