<template>
  <div class="top-bar">
    <div class="top-bar-left">
      <h1 class="page-title">{{ pageTitle }}</h1>
    </div>
    <div class="top-bar-center">
      <FilterBar />
    </div>
    <div class="top-bar-right">
      <ProfileMenu
        @show-profile-details="$emit('show-profile-details')"
        @show-tasks="$emit('show-tasks')"
      />
    </div>
  </div>
</template>

<script>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from '../composables/useI18n'
import FilterBar from './FilterBar.vue'
import ProfileMenu from './ProfileMenu.vue'

export default {
  name: 'TopBar',
  components: { FilterBar, ProfileMenu },
  emits: ['show-profile-details', 'show-tasks'],
  setup() {
    const route = useRoute()
    const { t } = useI18n()

    const pageTitle = computed(() => {
      const titles = {
        '/': t('nav.overview'),
        '/inventory': t('nav.inventory'),
        '/orders': t('nav.orders'),
        '/spending': t('nav.finance'),
        '/demand': t('nav.demandForecast'),
        '/restocking': t('nav.restocking'),
        '/reports': 'Reports'
      }
      return titles[route.path] || 'Page'
    })

    return { pageTitle }
  }
}
</script>

<style scoped>
.top-bar {
  display: flex;
  align-items: center;
  height: 56px;
  padding: 0 1.5rem;
  background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  position: sticky;
  top: 0;
  z-index: 100;
  gap: 1rem;
}

.top-bar-left {
  flex-shrink: 0;
}

.page-title {
  font-size: 1rem;
  font-weight: 600;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.top-bar-center {
  flex: 1;
  min-width: 0;
}

.top-bar-right {
  flex-shrink: 0;
}
</style>
