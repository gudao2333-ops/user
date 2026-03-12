<template>
  <div class="rounded-2xl border theme-panel-soft p-6 shadow-sm">
    <h3 class="text-lg font-bold theme-text-primary">{{ t('personalCenter.siteOwner.open.title') }}</h3>
    <p class="mt-1 text-sm theme-text-muted">{{ t('personalCenter.siteOwner.open.subtitle') }}</p>

    <div class="mt-4 grid grid-cols-1 gap-4 md:grid-cols-3">
      <div>
        <label class="mb-2 block text-sm font-medium theme-text-secondary">{{ t('personalCenter.siteOwner.open.siteName') }}</label>
        <input v-model.trim="form.site_name" type="text" class="w-full form-input-lg" :placeholder="t('personalCenter.siteOwner.open.siteNamePlaceholder')" />
      </div>
      <div>
        <label class="mb-2 block text-sm font-medium theme-text-secondary">{{ t('personalCenter.siteOwner.open.prefix') }}</label>
        <input v-model.trim="form.subdomain_prefix" type="text" class="w-full form-input-lg" :placeholder="t('personalCenter.siteOwner.open.prefixPlaceholder')" />
      </div>
      <div>
        <label class="mb-2 block text-sm font-medium theme-text-secondary">{{ t('personalCenter.siteOwner.open.suffix') }}</label>
        <select v-model="form.domain_suffix" class="w-full form-input-lg">
          <option value="">{{ t('personalCenter.siteOwner.open.suffixPlaceholder') }}</option>
          <option v-for="suffix in suffixes" :key="suffix" :value="suffix">{{ suffix }}</option>
        </select>
      </div>
    </div>

    <p class="mt-4 text-xs theme-text-muted">
      {{ t('personalCenter.siteOwner.open.finalDomain') }}: <span class="font-semibold theme-text-primary">{{ fullDomain }}</span>
    </p>
    <p class="mt-2 text-xs theme-text-muted">
      {{ t('personalCenter.siteOwner.open.price', { price: openingPrice || '0.00', currency: openingCurrency || '-' }) }}
    </p>

    <button
      type="button"
      :disabled="submitting"
      class="mt-5 inline-flex items-center rounded-xl theme-btn-primary px-4 py-2 text-sm font-bold disabled:cursor-not-allowed disabled:opacity-60"
      @click="$emit('submit', { ...form })"
    >
      {{ submitting ? t('personalCenter.siteOwner.open.submitting') : t('personalCenter.siteOwner.open.submit') }}
    </button>
  </div>
</template>

<script setup lang="ts">
import { computed, reactive } from 'vue'
import { useI18n } from 'vue-i18n'

const props = defineProps<{
  suffixes: string[]
  openingPrice?: string
  openingCurrency?: string
  submitting: boolean
}>()

defineEmits<{
  submit: [payload: { site_name: string; subdomain_prefix: string; domain_suffix: string }]
}>()

const { t } = useI18n()

const form = reactive({
  site_name: '',
  subdomain_prefix: '',
  domain_suffix: '',
})

const fullDomain = computed(() => {
  if (!form.subdomain_prefix || !form.domain_suffix) return '-'
  return `${form.subdomain_prefix}${form.domain_suffix}`
})
</script>
