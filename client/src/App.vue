<template>
  <div class="app">
    <aside class="sidebar" :class="{ collapsed }">
      <div class="sidebar-brand">
        <div class="brand-mark">{{ t('nav.companyName').charAt(0) }}</div>
        <div class="brand-text">
          <span class="brand-name">{{ t('nav.companyName') }}</span>
          <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
      </div>

      <nav class="sidebar-nav">
        <router-link
          v-for="item in navItems"
          :key="item.to"
          :to="item.to"
          class="nav-link"
          :class="{ active: $route.path === item.to }"
          :title="item.label"
          :aria-label="item.label"
        >
          <NavIcon :name="item.icon" />
          <span class="nav-label">{{ item.label }}</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <button
          class="collapse-btn"
          type="button"
          @click="toggleCollapsed"
          :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
          :aria-label="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
        >
          <NavIcon name="collapse" :size="18" class="collapse-icon" :class="{ flipped: collapsed }" />
          <span class="nav-label">Collapse</span>
        </button>
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="app-main">
      <header class="topbar">
        <FilterBar />
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
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'
import NavIcon from './components/icons/NavIcon.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher,
    NavIcon
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // --- Sidebar collapse (additive layout state) ---
    const MOBILE_QUERY = '(max-width: 1023px)'
    const storedCollapsed = () => localStorage.getItem('sidebar-collapsed') === 'true'
    const collapsed = ref(storedCollapsed())
    let mediaQuery = null

    const applyMediaState = (event) => {
      collapsed.value = event.matches ? true : storedCollapsed()
    }

    const toggleCollapsed = () => {
      collapsed.value = !collapsed.value
      localStorage.setItem('sidebar-collapsed', String(collapsed.value))
    }

    const navItems = computed(() => [
      { to: '/', icon: 'overview', label: t('nav.overview') },
      { to: '/inventory', icon: 'inventory', label: t('nav.inventory') },
      { to: '/orders', icon: 'orders', label: t('nav.orders') },
      { to: '/spending', icon: 'finance', label: t('nav.finance') },
      { to: '/demand', icon: 'demand', label: t('nav.demandForecast') },
      { to: '/reports', icon: 'reports', label: 'Reports' }
    ])

    onMounted(() => {
      mediaQuery = window.matchMedia(MOBILE_QUERY)
      applyMediaState(mediaQuery)
      mediaQuery.addEventListener('change', applyMediaState)
    })

    onUnmounted(() => {
      if (mediaQuery) mediaQuery.removeEventListener('change', applyMediaState)
    })

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
      collapsed,
      toggleCollapsed,
      navItems
    }
  }
}
</script>

<style>
:root {
  /* Surfaces & text (slate / gray) */
  --color-ink: #0f172a;
  --color-body: #334155;
  --color-muted: #64748b;
  --color-line: #e2e8f0;
  --color-line-strong: #cbd5e1;
  --color-surface: #ffffff;
  --color-bg: #f8fafc;
  --color-bg-hover: #f1f5f9;

  /* Accent */
  --color-accent: #2563eb;
  --color-accent-hover: #1d4ed8;
  --color-accent-soft: #eff6ff;

  /* Status */
  --color-success: #059669;
  --color-info: #2563eb;
  --color-warning: #d97706;
  --color-danger: #dc2626;

  /* Spacing scale (4 / 8 / 12 / 16 / 24 / 32 / 48 / 64) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 24px;
  --space-6: 32px;
  --space-7: 48px;
  --space-8: 64px;

  /* Radius */
  --radius-sm: 6px;
  --radius: 10px;
  --radius-lg: 14px;
  --radius-full: 9999px;

  /* Elevation */
  --shadow-hover: 0 4px 12px rgba(2, 6, 23, 0.06);

  /* Layout */
  --sidebar-width: 240px;
  --sidebar-width-collapsed: 64px;
  --content-max: 1600px;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--color-bg);
  color: var(--color-body);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ============ App shell ============ */
.app {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  width: var(--sidebar-width);
  flex-shrink: 0;
  background: var(--color-surface);
  border-right: 1px solid var(--color-line);
  display: flex;
  flex-direction: column;
  position: sticky;
  top: 0;
  height: 100vh;
  transition: width 0.18s ease;
  z-index: 100;
}

.sidebar.collapsed {
  width: var(--sidebar-width-collapsed);
}

/* Brand */
.sidebar-brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--color-line);
  min-height: 73px;
  overflow: hidden;
}

.sidebar.collapsed .sidebar-brand {
  justify-content: center;
  padding: var(--space-5) 0;
}

.brand-mark {
  width: 32px;
  height: 32px;
  flex-shrink: 0;
  border-radius: var(--radius-sm);
  background: linear-gradient(135deg, var(--color-accent) 0%, #1e40af 100%);
  color: #fff;
  font-weight: 700;
  font-size: 0.95rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.brand-name {
  font-size: 0.95rem;
  font-weight: 700;
  color: var(--color-ink);
  letter-spacing: -0.015em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-subtitle {
  font-size: 0.75rem;
  color: var(--color-muted);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Nav */
.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: var(--space-3) var(--space-2);
  overflow-x: hidden;
  overflow-y: auto;
}

.nav-link {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: 10px var(--space-4);
  border-radius: var(--radius-sm);
  color: var(--color-muted);
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 500;
  white-space: nowrap;
  position: relative;
  transition: background 0.15s ease, color 0.15s ease;
}

.nav-link:hover {
  background: var(--color-bg-hover);
  color: var(--color-ink);
}

.nav-link.active {
  background: var(--color-accent-soft);
  color: var(--color-accent);
}

.nav-link.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 6px;
  bottom: 6px;
  width: 3px;
  border-radius: 0 2px 2px 0;
  background: var(--color-accent);
}

.sidebar.collapsed .nav-link {
  justify-content: center;
  padding: var(--space-2);
}

.sidebar.collapsed .nav-label,
.sidebar.collapsed .brand-text {
  display: none;
}

/* Footer */
.sidebar-footer {
  border-top: 1px solid var(--color-line);
  padding: var(--space-3);
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
}

.sidebar.collapsed .sidebar-footer {
  align-items: center;
  padding: var(--space-3) var(--space-2);
}

.collapse-btn {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  width: 100%;
  padding: var(--space-2) var(--space-3);
  background: none;
  border: none;
  border-radius: var(--radius-sm);
  color: var(--color-muted);
  font: inherit;
  font-size: 0.85rem;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.15s ease, color 0.15s ease;
}

.collapse-btn:hover {
  background: var(--color-bg-hover);
  color: var(--color-ink);
}

.sidebar.collapsed .collapse-btn {
  justify-content: center;
  padding: var(--space-2);
}

.collapse-icon {
  transition: transform 0.18s ease;
}

.collapse-icon.flipped {
  transform: rotate(180deg);
}

/* Footer child components: trim to fit collapsed rail (global styles, App.vue <style> is not scoped) */
.sidebar.collapsed .sidebar-footer .profile-name,
.sidebar.collapsed .sidebar-footer .language-label,
.sidebar.collapsed .sidebar-footer .language-button .chevron,
.sidebar.collapsed .sidebar-footer .profile-button .chevron {
  display: none;
}

.sidebar-footer .language-button,
.sidebar-footer .profile-button {
  width: 100%;
}

.sidebar.collapsed .sidebar-footer .language-button,
.sidebar.collapsed .sidebar-footer .profile-button {
  width: auto;
  padding: var(--space-2);
}

.sidebar-footer .dropdown-menu {
  right: auto;
  left: 0;
  bottom: calc(100% + 0.5rem);
  top: auto;
}

/* ============ Main column ============ */
.app-main {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
}

.topbar {
  position: sticky;
  top: 0;
  z-index: 90;
  background: var(--color-surface);
  border-bottom: 1px solid var(--color-line);
}

.main-content {
  flex: 1;
  width: 100%;
  max-width: var(--content-max);
  margin: 0;
  padding: var(--space-6);
}

/* ============ Shared components ============ */
.page-header {
  margin-bottom: var(--space-5);
}

.page-header h2 {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--color-ink);
  margin-bottom: var(--space-1);
  letter-spacing: -0.02em;
}

.page-header p {
  color: var(--color-muted);
  font-size: 0.938rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-5);
  margin-bottom: var(--space-5);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--space-5);
  border-radius: var(--radius);
  border: 1px solid var(--color-line);
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.stat-card:hover {
  border-color: var(--color-line-strong);
  box-shadow: var(--shadow-hover);
}

.stat-label {
  color: var(--color-muted);
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: var(--space-2);
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: var(--color-ink);
  letter-spacing: -0.02em;
}

.stat-card.warning .stat-value {
  color: var(--color-warning);
}

.stat-card.success .stat-value {
  color: var(--color-success);
}

.stat-card.danger .stat-value {
  color: var(--color-danger);
}

.stat-card.info .stat-value {
  color: var(--color-info);
}

.card {
  background: var(--color-surface);
  border-radius: var(--radius);
  padding: var(--space-5);
  border: 1px solid var(--color-line);
  margin-bottom: var(--space-5);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-3);
  border-bottom: 1px solid var(--color-line);
}

.card-title {
  font-size: 1.125rem;
  font-weight: 650;
  color: var(--color-ink);
  letter-spacing: -0.02em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--color-bg);
  border-top: 1px solid var(--color-line);
  border-bottom: 1px solid var(--color-line);
}

th {
  text-align: left;
  padding: var(--space-2) var(--space-3);
  font-weight: 600;
  color: var(--color-muted);
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: var(--space-2) var(--space-3);
  border-top: 1px solid var(--color-line);
  color: var(--color-body);
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: var(--color-bg);
}

.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: var(--radius-sm);
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  text-align: center;
  padding: var(--space-7);
  color: var(--color-muted);
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: var(--space-4);
  border-radius: var(--radius-sm);
  margin: var(--space-4) 0;
  font-size: 0.938rem;
}

@media (max-width: 640px) {
  .main-content {
    padding: var(--space-4);
  }
}
</style>
