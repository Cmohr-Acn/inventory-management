# App.vue restructure pattern

The redesign keeps App.vue's responsibilities (shell layout, modal mounts, task state, i18n setup) but swaps the chrome from "top-nav + filter-bar + main" to "sidebar + main column".

## Before — current structure

```
<div class="app">                       ← flex column
  <header class="top-nav">              ← sticky top, 70px tall
    .nav-container (max 1600, padded)
      .logo (companyName + subtitle)
      <nav class="nav-tabs">            ← horizontal router-links
      <LanguageSwitcher />
      <ProfileMenu />
  <FilterBar />                         ← sticky at top:70px
  <main class="main-content">           ← max 1600, padded
    <router-view />
  <ProfileDetailsModal />
  <TasksModal />
```

## After — target structure

```
<div class="app-shell">                 ← flex row
  <Sidebar :collapsed @toggle-collapse>
    <template #footer>
      <ProfileMenu />
    </template>

  <div class="main-column">             ← flex 1, min-width 0
    <header class="main-header">        ← sticky, header-height (56px)
      .main-header__inner (gutter padding)
        <FilterBar :embedded="true" />  ← (FilterBar's own sticky removed when embedded)
        .main-header__actions
          <LanguageSwitcher />

    <main class="main-content">
      <router-view />

  <ProfileDetailsModal />               ← still here, unchanged
  <TasksModal />
```

## App.vue diff (template)

```html
<!-- BEFORE -->
<header class="top-nav">
  <div class="nav-container">
    <div class="logo">…</div>
    <nav class="nav-tabs">
      <router-link to="/">…</router-link>
      …
    </nav>
    <LanguageSwitcher />
    <ProfileMenu @show-profile-details="…" @show-tasks="…" />
  </div>
</header>
<FilterBar />
<main class="main-content">
  <router-view />
</main>

<!-- AFTER -->
<Sidebar :collapsed="sidebarCollapsed" @toggle-collapse="sidebarCollapsed = !sidebarCollapsed">
  <template #footer>
    <ProfileMenu @show-profile-details="showProfileDetails = true" @show-tasks="showTasks = true" />
  </template>
</Sidebar>

<div class="main-column">
  <header class="main-header">
    <FilterBar />
    <div class="main-header__actions">
      <LanguageSwitcher />
    </div>
  </header>
  <main class="main-content">
    <router-view />
  </main>
</div>
```

## App.vue diff (script)

```js
import Sidebar from './components/Sidebar.vue'

// Inside setup():
const sidebarCollapsed = ref(false)

// Add sidebarCollapsed to the return object.
```

Don't drop existing imports/refs. Drop only `LanguageSwitcher` import duplicates if you move it. Tasks state, modal state, i18n binding all stay.

## App.vue diff (global style block)

The global `<style>` block keeps everything below the navigation rules. Remove or rewrite **only** these blocks:

- `.top-nav`, `.nav-container`, `.logo`, `.subtitle`, `.nav-tabs`, `.nav-tabs a`, `.nav-tabs a:hover`, `.nav-tabs a.active`, `.nav-tabs a.active::after`
- `.main-content` (rewrite — see below)

Keep untouched:
- The `*` reset and `body` rules (apply tokens via `body { background: var(--surface-canvas); color: var(--ink-primary); }`)
- `.page-header`, `.page-header h2`, `.page-header p`
- `.stats-grid`, `.stat-card`, all `.stat-card.*` variants, `.stat-label`, `.stat-value`
- `.card`, `.card-header`, `.card-title`
- `.table-container`, `table`, `thead`, `th`, `td`, `tbody tr`, `tbody tr:hover`
- All `.badge.*` variants (success, warning, danger, info, increasing, decreasing, stable, high, medium, low)
- `.loading`, `.error`

### New shell rules

Add these after the body rule:

```css
.app-shell {
  display: flex;
  min-height: 100vh;
  background: var(--surface-canvas);
}

.main-column {
  flex: 1;
  min-width: 0;            /* prevents flex children from forcing horizontal scroll */
  display: flex;
  flex-direction: column;
}

.main-header {
  position: sticky;
  top: 0;
  z-index: 50;
  background: var(--surface-chrome);
  border-bottom: 1px solid var(--border-default);
  display: flex;
  align-items: center;
  gap: var(--space-4);
  padding: 0 var(--space-6);
  height: var(--header-height);
}

.main-header__actions {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  margin-left: auto;
}

.main-content {
  flex: 1;
  width: 100%;
  max-width: var(--content-max-width);
  margin: 0 auto;
  padding: var(--space-6);
}
```

## FilterBar adjustment

`FilterBar.vue` currently sets `position: sticky; top: 70px` because it sat below a 70px top-nav. In the new layout it lives **inside** the main-header, so its sticky positioning becomes redundant and conflicts with the parent header's sticky behavior.

Options, in order of preference:

1. **Drop FilterBar's outer wrapper styling.** Strip the `.filters-bar` background, border-bottom, sticky, and padding, and let the new `.main-header` provide all the chrome. Keep only the `.filters-grid` layout and the `.filter-select` styling.
2. **Add an `embedded` prop** that conditionally drops the wrapper styles. Use this if you want FilterBar to remain usable as a standalone bar elsewhere.

The skill should default to option 1 unless the user objects.

## Edge cases

- **Modals at app shell.** ProfileDetailsModal and TasksModal must stay inside `.app-shell` (not inside `.main-column`) so they can overlay the entire viewport. Place them as siblings to Sidebar/main-column.
- **Mobile / narrow viewports.** Below ~960px the sidebar should auto-collapse. Add a media query in App.vue's global style block:

  ```css
  @media (max-width: 960px) {
    .sidebar { width: var(--sidebar-width-collapsed); }
    .sidebar :deep(.nav-item-label),
    .sidebar :deep(.brand-text) { display: none; }
  }
  ```

  (Or pass `:collapsed="true"` based on a window-size watcher — pick whichever is simpler given the project's other reactivity patterns.)

- **Dashboard's KPI grid.** Some KPI grids hardcode `grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))`. With a 240px sidebar, the available content width drops by 240px — verify the grid still wraps cleanly at 1280px viewport. If it overflows, lower the minmax floor to 240px or expand to `1fr 1fr 1fr` at narrower breakpoints.
