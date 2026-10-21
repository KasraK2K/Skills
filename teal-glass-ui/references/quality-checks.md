# Quality Checks

Use the project's normal commands first. Add focused checks only when they test behavior not covered by the existing gates.

## Scope and preservation

- Confirm the user actually chose Teal Glass for every modified surface.
- Confirm unrelated pages, themes, behaviors, and dependencies remain intact.
- Review the diff for broad generated-file or formatting churn.
- Confirm sample data was not introduced into production paths.

## Visual hierarchy

- One clear primary action per region.
- Gold appears only for primary, selected, focus, or center.
- Glass surfaces remain distinguishable without stacking blur.
- Popovers and side panels are opaque enough for text.
- Status colors match meaning and include text or iconography.
- Long names, localized text, large numbers, empty values, and error messages fit.

## Responsive rendering

Verify the real rendered UI at minimum:

- Desktop around 1440×900.
- Narrow mobile around 375×812.
- One intermediate/tablet width when the layout has side panels or dense controls.

Check:

- No unintended horizontal page scroll.
- Header actions wrap or collapse deliberately.
- KPI values and labels do not overlap.
- Tables scroll inside their own region.
- Canvas overlays do not cover one another.
- Side panels become a usable sheet or full-width overlay.
- Safe-area insets are respected where relevant.

## Accessibility

- Keyboard reachability and visible focus for every control.
- Icon-only buttons have accessible names.
- Toggle state is exposed with `aria-pressed` or the native equivalent.
- Inputs have persistent labels and errors are programmatically associated.
- Status and charts do not rely on color alone.
- Representative foreground/muted text passes contrast on the actual composed glass surface.
- Reduced motion preserves content and state.
- Canvas has an accessible summary and an equivalent way to reach essential data.

Use automated accessibility tooling if already available, then manually test keyboard order and the most important states. Automated tools do not validate the quality of names, focus order, or canvas alternatives.

## Interaction and state

- Hover is supplementary.
- Destructive actions use the product's confirmation pattern.
- Loading, empty, error, permission-limited, and disabled states are present.
- Opening a detail panel does not hide the selected object or block required controls.
- Real data drives deltas, trends, counts, and status.

## Performance

- No idle animation loop or force simulation.
- Backdrop blur is limited to major surfaces.
- Canvas backing size is bounded and responds to DPR changes safely.
- Resize observers and animation frames clean up.
- Large data behavior is measured with representative volume.

## Engineering gates

Run the relevant available gates:

- Formatter or format check.
- Lint.
- Typecheck.
- Unit/component tests.
- Production build.
- Visual/browser smoke test for modified routes.

Report wrapper/tool failures separately from code failures. Do not claim a browser verification passed from source inspection alone.
