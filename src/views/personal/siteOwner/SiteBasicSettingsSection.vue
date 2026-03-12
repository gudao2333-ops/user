<template>
  <div class="rounded-2xl border theme-panel-soft p-6 shadow-sm">
    <h3 class="text-lg font-bold theme-text-primary">{{ t('personalCenter.siteOwner.settings.title') }}</h3>
    <p class="mt-1 text-sm theme-text-muted">{{ t('personalCenter.siteOwner.settings.subtitle') }}</p>

    <div class="mt-4 grid grid-cols-1 gap-4 md:grid-cols-2">
      <div>
        <label class="mb-2 block text-sm font-medium theme-text-secondary">{{ t('personalCenter.siteOwner.settings.siteName') }}</label>
        <input v-model.trim="form.site_name" type="text" class="w-full form-input-lg" />
      </div>
      <div>
        <label class="mb-2 block text-sm font-medium theme-text-secondary">{{ t('personalCenter.siteOwner.settings.logoUrl') }}</label>
        <input v-model.trim="form.logo_url" type="text" class="w-full form-input-lg" />
      </div>
      <div class="md:col-span-2">
        <label class="mb-2 block text-sm font-medium theme-text-secondary">{{ t('personalCenter.siteOwner.settings.description') }}</label>
        <textarea v-model.trim="form.description" rows="4" class="w-full form-input-lg"></textarea>
      </div>
    </div>

    <button
      type="button"
      :disabled="saving"
      class="mt-5 inline-flex items-center rounded-xl theme-btn-primary px-4 py-2 text-sm font-bold disabled:cursor-not-allowed disabled:opacity-60"
      @click="$emit('save', { ...form })"
    >
      {{ saving ? t('personalCenter.siteOwner.settings.saving') : t('personalCenter.siteOwner.settings.save') }}
    </button>
  </div>
</template>

<script setup lang="ts">
import { reactive, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import type { SiteOwnerProfileData } from '../../../api'

const props = defineProps<{ site: SiteOwnerProfileData | null; saving: boolean }>()
defineEmits<{ save: [payload: { site_name?: string; logo_url?: string; description?: string }] }>()
const { t } = useI18n()

const form = reactive({ site_name: '', logo_url: '', description: '' })

watch(
  () => props.site,
  (site) => {
    form.site_name = site?.site_name || ''
    form.logo_url = site?.logo_url || ''
    form.description = site?.description || ''
  },
  { immediate: true },
)
</script>
