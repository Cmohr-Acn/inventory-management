<!--
  Sidebar.vue — drop-in template for the saas-redesign skill.
  Save to client/src/components/Sidebar.vue and import in App.vue.

  Assumes:
  - Tokens from references/design-tokens.md are in App.vue's global <style>.
  - useI18n composable exists and exposes t().
  - Routes match main.js: /, /inventory, /orders, /demand, /spending, /reports.
  - i18n keys exist: nav.companyName, nav.subtitle, nav.overview, nav.inventory,
    nav.orders, nav.finance, nav.demandForecast. (Reports has no key today —
    add nav.reports to useI18n.js or render the literal "Reports" until added.)
-->
<template>
  <aside class="sidebar" :class="{ 'sidebar--collapsed': collapsed }">
    <div class="sidebar-brand">
      <div class="brand-mark" aria-hidden="true">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/>
          <polyline points="3.27 6.96 12 12.01 20.73 6.96"/>
          <line x1="12" y1="22.08" x2="12" y2="12"/>
        </svg>
      </div>
      <div class="brand-text" v-show="!collapsed">
        <div class="brand-name">{{ t('nav.companyName') }}</div>
        <div class="brand-subtitle">{{ t('nav.subtitle') }}</div>
      </div>
    </div>

    <nav class="sidebar-nav" aria-label="Primary">
      <router-link
        v-for="item in items"
        :key="item.to"
        :to="item.to"
        class="nav-item"
        :class="{ 'nav-item--active': isActive(item.to) }"
        :title="collapsed ? item.label : undefined"
      >
        <span class="nav-item-icon" v-html="item.icon" aria-hidden="true"></span>
        <span class="nav-item-label" v-show="!collapsed">{{ item.label }}</span>
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <slot name="footer" />
      <button
        class="collapse-btn"
        @click="$emit('toggle-collapse')"
        :aria-label="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
      >
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" :style="{ transform: collapsed ? 'rotate(180deg)' : 'none' }">
          <polyline points="15 18 9 12 15 6"/>
        </svg>
      </button>
    </div>
  </aside>
</template>

<script>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from '../composables/useI18n'

// Inline SVG icons keep us off icon-library deps.
// Style: Heroicons outline, 1.5px stroke, 20x20 viewBox.
const ICON_OVERVIEW  = '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="9" rx="1.5"/><rect x="14" y="3" width="7" height="5" rx="1.5"/><rect x="14" y="12" width="7" height="9" rx="1.5"/><rect x="3" y="16" width="7" height="5" rx="1.5"/></svg>'
const ICON_INVENTORY = '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M21 8L12 3 3 8v8l9 5 9-5V8z"/><path d="M3 8l9 5 9-5"/><path d="M12 13v8"/></svg>'
const ICON_ORDERS    = '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M9 11V7a3 3 0 016 0v4"/><rect x="4" y="11" width="16" height="10" rx="1.5"/></svg>'
const ICON_FINANCE   = '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M12 1v22"/><path d="M17 5H9.5a3.5 3.5 0 000 7h5a3.5 3.5 0 010 7H6"/></svg>'
const ICON_DEMAND    = '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 17 9 11 13 15 21 7"/><polyline points="14 7 21 7 21 14"/></svg>'
const ICON_REPORTS   = '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="8" y1="13" x2="16" y2="13"/><line x1="8" y1="17" x2="13" y2="17"/></svg>'

export default {
  name: 'Sidebar',
  props: {
    collapsed: { type: Boolean, default: false }
  },
  emits: ['toggle-collapse'],
  setup() {
    const route = useRoute()
    const { t } = useI18n()

    const items = computed(() => [
      { to: '/',          label: t('nav.overview'),       icon: ICON_OVERVIEW },
      { to: '/inventory', label: t('nav.inventory'),      icon: ICON_INVENTORY },
      { to: '/orders',    label: t('nav.orders'),         icon: ICON_ORDERS },
      { to: '/spending',  label: t('nav.finance'),        icon: ICON_FINANCE },
      { to: '/demand',    label: t('nav.demandForecast'), icon: ICON_DEMAND },
      // Reports has no i18n key in useI18n.js today — add 'nav.reports' there
      // before relying on translation, or keep this literal label as a stopgap.
      { to: '/reports',   label: 'Reports',               icon: ICON_REPORTS }
    ])

    const isActive = (to) => {
      // Exact match for "/" so it doesn't light up on every route.
      return to === '/' ? route.path === '/' : route.path.startsWith(to)
    }

    return { t, items, isActive }
  }
}
</script>

<style scoped>
.sidebar {
  width: var(--sidebar-width);
  background: var(--surface-chrome);
  border-right: 1px solid var(--border-default);
  display: flex;
  flex-direction: column;
  position: sticky;
  top: 0;
  height: 100vh;
  flex-shrink: 0;
  transition: width var(--motion-base);
}
.sidebar--collapsed {
  width: var(--sidebar-width-collapsed);
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--border-subtle);
  height: var(--header-height);
}
.brand-mark {
  width: 28px;
  height: 28px;
  border-radius: var(--radius-sm);
  background: var(--brand);
  color: var(--ink-inverse);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}
.brand-mark svg { width: 16px; height: 16px; }
.brand-text { overflow: hidden; }
.brand-name {
  font-size: 14px;
  font-weight: 700;
  color: var(--ink-primary);
  letter-spacing: -0.01em;
  white-space: nowrap;
}
.brand-subtitle {
  font-size: 11px;
  color: var(--ink-tertiary);
  white-space: nowrap;
}

.sidebar-nav {
  flex: 1;
  padding: var(--space-3);
  display: flex;
  flex-direction: column;
  gap: 2px;
  overflow-y: auto;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-md);
  color: var(--ink-secondary);
  text-decoration: none;
  font-size: 14px;
  font-weight: 500;
  transition: background var(--motion-fast), color var(--motion-fast);
  white-space: nowrap;
  height: 36px;
}
.nav-item:hover {
  background: var(--border-subtle);
  color: var(--ink-primary);
}
.nav-item--active {
  background: var(--brand-soft);
  color: var(--brand);
}
.nav-item--active .nav-item-icon { color: var(--brand); }

.nav-item-icon {
  display: inline-flex;
  width: 20px;
  height: 20px;
  flex-shrink: 0;
  color: var(--ink-tertiary);
}
.nav-item-icon :deep(svg) { width: 100%; height: 100%; }

.nav-item-label { overflow: hidden; text-overflow: ellipsis; }

.sidebar--collapsed .nav-item {
  justify-content: center;
  padding: var(--space-2);
}

.sidebar-footer {
  border-top: 1px solid var(--border-subtle);
  padding: var(--space-3);
  display: flex;
  align-items: center;
  gap: var(--space-2);
}

.collapse-btn {
  margin-left: auto;
  width: 28px;
  height: 28px;
  border: 1px solid var(--border-default);
  background: var(--surface-chrome);
  border-radius: var(--radius-sm);
  color: var(--ink-tertiary);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background var(--motion-fast), color var(--motion-fast);
  flex-shrink: 0;
}
.collapse-btn:hover {
  background: var(--border-subtle);
  color: var(--ink-primary);
}
.collapse-btn svg {
  width: 14px;
  height: 14px;
  transition: transform var(--motion-base);
}
</style>
