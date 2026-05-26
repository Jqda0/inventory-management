---
name: vue-saas-redesign
description: Redesigns a Vue 3 application's UI into a modern SaaS-style interface. Use when the user wants to replace a top navigation bar with a vertical sidebar, add consistent spacing, modernize the visual design, or apply a professional SaaS look-and-feel to this inventory management app.
---

# Skill: Vue 3 SaaS UI Redesign — Vertical Sidebar Layout

This skill covers how to redesign this inventory management app from its current horizontal top-nav layout into a modern SaaS-style interface with a **vertical navigation sidebar** on the left, consistent spacing tokens, and a polished professional look.

---

## Codebase Context

**App entry:** `client/src/App.vue` — owns the shell layout (nav, FilterBar, main content, modals).  
**Router:** `client/src/main.js` — 6 routes: `/`, `/inventory`, `/orders`, `/spending`, `/demand`, `/reports`.  
**Nav translation keys:** `t('nav.overview')`, `t('nav.inventory')`, `t('nav.orders')`, `t('nav.finance')`, `t('nav.demandForecast')` — plus `t('nav.companyName')` and `t('nav.subtitle')` for the logo.  
**FilterBar:** `client/src/components/FilterBar.vue` — currently sticky at `top: 70px` (the nav height). Must be updated when layout changes.  
**ProfileMenu:** `client/src/components/ProfileMenu.vue` — emits `show-profile-details` and `show-tasks`.  
**LanguageSwitcher:** `client/src/components/LanguageSwitcher.vue`.  
**No CSS variables exist yet** — all tokens are hardcoded hex values in scoped `<style>` blocks.

---

## Design System

### Color Tokens (introduce as CSS custom properties on `:root`)

```css
:root {
  /* Surfaces */
  --color-bg:           #f8fafc;
  --color-surface:      #ffffff;
  --color-sidebar-bg:   #0f172a;   /* deep slate — sidebar background */
  --color-sidebar-hover:#1e293b;   /* slightly lighter for hover */
  --color-sidebar-active:#2563eb;  /* blue pill for active item */

  /* Text */
  --color-text-primary: #0f172a;
  --color-text-body:    #1e293b;
  --color-text-muted:   #64748b;
  --color-text-subtle:  #94a3b8;
  --color-text-sidebar: #cbd5e1;   /* light on dark sidebar */
  --color-text-sidebar-active: #ffffff;

  /* Borders */
  --color-border:       #e2e8f0;
  --color-border-hover: #cbd5e1;
  --color-border-subtle:#f1f5f9;

  /* Semantic */
  --color-primary:      #2563eb;
  --color-primary-light:#eff6ff;
  --color-success:      #059669;
  --color-warning:      #ea580c;
  --color-danger:       #dc2626;
  --color-info:         #2563eb;

  /* Layout */
  --sidebar-width:      240px;
  --sidebar-collapsed:  64px;
  --topbar-height:      0px;       /* removed in sidebar layout */
  --content-max-width:  1600px;
  --content-padding:    1.5rem 2rem;

  /* Spacing scale */
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
  --radius-xl: 16px;

  /* Shadows */
  --shadow-card:  0 1px 3px rgba(0,0,0,0.06), 0 1px 2px rgba(0,0,0,0.04);
  --shadow-hover: 0 4px 12px rgba(0,0,0,0.08);
  --shadow-modal: 0 20px 60px rgba(0,0,0,0.15);
}
```

### Typography (unchanged font stack)
```css
body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  -webkit-font-smoothing: antialiased;
}
```

---

## Layout Architecture

Replace the single-column (`flex-direction: column`) app shell with a two-column grid:

```
┌──────────────────────────────────────────┐
│  Sidebar (240px, fixed)  │  Main area     │
│  ─────────────────────── │  ───────────   │
│  Logo + subtitle         │  TopBar        │
│                          │  (profile,     │
│  Nav items:              │   lang, search)│
│  • Overview              │  ───────────   │
│  • Inventory             │  FilterBar     │
│  • Orders                │  ───────────   │
│  • Finance               │  <router-view> │
│  • Demand Forecast       │                │
│  • Reports               │                │
│                          │                │
│  ─────────────────────── │                │
│  Profile / Settings      │                │
└──────────────────────────────────────────┘
```

### App shell CSS structure

```css
.app {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  grid-template-rows: 1fr;
  min-height: 100vh;
}

.sidebar {
  grid-column: 1;
  grid-row: 1;
  background: var(--color-sidebar-bg);
  position: fixed;
  top: 0; left: 0;
  width: var(--sidebar-width);
  height: 100vh;
  display: flex;
  flex-direction: column;
  z-index: 200;
  overflow-y: auto;
}

.content-area {
  grid-column: 2;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin-left: var(--sidebar-width);
}

.top-bar {
  background: var(--color-surface);
  border-bottom: 1px solid var(--color-border);
  height: 56px;
  display: flex;
  align-items: center;
  padding: 0 var(--space-8);
  gap: var(--space-4);
  position: sticky;
  top: 0;
  z-index: 100;
}

.main-content {
  flex: 1;
  padding: var(--content-padding);
  max-width: var(--content-max-width);
  width: 100%;
}
```

---

## `App.vue` Rewrite Pattern

### Template structure

```vue
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
        <div class="top-bar-actions">
          <!-- slot for breadcrumb or actions -->
        </div>
      </header>
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <!-- Modals (unchanged) -->
    <ProfileDetailsModal ... />
    <TasksModal ... />
  </div>
</template>
```

### navItems computed property

```js
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
```

---

## Sidebar Styles

```css
/* === SIDEBAR === */
.sidebar {
  background: var(--color-sidebar-bg);
  position: fixed;
  top: 0; left: 0;
  width: var(--sidebar-width);
  height: 100vh;
  display: flex;
  flex-direction: column;
  z-index: 200;
  overflow-y: auto;
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-6) var(--space-5);
  border-bottom: 1px solid rgba(255,255,255,0.08);
  flex-shrink: 0;
}

.logo-mark {
  width: 36px; height: 36px;
  background: var(--color-primary);
  border-radius: var(--radius-sm);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.875rem;
  color: white;
  flex-shrink: 0;
}

.logo-name {
  display: block;
  font-size: 0.9rem;
  font-weight: 700;
  color: white;
  letter-spacing: -0.01em;
  line-height: 1.2;
}

.logo-sub {
  display: block;
  font-size: 0.7rem;
  color: var(--color-text-subtle);
  line-height: 1.3;
  margin-top: 2px;
}

.sidebar-nav {
  flex: 1;
  padding: var(--space-4) var(--space-3);
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sidebar-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) var(--space-3);
  border-radius: var(--radius-sm);
  color: var(--color-text-sidebar);
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  transition: background 0.15s ease, color 0.15s ease;
  white-space: nowrap;
}

.sidebar-item:hover {
  background: var(--color-sidebar-hover);
  color: white;
}

.sidebar-item.active {
  background: var(--color-sidebar-active);
  color: var(--color-text-sidebar-active);
}

.sidebar-icon {
  display: flex;
  align-items: center;
  flex-shrink: 0;
  opacity: 0.85;
}

.sidebar-item.active .sidebar-icon,
.sidebar-item:hover .sidebar-icon {
  opacity: 1;
}

.sidebar-footer {
  padding: var(--space-4) var(--space-3);
  border-top: 1px solid rgba(255,255,255,0.08);
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
}
```

---

## FilterBar Update

The FilterBar's sticky offset must change from `top: 70px` (old nav height) to `top: 56px` (the new top-bar height). Update `.filters-bar` in `client/src/components/FilterBar.vue`:

```css
/* BEFORE */
.filters-bar {
  position: sticky;
  top: 70px;
  z-index: 90;
}

/* AFTER */
.filters-bar {
  position: sticky;
  top: 56px;
  z-index: 90;
}
```

---

## ProfileMenu & LanguageSwitcher Adaptations

When these components move into `.sidebar-footer` (on a dark background), their dropdowns must open **upward** (`bottom: 100%` instead of `top: 100%`) to avoid clipping off-screen.

In `ProfileMenu.vue`, update the dropdown positioning:

```css
/* BEFORE (top-nav position — opens downward) */
.dropdown-menu {
  top: calc(100% + 8px);
  right: 0;
}

/* AFTER (sidebar-footer position — opens upward) */
.dropdown-menu {
  bottom: calc(100% + 8px);
  left: 0;
  right: auto;
}
```

Apply the same `bottom`/`left` adjustment to `.dropdown-menu` in `LanguageSwitcher.vue`.

Also, update the profile button to suit a dark sidebar — remove any light-background border and use white text:

```css
.profile-button {
  width: 100%;
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-sm);
  background: transparent;
  border: none;
  color: var(--color-text-sidebar);
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: var(--space-3);
}
.profile-button:hover {
  background: var(--color-sidebar-hover);
  color: white;
}
```

---

## Content Area & Top Bar

The content area's top bar replaces the old nav's right-side controls. It shows the current page title and has a right-aligned slot for future actions (search, notifications).

```css
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
}

.top-bar-title {
  font-size: 1rem;
  font-weight: 600;
  color: var(--color-text-primary);
}

.content-area {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin-left: var(--sidebar-width);
  background: var(--color-bg);
}

.main-content {
  flex: 1;
  padding: var(--content-padding);
  max-width: var(--content-max-width);
  width: 100%;
}
```

---

## Global Style Improvements

Replace the `.top-nav` block and update global `.app` in `App.vue`'s `<style>` (unscoped):

```css
/* === GLOBAL RESET === */
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: var(--color-bg);
  color: var(--color-text-body);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app {
  display: flex;
  min-height: 100vh;
}
```

Cards, tables, badges — keep existing styles but replace hardcoded tokens with CSS variable references where feasible:

```css
.card {
  background: var(--color-surface);
  border-radius: var(--radius-md);
  padding: var(--space-5);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-card);
  margin-bottom: var(--space-5);
}

.card:hover {
  border-color: var(--color-border-hover);
  box-shadow: var(--shadow-hover);
}
```

---

## Step-by-Step Implementation Order

1. **Add CSS custom properties** to the `:root` block at the top of `App.vue`'s global `<style>`.
2. **Rewrite `App.vue` template** — replace `<header class="top-nav">` with `<aside class="sidebar">` and add `.content-area` wrapper.
3. **Add `navItems` computed + `currentPageTitle` computed** to `App.vue`'s `setup()`. Import `useRoute` from `vue-router`.
4. **Add sidebar and content-area CSS** to `App.vue`'s global `<style>`, removing the old `.top-nav` / `.nav-container` / `.nav-tabs` rules.
5. **Update FilterBar** sticky offset: `top: 70px` → `top: 56px`.
6. **Update ProfileMenu dropdown** to open upward (`bottom: calc(100% + 8px)`) and adapt button styling for dark background.
7. **Update LanguageSwitcher dropdown** to open upward.
8. **Verify** with `npm run dev` in `client/` — check all 6 routes, filter bar stickiness, and dropdown positions.

---

## Common Pitfalls

- **Sidebar clips dropdowns:** If `ProfileMenu` or `LanguageSwitcher` dropdowns appear cut off, add `overflow: visible` to `.sidebar` or move the dropdowns to a portal. Alternatively, use `position: fixed` for the dropdown itself.
- **FilterBar offset wrong:** Always set `top` to the exact pixel height of the top bar (56px in this design). If you change the top-bar height, update FilterBar too.
- **`v-html` for icons:** Use `v-html` on a wrapper `<span>` to render inline SVG icon strings. This is safe since the SVG content is hardcoded in the component, not from user input.
- **`useRoute` import:** Add `import { useRoute, useRouter } from 'vue-router'` and call `const route = useRoute()` inside `setup()` to access `$route.path` in script.
- **Dark sidebar text contrast:** Sidebar text must be `#cbd5e1` (light) against `#0f172a` (dark), not the page's default `#1e293b` body color.
- **`margin-left` vs grid:** Using `margin-left: var(--sidebar-width)` on `.content-area` (with sidebar `position: fixed`) is simpler than CSS Grid for this fixed-sidebar pattern and avoids grid track sizing pitfalls.

---

## Responsive Considerations (optional enhancement)

For narrower viewports, collapse the sidebar to icon-only mode:

```css
@media (max-width: 1024px) {
  .sidebar { width: var(--sidebar-collapsed); }
  .logo-text, .sidebar-label { display: none; }
  .content-area { margin-left: var(--sidebar-collapsed); }
  .filters-bar { top: 56px; } /* top-bar height unchanged */
}
```

Add a toggle button in the top bar to programmatically toggle a `.collapsed` class on `.sidebar` and adjust `.content-area`'s `margin-left` accordingly via a reactive ref in `App.vue`.
