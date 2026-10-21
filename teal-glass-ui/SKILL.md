---
name: teal-glass-ui
description: The "teal glass" dark design system — deep teal-black ground with radial glows and a star field, frosted glass panels, champagne-gold primary, rose / lavender / mint / teal / coral accents, thin-numeral KPIs, condensed labels, and a glowing orb network graph (canvas + d3-force) with legend, toolbar, ask bar and a click-to-open detail panel. Use this skill whenever the user wants this look in any app — "use my glass design", "teal and gold dark theme", "make it look like Reachly", "dark glass dashboard", "Fuselab / Control AI style", "orb graph", "knowledge graph like before", "glowing network graph" — and whenever they ask to restyle, theme, or build a dashboard, admin panel, sidebar, KPI row, data table, chat UI, or node/graph visualization and have previously chosen this style, even if they don't name it. Covers exact tokens, Tailwind v4 + shadcn setup, every component recipe, the canvas graph renderer, motion, accessibility, and the mistakes to avoid.
---

# Teal Glass UI

A dark-only design system distilled from a production app (Reachly) that was modeled on Fuselab Creative's "Control AI Policy Platform" shots and the "meely" agents dashboard. The feel: a quiet teal-black space, frosted panels floating over soft glows, one warm gold accent for anything primary or selected, and a small family of jewel accents for categories. Numbers are big and thin, labels are tiny and tracked, nothing is loud except the thing that matters.

Read the whole file once before building. Sections 1–4 are the foundation (do these first, they make most shadcn components look right automatically); 5–7 are component recipes; 8 is the network graph; 9 is the list of things that went wrong last time and why.

---

## 0. Principles (why it looks the way it does)

- **Dark only.** The glass, glow and orb effects only read on a dark ground. Don't build a light theme; remove theme toggles and `next-themes`. Put `class="dark"` on `<html>` so any `dark:` variants inside third-party primitives still resolve.
- **One warm accent.** Champagne gold means "primary / selected / focus". Everything else is a cool teal family. If two things are gold, neither stands out.
- **Glass, not boxes.** Surfaces are translucent teal with a 1px hairline and a faint top highlight, over a blurred background. Cards don't have heavy borders or solid fills.
- **Colour is a category, never the only signal.** Every coloured status also carries text (a pill label, an icon, a word).
- **Big thin numbers, tiny tracked labels.** KPIs are `font-light` at 40–56px; section labels are 11px uppercase with wide tracking.
- **Motion is ambient and optional.** Soft fades, staggered reveals, slow pulses on "live" things. All of it switches off under `prefers-reduced-motion`.
- **Details on click, not hover.** Hover should only signal clickability (cursor). Anything informative opens in a side panel on click (see §9 for why).

---

## 1. Stack and setup

Written for **Next.js (App Router) + Tailwind CSS v4 + shadcn/ui (radix) + tw-animate-css + lucide-react**. It ports to any React + Tailwind v4 app; the CSS tokens port to anything.

```bash
pnpm add tw-animate-css lucide-react
pnpm add d3-force && pnpm add -D @types/d3-force   # only for the network graph (§8)
pnpm remove next-themes                            # dark only
```

Fonts (Next.js `next/font/google`):

```tsx
// app/layout.tsx
import { Geist, Geist_Mono, Roboto_Condensed } from "next/font/google"

const geistSans = Geist({ variable: "--font-geist-sans", subsets: ["latin"] })
const geistMono = Geist_Mono({ variable: "--font-geist-mono", subsets: ["latin"] })
/** Canvas labels (the graph reads this CSS variable by name). */
const condensed = Roboto_Condensed({ variable: "--font-condensed", subsets: ["latin"], weight: ["300", "400", "600"] })

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`dark ${geistSans.variable} ${geistMono.variable} ${condensed.variable}`}>
      <body className="antialiased">
        {children}
        <Toaster richColors position="bottom-right" /> {/* sonner, with theme="dark" hard-coded */}
      </body>
    </html>
  )
}
```

- Geist Sans for all UI text, Geist Mono for timestamps/ids, Roboto Condensed only for canvas graph labels.
- In the sonner wrapper (`components/ui/sonner.tsx`) delete the `useTheme` call and pass `theme="dark"`.
- If you use shadcn's Sidebar with `tooltip` props, wrap the app shell in `<TooltipProvider>` — otherwise every page crashes with "`Tooltip` must be used within `TooltipProvider`".

---

## 2. Tokens (globals.css)

Replace shadcn's `:root` and `.dark` blocks with this single `:root` (keep shadcn's `@theme inline` mapping block as generated; make sure it maps `--font-sans` to `--font-geist-sans`, `--font-mono` to `--font-geist-mono`, and derives `--radius-sm…4xl` from `--radius`).

```css
@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";

@custom-variant dark (&:is(.dark *));

/* @theme inline { …shadcn's generated mapping, unchanged… } */

:root {
  color-scheme: dark;
  --background: oklch(0.17 0.018 200);          /* teal-black ground ≈ #0a1214 */
  --foreground: oklch(0.95 0.01 190);
  --card: oklch(0.32 0.04 195 / 0.28);          /* glass fill */
  --card-foreground: var(--foreground);
  --popover: oklch(0.22 0.03 198 / 0.92);       /* near-opaque glass for menus, side panels */
  --popover-foreground: var(--foreground);
  --primary: oklch(0.85 0.08 80);               /* champagne gold */
  --primary-foreground: oklch(0.2 0.02 80);     /* near-black — only on solid gold */
  --secondary: oklch(0.35 0.04 195 / 0.45);
  --secondary-foreground: var(--foreground);
  --muted: oklch(0.32 0.03 195 / 0.4);
  --muted-foreground: oklch(0.74 0.02 195);
  --accent: oklch(0.38 0.05 190 / 0.5);         /* hover fill on glass */
  --accent-foreground: var(--foreground);
  --destructive: oklch(0.72 0.17 30);           /* coral */
  --border: oklch(1 0 0 / 9%);                  /* hairline */
  --input: oklch(1 0 0 / 12%);
  --ring: oklch(0.85 0.08 80);                  /* gold focus ring */
  --chart-1: oklch(0.85 0.08 80);               /* gold */
  --chart-2: oklch(0.76 0.11 355);              /* rose */
  --chart-3: oklch(0.74 0.12 290);              /* lavender */
  --chart-4: oklch(0.85 0.12 160);              /* mint */
  --chart-5: oklch(0.74 0.07 185);              /* teal */
  --radius: 1rem;
  --sidebar: oklch(0.2 0.025 198 / 0.7);
  --sidebar-foreground: var(--foreground);
  --sidebar-primary: var(--primary);
  --sidebar-primary-foreground: var(--primary-foreground);
  --sidebar-accent: oklch(0.85 0.08 80 / 0.12);
  --sidebar-accent-foreground: oklch(0.9 0.07 80);
  --sidebar-border: oklch(1 0 0 / 8%);
  --sidebar-ring: var(--ring);
}
```

### 2.1 Background: glows + star field

```css
@layer base {
  * { @apply border-border outline-ring/50; }
  body {
    @apply text-foreground;
    min-height: 100svh;
    background:
      radial-gradient(60rem 40rem at 50% -10%, oklch(0.45 0.07 190 / 0.35), transparent 70%),
      radial-gradient(50rem 40rem at -10% 110%, oklch(0.4 0.06 185 / 0.3), transparent 70%),
      var(--background);
    background-attachment: fixed;
  }
  /* CSS-only star field: three tiled layers of 1px dots, one in mint. */
  body::before {
    content: "";
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 0;
    opacity: 0.35;
    background-image:
      radial-gradient(1px 1px at 20% 30%, white 50%, transparent 51%),
      radial-gradient(1px 1px at 70% 80%, white 50%, transparent 51%),
      radial-gradient(1px 1px at 40% 60%, oklch(0.85 0.12 160) 50%, transparent 51%);
    background-size: 340px 340px, 460px 460px, 580px 580px;
  }
  body > * { position: relative; z-index: 1; }   /* content above the stars; utilities still win */
  html { @apply font-sans; }
}
```

### 2.2 Utilities and motion

```css
@utility glass {
  background: var(--card);
  backdrop-filter: blur(20px) saturate(140%);
  box-shadow: inset 0 1px 0 rgb(255 255 255 / 0.06), 0 10px 40px -12px rgb(0 0 0 / 0.6);
  border: 1px solid var(--border);
}
@utility glow-gold {
  box-shadow: 0 0 0 1px oklch(0.85 0.08 80 / 0.6), 0 0 24px -4px oklch(0.85 0.08 80 / 0.55);
}
@keyframes pulse-glow {
  0%, 100% { opacity: 1; filter: drop-shadow(0 0 0 currentColor); }
  50% { opacity: 0.7; filter: drop-shadow(0 0 6px currentColor); }
}
@utility animate-pulse-glow { animation: pulse-glow 2.4s ease-in-out infinite; }
/* Pair with tw-animate's animate-in classes; set --i per item for a 60ms cascade. */
@utility stagger { animation-delay: calc(var(--i, 0) * 60ms); animation-fill-mode: both; }

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation: none !important; transition: none !important; }
}

/* Slow drifting stars, only on auth screens (<main data-auth>). */
@keyframes drift { to { background-position: 340px 680px, -460px 460px, 580px -580px; } }
body:has(main[data-auth])::before { animation: drift 120s linear infinite; }
```

`glass`, `glow-gold` etc. are `@utility` definitions, so variants work: `hover:glow-gold`, `aria-pressed:glow-gold`, `data-[active=true]:glow-gold`.

---

## 3. Palette and semantics

| Role | Token | oklch | Hex (canvas) |
|---|---|---|---|
| Primary, selected, focus, "the centre" | `--primary` / `--chart-1` | 0.85 0.08 80 | `#e9c98b` gold |
| Category A / hubs / "replied" | `--chart-2` | 0.76 0.11 355 | `#ec8fac` rose |
| Category B / secondary items / "contacted, sent" | `--chart-3` | 0.74 0.12 290 | `#a99df2` lavender |
| Success, live, ready, company-level items | `--chart-4` | 0.85 0.12 160 | `#7fe3b8` mint |
| Neutral / new / draft | `--chart-5` | 0.74 0.07 185 | `#5fb8ad` teal |
| Error, failed, destructive, "not interested" | `--destructive` | 0.72 0.17 30 | `#ff7a6b` coral |

Mapping rule when restyling an existing app: grep for hard-coded colours and replace by meaning, not by hue.

```bash
grep -rnE "(bg|text|border|ring|stroke|fill)-(zinc|gray|slate|neutral|blue|violet|emerald|green|red|amber|yellow|sky|indigo|purple)-[0-9]{2,3}" app components --include=*.tsx
```

| Meaning | Use |
|---|---|
| neutral / new / draft | `--chart-5` or `text-muted-foreground` |
| info / sent / contacted | `--chart-3` |
| reply / highlight | `--chart-2` |
| success / converted / active / live | `--chart-4` |
| primary / selected / completed | `--chart-1` / `primary` |
| warning / paused | `--chart-1` in a gold `GlowBar` |
| error / failed | `destructive` |

e.g. `bg-emerald-500` → `bg-[var(--chart-4)]`, `text-red-500` → `text-destructive`. Delete `dark:` variants from app code (dark only).

Contrast: gold (L≈0.85) and mint text on tinted glass are far above 4.5:1. **Never** put `text-primary-foreground` (near-black) on anything except solid gold — it becomes invisible on gold-tinted glass.

---

## 4. Type, shape, spacing

**Type scale**
- Page title: `text-lg font-medium tracking-tight` (in the glass header bar).
- Section/card title: shadcn default `text-base font-medium`.
- Body: `text-sm`; secondary text `text-xs text-muted-foreground`.
- Section label / eyebrow: `text-[11px] font-medium tracking-[0.12em] text-muted-foreground uppercase` (legend uses `tracking-[0.14em]`).
- KPI numeral: `text-4xl md:text-5xl font-light tracking-tight tabular-nums`.
- Tile numeral: `text-2xl font-light tabular-nums`. Small stat in a strip: `text-base font-light tabular-nums`.
- Timestamps / ids: `font-mono text-xs text-muted-foreground`.
- Numbers always `tabular-nums`.

**Shape**
- `--radius: 1rem`. Cards `rounded-2xl`; floating bars/panels `rounded-2xl`–`rounded-3xl`; auth card `rounded-3xl`; pills `rounded-full`; icon buttons `rounded-xl`.
- Borders are the 9% white hairline from `glass`; internal dividers `border-white/10` (or `/5` for table rows).

**Spacing**
- Page content: `mx-auto w-full max-w-6xl px-4 py-6 md:px-8 md:py-8`.
- Card padding comes from shadcn's `--card-spacing` (16px; 12px for `size="sm"`).
- Gaps: 2 (8px) inside clusters, 4–6 between blocks.

**Elevation** is the `glass` shadow (`0 10px 40px -12px rgb(0 0 0 / .6)`). Don't stack extra shadows; use `glow-gold` for emphasis instead.

---

## 5. Primitive overrides (shadcn)

### 5.1 Button (`components/ui/button.tsx`)

Keep shadcn's base class string and sizes, change these:

```ts
default:   "bg-gradient-to-b from-[oklch(0.9_0.08_80)] to-[oklch(0.78_0.09_75)] text-primary-foreground shadow-[0_0_0_1px_oklch(0.85_0.08_80/0.5),0_6px_20px_-6px_oklch(0.85_0.08_80/0.6)] hover:shadow-[0_0_0_1px_oklch(0.85_0.08_80/0.8),0_8px_28px_-4px_oklch(0.85_0.08_80/0.8)]",
outline:   "glass hover:bg-accent hover:text-foreground aria-expanded:bg-accent",
secondary: "glass text-secondary-foreground hover:bg-accent aria-expanded:bg-accent",
// ghost / destructive / link: shadcn defaults
// sizes:
icon:      "size-9 rounded-xl",
"icon-lg": "size-10 rounded-xl",
```

Primary = gold gradient pill with a soft gold halo. Secondary/outline = glass. One primary button per area.

### 5.2 Card (`components/ui/card.tsx`)

In `Card`, replace `rounded-xl bg-card … ring-1 ring-foreground/10` with `rounded-2xl glass` (keep `overflow-hidden` so the blur clips to the radius). In `CardFooter`, replace `bg-muted/50` with `bg-white/[0.02]`.

### 5.3 Table

- `TableRow`: `border-b border-white/5 transition-colors hover:bg-white/[0.03]`.
- `TableHeader`: just `[&_tr]:border-b`. Don't add `sticky top-0` — shadcn wraps `<table>` in an `overflow-x-auto` div, which becomes the sticky container, so it never sticks. A sticky header needs its own scroll container per table.

### 5.4 Inputs

shadcn defaults read fine on glass. For an input sitting on glass, add `rounded-xl bg-black/30`. Don't try a white input — shadcn's `dark:bg-input/30` wins over `bg-white/90` (html has `.dark`) and you get dark text on a dark field.

---

## 6. Component recipes

### 6.1 `fx.tsx` — Pill, Stat, GlowBar

```tsx
import { cn } from "@/lib/utils"

type Tone = "gold" | "rose" | "lavender" | "mint" | "coral" | "muted"
const TONE: Record<Tone, string> = {
  gold: "text-[oklch(0.88_0.08_80)] bg-[oklch(0.85_0.08_80/0.12)] ring-[oklch(0.85_0.08_80/0.35)]",
  rose: "text-[oklch(0.84_0.1_355)] bg-[oklch(0.76_0.11_355/0.12)] ring-[oklch(0.76_0.11_355/0.35)]",
  lavender: "text-[oklch(0.84_0.1_290)] bg-[oklch(0.74_0.12_290/0.12)] ring-[oklch(0.74_0.12_290/0.35)]",
  mint: "text-[oklch(0.88_0.11_160)] bg-[oklch(0.85_0.12_160/0.12)] ring-[oklch(0.85_0.12_160/0.35)]",
  coral: "text-[oklch(0.82_0.14_30)] bg-[oklch(0.72_0.17_30/0.14)] ring-[oklch(0.72_0.17_30/0.4)]",
  muted: "text-muted-foreground bg-white/5 ring-white/10",
}

/** Count, percent or status badge. Always carries text, so colour isn't the only signal. */
export function Pill({ children, tone = "muted", className }: { children: React.ReactNode; tone?: Tone; className?: string }) {
  return <span className={cn("inline-flex items-center gap-1 rounded-full px-2 py-0.5 text-[11px] font-medium tabular-nums ring-1", TONE[tone], className)}>{children}</span>
}

/** Thin-numeral KPI. */
export function Stat({ label, value, hint }: { label: string; value: React.ReactNode; hint?: React.ReactNode }) {
  return (
    <div className="grid gap-1">
      <span className="text-[11px] font-medium tracking-[0.12em] text-muted-foreground uppercase">{label}</span>
      <span className="text-4xl font-light tracking-tight tabular-nums md:text-5xl">{value}</span>
      {hint && <span className="text-xs text-muted-foreground">{hint}</span>}
    </div>
  )
}

/** Alert-ish row with a tinted gradient fading to the right ("Critical concerns" style). */
export function GlowBar({ tone, role, children }: { tone: "coral" | "gold" | "mint"; role?: "alert" | "status"; children: React.ReactNode }) {
  const bar = { coral: "from-[oklch(0.72_0.17_30/0.35)]", gold: "from-[oklch(0.85_0.08_80/0.3)]", mint: "from-[oklch(0.85_0.12_160/0.25)]" }[tone]
  return <div role={role} className={cn("rounded-xl border bg-gradient-to-r to-transparent p-3", bar)}>{children}</div>
}
```

Status → tone map used everywhere: New/Draft `muted`, Contacted `lavender`, Replied `rose`, Converted/Active/Ready `mint`, Paused/Processing `gold`, Completed `lavender`, NotInterested/Failed `coral`. Give alerts `role="alert"`, informational bars `role="status"`.

### 6.2 App shell

- **Sidebar = slim glass icon rail**: shadcn `<Sidebar collapsible="icon" variant="floating">`, `<SidebarProvider defaultOpen={false}>`. Each `SidebarMenuButton` gets `tooltip={label}` and `className="data-[active=true]:glow-gold data-[active=true]:text-sidebar-accent-foreground"` (active item = gold glyph on a gold-glow tile). Hide the wordmark when collapsed: `group-data-[collapsible=icon]:hidden`. Sign-out at the bottom, tooltip only (no duplicate `title`). No theme toggle.
- **Inset**: `<SidebarInset className="bg-transparent">` so the body glow shows through.
- **Mobile bar** (below md): `<header className="glass sticky top-0 z-20 flex h-12 items-center gap-2 rounded-none border-x-0 border-t-0 px-4 md:hidden"><SidebarTrigger /></header>`.
- **Content wrapper with full-bleed opt-out (pure CSS)**: `<div className="mx-auto w-full max-w-6xl px-4 py-6 md:px-8 md:py-8 has-[[data-bleed]]:max-w-none has-[[data-bleed]]:p-0">`. A page whose root has `data-bleed` goes edge to edge (used for the graph page).
- **Page header = sticky glass bar** (the "top bar"; pages pass actions as children):

```tsx
export function PageHeader({ title, description, children }: { title: string; description?: string; children?: React.ReactNode }) {
  return (
    <div className="glass sticky top-15 z-10 md:top-3 mb-6 flex flex-wrap items-center justify-between gap-4 rounded-2xl px-5 py-3">
      <div className="min-w-0">
        <h1 className="truncate text-lg font-medium tracking-tight">{title}</h1>
        {description && <p className="mt-0.5 text-xs text-muted-foreground">{description}</p>}
      </div>
      {children && <div className="flex flex-wrap gap-2">{children}</div>}
    </div>
  )
}
```
`top-15` on mobile clears the 48px mobile bar; `md:top-3` on desktop.

### 6.3 Dashboard pieces

- **KPI hero row**: `<div className="glass mb-6 grid grid-cols-2 gap-6 rounded-2xl p-6 md:grid-cols-4">` with four `<Stat>`s. Wraps to 2×2 on mobile. Only show deltas if you have real history — never invent them.
- **Segmented glowing bar** (pipeline/distribution): track `flex h-2.5 overflow-hidden rounded-full bg-white/5`; each segment `bg-[var(--chart-N)] text-[var(--chart-N)] shadow-[0_0_12px_currentColor]` with `style={{ width: % }}` and a `title`. The matching `text-` colour matters: the glow uses `currentColor`.
- **Status tiles** under the bar: `glass rounded-xl p-3 transition-colors hover:bg-accent`, a `size-2 rounded-full` dot + `text-xs text-muted-foreground` label, value `text-2xl font-light tabular-nums`. Grid `grid-cols-2 sm:grid-cols-5`.
- **LIVE indicator**: `<Pill tone="mint"><span className="size-1.5 rounded-full bg-current animate-pulse-glow" /> LIVE</Pill>`.
- **Progress ring**: SVG `viewBox="0 0 52 52"` `size-14 -rotate-90`, r=22, stroke-width 5, track `stroke-muted`, progress `stroke-primary` with `strokeLinecap="round"` and dasharray/dashoffset; centre label rotated back.
- **Trace timeline** ("Reasoning Trace" look): `<ul className="grid gap-3 border-l border-white/10 pl-3">`; each row: mono timestamp `font-mono text-xs text-muted-foreground w-28 shrink-0`, then a coloured icon, then text.
- **Checklist**: done = `CheckCircle2` in mint + muted line-through text; todo = `Circle` muted.

### 6.4 Lists, steps, chat

- **Campaign/entity cards**: `glass` card, status `Pill` top-right, meta line muted.
- **Step checklist**: number in `glass size-8 rounded-full` (gold glow + check when done). Don't try a `::before` connector line between card steps; the card's backdrop-filter stacking context paints over it.
- **Chat bubbles**: theirs `glass rounded-2xl rounded-bl-md px-3 py-2 text-sm`; ours `rounded-2xl rounded-br-md bg-[oklch(0.85_0.08_80/0.14)] ring-1 ring-[oklch(0.85_0.08_80/0.3)]`; timestamps `text-[11px] text-muted-foreground` in both.
- **Suggestion / AI cards**: `glass rounded-xl p-3 transition-shadow hover:glow-gold`.
- **Auth screens**: `<main data-auth className="grid min-h-svh place-items-center px-4 py-10">` + `<div className="glass w-full max-w-sm space-y-6 rounded-3xl p-8">`. Stars drift slowly behind.

### 6.5 Floating control surfaces (used on canvases, maps, editors)

- **Toolbar**: `absolute top-4 left-1/2 -translate-x-1/2 flex gap-2`. Groups of related icon buttons in `glass flex rounded-2xl p-1` with `variant="ghost" size="icon"` buttons. Pressed toggle: `aria-pressed:bg-black/40 aria-pressed:text-primary aria-pressed:glow-gold`. **Standalone** buttons next to groups must be `size-[46px] rounded-2xl` (36px button + 2×4px padding + 2×1px border) or they look shorter than the groups. The single primary action (Add/Upload) is the gold default button with `glow-gold`. Count badge on a button: `absolute -top-1 -right-1 grid size-4 place-items-center rounded-full bg-primary text-[10px] text-primary-foreground`.
- **Legend**: `glass absolute bottom-4 left-4 z-10 hidden w-48 rounded-2xl p-4 text-sm sm:block`; eyebrow "LEGEND"; rows are buttons (`aria-pressed`, `aria-[pressed=false]:opacity-40`) with a `glass rounded-full px-1.5 py-0.5` capsule holding a glowing dot (`rounded-full shadow-[0_0_8px_currentColor]`, background+color = hue) and a count; rows reveal with `stagger animate-in fade-in slide-in-from-left-2` and `--i`. Footer rows separated by `border-t border-white/10` with gold/coral pills.
- **Ask / command bar**: `absolute inset-x-4 bottom-4 z-10 sm:left-56 mx-auto max-w-xl`; a `glass rounded-2xl p-3` form with a 4-column stat strip (`text-[10px] tracking-wider uppercase` labels, `text-base font-light tabular-nums` values) above an input (`rounded-xl bg-black/30`) + gold icon submit. Results card above it: `glass mb-2 max-h-56 overflow-y-auto rounded-2xl p-3 animate-in fade-in slide-in-from-bottom-2`, items `rounded-xl bg-white/[0.03] p-2 text-xs` with a mint score pill.
- **Detail side panel** (click a thing → details): `glass absolute top-4 right-4 bottom-4 z-20 flex w-[min(22rem,calc(100%-2rem))] flex-col gap-4 overflow-y-auto rounded-3xl p-5 animate-in fade-in slide-in-from-right-6` with **inline** `style={{ background: "var(--popover)" }}` (both `glass` and a `bg-` class set `background`; inline makes the opaque one win deterministically). Contents, top to bottom:
  1. Header: glowing colour dot (`size-4 rounded-full shadow-[0_0_14px_currentColor]`), eyebrow with the type ("FILE", "CAMPAIGN"), name `text-lg font-medium break-words`, ghost close button.
  2. Two count cards: `glass stagger rounded-2xl p-3 animate-in fade-in slide-in-from-left-4` with `--i` 0/1, value `text-2xl font-light`, label 11px uppercase.
  3. Facts list: `<dl className="grid grid-cols-[auto_1fr] gap-x-4 gap-y-2 rounded-2xl bg-white/[0.03] p-3 text-sm">`, `dt` muted, `dd` right-aligned; status values as Pills; dates formatted; sizes "6 KB"/"1.2 MB"; mime → friendly type (PDF, Word, Markdown, CSV, Image (PNG)).
  4. Error (if any) in a coral gradient box `bg-gradient-to-r from-[oklch(0.72_0.17_30/0.3)] to-transparent rounded-2xl p-3`.
  5. Actions as small outline buttons; destructive actions confirm **inline** in a coral gradient box (no `confirm()`), and buttons disable while their request runs.
  When the panel is open on ≥lg, push other bottom overlays left (`lg:right-[24rem]`), and hide them below lg (`max-lg:hidden`) rather than squeezing them. If selecting something near the right edge, pan the view so it isn't hidden behind the panel.

---

## 7. Motion and accessibility

- Entrances: tw-animate `animate-in fade-in` + `slide-in-from-left-2 / -right-6 / -bottom-2` or `zoom-in-50`; cascade with `stagger` + `--i`.
- Ambient: `animate-pulse-glow` on live dots and processing states only.
- Canvas animation loops must stop when idle (see §8.6) and respect reduced motion.
- Focus: gold ring via `--ring` (shadcn's `focus-visible:ring-3 ring-ring/50`) — keep it visible on every control.
- Keyboard: every overlay control is a real `<button>`/input; legend rows are toggle buttons with `aria-pressed`; icon buttons have `aria-label`.
- Canvas gets `role="img"` and an `aria-label` that summarises the counts.
- Text on glass ≥ 4.5:1 (foreground/muted-foreground on `--card` pass). Status = colour + text.
- Responsive: rail → drawer below md; KPI rows 2×2; tables scroll horizontally inside their wrapper; no horizontal page scroll at 375px (check `document.documentElement.scrollWidth > innerWidth` is false).

---

## 8. The orb network graph (canvas)

A full-bleed glass card containing a `<canvas>` drawn by a small renderer, laid out with d3-force, with the HTML overlays from §6.5 on top. Built for a "workspace → campaigns → files → chunks" knowledge graph, but the recipe fits any hub-and-spoke hierarchy (org → teams → people, product → features → issues).

### 8.1 Files

```
components/graph/
  layout.ts        pure: build nodes/links from data, tree positions, ring radius, scale guard (no imports → unit-testable with node --test)
  layout.test.ts
  render.ts        canvas drawing only: orbs, rings, edges, labels, badges, hit-testing
  overlays.tsx     Toolbar, Legend, AskBar
  detail-panel.tsx click-to-open side panel
  index.tsx        data fetch, simulation, rAF loop, pointer/drag/drop, composition
```

### 8.2 Node model

```ts
type NodeKind = "workspace" | "campaign" | "doc" | "chunk"   // root → hub → item → leaf
interface GNode { id: string; kind: NodeKind; label: string; r: number; parent: string | null; company: boolean; dim: boolean;
  status: "Processing" | "Ready" | "Failed" | null; count: number; ref: string; x?: number; y?: number; fx?: number | null; fy?: number | null; vx?: number; vy?: number }
interface GLink { source: string | GNode; target: string | GNode; kind: "spoke" | "branch" | "leaf" }

const RADIUS = { workspace: 26, campaign: 13, doc: 7, chunk: 2.2 }   // world units
```

- Ids are prefixed (`ws`, `c:<id>`, `d:<id>`, `k:<id>`); `ref` is the raw id.
- Root is pinned at the origin (`fx: 0, fy: 0`).
- Links: root→hub and root→root-level-item are **spokes**; hub→item **branch**; item→leaf **leaf**.
- Root-level items (e.g. company files) are mint; hub items lavender; failed items coral.
- Scoped view (e.g. one campaign): that hub becomes the pinned centre; root-level items show as `dim` context.
- Build with `Map` lookups (byId, per-hub counts); the graph rebuilds on selection/filter changes, and linear scans get quadratic at a few thousand nodes.

### 8.3 Colours and drawing primitives (render.ts)

```ts
export const COLOR = { gold: "#e9c98b", rose: "#ec8fac", lavender: "#a99df2", mint: "#7fe3b8", teal: "#5fb8ad", coral: "#ff7a6b" }
const hex = (c: string) => [1, 3, 5].map((i) => parseInt(c.slice(i, i + 2), 16))
const rgba = (c: string, a: number) => `rgba(${hex(c).join(",")},${a})`
const shade = (c: string, f: number) => `rgb(${hex(c).map((v) => Math.round(v * f)).join(",")})`
const TAU = Math.PI * 2

export function nodeColor(n: GNode) {
  if (n.status === "Failed") return COLOR.coral
  if (n.kind === "workspace") return COLOR.gold
  if (n.kind === "campaign") return COLOR.rose
  return n.company ? COLOR.mint : COLOR.lavender
}

/** Glossy sphere: additive halo, white specular highlight top-left, darkened rim, thin white edge. */
function orb(ctx: CanvasRenderingContext2D, x: number, y: number, r: number, c: string, glow: number) {
  if (glow > 0) {
    const g = ctx.createRadialGradient(x, y, r * 0.6, x, y, r * 3.4)
    g.addColorStop(0, rgba(c, 0.4 * glow)); g.addColorStop(1, rgba(c, 0))
    ctx.globalCompositeOperation = "lighter"
    ctx.fillStyle = g; ctx.beginPath(); ctx.arc(x, y, r * 3.4, 0, TAU); ctx.fill()
    ctx.globalCompositeOperation = "source-over"
  }
  const body = ctx.createRadialGradient(x - r * 0.35, y - r * 0.4, r * 0.08, x, y, r)
  body.addColorStop(0, "#ffffff"); body.addColorStop(0.28, c); body.addColorStop(1, shade(c, 0.4))
  ctx.fillStyle = body; ctx.beginPath(); ctx.arc(x, y, r, 0, TAU); ctx.fill()
  ctx.strokeStyle = "rgba(255,255,255,0.28)"; ctx.lineWidth = 0.75; ctx.stroke()
}

/** Fine radial ticks around an orb; the root gets 64 longer, uneven, slowly turning "spiky halo" ticks. */
function tickRing(ctx: CanvasRenderingContext2D, x: number, y: number, r: number, c: string, k: number, t: number, spiky: boolean) {
  const count = spiky ? 64 : 44
  ctx.strokeStyle = rgba(c, spiky ? 0.55 : 0.4)
  ctx.lineWidth = (spiky ? 0.8 : 0.6) / Math.max(k, 0.5)
  ctx.beginPath()
  for (let i = 0; i < count; i++) {
    const a = (i / count) * TAU + (spiky ? t / 8000 : 0)
    const from = r * (spiky ? 1.08 : 1.2)
    const to = spiky ? r * (1.18 + 0.22 * Math.abs(Math.sin(i * 1.7))) : r * 1.5
    ctx.moveTo(x + Math.cos(a) * from, y + Math.sin(a) * from)
    ctx.lineTo(x + Math.cos(a) * to, y + Math.sin(a) * to)
  }
  ctx.stroke()
}
```

Orb glow levels: root always 1; hub/items 0.35 at rest, pulsing (`0.6 + 0.4·sin(t/400)`) when selected, a drop target, or a search hit. Processing items draw as a hollow ring whose alpha pulses. Selected node draws at 1.5× radius. Chunks are plain dots (`rgba(hue, .75)`); search hits become mint, 1.6× and get an orb glow.

### 8.4 Labels (the part people notice)

Copy the reference exactly; this was the most-iterated detail.

- **Hubs and root**: a gold stem line out of the orb's right edge, the name in light condensed type above it, and a slanted dark tag with the count + a gold raised unit below it.
- **Items (files)**: no name in the overview — only a small dark count badge at the top-right. Show the name only when selected or zoomed ≥2×.
- **Leaves**: no label.
- All label sizes are screen px divided by zoom `k`, so text stays constant on screen.

```ts
let family = ""
const font = (weight: number, px: number) => {
  if (!family && typeof document !== "undefined") family = getComputedStyle(document.documentElement).getPropertyValue("--font-condensed").trim()
  return `${weight} ${px}px ${family || "sans-serif"}, ui-sans-serif, system-ui`
}
const trunc = (s: string, n: number) => (s.length > n ? `${s.slice(0, n - 1)}…` : s)

/** Slanted tag (parallelogram): value + tiny gold unit raised after it, like "11 %" / "3 campaigns". */
function tag(ctx: CanvasRenderingContext2D, x: number, y: number, value: string, unit: string, k: number) {
  const h = 18 / k, skew = 5 / k
  ctx.font = font(400, 13 / k); const vw = ctx.measureText(value).width
  ctx.font = font(600, 8 / k); const uw = unit ? ctx.measureText(unit).width + 2 / k : 0
  const w = 14 / k + vw + uw
  ctx.beginPath(); ctx.moveTo(x + skew, y); ctx.lineTo(x + w + skew, y); ctx.lineTo(x + w, y + h); ctx.lineTo(x, y + h); ctx.closePath()
  ctx.fillStyle = "rgba(5,16,18,0.72)"; ctx.fill()
  ctx.strokeStyle = "rgba(255,255,255,0.12)"; ctx.lineWidth = 1 / k; ctx.stroke()
  ctx.textBaseline = "alphabetic"
  ctx.font = font(400, 13 / k); ctx.fillStyle = "rgba(255,255,255,0.92)"; ctx.fillText(value, x + 7 / k, y + 13.5 / k)
  if (unit) { ctx.font = font(600, 8 / k); ctx.fillStyle = COLOR.gold; ctx.fillText(unit, x + 7 / k + vw + 2 / k, y + 8 / k) }
}

/** Small dark rounded count badge. */
function badge(ctx: CanvasRenderingContext2D, x: number, y: number, text: string, k: number) {
  ctx.font = font(600, 10 / k)
  const w = ctx.measureText(text).width + 8 / k, h = 14 / k
  ctx.fillStyle = "rgba(4,12,13,0.82)"; ctx.beginPath(); ctx.roundRect(x, y - h, w, h, 3 / k); ctx.fill()
  ctx.fillStyle = "rgba(255,255,255,0.9)"; ctx.textBaseline = "alphabetic"; ctx.fillText(text, x + 4 / k, y - 3.5 / k)
}

function hubLabel(ctx: CanvasRenderingContext2D, n: GNode, r: number, k: number, unit: string) {
  const x0 = n.x! + r * 1.55, y = n.y!, end = x0 + 92 / k
  const g = ctx.createLinearGradient(n.x! + r, 0, end, 0)
  g.addColorStop(0, rgba(COLOR.gold, 0.85)); g.addColorStop(1, rgba(COLOR.gold, 0))
  ctx.strokeStyle = g; ctx.lineWidth = 1 / k
  ctx.beginPath(); ctx.moveTo(n.x! + r * 1.05, y); ctx.lineTo(end, y); ctx.stroke()
  ctx.font = font(300, (n.kind === "workspace" ? 17 : 15) / k); ctx.fillStyle = "rgba(236,246,244,0.92)"; ctx.textBaseline = "alphabetic"
  ctx.fillText(trunc(n.label, 24), x0 + 4 / k, y - 6 / k)
  tag(ctx, x0 + 8 / k, y + 5 / k, String(n.count), unit, k)
}
// item: badge(ctx, n.x + r * 0.9, n.y - r * 1.1, String(n.count), k)
// item name when selected/zoomed: font(selected ? 400 : 300, (selected ? 14 : 12) / k) at x = n.x + max(r * 2.3, 20 / k), y = n.y + 4 / k
```

### 8.5 Edges and scene order

- Spoke: linear gradient gold `0.9 → 0.15` alpha from root to target, width `1.8/k`.
- Branch: teal at 0.45 alpha, width `1/k`.
- Leaf: mint at 0.25 alpha, width `0.6/k`, dashed `[2/k, 2/k]`.
- While a search is active, non-hit targets fade to 0.25 and non-hit nodes to 0.35 alpha; `dim` context nodes 0.35–0.4.
- Draw order: links, then nodes sorted chunk → doc → hub → root (so big orbs sit on top), each node's label right after its orb.
- World transform: `ctx.setTransform(dpr*k, 0, 0, dpr*k, dpr*(w/2 + v.x), dpr*(h/2 + v.y))`; canvas sized by `devicePixelRatio`; screen↔world: `sx = w/2 + v.x + x*k`.

### 8.6 Layout and performance (index.tsx)

```ts
const DIST = { spoke: 0, branch: 70, leaf: 16 }
const CHARGE = { workspace: -600, campaign: -260, doc: -60, chunk: -4 }
const ringRadius = (n: number) => Math.max(220, Math.round((n * 90) / (2 * Math.PI)))   // hub ring

forceSimulation(nodes)
  .force("link", forceLink(links).id((n) => n.id)
    .distance((l) => (l.kind === "spoke" ? ringRadius(hubs) * 0.5 : DIST[l.kind]))
    .strength((l) => (l.kind === "spoke" ? 0.05 : 0.8)))
  .force("charge", forceManyBody().strength((n) => CHARGE[n.kind]))
  .force("collide", forceCollide((n) => n.r + 3))
  .force("ring", forceRadial(ringRadius(hubs)).strength((n) => (n.kind === "campaign" && n.parent ? 0.9 : 0)))
```

- **Seed** from deterministic radial-tree positions; first load starts every node at 5% of its seed so the graph "bursts" out of the centre.
- **Carry positions across rebuilds** by id from the previous simulation; only new nodes get seeded (near their parent). If most nodes were carried, start at `alpha(0.3)`. Without this, every click re-explodes the graph.
- **Tree mode** = pin every node to its seed (`fx/fy`), `s.stop(); s.alpha(0)`. **Force mode** clears `fx/fy` on non-root nodes (otherwise switching back from tree stays frozen).
- **One rAF loop** that keeps running only while `alpha > alphaMin` or something pulses (selection, drop target, search hits, processing items) — idle CPU ≈ 0. Cancel the frame on unmount. Keep the latest `kick` in a ref for the ResizeObserver.
- **Reduced motion**: `sim.stop(); sim.tick(300)` synchronously, no burst, no pulse.
- **ResizeObserver** with `[]` deps; only reassign `canvas.width/height` when the rounded size changed (assigning clears the bitmap → flicker).
- **Wheel zoom** via a native `addEventListener("wheel", h, { passive: false })` + `preventDefault()` (React's `onWheel` is passive, the page scrolls). Zoom around the pointer; clamp k to [0.25, 6].
- **Scale guard**: render chunk dots only if total ≤ 1,500, or zoom ≥ 1.6, or for the selected item; the API can omit chunks above 5,000 and send counts only.
- **Fit**: bounding box of all nodes (or the selection and its children) + 120px padding; `k = clamp(min(w/bw, h/bh), 0.25, 4)`; return early if the canvas has no size yet.

### 8.7 Interaction

- **Hover does nothing visual.** Only set `canvas.style.cursor = hitTest(...) ? "pointer" : ""`.
- **Click** selects (opens the detail panel, §6.5); click on empty space deselects. A pointer that moved >3px is a pan, not a click.
- **Pan** by dragging empty space; **zoom** with the wheel or toolbar ±.
- **Admin drag a node to another hub** (re-parent): pin it under the pointer, `sim.alpha(0.1).restart()`, highlight the hub under the pointer (glow). Hit-test hubs **excluding the dragged node and its children** — otherwise the dragged node is always nearest and you can never drop it anywhere. Only re-parent when released actually over a hub; release over empty space does nothing. Handle `pointercancel`/`lostpointercapture` (unpin). PATCH, toast, reload.
- **Drop files** onto a hub to add them there (empty space = root); `dragover` only `preventDefault()`s when `dataTransfer.types.includes("Files")`.
- **Search** (ask bar): results carry the matching leaf id; highlight those leaves, pan them into the clear area above the results card (`y offset −22% of height`) and zoom to ≥2× so 2px dots are visible.
- **Members vs admins**: hide every write control (add, drag, drop, edit, delete) for non-admins; the API enforces it too.
- **Empty state**: just the root orb and a centered muted hint ("Drop files here…" for admins, "No knowledge yet" otherwise).

---

## 9. Lessons learned (don't repeat these)

1. **No informative hover on dense graphs.** A magnifying "reticle lens" (from the reference video) covered node names and overlapped neighbours; a smaller reticle still collided with labels. The user asked to remove hover entirely and put all details in a click-to-open side panel. Keep hover = cursor only.
2. **Don't label every node.** Names next to every orb looked messy. Hubs get the stem + name + tag treatment; items get a count badge; leaves nothing.
3. **Don't use internal jargon in UI copy.** "1 part" (a chunk) confused the user. Show things people care about: status, size, dates, counts of real objects.
4. **Toolbar heights**: standalone buttons beside padded groups must be 46px, or the row looks uneven. `aspect-square` + `self-stretch` doesn't work (height from stretch isn't definite).
5. **TooltipProvider** is required once shadcn sidebar buttons get `tooltip`.
6. **Sticky table headers** don't work inside shadcn's `overflow-x-auto` wrapper; don't ship a fake one.
7. **Dark text on tinted glass**: `text-primary-foreground` on a gold-tinted bubble was invisible; use `text-muted-foreground`/`foreground`.
8. **White inputs lose** to `dark:bg-input/30`; style inputs dark.
9. **Glow uses currentColor** — set the matching `text-` colour on glowing segments.
10. **Two `background`s on one element** (`glass` utility + a `bg-` class) resolve by CSS order; use an inline style for the one that must win.
11. **Overlays on top of a canvas compete for space**: when a side panel opens, move or hide bottom bars instead of shrinking them to slivers; pan the selected node clear of the panel.
12. **Don't reformat files** with a formatter that doesn't match the repo's style; it buries the real diff.
13. **Fake numbers**: no delta pills or sparklines without real historical data.

---

## 10. Build checklist

1. Install deps, fonts on `<html class="dark …">`, remove theme toggle/`next-themes`, sonner `theme="dark"`.
2. Paste tokens, background, star field, utilities, reduced-motion rule.
3. Override Button, Card, Table row; add `fx.tsx` (Pill, Stat, GlowBar).
4. Shell: icon rail sidebar (+ TooltipProvider), transparent inset, mobile glass bar, content wrapper with `data-bleed` opt-out, sticky glass PageHeader.
5. Sweep hard-coded colours with the grep in §3 and map by meaning.
6. Dashboards: KPI row, glowing segmented bar + tiles, live pills, trace timeline.
7. Graph (if needed): layout.ts (+ test), render.ts, overlays, detail panel, index.tsx per §8.
8. Verify in a browser at desktop and 375px: no horizontal scroll, focus rings visible, panels opaque, toolbar heights equal, idle CPU stops, reduced motion static, admin-only controls hidden for members.
