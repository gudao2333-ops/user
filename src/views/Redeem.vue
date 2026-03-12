<template>
  <div class="pt-24 pb-12 theme-page min-h-screen">
    <div class="container mx-auto px-4 max-w-4xl space-y-5">
      <div class="rounded-2xl border theme-surface-soft px-4 py-3 text-sm">
        <p class="font-semibold theme-text-primary">{{ t('personalCenter.giftCard.loginRequiredTitle') }}</p>
        <p class="mt-1 theme-text-secondary">{{ t('personalCenter.giftCard.loginRequiredHint') }}</p>
        <p class="mt-1 theme-text-muted">{{ t('personalCenter.giftCard.redeemOwnershipHint') }}</p>
        <router-link
          v-if="!userAuthStore.isAuthenticated"
          :to="`/auth/login?redirect=${encodedRedirect}`"
          class="mt-3 inline-flex h-9 items-center justify-center rounded-lg theme-btn-primary px-3 text-xs font-semibold"
        >
          {{ t('personalCenter.giftCard.loginNow') }}
        </router-link>
      </div>
    </div>
    <div class="container mx-auto px-4 max-w-4xl">
      <GiftCardPanel />
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import { useUserAuthStore } from '../stores/userAuth'
import GiftCardPanel from './personal/GiftCardPanel.vue'

const { t } = useI18n()
const userAuthStore = useUserAuthStore()
const encodedRedirect = computed(() => encodeURIComponent('/redeem'))
</script>
