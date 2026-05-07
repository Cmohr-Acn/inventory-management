---
name: saas-redesign
description: "Redesign this Vue 3 client into a modern SaaS-style UI — vertical left sidebar replacing the top nav, consistent spacing rhythm, and a polished neutral surface. Source-anchored: audit current App.vue + FilterBar before changing anything. Triggers: 'redesign the UI', 'modernize the layout', 'switch to a sidebar', 'make it look more SaaS-y'."
metadata:
    category: ui-redesign
    target: client/src (Vue 3 + Vite)
    version: 1.0.0
---

# saas-redesign

Restructure the inventory-management Vue 3 client from its current top-nav-plus-filter-bar layout into a modern SaaS-style shell with a vertical sidebar on the left, a clean header strip on top of the main column, and consistent spacing across all views.

This skill is scoped to **this project**. It assumes the codebase shape under `client/src/` (App.vue with global styles, FilterBar component, six views, three composables, modals at app-shell level). Don't apply it to unrelated Vue projects without re-reading the audit step.

---

## Trigger when

- The user asks to "redesign", "modernize", "polish", or "improve the look" of the UI
- The user asks to switch from a top nav to a sidebar
- The user mentions "SaaS-style", "Linear-style", "Notion-style", "modern dashboard" layout
- The user asks for consistent spacing or design-system cleanup across views

## Don't use this skill for

- Single-component tweaks (just edit the component directly)
- Adding a brand-new view (use the vue-expert agent)
- Backend-only changes
- Building a new design system from scratch for a non-Vue project

---

## Hard rules

These are non-negotiable. Violating them produces a half-redesign that breaks the app.

1. **Audit before editing.** Read `client/src/App.vue` and `client/src/components/FilterBar.vue` in full before proposing any layout. The current sticky-positioning math (`top: 70px` on FilterBar) and global styles (`.stat-card`, `.card`, `.badge`, `.page-header`, `.kpi-*`) are referenced from views — breaking them silently regresses every page.
2. **Preserve i18n keys.** Every nav label currently uses `t('nav.companyName')`, `t('nav.overview')`, `t('nav.inventory')`, etc. Replace markup, not translation calls. If a key doesn't exist for a new label, add it to the i18n composable rather than hardcoding English.
3. **Preserve composables.** `useAuth`, `useFilters`, `useI18n` are the source of truth. The redesigned shell still consumes them — don't refactor state into the layout.
4. **Preserve modals at app shell.** `ProfileDetailsModal` and `TasksModal` mount in App.vue and are wired to `ProfileMenu` events. Keep that wiring; just move where ProfileMenu sits in the new chrome.
5. **No new dependencies.** No icon library, no Tailwind, no UI kit. Use inline SVGs and scoped CSS, like the rest of the project.
6. **No hardcoded colors in components.** Centralize tokens in App.vue's global `<style>` block as CSS custom properties. Sidebar and views consume `var(--*)`, not raw hex.
7. **Render and verify before declaring done.** Run `npm run dev`, open `http://localhost:3000`, walk all six routes, and take a screenshot. Don't claim "done" without visual confirmation across views.

---

## The 6-step workflow

### Step 1: Audit the current shell

Read in full:
- `client/src/App.vue` — top nav structure, global styles, modal mounts, font/color constants
- `client/src/components/FilterBar.vue` — sticky position depending on nav height
- `client/src/main.js` — router routes (drives the sidebar items)
- `client/src/composables/useI18n.js` — existing translation keys for nav labels

Skim (one each is enough):
- `client/src/views/Dashboard.vue` — sample of how views consume global styles
- `client/src/components/ProfileMenu.vue` — what events it emits (you'll re-mount it in the new shell)

Produce a one-paragraph audit summary listing:
- Current nav routes and their i18n keys
- Global style classes views depend on (so you don't accidentally rename them)
- Sticky-position math currently in play
- Where modals attach

### Step 2: Propose the layout in text and get an OK

Before writing any Vue or CSS, write a short proposal:

- Sidebar width (default 240px expanded, 64px collapsed)
- Sidebar contents top-to-bottom (logo → primary nav → divider → secondary actions → user/profile area at the bottom)
- Where FilterBar lives (sticky inside the main column, **not** inside the sidebar)
- Where LanguageSwitcher and ProfileMenu move (header strip on the main column or sidebar footer)
- Page padding rhythm (24px gutter, 32px between major sections)
- One ASCII sketch of the layout

Wait for the user's "yes" or a redirect. Skip the wait only if the user has already explicitly told you to proceed without confirmation; even then, write the proposal as the first paragraph of your build reply.

### Step 3: Drop in design tokens

Add CSS custom properties to `App.vue`'s global (non-scoped) `<style>` block — see `references/design-tokens.md` for the full set. These replace ad-hoc hex values throughout the redesign.

Keep the existing global utility classes (`.stat-card`, `.card`, `.badge`, `.page-header`, `.kpi-*`) — they're consumed by views. Only add tokens; don't rename or remove existing classes in this step.

### Step 4: Build the Sidebar component

Create `client/src/components/Sidebar.vue` from `references/sidebar-template.vue`. The template:
- Uses i18n via `useI18n`
- Uses inline SVG icons (Heroicons-style outline)
- Driven by a static nav-items array that matches `main.js` routes
- Renders `<router-link>` for each item, active-class styling driven by route match
- Has a collapse toggle and a footer slot for ProfileMenu
- Scoped styles consuming tokens from App.vue

### Step 5: Restructure App.vue

Replace the `<template>` to use a flex shell:
- `aside.sidebar` (left, sticky, full viewport height)
- `div.main-column` (right, flex-1, contains a slim header strip with FilterBar + LanguageSwitcher + ProfileMenu, then `<main>` with router-view)

Update the global `<style>` block:
- Replace `.top-nav`, `.nav-container`, `.nav-tabs`, `.logo`, `.subtitle` with the new shell classes
- Keep `.stats-grid`, `.stat-card`, `.card`, `.card-header`, `.badge`, `.kpi-*`, table styles — views depend on these
- Update `.main-content` padding to use new tokens

See `references/app-layout.md` for the before/after structure.

### Step 6: Verify across all six views

Run the dev server and open the browser:

```powershell
# Start (if not already running)
cd client; npm run dev
# Open
Start-Process "http://localhost:3000"
```

Walk every route — Overview (`/`), Inventory, Orders, Spending, Demand, Reports — and confirm:

- Sidebar renders and active state highlights the current route
- FilterBar still sticks correctly under the main-column header
- KPI cards, tables, and badges look unchanged inside views (no global style breakage)
- Modals still open from ProfileMenu
- No horizontal scroll at 1280px and 1920px viewport widths
- Collapsed-sidebar mode works (if implemented)

Take a screenshot of at least the Dashboard view and include it in the final reply. Use `references/checklist.md` to confirm nothing was missed.

---

## Anti-patterns

- **Don't wipe and rebuild.** Augment the existing shell. The current global styles, modal mounts, and composables are the load-bearing scaffolding — replace the chrome only.
- **Don't strip page headers from views.** Each view has its own `<div class="page-header">` block. Leave them. Only the *outer* shell changes.
- **Don't reposition FilterBar inside the sidebar.** Filters are page-level state; they belong in the main column header strip.
- **Don't introduce a router-link `to` change without checking `main.js`.** The six routes are fixed: `/`, `/inventory`, `/orders`, `/demand`, `/spending`, `/reports`.
- **Don't switch fonts.** Project uses Inter via `body` font-family. Keep it.

---

## File map

```
.claude/skills/saas-redesign/
├── SKILL.md                              # this file
└── references/
    ├── design-tokens.md                  # CSS custom properties for the new design system
    ├── sidebar-template.vue              # drop-in Sidebar.vue scaffold
    ├── app-layout.md                     # App.vue before/after restructure pattern
    └── checklist.md                      # final verification list
```

When invoked, the skill loads SKILL.md plus any references it needs for the active step. Reference files are not pre-loaded — read them only when you reach the corresponding step.
