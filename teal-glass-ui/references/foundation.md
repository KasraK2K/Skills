# Foundation

Use this reference for any implementation. Adapt it to the project's existing architecture instead of treating it as a replacement template.

## Adoption modes

Choose one mode from the user's scope:

- **Full dark system:** apply the tokens at the application root and make Teal Glass the product's visual language.
- **Scoped experience:** attach the variables to a wrapper such as `.teal-glass-theme`; preserve the rest of the product and its light/dark behavior.
- **Single component:** use only the relevant tokens and effects. Do not add a star field, new fonts, or global theme changes for one card.
- **Review:** compare the rendered interface against this reference and report concrete mismatches before editing.

Do not remove `next-themes`, theme toggles, or existing token sets unless a full dark-only conversion is explicitly requested. If the current product supports multiple themes, introduce a scoped dark theme or map these semantic roles into its theme architecture.

## Core tokens

Keep semantic names in the target project where possible. These values are the canonical visual baseline:

```css
:root,
.teal-glass-theme {
  color-scheme: dark;
  --background: oklch(0.17 0.018 200);
  --foreground: oklch(0.95 0.01 190);
  --card: oklch(0.32 0.04 195 / 0.28);
  --card-foreground: var(--foreground);
  --popover: oklch(0.22 0.03 198 / 0.94);
  --popover-foreground: var(--foreground);
  --primary: oklch(0.85 0.08 80);
  --primary-foreground: oklch(0.2 0.02 80);
  --secondary: oklch(0.35 0.04 195 / 0.45);
  --secondary-foreground: var(--foreground);
  --muted: oklch(0.32 0.03 195 / 0.4);
  --muted-foreground: oklch(0.74 0.02 195);
  --accent: oklch(0.38 0.05 190 / 0.5);
  --accent-foreground: var(--foreground);
  --destructive: oklch(0.72 0.17 30);
  --border: oklch(1 0 0 / 9%);
  --input: oklch(1 0 0 / 12%);
  --ring: oklch(0.85 0.08 80);
  --chart-1: oklch(0.85 0.08 80);
  --chart-2: oklch(0.76 0.11 355);
  --chart-3: oklch(0.74 0.12 290);
  --chart-4: oklch(0.85 0.12 160);
  --chart-5: oklch(0.74 0.07 185);
  --radius: 1rem;
}
```

Canvas hex values for APIs that do not accept OKLCH:

| Role | Hex | Meaning |
|---|---:|---|
| Gold | `#e9c98b` | primary, selected, focus, graph center |
| Rose | `#ec8fac` | highlight, hub, replied |
| Lavender | `#a99df2` | secondary category, contacted, sent |
| Mint | `#7fe3b8` | success, active, ready |
| Teal | `#5fb8ad` | neutral, new, draft |
| Coral | `#ff7a6b` | error, failed, destructive |

Map status by meaning, not by the source color. Preserve the product's established status language if it already has one.

## Surface hierarchy

Use at most three visually distinct layers:

1. **Canvas:** opaque teal-black ground and broad radial light.
2. **Glass surface:** cards, bars, and rails that benefit from seeing the canvas behind them.
3. **Popover surface:** near-opaque menus, sheets, and detail panels where readability matters more than translucency.

```css
.teal-glass-canvas {
  color: var(--foreground);
  background:
    radial-gradient(60rem 40rem at 50% -10%, oklch(0.45 0.07 190 / 0.35), transparent 70%),
    radial-gradient(50rem 40rem at -10% 110%, oklch(0.4 0.06 185 / 0.3), transparent 70%),
    var(--background);
}

.glass {
  background: var(--card);
  border: 1px solid var(--border);
  backdrop-filter: blur(20px) saturate(140%);
  box-shadow:
    inset 0 1px 0 rgb(255 255 255 / 0.06),
    0 10px 40px -12px rgb(0 0 0 / 0.6);
}

.glass-popover {
  background: var(--popover);
  border: 1px solid var(--border);
  box-shadow:
    inset 0 1px 0 rgb(255 255 255 / 0.06),
    0 18px 60px -18px rgb(0 0 0 / 0.72);
}

.glow-gold {
  box-shadow:
    0 0 0 1px oklch(0.85 0.08 80 / 0.6),
    0 0 24px -4px oklch(0.85 0.08 80 / 0.55);
}
```

Avoid nested backdrop blur. A glass card inside another glass card should usually become a low-opacity fill with a hairline, not another blur layer. If `backdrop-filter` is unavailable or too expensive, use an opaque teal surface while retaining the border and highlight.

The optional star field belongs on immersive canvases, auth screens, or graph views. Skip it on dense administrative pages when it competes with data.

## Typography and spacing

Use the project's font when it already has a strong identity. Otherwise:

- UI: Geist or a neutral modern sans.
- IDs, timestamps, and technical values: Geist Mono or the existing monospace.
- Canvas labels: Roboto Condensed or the existing condensed face.

Scale:

- Page title: 18–20px, medium, tight tracking.
- Card title: 15–16px, medium.
- Body: 14px; supporting text: 12–13px.
- Eyebrow: 11px, medium, uppercase, 0.12em tracking.
- KPI: 40–56px, weight 300, tight tracking, tabular numerals.
- Tile value: 24px, weight 300, tabular numerals.

Use a 4px spacing base, 16px card radius, 20–24px floating-panel radius, full pills, and 12px icon-button radius. Keep 8px inside tight control clusters and 16–24px between content groups.

## Background and focus

If adding a star layer, put it behind content, set `pointer-events: none`, and keep opacity around 0.2–0.35. Do not change the positioning of every direct body child; use an explicit app wrapper with an isolation context when possible.

```css
.teal-glass-root {
  position: relative;
  isolation: isolate;
  min-height: 100svh;
}

.teal-glass-root::before {
  content: "";
  position: fixed;
  inset: 0;
  z-index: -1;
  pointer-events: none;
  opacity: 0.28;
  background-image:
    radial-gradient(1px 1px at 20% 30%, white 50%, transparent 51%),
    radial-gradient(1px 1px at 70% 80%, white 50%, transparent 51%),
    radial-gradient(1px 1px at 40% 60%, oklch(0.85 0.12 160) 50%, transparent 51%);
  background-size: 340px 340px, 460px 460px, 580px 580px;
}

:where(button, a, input, select, textarea):focus-visible {
  outline: 2px solid var(--ring);
  outline-offset: 2px;
}
```

Do not infer WCAG conformance from the palette. Test text over the actual composed surface because blur and content behind glass change effective contrast.

## Motion

- Entrances: 160–240ms fade plus 4–8px translation.
- Live state: a slow 2.4s opacity/glow pulse.
- Selection: shadow or ring transition, not scaling that shifts layout.
- Graph animation: stop when the simulation and state are idle.

```css
@keyframes pulse-glow {
  0%, 100% { opacity: 1; filter: drop-shadow(0 0 0 currentColor); }
  50% { opacity: 0.72; filter: drop-shadow(0 0 6px currentColor); }
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

Reduced motion should preserve state and meaning, not simply hide content.

## Tailwind and shadcn adaptation

For Tailwind CSS v4, map the project's semantic theme variables to these values and expose `glass` and `glow-gold` as utilities. Do not replace a generated `@theme` block blindly.

For shadcn/ui:

- Keep component APIs and variant names stable.
- Adjust semantic classes in the project's existing component files.
- Preserve generated accessibility behavior, focus handling, and data attributes.
- Use `TooltipProvider` when the chosen sidebar implementation requires it.
- Avoid depending on a specific shadcn release's exact source string; inspect the installed components first.
