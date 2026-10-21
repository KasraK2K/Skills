# Component Recipes

Use these as intent-level recipes. Match the project's existing component API and naming rather than creating duplicate primitives.

## Buttons

- **Primary:** solid or vertical champagne-gold gradient, dark text, soft gold halo. Keep one clear primary action per region.
- **Secondary/outline:** glass or translucent teal with foreground text.
- **Ghost:** transparent until hover/focus.
- **Destructive:** coral text or a restrained coral tint; require confirmation for destructive data actions.
- **Icon button:** 36–40px target with 12px radius and an accessible name.

Do not use `primary-foreground` on translucent gold. It is intended for a solid gold surface.

Example Tailwind-style variants:

```ts
const buttonTones = {
  primary:
    "bg-gradient-to-b from-[oklch(0.9_0.08_80)] to-[oklch(0.78_0.09_75)] text-primary-foreground shadow-[0_0_0_1px_oklch(0.85_0.08_80/0.5),0_6px_20px_-6px_oklch(0.85_0.08_80/0.6)]",
  secondary: "glass text-secondary-foreground hover:bg-accent",
  destructive: "bg-destructive/15 text-destructive hover:bg-destructive/22",
}
```

## Cards and panels

- Major cards: `glass`, 16px radius, 16–24px padding.
- Nested tiles: translucent fill plus hairline; omit backdrop blur.
- Menus, sheets, and detail panels: near-opaque `glass-popover`.
- Use a gold glow only for selected or primary elements.

Keep content grouping legible without the glass effect. The information hierarchy should still work when blur is unavailable.

## Status pills

Every pill includes text or an icon with an accessible label:

| State | Tone |
|---|---|
| New, draft, neutral | teal or muted |
| Contacted, sent | lavender |
| Replied, highlighted | rose |
| Active, ready, converted, live | mint |
| Paused, processing, warning | gold |
| Failed, blocked, destructive | coral |

Use compact pills for state, not for paragraphs or primary actions. A live dot may pulse, but the word “Live” must remain.

## App shell

- Desktop navigation can be a slim floating rail when icons are familiar and tooltips are available.
- Mobile navigation becomes a drawer or bottom pattern already supported by the product.
- Keep the content inset transparent so the canvas light can show through.
- Use a sticky glass page bar only when it does not stack with another sticky header.
- If the sidebar is collapsed, retain accessible names and keyboard reachability.

For a full-bleed graph page, opt out explicitly at the page boundary; do not make all pages edge-to-edge.

## Dashboard composition

A reliable desktop hierarchy:

1. Compact page bar with title, description, secondary action, and one primary action.
2. One glass KPI strip with two to four metrics.
3. Main content grid with a wider analytical column and narrower operational column.
4. Detail cards, timelines, or tables below.

At narrow widths:

- KPI row becomes two columns, then one if labels or localized values collide.
- Analytical and operational columns stack.
- Tables scroll inside their own labeled region.
- Header actions wrap or move into a menu.
- Never force desktop overlay controls to remain side by side.

Only render deltas or trends when the data exists. Use `tabular-nums` for all changing numbers.

## KPI and distribution patterns

- KPI label: 11px uppercase tracked eyebrow.
- KPI value: 40–56px, light, tabular.
- Hint: 12–13px muted copy.
- Segmented distribution: low-contrast track, semantically colored segments, and a separate legend or labeled tiles.
- Progress ring: include a text value; do not rely on the arc alone.

For glowing segments, set both `background-color` and `color` when the shadow uses `currentColor`.

## Tables and lists

- Use 5–9% white separators and a very slight row hover.
- Keep headers readable without making them the brightest element.
- Align numeric columns right and use tabular numerals.
- Status remains a text pill.
- A sticky table header needs a deliberate vertical scroll container. Do not apply `sticky` inside a wrapper that only scrolls horizontally and assume it works.

## Forms

- On glass, inputs use a dark translucent fill, hairline border, and visible gold focus ring.
- Labels remain persistent; placeholder text is not a label.
- Validation uses an icon or text as well as coral.
- Disabled controls must remain distinguishable from low-priority controls.
- Keep autocomplete, input type, names, and validation behavior from the existing form.

## Alerts and activity

Use a left-to-right tint that fades to transparent:

- Coral for failed or blocked.
- Gold for paused, processing, or warning.
- Mint for success or new activity.

Give urgent messages `role="alert"`; use `role="status"` for informational updates. Do not put every notification in an assertive live region.

Activity timelines use a subtle vertical rule, mono timestamps, a semantic icon, primary event text, and optional muted detail. Keep timestamp width flexible for localization.

## Chat

- Incoming: glass bubble with a slightly squared lower-left corner.
- Outgoing: translucent gold tint with foreground text and gold hairline.
- Timestamp: 11px muted text.
- Preserve sender identity and delivery state.
- On mobile, cap bubble width and prevent long URLs or code from causing horizontal page overflow.

## Floating control surfaces

For canvases, maps, and editors:

- Group related icon buttons in one glass cluster.
- Match standalone button height to the clustered controls.
- Use `aria-pressed` for toggles and visibly distinguish the pressed state.
- Place the single primary action in gold.
- Use real buttons and inputs, not interactive spans.

A detail panel should be near-opaque, independently scrollable, and responsive:

- Desktop: 320–380px side panel.
- Mobile: bottom sheet or full-width overlay.
- Move, hide, or reflow competing overlays instead of squeezing them.
- Keep the selected canvas object visible when the panel opens.

## Empty, loading, and error states

- Empty state: one concise explanation and one valid next action.
- Loading: preserve the final layout dimensions; avoid a page full of glowing skeletons.
- Error: coral treatment plus plain-language recovery guidance.
- Permission-limited users: hide unavailable write actions when appropriate, but enforce permission at the API too.

## Interaction rules

- Hover signals affordance; it does not carry required content.
- Click, keyboard activation, or focus opens details.
- Focus rings remain visible on all controls.
- Tooltips supplement accessible names; they do not replace them.
- Confirm destructive operations inline or with the project's established dialog component.
