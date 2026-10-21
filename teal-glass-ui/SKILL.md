---
name: teal-glass-ui
description: "Build, restyle, or review interfaces with the Teal Glass UI system: teal-black depth, restrained frosted glass, champagne-gold emphasis, jewel status accents, thin-number KPIs, and optional glowing orb graphs. Use when the user explicitly asks for teal-glass-ui, a teal-and-gold glass interface, or an existing project already uses these tokens. Do not select it for a generic dashboard or dark UI unless the user chooses this visual direction."
metadata:
  version: "2.0.0"
---

# Teal Glass UI

Create quiet, information-dense interfaces on a teal-black canvas. Use translucent depth sparingly, reserve champagne gold for primary or selected states, use jewel colors semantically, and keep interaction behavior clearer than the visual effects.

## Start by understanding the project

Before editing:

1. Inspect the existing framework, styling system, components, tokens, routes, and verification commands.
2. Identify whether the user wants a full visual-system adoption, a focused component, a design review, or the orb graph.
3. Preserve the project's framework, behavior, data, accessibility, and responsive contracts. Reuse its components and tokens when practical.
4. Do not remove theme support, dependencies, or existing features merely because the reference implementation is dark-only. Make destructive design-system changes only when the user explicitly requests a full replacement.
5. If the request is ambiguous, apply the style only to the requested surface. Do not silently restyle unrelated pages.

## Visual signature

- Deep teal-black background with broad, low-contrast radial light; an optional sparse star field on immersive screens.
- Frosted teal glass with a thin light border, faint top highlight, and one soft shadow. Not every container needs glass.
- Champagne gold only for the primary action, selected item, focus ring, or graph center.
- Rose, lavender, mint, teal, and coral communicate categories and status. Color is never the only signal.
- Large light-weight tabular numbers, small tracked labels, compact body copy, and generous spacing around major data.
- Ambient motion only: short entrances, restrained live-state pulses, and no required hover-only information.

## Choose the relevant references

Read only what the task needs:

- Read [references/foundation.md](references/foundation.md) for tokens, typography, glass, backgrounds, motion, stack adaptation, and theming boundaries.
- Read [references/components.md](references/components.md) when building or restyling dashboards, shells, cards, tables, forms, chat, status, and floating controls.
- Read [references/orb-graph.md](references/orb-graph.md) only for canvas/network/hierarchy visualizations.
- Read [references/quality-checks.md](references/quality-checks.md) before finishing an implementation or audit.

Use the runnable vanilla examples only when they materially help:

- [examples/dashboard.html](examples/dashboard.html): dashboard composition.
- [examples/components.html](examples/components.html): tokens and component states.
- [examples/graph.html](examples/graph.html): orb renderer and graph overlays.
- [examples/glass.css](examples/glass.css): framework-neutral CSS implementation.

## Implementation workflow

1. Map the existing product's semantic roles to the Teal Glass tokens. Do not replace colors by visual similarity alone.
2. Establish foundation tokens and surface hierarchy before tuning individual components.
3. Keep one clear primary action per region. Use glass, glow, and saturated color as emphasis, not decoration.
4. Adapt recipes to the current stack. Prefer existing primitives over installing a parallel component system.
5. Maintain real content and state. Never invent KPI deltas, status, historical trends, or product claims to fill the design.
6. Make responsive behavior explicit. Dense desktop layouts must collapse intentionally rather than squeeze.
7. Verify the actual rendered UI at desktop and narrow viewport sizes, plus keyboard focus and reduced motion. Use the project's normal lint, typecheck, test, and build gates.

## Non-negotiable boundaries

- Never make color the only status indicator.
- Never hide required information behind hover.
- Never put dark `primary-foreground` text on translucent gold; reserve it for solid gold surfaces.
- Never layer blur on every nested surface; excessive glass destroys hierarchy and performance.
- Never ship a graph that animates continuously while idle.
- Never claim accessibility from token values alone; verify representative rendered states.
- Never overwrite a repository's theme or generated component files without first inspecting local conventions and the requested scope.

## Completion report

Summarize the surfaces changed, the main design decisions, responsive and accessibility behavior, and the verification performed. Mention any checks that could not run.
