<template>
  <div class="app">
    <!-- Left sidebar -->
    <aside class="sidebar">
      <div class="sidebar-logo">
        <div class="logo-mark">CC</div>
        <div class="logo-text">
          <span class="logo-name">{{ t('nav.companyName') }}</span>
          <span class="logo-sub">{{ t('nav.subtitle') }}</span>
        </div>
      </div>

      <nav class="sidebar-nav">
        <router-link
          v-for="item in navItems"
          :key="item.to"
          :to="item.to"
          class="sidebar-item"
          :class="{ active: $route.path === item.to }"
        >
          <span class="sidebar-icon" v-html="item.icon"></span>
          <span class="sidebar-label">{{ item.label }}</span>
        </router-link>
      </nav>

      <div class="sidebar-filters">
        <FilterBar />
      </div>

      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <!-- Right content area -->
    <div class="content-area">
      <header class="top-bar">
        <div class="top-bar-title">{{ currentPageTitle }}</div>
        <div class="top-bar-actions"></div>
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
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const route = useRoute()
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const navItems = computed(() => [
      {
        to: '/',
        label: t('nav.overview'),
        icon: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>`
      },
      {
        to: '/inventory',
        label: t('nav.inventory'),
        icon: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/></svg>`
      },
      {
        to: '/orders',
        label: t('nav.orders'),
        icon: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2"/><rect x="9" y="3" width="6" height="4" rx="2"/><line x1="9" y1="12" x2="15" y2="12"/><line x1="9" y1="16" x2="13" y2="16"/></svg>`
      },
      {
        to: '/spending',
        label: t('nav.finance'),
        icon: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>`
      },
      {
        to: '/demand',
        label: t('nav.demandForecast'),
        icon: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/></svg>`
      },
      {
        to: '/reports',
        label: 'Reports',
        icon: `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/></svg>`
      }
    ])

    const currentPageTitle = computed(() => {
      const item = navItems.value.find(n => n.to === route.path)
      return item ? item.label : ''
    })

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
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)
        if (isMockTask) {
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) currentUser.value.tasks.splice(index, 1)
        } else {
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)
        if (mockTask) {
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) apiTasks.value[index] = updatedTask
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      navItems,
      currentPageTitle,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
/* === CSS DESIGN TOKENS === */
:root {
  /* Surfaces */
  --color-bg:             #f8fafc;
  --color-surface:        #ffffff;
  --color-sidebar-bg:     #0f172a;
  --color-sidebar-hover:  #1e293b;
  --color-sidebar-active: #2563eb;

  /* Text */
  --color-text-primary:        #0f172a;
  --color-text-body:           #1e293b;
  --color-text-muted:          #64748b;
  --color-text-subtle:         #94a3b8;
  --color-text-sidebar:        #94a3b8;
  --color-text-sidebar-active: #ffffff;

  /* Borders */
  --color-border:        #e2e8f0;
  --color-border-hover:  #cbd5e1;
  --color-border-subtle: #f1f5f9;

  /* Semantic */
  --color-primary:       #2563eb;
  --color-primary-light: #eff6ff;
  --color-success:       #059669;
  --color-warning:       #ea580c;
  --color-danger:        #dc2626;

  /* Layout */
  --sidebar-width:        260px;
  --content-max-width:    1600px;

  /* Spacing */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.25rem;
  --space-6: 1.5rem;
  --space-8: 2rem;

  /* Radii */
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 12px;

  /* Shadows */
  --shadow-card:  0 1px 3px rgba(0,0,0,0.06), 0 1px 2px rgba(0,0,0,0.04);
  --shadow-hover: 0 4px 12px rgba(0,0,0,0.08);
  --shadow-modal: 0 20px 60px rgba(0,0,0,0.15);
}

/* === GLOBAL RESET === */
*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--color-bg);
  color: var(--color-text-body);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* === APP SHELL === */
.app {
  display: flex;
  min-height: 100vh;
}

/* === SIDEBAR === */
.sidebar {
  background: var(--color-sidebar-bg);
  position: fixed;
  top: 0;
  left: 0;
  width: var(--sidebar-width);
  height: 100vh;
  display: flex;
  flex-direction: column;
  z-index: 200;
  overflow-y: auto;
  overflow-x: hidden;
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-5) var(--space-5);
  border-bottom: 1px solid rgba(255, 255, 255, 0.07);
  flex-shrink: 0;
}

.logo-mark {
  width: 34px;
  height: 34px;
  background: var(--color-primary);
  border-radius: var(--radius-sm);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.8rem;
  color: white;
  flex-shrink: 0;
  letter-spacing: 0.01em;
}

.logo-text {
  min-width: 0;
}

.logo-name {
  display: block;
  font-size: 0.875rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.01em;
  line-height: 1.25;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.logo-sub {
  display: block;
  font-size: 0.688rem;
  color: var(--color-text-subtle);
  line-height: 1.3;
  margin-top: 2px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* === SIDEBAR NAV === */
.sidebar-nav {
  flex-shrink: 0;
  padding: var(--space-3) var(--space-3);
  display: flex;
  flex-direction: column;
  gap: 1px;
}

.sidebar-nav::before {
  content: 'NAVIGATION';
  display: block;
  font-size: 0.625rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  color: rgba(148, 163, 184, 0.5);
  padding: var(--space-3) var(--space-2) var(--space-2);
}

.sidebar-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: 0.563rem var(--space-3);
  border-radius: var(--radius-sm);
  color: var(--color-text-sidebar);
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  transition: background 0.15s ease, color 0.15s ease;
  white-space: nowrap;
}

.sidebar-item:hover {
  background: rgba(255, 255, 255, 0.06);
  color: #e2e8f0;
}

.sidebar-item.active {
  background: var(--color-sidebar-active);
  color: var(--color-text-sidebar-active);
}

.sidebar-icon {
  display: flex;
  align-items: center;
  flex-shrink: 0;
  opacity: 0.7;
}

.sidebar-item:hover .sidebar-icon {
  opacity: 0.9;
}

.sidebar-item.active .sidebar-icon {
  opacity: 1;
}

/* === SIDEBAR FILTERS SECTION === */
.sidebar-filters {
  flex: 1;
  border-top: 1px solid rgba(255, 255, 255, 0.07);
  padding: var(--space-3) 0;
  overflow-y: auto;
}

/* === SIDEBAR FOOTER === */
.sidebar-footer {
  flex-shrink: 0;
  padding: var(--space-3) var(--space-3);
  border-top: 1px solid rgba(255, 255, 255, 0.07);
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
}

/* === CONTENT AREA === */
.content-area {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin-left: var(--sidebar-width);
  background: var(--color-bg);
  width: calc(100% - var(--sidebar-width));
}

/* === TOP BAR === */
.top-bar {
  background: var(--color-surface);
  border-bottom: 1px solid var(--color-border);
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 var(--space-8);
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 1px 0 var(--color-border);
}

.top-bar-title {
  font-size: 0.938rem;
  font-weight: 600;
  color: var(--color-text-primary);
  letter-spacing: -0.01em;
}

.top-bar-actions {
  display: flex;
  align-items: center;
  gap: var(--space-3);
}

/* === MAIN CONTENT === */
.main-content {
  flex: 1;
  max-width: var(--content-max-width);
  width: 100%;
  padding: var(--space-6) var(--space-8);
}

/* === PAGE HEADERS === */
.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--color-text-primary);
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--color-text-muted);
  font-size: 0.938rem;
}

/* === STAT CARDS === */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--space-5);
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--space-5);
  border-radius: var(--radius-md);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-card);
  transition: box-shadow 0.2s ease, border-color 0.2s ease;
}

.stat-card:hover {
  border-color: var(--color-border-hover);
  box-shadow: var(--shadow-hover);
}

.stat-label {
  color: var(--color-text-muted);
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  margin-bottom: var(--space-3);
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: var(--color-text-primary);
  letter-spacing: -0.03em;
  line-height: 1;
}

.stat-card.warning .stat-value { color: var(--color-warning); }
.stat-card.success .stat-value { color: var(--color-success); }
.stat-card.danger  .stat-value { color: var(--color-danger); }
.stat-card.info    .stat-value { color: var(--color-primary); }

/* === CARDS === */
.card {
  background: var(--color-surface);
  border-radius: var(--radius-md);
  padding: var(--space-5);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-card);
  margin-bottom: var(--space-5);
  transition: box-shadow 0.2s ease, border-color 0.2s ease;
}

.card:hover {
  border-color: var(--color-border-hover);
  box-shadow: var(--shadow-hover);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-4);
  border-bottom: 1px solid var(--color-border);
}

.card-title {
  font-size: 1rem;
  font-weight: 700;
  color: var(--color-text-primary);
  letter-spacing: -0.015em;
}

/* === TABLES === */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--color-bg);
  border-top: 1px solid var(--color-border);
  border-bottom: 1px solid var(--color-border);
}

th {
  text-align: left;
  padding: 0.563rem 0.875rem;
  font-weight: 600;
  color: #475569;
  font-size: 0.688rem;
  text-transform: uppercase;
  letter-spacing: 0.07em;
}

td {
  padding: 0.625rem 0.875rem;
  border-top: 1px solid var(--color-border-subtle);
  color: #334155;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.12s ease;
}

tbody tr:hover {
  background: var(--color-bg);
}

/* === BADGES === */
.badge {
  display: inline-block;
  padding: 0.25rem 0.625rem;
  border-radius: var(--radius-sm);
  font-size: 0.688rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.badge.success    { background: #d1fae5; color: #065f46; }
.badge.warning    { background: #fed7aa; color: #92400e; }
.badge.danger     { background: #fecaca; color: #991b1b; }
.badge.info       { background: #dbeafe; color: #1e40af; }
.badge.increasing { background: #d1fae5; color: #065f46; }
.badge.decreasing { background: #fecaca; color: #991b1b; }
.badge.stable     { background: #e0e7ff; color: #3730a3; }
.badge.high       { background: #fecaca; color: #991b1b; }
.badge.medium     { background: #fed7aa; color: #92400e; }
.badge.low        { background: #dbeafe; color: #1e40af; }

/* === UTILITY === */
.loading {
  text-align: center;
  padding: 3rem;
  color: var(--color-text-muted);
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: var(--space-4);
  border-radius: var(--radius-sm);
  margin: var(--space-4) 0;
  font-size: 0.875rem;
}
</style>
