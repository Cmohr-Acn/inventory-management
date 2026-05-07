# Design tokens for the SaaS-style redesign

Drop these CSS custom properties into the **global** (non-scoped) `<style>` block of `client/src/App.vue`, at the top before any rules. The existing rules then reference them via `var(--*)` instead of raw hex.

## Token block to add

```css
:root {
  /* Surface — the canvas, the chrome, the cards */
  --surface-canvas: #f8fafc;       /* page background (slate-50) */
  --surface-chrome: #ffffff;       /* sidebar, header strip, cards */
  --surface-raised: #ffffff;       /* modals, popovers */
  --surface-muted:  #f1f5f9;       /* hover state on subtle interactive elements */
  --surface-sunken: #f8fafc;       /* inset areas, table headers */

  /* Border — three weights, neutral */
  --border-subtle:  #f1f5f9;
  --border-default: #e2e8f0;
  --border-strong:  #cbd5e1;

  /* Ink — text and icon color, four weights */
  --ink-primary:   #0f172a;        /* headings, key values */
  --ink-secondary: #334155;        /* body */
  --ink-tertiary:  #64748b;        /* labels, helper text */
  --ink-quaternary:#94a3b8;        /* placeholders, disabled */
  --ink-inverse:   #ffffff;        /* text on filled brand surfaces */

  /* Brand — single accent, used sparingly for active state, primary CTAs, focus rings */
  --brand:         #2563eb;        /* keep current blue or swap to Accenture purple #A100FF if requested */
  --brand-hover:   #1d4ed8;
  --brand-soft:    #eff6ff;        /* tinted background for active sidebar item */
  --brand-soft-border: #dbeafe;
  --focus-ring:    rgba(37, 99, 235, 0.18);

  /* Status — green / amber / red, used in badges and KPI tones */
  --success:       #059669;
  --success-soft:  #d1fae5;
  --warning:       #ea580c;
  --warning-soft:  #fed7aa;
  --danger:        #dc2626;
  --danger-soft:   #fecaca;
  --info:          #3b82f6;
  --info-soft:     #dbeafe;

  /* Radius — three steps, no decoration radii */
  --radius-sm: 6px;                /* selects, badges, small buttons */
  --radius-md: 8px;                /* cards, inputs, sidebar items */
  --radius-lg: 12px;               /* modal, KPI cards */

  /* Spacing — 4px base, named for the SaaS rhythm */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;                 /* default page gutter */
  --space-8: 32px;                 /* between major sections */
  --space-12: 48px;                /* page top padding */

  /* Shadow — only one, for raised surfaces (popovers, modals) */
  --shadow-sm: 0 1px 2px 0 rgba(15, 23, 42, 0.04);
  --shadow-md: 0 4px 12px 0 rgba(15, 23, 42, 0.06);
  --shadow-lg: 0 12px 32px -8px rgba(15, 23, 42, 0.18);

  /* Layout constants */
  --sidebar-width:           240px;
  --sidebar-width-collapsed: 64px;
  --header-height:           56px;
  --content-max-width:       1440px;

  /* Motion */
  --motion-fast: 120ms ease;
  --motion-base: 180ms ease;
}
```

## Mapping from existing hex values

When restructuring App.vue, replace the current ad-hoc colors with tokens:

| Current value     | Replace with              | Used by                       |
|-------------------|---------------------------|-------------------------------|
| `#f8fafc`         | `var(--surface-canvas)`   | `body` background, FilterBar  |
| `#ffffff` (chrome)| `var(--surface-chrome)`   | top-nav, cards                |
| `#e2e8f0`         | `var(--border-default)`   | borders throughout            |
| `#cbd5e1`         | `var(--border-strong)`    | hover borders, select borders |
| `#f1f5f9`         | `var(--border-subtle)`    | hover surfaces                |
| `#0f172a`         | `var(--ink-primary)`      | headings, stat values         |
| `#1e293b`         | `var(--ink-primary)`      | body text                     |
| `#334155`, `#475569` | `var(--ink-secondary)` | table cells, etc.             |
| `#64748b`         | `var(--ink-tertiary)`     | subtitles, labels             |
| `#2563eb`         | `var(--brand)`            | active nav, focused inputs    |
| `#eff6ff`         | `var(--brand-soft)`       | active nav background         |

Status badges already cluster well — leave their soft/strong pairs alone unless the user explicitly asks for a status palette change.

## Spacing rhythm

The redesign uses a 4px base with three load-bearing steps:

- `var(--space-6)` (24px) → page gutter, gap between top-level sections
- `var(--space-4)` (16px) → padding inside cards, gap between siblings inside a section
- `var(--space-2)` (8px) → tight gap (icon + label, badge padding)

Avoid arbitrary values like `1.25rem`, `0.625rem`, `0.875rem` that pepper the current code. Snap to the scale.

## Typography

Don't change the font (Inter stays). Snap to a small scale:

| Use                   | Size       | Weight | Line height |
|-----------------------|------------|--------|-------------|
| Page title (h2)       | 24px       | 600    | 1.25        |
| Section title (h3)    | 16px       | 600    | 1.4         |
| Body                  | 14px       | 400    | 1.55        |
| Label / helper        | 12px       | 500    | 1.4         |
| KPI value             | 32–36px    | 700    | 1.1         |
| Sidebar nav item      | 14px       | 500    | 1           |
