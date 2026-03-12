<template>
  <div class="rounded-2xl border theme-panel-soft p-6 shadow-sm">
    <div class="mb-4 flex items-center justify-between">
      <h3 class="text-lg font-bold theme-text-primary">{{ t('personalCenter.siteOwner.orders.title') }}</h3>
      <button type="button" class="rounded-lg border theme-btn-secondary px-3 py-1.5 text-xs font-semibold" @click="$emit('reload')">{{ t('orders.filters.refresh') }}</button>
    </div>
    <p class="mb-4 text-xs theme-text-muted">{{ t('personalCenter.siteOwner.orders.safeHint') }}</p>

    <div class="overflow-x-auto rounded-xl border border-gray-200/70 dark:border-white/10">
      <table class="min-w-full divide-y divide-gray-200 text-left text-sm dark:divide-white/10">
        <thead class="bg-gray-50/80 text-xs uppercase tracking-wide text-gray-500 dark:bg-white/5 dark:text-gray-400">
          <tr>
            <th class="px-4 py-3 font-semibold">{{ t('orders.orderNo') }}</th>
            <th class="px-4 py-3 font-semibold">{{ t('personalCenter.siteOwner.orders.status') }}</th>
            <th class="px-4 py-3 font-semibold">{{ t('personalCenter.siteOwner.orders.paidAmount') }}</th>
            <th class="px-4 py-3 font-semibold">{{ t('personalCenter.siteOwner.orders.siteProfit') }}</th>
            <th class="px-4 py-3 font-semibold">{{ t('personalCenter.siteOwner.orders.createdAt') }}</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-200 dark:divide-white/10">
          <tr v-for="item in items" :key="item.id">
            <td class="px-4 py-3 font-mono text-xs theme-text-primary">{{ item.order_no }}</td>
            <td class="px-4 py-3 text-xs theme-text-secondary">{{ item.status }}</td>
            <td class="px-4 py-3 font-mono text-xs theme-text-primary">{{ item.paid_amount }} {{ item.currency }}</td>
            <td class="px-4 py-3 font-mono text-xs theme-text-primary">{{ item.site_profit }} {{ item.currency }}</td>
            <td class="px-4 py-3 text-xs theme-text-muted">{{ item.created_at || '-' }}</td>
          </tr>
          <tr v-if="items.length === 0"><td colspan="5" class="px-4 py-6 text-center text-sm theme-text-muted">{{ t('personalCenter.siteOwner.orders.empty') }}</td></tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import type { SiteOwnerOrderSummaryData } from '../../../api'

defineProps<{ items: SiteOwnerOrderSummaryData[] }>()
defineEmits<{ reload: [] }>()
const { t } = useI18n()
</script>
