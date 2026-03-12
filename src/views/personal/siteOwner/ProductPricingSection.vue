<template>
  <div class="rounded-2xl border theme-panel-soft p-6 shadow-sm">
    <div class="mb-4 flex items-center justify-between">
      <h3 class="text-lg font-bold theme-text-primary">{{ t('personalCenter.siteOwner.pricing.title') }}</h3>
      <button type="button" class="rounded-lg border theme-btn-secondary px-3 py-1.5 text-xs font-semibold" @click="$emit('reload')">
        {{ t('orders.filters.refresh') }}
      </button>
    </div>

    <div class="overflow-x-auto rounded-xl border border-gray-200/70 dark:border-white/10">
      <table class="min-w-full divide-y divide-gray-200 text-left text-sm dark:divide-white/10">
        <thead class="bg-gray-50/80 text-xs uppercase tracking-wide text-gray-500 dark:bg-white/5 dark:text-gray-400">
          <tr>
            <th class="px-4 py-3 font-semibold">{{ t('personalCenter.siteOwner.pricing.product') }}</th>
            <th class="px-4 py-3 font-semibold">{{ t('personalCenter.siteOwner.pricing.basePrice') }}</th>
            <th class="px-4 py-3 font-semibold">{{ t('personalCenter.siteOwner.pricing.sitePrice') }}</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-200 dark:divide-white/10">
          <tr v-for="item in localItems" :key="item.product_id">
            <td class="px-4 py-3 text-xs theme-text-secondary">{{ item.product_name }}</td>
            <td class="px-4 py-3 font-mono text-xs theme-text-primary">{{ item.base_price }} {{ item.currency }}</td>
            <td class="px-4 py-3">
              <input v-model.trim="item.site_price" type="text" class="w-36 form-input-lg text-xs" />
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <button type="button" :disabled="saving" class="mt-5 inline-flex items-center rounded-xl theme-btn-primary px-4 py-2 text-sm font-bold disabled:cursor-not-allowed disabled:opacity-60" @click="$emit('save', localItems)">
      {{ saving ? t('personalCenter.siteOwner.pricing.saving') : t('personalCenter.siteOwner.pricing.save') }}
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import type { SiteOwnerProductPricingData } from '../../../api'

const props = defineProps<{ items: SiteOwnerProductPricingData[]; saving: boolean }>()
defineEmits<{ reload: []; save: [items: SiteOwnerProductPricingData[]] }>()
const { t } = useI18n()

const localItems = ref<SiteOwnerProductPricingData[]>([])
watch(() => props.items, (items) => {
  localItems.value = items.map((it) => ({ ...it }))
}, { immediate: true })
</script>
