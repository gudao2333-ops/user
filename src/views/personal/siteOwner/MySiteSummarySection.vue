<template>
  <div class="rounded-2xl border theme-panel-soft p-6 shadow-sm">
    <h3 class="text-lg font-bold theme-text-primary">{{ t('personalCenter.siteOwner.summary.title') }}</h3>
    <div class="mt-4 grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4">
      <div class="rounded-xl border theme-surface-soft p-4">
        <div class="text-xs theme-text-muted">{{ t('personalCenter.siteOwner.summary.siteName') }}</div>
        <div class="mt-2 font-semibold theme-text-primary">{{ site?.site_name || '-' }}</div>
      </div>
      <div class="rounded-xl border theme-surface-soft p-4">
        <div class="text-xs theme-text-muted">{{ t('personalCenter.siteOwner.summary.domain') }}</div>
        <div class="mt-2 font-semibold theme-text-primary">{{ site?.domain || '-' }}</div>
      </div>
      <div class="rounded-xl border theme-surface-soft p-4">
        <div class="text-xs theme-text-muted">{{ t('personalCenter.siteOwner.summary.productCount') }}</div>
        <div class="mt-2 font-semibold theme-text-primary">{{ stats?.product_count || 0 }}</div>
      </div>
      <div class="rounded-xl border theme-surface-soft p-4">
        <div class="text-xs theme-text-muted">{{ t('personalCenter.siteOwner.summary.orderCount') }}</div>
        <div class="mt-2 font-semibold theme-text-primary">{{ stats?.order_count || 0 }}</div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import type { SiteOwnerProfileData } from '../../../api'

defineProps<{ site: SiteOwnerProfileData | null; stats?: any }>()
const { t } = useI18n()
</script>
