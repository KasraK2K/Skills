# Teal Glass UI

A dark-only design system for dashboards, admin panels and data-heavy apps: a deep teal-black background with soft glows and a faint star field, frosted glass panels, one champagne-gold accent for anything primary or selected, and a small family of jewel colours for categories. It also includes a glowing **orb network graph** (canvas + d3-force) for hierarchies like workspace → groups → items.

This folder is a Claude skill. Install it and ask Claude for "my teal glass design" (or "make it look like Reachly", "dark glass dashboard", "orb graph like before") in any project, and it follows these exact tokens and recipes.

![Dashboard](images/dashboard.png)

## What it looks like

### Network graph with legend, toolbar, search bar and detail panel

Hubs get a name above a gold stem line and a slanted count tag; files show only a small count badge; parts orbit their file as dots. Clicking a node opens the detail panel on the right. Hover does nothing except show a pointer cursor.

![Knowledge graph](images/graph.png)

### Tokens and components

Palette, type scale, status pills, buttons, input, alert bars, chat bubbles, table and step markers.

![Components](images/components.png)

## What's in this folder

| File | What it is |
|---|---|
| `SKILL.md` | The skill itself: every token, component recipe, the graph renderer, motion, accessibility rules and lessons learned. Readable on its own as a design-system spec. |
| `teal-glass-ui.skill` | The same skill packaged for one-click install in Claude (Save skill / install). |
| `images/` | The preview images above. |
| `examples/` | The standalone pages the images were rendered from. `glass.css` is a framework-free version of the system (plain CSS, no Tailwind), and `graph.html` contains the canvas drawing code for the orbs, tick rings, labels and badges. Open any `.html` file in a browser. |

## Installing the skill

- **Claude (web / desktop):** open `teal-glass-ui.skill`, or upload it under Settings → Capabilities → Skills, and choose **Save skill**.
- **Claude Code:** copy this folder (at least `SKILL.md`) to `~/.claude/skills/teal-glass-ui/` for all projects, or to `.claude/skills/teal-glass-ui/` inside one project.

## The system at a glance

**Colours**

| | oklch | Hex | Used for |
|---|---|---|---|
| Background | 0.17 0.018 200 | ≈ `#0a1214` | The ground, with two radial teal glows and a star field |
| Glass | 0.32 0.04 195 / 0.28 | | Cards and panels, with `backdrop-filter: blur(20px) saturate(140%)` and a 9% white hairline |
| Gold | 0.85 0.08 80 | `#e9c98b` | Primary buttons, selection, focus ring, the graph's centre |
| Rose | 0.76 0.11 355 | `#ec8fac` | Hubs, "replied" |
| Lavender | 0.74 0.12 290 | `#a99df2` | Items, "sent / contacted" |
| Mint | 0.85 0.12 160 | `#7fe3b8` | Success, live, ready |
| Teal | 0.74 0.07 185 | `#5fb8ad` | Neutral, new, draft |
| Coral | 0.72 0.17 30 | `#ff7a6b` | Errors, failed, destructive |

**Type:** Geist for UI, Geist Mono for timestamps, Roboto Condensed for graph labels. KPI numbers are big and light (48px, weight 300), section labels are 11px tracked capitals, all numbers use tabular figures.

**Shape:** 1rem base radius, 2xl cards, 3xl floating panels, full pills; elevation comes from the glass shadow, emphasis from a soft gold glow.

**Motion:** fades and staggered reveals, a slow glow pulse on live things, all switched off with `prefers-reduced-motion`.

## Built for

Next.js App Router + Tailwind CSS v4 + shadcn/ui + tw-animate-css + lucide-react, with d3-force for the graph. The tokens and the plain-CSS example work anywhere.

## Origin

Extracted from Reachly's redesign (October 2026), which was modelled on Fuselab Creative's "Control AI Policy Platform" concept and the "meely" agents dashboard. The preview images use made-up sample data.
