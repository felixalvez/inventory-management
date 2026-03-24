<template>
  <aside class="sidebar" :class="{ collapsed }">
    <div class="sidebar-header">
      <div class="sidebar-logo">
        <svg width="28" height="28" viewBox="0 0 28 28" fill="none">
          <rect width="28" height="28" rx="6" fill="#3b82f6"/>
          <path d="M8 10H20M8 14H20M8 18H14" stroke="white" stroke-width="2" stroke-linecap="round"/>
        </svg>
        <span v-show="!collapsed" class="sidebar-title">{{ t('nav.companyName') }}</span>
      </div>
      <button class="collapse-btn" @click="$emit('toggle-collapse')" :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'">
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path v-if="collapsed" d="M7 4L12 9L7 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          <path v-else d="M11 4L6 9L11 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>
    </div>

    <nav class="sidebar-nav">
      <div v-show="!collapsed" class="nav-section-label">NAVIGATION</div>

      <router-link
        v-for="item in navItems"
        :key="item.path"
        :to="item.path"
        class="nav-item"
        :class="{ active: isActive(item.path) }"
        :title="collapsed ? item.label : ''"
      >
        <svg width="20" height="20" viewBox="0 0 20 20" fill="none" v-html="item.icon"></svg>
        <span v-show="!collapsed" class="nav-label">{{ item.label }}</span>
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <div v-show="!collapsed" class="footer-expanded">
        <LanguageSwitcher />
      </div>
      <button class="user-card" @click="$emit('show-profile-details')" :title="collapsed ? currentUser.name : ''">
        <div class="user-avatar">{{ getInitials(currentUser.name) }}</div>
        <div v-show="!collapsed" class="user-info">
          <div class="user-name">{{ currentUser.name }}</div>
          <div class="user-role">{{ currentUser.jobTitle }}</div>
        </div>
      </button>
    </div>
  </aside>
</template>

<script>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from '../composables/useI18n'
import { useAuth } from '../composables/useAuth'
import LanguageSwitcher from './LanguageSwitcher.vue'

export default {
  name: 'SidebarNav',
  components: { LanguageSwitcher },
  props: {
    collapsed: {
      type: Boolean,
      default: false
    }
  },
  emits: ['show-profile-details', 'show-tasks', 'toggle-collapse'],
  setup() {
    const route = useRoute()
    const { t } = useI18n()
    const { currentUser, getInitials } = useAuth()

    const navItems = computed(() => [
      {
        path: '/',
        label: t('nav.overview'),
        icon: '<rect x="3" y="3" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/><rect x="11" y="3" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/><rect x="3" y="11" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/><rect x="11" y="11" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/>'
      },
      {
        path: '/inventory',
        label: t('nav.inventory'),
        icon: '<path d="M3 7L10 3L17 7V13L10 17L3 13V7Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><path d="M10 17V10M10 10L3 7M10 10L17 7" stroke="currentColor" stroke-width="1.5"/>'
      },
      {
        path: '/orders',
        label: t('nav.orders'),
        icon: '<rect x="4" y="2" width="12" height="16" rx="1" stroke="currentColor" stroke-width="1.5"/><path d="M7 6H13M7 9H13M7 12H10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>'
      },
      {
        path: '/spending',
        label: t('nav.finance'),
        icon: '<circle cx="10" cy="10" r="7" stroke="currentColor" stroke-width="1.5"/><path d="M10 6V14M12 7.5H9C8.17 7.5 7.5 8.17 7.5 9C7.5 9.83 8.17 10.5 9 10.5H11C11.83 10.5 12.5 11.17 12.5 12C12.5 12.83 11.83 13.5 11 13.5H8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>'
      },
      {
        path: '/demand',
        label: t('nav.demandForecast'),
        icon: '<path d="M3 17L7 12L11 14L17 3" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/><path d="M14 3H17V6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>'
      },
      {
        path: '/restocking',
        label: t('nav.restocking'),
        icon: '<path d="M4 2V8H10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/><path d="M4 8C6 4 10 2 14 4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/><path d="M16 18V12H10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/><path d="M16 12C14 16 10 18 6 16" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>'
      },
      {
        path: '/reports',
        label: 'Reports',
        icon: '<rect x="3" y="10" width="3" height="7" rx="0.5" stroke="currentColor" stroke-width="1.5"/><rect x="8.5" y="6" width="3" height="11" rx="0.5" stroke="currentColor" stroke-width="1.5"/><rect x="14" y="3" width="3" height="14" rx="0.5" stroke="currentColor" stroke-width="1.5"/>'
      }
    ])

    const isActive = (path) => {
      if (path === '/') return route.path === '/'
      return route.path.startsWith(path)
    }

    return {
      t,
      currentUser,
      getInitials,
      navItems,
      isActive
    }
  }
}
</script>

<style scoped>
.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  width: 240px;
  height: 100vh;
  background: #0f172a;
  display: flex;
  flex-direction: column;
  z-index: 200;
  transition: width 0.2s ease;
}

.sidebar.collapsed {
  width: 64px;
}

.sidebar-header {
  padding: 1.25rem 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
}

.sidebar.collapsed .sidebar-header {
  padding: 1.25rem 0.5rem;
  justify-content: center;
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  min-width: 0;
}

.sidebar-logo svg {
  flex-shrink: 0;
}

.sidebar-title {
  color: #ffffff;
  font-weight: 700;
  font-size: 1.125rem;
  letter-spacing: -0.025em;
  white-space: nowrap;
  overflow: hidden;
}

.collapse-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 6px;
  color: #64748b;
  cursor: pointer;
  transition: all 0.15s ease;
  flex-shrink: 0;
}

.collapse-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #94a3b8;
}

.sidebar.collapsed .collapse-btn {
  display: none;
}

.sidebar-nav {
  flex: 1;
  padding: 1rem 0;
  overflow-y: auto;
}

.nav-section-label {
  color: #475569;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  padding: 0 1rem;
  margin-bottom: 0.5rem;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.625rem 1rem;
  margin: 2px 8px;
  border-radius: 6px;
  color: #94a3b8;
  text-decoration: none;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.15s ease;
  border-left: 3px solid transparent;
  white-space: nowrap;
  overflow: hidden;
}

.nav-item:hover {
  background: rgba(255, 255, 255, 0.05);
  color: #e2e8f0;
}

.nav-item.active {
  background: rgba(59, 130, 246, 0.1);
  color: #ffffff;
  border-left-color: #3b82f6;
}

.nav-item svg {
  flex-shrink: 0;
}

.sidebar.collapsed .nav-item {
  justify-content: center;
  padding: 0.625rem 0;
  margin: 2px 6px;
  border-left: none;
}

.sidebar.collapsed .nav-item.active {
  border-left: none;
  background: rgba(59, 130, 246, 0.15);
}

.nav-label {
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar-footer {
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  padding: 0.75rem;
}

.footer-expanded {
  margin-bottom: 0.5rem;
}

.sidebar-footer :deep(.language-button) {
  width: 100%;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  color: #94a3b8;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  font-size: 0.8125rem;
}

.sidebar-footer :deep(.language-button:hover) {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.15);
  color: #e2e8f0;
}

.sidebar-footer :deep(.language-button .globe-icon) {
  color: #94a3b8;
}

.sidebar-footer :deep(.language-button .chevron) {
  color: #94a3b8;
}

.sidebar-footer :deep(.dropdown-menu) {
  bottom: calc(100% + 0.5rem);
  top: auto;
  left: 0;
  right: auto;
}

.user-card {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  width: 100%;
  padding: 0.625rem 0.75rem;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.15s ease;
  text-align: left;
  font-family: inherit;
}

.user-card:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.15);
}

.sidebar.collapsed .user-card {
  justify-content: center;
  padding: 0.625rem;
}

.user-avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: linear-gradient(135deg, #3b82f6 0%, #1e40af 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 0.75rem;
  flex-shrink: 0;
}

.user-info {
  flex: 1;
  min-width: 0;
}

.user-name {
  color: #e2e8f0;
  font-size: 0.8125rem;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.user-role {
  color: #64748b;
  font-size: 0.6875rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
