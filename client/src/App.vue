<template>
  <div class="app-shell">
    <Sidebar :collapsed="sidebarCollapsed" @toggle-collapse="sidebarCollapsed = !sidebarCollapsed">
      <template #footer>
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
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

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import Sidebar from './components/Sidebar.vue'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    Sidebar,
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])
    const sidebarCollapsed = ref(false)

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      sidebarCollapsed
    }
  }
}
</script>

<style>
/* ─── Design tokens ─────────────────────────────────────────────────────── */
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

  /* Brand — single accent, active state, primary CTAs, focus rings */
  --brand:         #2563eb;
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

  /* Radius — three steps */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;

  /* Spacing — 4px base */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;

  /* Shadow */
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

/* ─── Reset ─────────────────────────────────────────────────────────────── */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--surface-canvas);
  color: var(--ink-primary);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ─── Shell ─────────────────────────────────────────────────────────────── */
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

/* ─── Narrow viewport: auto-collapse sidebar ────────────────────────────── */
@media (max-width: 960px) {
  .sidebar { width: var(--sidebar-width-collapsed); }
  .sidebar :deep(.nav-item-label),
  .sidebar :deep(.brand-text) { display: none; }
}

/* ─── Page header ───────────────────────────────────────────────────────── */
.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: var(--ink-primary);
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--ink-tertiary);
  font-size: 0.938rem;
}

/* ─── Stats grid + stat cards ───────────────────────────────────────────── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: var(--surface-chrome);
  padding: 1.25rem;
  border-radius: var(--radius-lg);
  border: 1px solid var(--border-default);
  transition: all var(--motion-base);
}

.stat-card:hover {
  border-color: var(--border-strong);
  box-shadow: var(--shadow-md);
}

.stat-label {
  color: var(--ink-tertiary);
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: var(--ink-primary);
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value { color: var(--warning); }
.stat-card.success .stat-value { color: var(--success); }
.stat-card.danger  .stat-value { color: var(--danger); }
.stat-card.info    .stat-value { color: var(--info); }

/* ─── Card ──────────────────────────────────────────────────────────────── */
.card {
  background: var(--surface-chrome);
  border-radius: var(--radius-lg);
  padding: 1.25rem;
  border: 1px solid var(--border-default);
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid var(--border-default);
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--ink-primary);
  letter-spacing: -0.025em;
}

/* ─── Table chrome ──────────────────────────────────────────────────────── */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--surface-sunken);
  border-top: 1px solid var(--border-default);
  border-bottom: 1px solid var(--border-default);
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: var(--ink-secondary);
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid var(--border-subtle);
  color: var(--ink-secondary);
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color var(--motion-fast);
}

tbody tr:hover {
  background: var(--surface-sunken);
}

/* ─── Badges ────────────────────────────────────────────────────────────── */
.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: var(--radius-sm);
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success    { background: var(--success-soft); color: #065f46; }
.badge.warning    { background: var(--warning-soft); color: #92400e; }
.badge.danger     { background: var(--danger-soft);  color: #991b1b; }
.badge.info       { background: var(--info-soft);    color: #1e40af; }
.badge.increasing { background: var(--success-soft); color: #065f46; }
.badge.decreasing { background: var(--danger-soft);  color: #991b1b; }
.badge.stable     { background: #e0e7ff; color: #3730a3; }
.badge.high       { background: var(--danger-soft);  color: #991b1b; }
.badge.medium     { background: var(--warning-soft); color: #92400e; }
.badge.low        { background: var(--info-soft);    color: #1e40af; }

/* ─── States ────────────────────────────────────────────────────────────── */
.loading {
  text-align: center;
  padding: 3rem;
  color: var(--ink-tertiary);
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid var(--danger-soft);
  color: #991b1b;
  padding: 1rem;
  border-radius: var(--radius-md);
  margin: 1rem 0;
  font-size: 0.938rem;
}
</style>
