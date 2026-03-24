# Agent Skills — CLAUDE.md

## Project Overview

Claude Code plugin providing 12 GSAP animation skills for Vue 3 / Nuxt 3 and React projects. Published as `iotron-agent-skills` on the Claude Code marketplace.

- **Repository**: `iotron/agent-skills`
- **Plugin name**: `iotron-agent-skills`
- **Install**: `/plugin marketplace add iotron/agent-skills` → `/plugin install iotron-agent-skills`
- **Invoke**: `/iotron-agent-skills:<skill-name>` (e.g., `/iotron-agent-skills:gsap-animate`)

## Architecture

### Skill Flow

```
gsap-setup → gsap-animate → gsap-optimise → gsap-test
                  ↓ dispatches to:
    gsap-scroll | gsap-interact | gsap-cursor | gsap-text | gsap-svg | gsap-vfx | gsap-canvas | gsap-showcase
```

### Progressive Disclosure (3-tier)

1. **YAML frontmatter** — always loaded (~100 words). Contains triggers, non-triggers, expected outcome.
2. **SKILL.md body** — loaded on activation. **Must stay under 200 lines.**
3. **`references/` files** — loaded only when a specific pattern is needed.

### Companion Plugin Relationship

This plugin is cookbook-only — it NEVER duplicates API reference from greensock/gsap-skills. Each skill intrinsically invokes official GSAP skills for API context via companion directives. When adding content, ask: "Is this a recipe or API docs?" — only recipes belong here.

### Directory Structure

```
.claude-plugin/
  marketplace.json    # Marketplace listing metadata
  plugin.json         # Plugin manifest (name, version, keywords)
skills/
  gsap-animate/       # Core orchestrator — dispatches to sub-skills
    SKILL.md
    references/
      vue-examples.md
  gsap-scroll/        # ScrollTrigger patterns
  gsap-interact/      # Mouse-driven animations
  gsap-text/          # SplitText, ScrambleText
  gsap-svg/           # DrawSVG, morph, circuit
  gsap-vfx/           # Glitch, marquee, counters, floating
  gsap-optimise/      # Performance audit & patterns
  gsap-test/          # Testing & debugging
  gsap-cursor/        # Cursor followers, trails, magnetic, spotlight
  gsap-canvas/        # Canvas + GSAP rendering pipeline
  gsap-showcase/      # Premium multi-plugin animation compositions
  gsap-setup/         # Project setup & configuration
```

## Skill Authoring Rules

### SKILL.md Format

```yaml
---
name: gsap-<name>
description: >
  One-paragraph description.
  Triggers: comma-separated context triggers.
  Non-triggers: what this skill does NOT handle (point to correct skill).
  Outcome: what the skill produces.
---
```

- Frontmatter `description` drives auto-invocation — be specific about triggers
- Include `Non-triggers` to prevent incorrect dispatch
- Include `Outcome` so Claude knows what success looks like

### Content Rules

- **200-line max** for SKILL.md body (excluding frontmatter)
- Use `references/` for full implementations, deep dives, production learnings
- Reference files via relative path: `See [references/vue-examples.md](references/vue-examples.md)`
- All code examples must be real patterns from production codebases, not hypothetical
- Mark custom patterns (e.g., `.reveal`, `.text-reveal` CSS classes) as **custom, not GSAP-native**

### Key Conventions

- Vue 3 Composition API only (no Options API)
- Always use `gsap.context()` for cleanup in components
- `autoAlpha` over `opacity` (handles `visibility: hidden` automatically)
- `force3D: true` for GPU compositing on animated elements
- `will-change` lifecycle: set on animation start, release on complete
- Accessibility: always include `prefers-reduced-motion` handling

## Commit Convention

Follow existing pattern: `type: description`

- `fix:` — corrections to skill content or configuration
- `refactor:` — structural changes (line count, reorganization)
- `docs:` — README or non-skill documentation changes
- `feat:` — new skills or major new patterns

## Known Decisions

- **No symlinks**: Previously used `.claude/skills/` symlinks for short names but removed them to prevent duplicate loading. Plugin namespaced invocation only.
- **Companion plugin**: Designed to work alongside `greensock/gsap-skills` (official API reference). This plugin covers *how* (patterns), official covers *what* (API).
- **`strict: false`** not set in plugin.json — skills can be auto-invoked by context matching.
