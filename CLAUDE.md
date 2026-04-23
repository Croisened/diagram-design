# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo is **not application code** — it ships a single Claude Code skill (`diagram-design`) that teaches Claude how to generate editorial-quality diagrams as self-contained HTML files with inline SVG. The same tree is published three ways: as a standalone skill (symlinked into `~/.claude/skills/`), as a Claude Code plugin (via `.claude-plugin/`), and as a Codex plugin (via `.codex-plugin/`). All three point at the inner `skills/diagram-design/` directory.

There is no build, no test runner, no lint, no package manager. "Running" a diagram means opening its `.html` file in a browser — the gallery is [skills/diagram-design/assets/index.html](skills/diagram-design/assets/index.html).

## Architecture: progressive disclosure

The skill is designed so Claude's working context stays tight regardless of how many diagram types exist. Understanding this flow is critical before editing:

- **[skills/diagram-design/SKILL.md](skills/diagram-design/SKILL.md)** is the only file always in context. It holds the philosophy, the type-selection table, the universal anti-patterns, the 4px-grid layout rules, and the pre-output taste gate.
- **[skills/diagram-design/references/](skills/diagram-design/references/)** holds one `type-<name>.md` per diagram type, loaded *only* when that type is chosen. Adding a new type = dropping a new `type-<name>.md` + one row in SKILL.md's selection table. Nothing else changes.
- **[skills/diagram-design/references/style-guide.md](skills/diagram-design/references/style-guide.md)** is the single source of truth for colors + fonts. Every `type-*.md` and every generated diagram references semantic role names (`accent`, `ink`, `paper`, `muted`) — never raw hex. When editing anywhere downstream, preserve this indirection.
- **[skills/diagram-design/references/onboarding.md](skills/diagram-design/references/onboarding.md)** defines the URL-to-tokens flow that rewrites `style-guide.md` from a user's website.
- **[skills/diagram-design/assets/](skills/diagram-design/assets/)** holds three variants per type (`example-<type>.html`, `example-<type>-dark.html`, `example-<type>-full.html`) plus three matching templates. The gallery `index.html` tabs between them.

Because SKILL.md is always loaded and everything else is lazy-loaded, the skill stays fast even at 15+ reference files. Preserve this property — don't inline type-specific content into SKILL.md, and don't cross-link between `type-*.md` files when a pointer back to SKILL.md would do.

## First-run style-guide gate

Before generating any diagram in a new project, the skill is required to check whether `style-guide.md` still holds the shipped default (accent `#b5523a`). If it does, it must pause and ask the user whether to run onboarding, paste tokens manually, or proceed with defaults. This gate exists to prevent silently shipping default-skinned diagrams into a branded project — see [SKILL.md §0](skills/diagram-design/SKILL.md). Keep the gate behavior in sync across SKILL.md, onboarding.md, and style-guide.md if you touch any of them.

## Non-negotiable design rules

These are the rules the skill is built to enforce — if you edit examples, templates, or references, don't break them:

- **4px grid.** Every font size, coord, width, height, gap must be divisible by 4. Exempt: stroke widths, opacity, the 22×22 dot pattern. SKILL.md §7 has the allowed-values tables.
- **Complexity budget.** Per-type caps (e.g. max 9 nodes, 12 arrows, 2 coral accents, 5 lanes). Exceeding means split into two diagrams. See SKILL.md §7.
- **Focal rule.** `accent` goes on 1–2 elements max per diagram. Not a signaling system.
- **Font roles.** Instrument Serif = titles + italic callouts. Geist sans = human-readable node names. Geist Mono = technical sublabels (ports, URLs, field types) — never a blanket "dev" font. No JetBrains Mono anywhere.
- **Self-contained output.** Every generated diagram is one `.html` file: embedded CSS, inline SVG, no JS, no external images. Only external dependency allowed is Google Fonts.
- **Arrows before boxes.** Z-order matters. Arrow labels always need an opaque masking rect behind them. Legends are horizontal bottom strips, never floating inside the diagram area.

The full pre-output checklist lives at [SKILL.md §9](skills/diagram-design/SKILL.md). When producing or reviewing diagrams, run it.

## Working on this repo

- **Editing a type's conventions**: change `skills/diagram-design/references/type-<name>.md` and update the matching `example-<type>*.html` files in `assets/` so the gallery stays truthful.
- **Adding a new diagram type**: (1) new `type-<name>.md`, (2) new row in the SKILL.md selection table + a legend entry wherever relevant, (3) three new `example-<name>*.html` variants, (4) wire it into `assets/index.html`.
- **Changing colors or fonts**: edit `references/style-guide.md` only. Everything downstream reads semantic roles from there.
- **Verifying changes**: open the affected `example-*.html` (or `index.html`) in a browser. There is no automated test.

## Install paths (for context when debugging user reports)

- Standalone skill: `~/.claude/skills/diagram-design` symlinked to `<repo>/skills/diagram-design`.
- Claude Code / Codex plugin: installed via `/plugin install diagram-design@diagram-design`; lives in the plugin cache, so edits to `references/style-guide.md` don't survive plugin updates. The clone+symlink route is what users pick when they want to customize the style guide by hand.
