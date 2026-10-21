# Orb Graph

Use this reference only when the requested interface needs a network, hierarchy, or relationship view. For ordinary metrics and workflows, prefer simpler charts or lists.

The working vanilla renderer is in [../examples/graph.html](../examples/graph.html). Adapt its visual primitives rather than copying its sample data or product language.

## Visual model

The graph uses a clear four-level hierarchy:

- **Root:** champagne-gold orb with a spiky radial halo.
- **Hub:** rose orb with a fine tick ring.
- **Item:** lavender or mint orb with a small count badge.
- **Leaf:** small dot; highlight search matches in mint.

Edges become quieter with depth:

- Root to hub/item: gold gradient, strongest at the root.
- Hub to item: translucent teal.
- Item to leaf: thin dashed mint.

Do not label every object. Label the root and hubs; show item names only when selected or sufficiently zoomed; leave leaves unlabeled.

## Suggested data model

```ts
type NodeKind = "root" | "hub" | "item" | "leaf"

interface GraphNode {
  id: string
  kind: NodeKind
  label: string
  parentId: string | null
  count: number
  tone?: "gold" | "rose" | "lavender" | "mint" | "teal" | "coral"
  status?: "processing" | "ready" | "failed" | null
  dimmed?: boolean
  x?: number
  y?: number
  fx?: number | null
  fy?: number | null
}

interface GraphLink {
  source: string | GraphNode
  target: string | GraphNode
  kind: "spoke" | "branch" | "leaf"
}
```

Prefix IDs by kind and retain the raw domain ID separately if the application needs it. Build parent and node maps once; avoid repeated linear scans during layout and interaction.

Canonical world radii:

```ts
const NODE_RADIUS = { root: 26, hub: 13, item: 7, leaf: 2.2 }
const COLOR = {
  gold: "#e9c98b",
  rose: "#ec8fac",
  lavender: "#a99df2",
  mint: "#7fe3b8",
  teal: "#5fb8ad",
  coral: "#ff7a6b",
}
```

## Renderer

Render on a high-DPI canvas:

1. Size the backing bitmap with `devicePixelRatio`.
2. Apply one world transform for pan and zoom.
3. Draw links first.
4. Draw nodes from smallest to largest: leaf, item, hub, root.
5. Draw each node's badge or label immediately after the node.
6. Restore canvas state after temporary alpha, composite, dash, or transform changes.

An orb consists of:

- A radial additive halo when selected, active, or highlighted.
- A radial body gradient with a white top-left specular point.
- A darker rim and thin white edge.
- An optional radial tick ring around root, hub, or item.

Keep labels a constant screen size by dividing font size, line width, and padding by the current zoom. Use the project's condensed font when available. Truncate long overview labels and provide the full value in the detail panel.

Root and hub label treatment:

- A thin gold stem from the orb's right edge.
- Light condensed name above the stem.
- A dark slanted count tag below it.
- Small gold unit text after the count.

Item treatment:

- Small dark count badge at the top-right.
- Full name only while selected or above a readable zoom threshold.

## Layout

Seed a deterministic radial tree before starting a force simulation. Determinism makes tests, screenshots, and user orientation reliable.

Suggested d3-force behavior:

- Pin the root at the origin.
- Place hubs on a ring whose radius grows with hub count.
- Keep item-to-leaf links short.
- Use collision radius slightly larger than the drawn orb.
- Use mild spoke strength so the radial structure remains legible.
- Carry positions by ID across data/filter rebuilds; seed only new nodes near their parent.

Two useful modes:

- **Tree:** pin all nodes to deterministic seed positions and stop the simulation.
- **Force:** clear pins on non-root nodes and let the simulation settle.

When changing modes, explicitly clear or restore `fx` and `fy`. Otherwise a force view can remain accidentally frozen.

## Interaction

- Hover changes only the cursor. Required information appears on click or keyboard-accessible selection.
- Click selects and opens the detail panel; click empty canvas clears selection.
- Treat a pointer move greater than a small threshold as a pan, not a click.
- Wheel or controls zoom around the pointer and clamp scale to a useful range.
- Dragging a node is optional and belongs to authorized edit mode only.
- File drop is optional; call `preventDefault()` only for file drag types.
- Handle `pointercancel` and `lostpointercapture`.

If re-parenting:

- Exclude the dragged node and its descendants from drop-target hit testing.
- Highlight only a valid hub.
- Do nothing when released over empty space.
- Persist through the application's API, report success/failure, and reconcile state.

Search results should carry the exact matching node ID. Pan the selected result into an unobstructed area and zoom enough for small leaves to be visible.

## Overlay composition

Typical overlay regions:

- Center-top toolbar.
- Bottom-left legend.
- Bottom-center search/command bar.
- Right-side detail panel.

They compete for space. When the detail panel opens, reflow or hide lower-priority overlays and pan the selection away from the covered region. On mobile, use a bottom sheet and reduce the number of always-visible controls.

Legend rows are toggle buttons with `aria-pressed`, text labels, a count, and a colored dot. The color is supplementary.

The detail panel includes:

1. Type, name, tone dot, and close control.
2. Key count cards.
3. Description list for state and metadata.
4. Error detail when present.
5. Valid actions for the current user's permissions.

## Accessibility fallback

Canvas is not an accessible data structure. Provide:

- A concise `aria-label` or accessible description with visible counts and current selection.
- Keyboard-operable overlay controls.
- A list/table or detail representation of the same essential information.
- No interaction that requires precise dragging.
- Reduced-motion behavior that synchronously settles or uses deterministic positions.

## Performance

- Run one animation frame loop only while the simulation is active or a visible state is pulsing.
- Cancel animation and observers on unmount.
- Do not resize the backing bitmap unless rounded dimensions actually changed.
- Cull leaves when zoomed out or above a project-appropriate threshold.
- Keep hit testing spatially bounded for large graphs.
- Profile on representative data, not only the small demo.

The demo's 1,500-leaf drawing guard and 5,000-leaf API threshold are starting points, not universal limits.

## Common failures

- Continuous idle animation.
- Labels on every node.
- Hover lens or tooltip obscuring dense content.
- Re-exploding layout after every filter or selection.
- Side panel covering the selected node.
- Unbounded high-DPI canvas memory.
- Drag-and-drop enabled for users who cannot persist changes.
- Internal data-model words used as product-facing labels.
