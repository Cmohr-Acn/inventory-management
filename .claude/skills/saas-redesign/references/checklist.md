# Final verification checklist

Run through this **before** declaring the redesign done. Skipping items here is what produces "the sidebar looks great but Reports is broken" handoffs.

## Build and dev server

- [ ] `npm run dev` starts without console errors or warnings
- [ ] `npm run build` produces a clean dist (no Vue compiler warnings, no missing-import errors)
- [ ] No new dependencies added to `client/package.json` (confirm via `git diff client/package.json`)

## Per-route walkthrough

Open each route and confirm:

| Route        | Active sidebar item lights up | Page header renders | Cards/KPI/tables intact | Modals reachable |
|--------------|-------------------------------|---------------------|-------------------------|------------------|
| `/` (Overview) | [ ]                         | [ ]                 | [ ]                     | [ ]              |
| `/inventory` | [ ]                           | [ ]                 | [ ]                     | [ ]              |
| `/orders`    | [ ]                           | [ ]                 | [ ]                     | [ ]              |
| `/spending`  | [ ]                           | [ ]                 | [ ]                     | [ ]              |
| `/demand`    | [ ]                           | [ ]                 | [ ]                     | [ ]              |
| `/reports`   | [ ]                           | [ ]                 | [ ]                     | [ ]              |

## Layout integrity

- [ ] Sidebar sticks to viewport while main content scrolls
- [ ] Main header sticks below the top of the main column (FilterBar visible while scrolling)
- [ ] No horizontal scroll at 1920px, 1440px, 1280px viewport widths
- [ ] No horizontal scroll at 960px (sidebar should be in collapsed mode)
- [ ] FilterBar's selects line up vertically centered with the header strip
- [ ] LanguageSwitcher and ProfileMenu align right in the header strip without wrapping
- [ ] Active nav-item background and text color use `var(--brand-soft)` and `var(--brand)` — no hardcoded blue

## Spacing rhythm

- [ ] Main content uses `var(--space-6)` (24px) gutter padding
- [ ] Page header has `var(--space-6)` margin-bottom
- [ ] Cards have `var(--space-4)` (16px) inner padding
- [ ] Sidebar nav items are 36px tall, 4px gap between items

## Token discipline

- [ ] Sidebar.vue uses `var(--*)` for every color value — grep for hex codes (`#`) inside `<style scoped>`. Allowed: only inside SVG `fill="none"` or `stroke="currentColor"`.
- [ ] App.vue's restructured rules use tokens, not raw hex
- [ ] No new `1.25rem`, `0.625rem`, `0.875rem`, `0.813rem` values introduced — snap to `var(--space-*)`

## Functional regressions

- [ ] Filter changes still update views (pick a filter, see the page data change)
- [ ] Reset filters button still clears state
- [ ] ProfileMenu opens, both modal triggers fire (`show-profile-details`, `show-tasks`)
- [ ] LanguageSwitcher still toggles between languages
- [ ] Backend at `localhost:8001` still serves data — sidebar redesign is frontend-only

## Visual confirmation

- [ ] Take a Dashboard screenshot at 1440px width and include in the handoff message
- [ ] Diff against `docs/dashboard-screenshot.png` if helpful — note that screenshot is the *old* layout, so it's a sanity check, not a target

## i18n

- [ ] All sidebar labels render in the active language (toggle to a non-default language and verify)
- [ ] If `nav.reports` was added to `useI18n.js`, confirm the key exists in every language map. If not added, the literal "Reports" rendering is acceptable as a stopgap — call it out in the handoff.

## Memory / persistence

- [ ] If sidebar collapse state should persist across reloads, persist it via `localStorage`. If transient is fine, skip — but mention it in the handoff so the user can request persistence if they want it.
