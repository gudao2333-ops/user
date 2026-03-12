<template>
  <div class="rounded-2xl border theme-panel-soft p-6 shadow-sm">
    <h3 class="text-lg font-bold theme-text-primary">{{ t('personalCenter.siteOwner.withdrawApply.title') }}</h3>
    <p class="mt-1 text-sm theme-text-muted">{{ t('personalCenter.siteOwner.withdrawApply.subtitle') }}</p>

    <form class="mt-5 grid grid-cols-1 gap-4 md:grid-cols-3" @submit.prevent="$emit('submit', { ...form })">
      <div>
        <label class="mb-2 block text-sm font-medium theme-text-secondary">{{ t('personalCenter.siteOwner.withdrawApply.amount') }}</label>
        <input v-model.trim="form.amount" type="text" class="w-full form-input-lg" />
      </div>
      <div>
        <label class="mb-2 block text-sm font-medium theme-text-secondary">{{ t('personalCenter.siteOwner.withdrawApply.channel') }}</label>
        <input v-model.trim="form.channel" type="text" class="w-full form-input-lg" />
      </div>
      <div>
        <label class="mb-2 block text-sm font-medium theme-text-secondary">{{ t('personalCenter.siteOwner.withdrawApply.account') }}</label>
        <input v-model.trim="form.account" type="text" class="w-full form-input-lg" />
      </div>
      <div class="md:col-span-3">
        <button type="submit" :disabled="submitting" class="inline-flex items-center rounded-xl theme-btn-primary px-4 py-2 text-sm font-bold disabled:cursor-not-allowed disabled:opacity-60">
          {{ submitting ? t('personalCenter.siteOwner.withdrawApply.submitting') : t('personalCenter.siteOwner.withdrawApply.submit') }}
        </button>
      </div>
    </form>
  </div>
</template>

<script setup lang="ts">
import { reactive } from 'vue'
import { useI18n } from 'vue-i18n'

defineProps<{ submitting: boolean }>()
defineEmits<{ submit: [payload: { amount: string; channel: string; account: string }] }>()
const { t } = useI18n()
const form = reactive({ amount: '', channel: '', account: '' })
</script>
