---
name: frontend-design
description: Redesign a Vue 3 + Vite application's UI into a modern, polished SaaS-style interface — vertical left navigation sidebar (replacing a top nav bar), consistent spacing scale, clean card layouts, and a professional look. Use this skill when asked to redesign, restyle, modernize, or apply a SaaS look to a Vue 3 frontend, or to convert a top nav to a sidebar.
---

# Frontend SaaS Redesign (Vue 3)

Guidance and a checklist for transforming a plain Vue 3 + Vite app into a modern SaaS-style
interface. This skill is **guidance, not templates** — read it, adapt the rules to the target
app's existing structure, and keep changes minimal and reviewable.

The Factory Inventory Management System (this repo) is the worked example throughout, but the
rules apply to any Vue 3 + Composition API + Vite project.

## Non-negotiables

- **Keep it reversible and scoped.** Restructure layout and styling only. Do not touch API
  calls, router route definitions, data fetching, or business logic.
- **One global stylesheet is the source of truth.** In this repo that is the `<style>` block in
  `client/src/App.vue`. Keep design tokens and shared component classes there; view files should
  rely on those classes, not redefine them.
- **No emojis in the UI.** Use text labels or inline SVG icons.
- **Delegate `.vue` edits to the `vue-expert` subagent** (per this repo's `CLAUDE.md`), passing
  it the specific rules from this skill.

## Design system (slate / gray)

Reuse the existing palette. Define these as CSS custom properties on `:root` and reference them
everywhere instead of hard-coded hex values.

| Token | Value | Use |
|---|---|---|
| `--color-ink` | `#0f172a` | Primary headings, active text |
| `--color-body` | `#334155` | Body text, table cells |
| `--color-muted` | `#64748b` | Secondary text, labels, inactive nav |
| `--color-line` | `#e2e8f0` | Borders, dividers |
| `--color-surface` | `#ffffff` | Cards, sidebar, top bar |
| `--color-bg` | `#f8fafc` | App background, table headers |
| `--color-bg-hover` | `#f1f5f9` | Hover fills |
| `--color-accent` | `#2563eb` | Active nav, primary buttons, focus |
| `--color-accent-soft` | `#eff6ff` | Active nav background |
| Status | green `#059669` · blue `#2563eb` · amber `#d97706` · red `#dc2626` | Badges, stat cards |

**Spacing scale** (use only these; expose as `--space-1`…`--space-8`):
`4, 8, 12, 16, 24, 32, 48, 64` px.

**Radius:** `--radius-sm: 6px`, `--radius: 10px`, `--radius-lg: 14px`.

**Elevation:** flat by default (1px `--color-line` border). One hover shadow only:
`0 4px 12px rgba(2, 6, 23, 0.06)`.

**Typography:** system font stack already in `body`. Sizes: page title `1.75rem/700`,
card title `1.125rem/650`, body `0.875rem`, label/overline `0.75rem/600 uppercase
letter-spacing 0.05em`. Headings use `letter-spacing: -0.02em`.

## Layout: left sidebar shell

Target structure for `App.vue`:

```
.app (display:flex, min-height:100vh)
├── .sidebar (fixed width, full height, vertical)
│   ├── .sidebar-brand      → company name + subtitle (hidden when collapsed)
│   ├── .sidebar-nav        → router-links, stacked vertically, icon + label
│   └── .sidebar-footer     → LanguageSwitcher + ProfileMenu
└── .app-main (flex:1, min-width:0, display:flex, flex-direction:column)
    ├── .topbar             → page context left, FilterBar right (sticky, top:0)
    └── .main-content       → <router-view />
```

Rules:
- Sidebar width `240px` expanded. Background `--color-surface`, right border `--color-line`.
- Nav links: full-width, `padding: 10px 16px`, `--radius-sm`, `--color-muted` text,
  `gap: 12px` between icon and label. Hover → `--color-bg-hover`. Active
  (`router-link-active` / exact match for `/`) → `--color-accent` text on
  `--color-accent-soft`, with a `3px` accent bar on the left edge.
- Keep the existing 6 routes and their labels/i18n keys exactly. Do not add or remove routes.
- `.main-content` keeps the current `max-width` behavior but left-aligned within `.app-main`
  (the sidebar now provides the left gutter). Padding `--space-6`.
- Move `FilterBar` into `.topbar`. It keeps its component API unchanged — only where it renders
  and its container styling change.

## Collapsible sidebar (icons-only mode)

- A toggle button pinned in `.sidebar-brand` (or top of nav) flips a `collapsed` ref.
- Collapsed width `64px`: brand text and nav labels hidden, icons centered, footer controls
  reduced to icon triggers.
- Persist the choice in `localStorage` (`sidebar-collapsed`).
- Auto-collapse under `1024px` viewport via a `matchMedia` listener; allow manual override
  above that breakpoint.
- Transition `width 0.18s ease`. Content must not reflow-jump — animate width only.
- Provide `title`/`aria-label` on links so collapsed icons stay accessible.

## Icons

No icon library dependency. Add a tiny local set of inline SVG components (16–20px,
`stroke: currentColor`, `stroke-width: 1.75`, `fill: none`) — one per nav item plus a
collapse/expand chevron. Keep them in `client/src/components/icons/` as single-file components
or a small `NavIcon.vue` with a `name` prop.

## Cards & tables (shared classes in App.vue)

- `.card`: `--color-surface`, `1px --color-line`, `--radius`, `padding: --space-5`,
  `margin-bottom: --space-5`.
- `.card-header`: flex space-between, bottom border `--color-line`, `padding-bottom: --space-3`.
- `.stat-card`: same shell; `.stat-label` overline; `.stat-value` `2rem/700 --color-ink`;
  modifier classes `.success/.warning/.danger/.info` recolor `.stat-value` only.
- Tables: `thead` on `--color-bg`, `th` overline style, `td` `0.875rem --color-body` with
  `1px --color-line` top border, `tbody tr:hover` → `--color-bg`.
- `.badge`: `--radius-sm`, `0.75rem/600 uppercase`, soft bg + strong fg per status
  (green/blue/amber/red and the existing trend/priority variants — keep every existing
  badge modifier class name).

## Redesign checklist

1. **Read first.** `client/src/App.vue` (template + full `<style>`), `client/src/main.js`
   (routes), `client/src/components/FilterBar.vue`, `ProfileMenu.vue`, `LanguageSwitcher.vue`,
   and skim each `client/src/views/*.vue` for classes they depend on.
2. **Tokens.** Add the `:root` custom properties + spacing/radius vars to the top of the
   `App.vue` `<style>`. Do not delete existing shared classes yet.
3. **Shell.** Rewrite the `App.vue` template to the sidebar structure above. Keep all
   component imports, props, events, modals, and script logic unchanged.
4. **Sidebar styles.** Add `.sidebar*` and `.topbar` styles. Convert `.nav-tabs` rules into
   `.sidebar-nav` rules (vertical).
5. **Icons.** Add the inline SVG icon components and wire one to each nav link.
6. **Refactor shared classes** to use the new tokens (find/replace hex → `var(--…)`), keeping
   class names identical so views keep working.
7. **Collapsible behavior.** Add the `collapsed` ref, toggle, `localStorage`, and `matchMedia`.
8. **Verify.** Start the app (`.claude/commands/start.md` / `./scripts/start.sh`), open
   `http://localhost:3000`, and check every route: nav active state, FilterBar in the topbar,
   collapse/expand, and the sub-1024px auto-collapse. Take screenshots.
9. **Diff review.** Run `git diff --stat` — changes should be concentrated in `App.vue` plus
   the new icon files, with at most cosmetic container tweaks elsewhere.

## What "done" looks like

- Left sidebar with vertical nav; no top nav bar remains.
- Filters live in a sticky topbar.
- Every page renders with consistent spacing, one card style, one table style.
- Sidebar collapses to icons-only, remembers the setting, and auto-collapses on narrow screens.
- No route, API, or business-logic changes; `npm run build` succeeds.
