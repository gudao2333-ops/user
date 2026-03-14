<template>
  <div class="relative min-h-screen overflow-hidden theme-page pb-16 pt-20 lg:pt-24">
    <div class="container relative z-10 mx-auto px-4">
      <header class="mb-6 rounded-3xl border theme-panel-soft p-4 shadow-xl backdrop-blur-sm lg:mb-8 lg:p-8">
        <div class="flex flex-col gap-6">
          <div class="flex flex-col gap-6 lg:flex-row lg:items-center lg:justify-between">
            <div>
              <p class="text-xs font-semibold uppercase tracking-[0.24em] theme-text-accent">
                {{ t('personalCenter.title') }}
              </p>
              <h1 class="mt-3 text-2xl font-black theme-text-primary lg:text-[2.1rem]">{{ userProfileStore.displayName }}</h1>
              <p class="mt-2 text-sm theme-text-muted">{{ t('personalCenter.subtitle') }}</p>
            </div>

            <div class="grid w-full grid-cols-2 gap-3 lg:w-auto">
              <div class="rounded-2xl border theme-surface-soft px-4 py-3">
                <p class="text-[11px] uppercase tracking-[0.16em] text-gray-400">{{ t('personalCenter.tabs.orders') }}</p>
                <p class="mt-2 text-sm font-semibold theme-text-secondary">
                  {{ userProfileStore.loadingOrders ? '-' : userProfileStore.recentOrders.length }}
                </p>
              </div>
              <div class="rounded-2xl border theme-surface-soft px-4 py-3">
                <p class="text-[11px] uppercase tracking-[0.16em] text-gray-400">{{ t('personalCenter.overview.accountLabel') }}</p>
                <span class="mt-2 theme-badge px-2.5 py-1 text-xs font-semibold" :class="emailVerifiedClass">
                  {{ emailVerifiedLabel }}
                </span>
              </div>
            </div>
          </div>

          <div class="grid grid-cols-2 gap-2 sm:grid-cols-3 lg:grid-cols-4 xl:grid-cols-7">
            <button
              v-for="item in sectionItems"
              :key="item.key"
              type="button"
              @click="switchSection(item.key)"
              class="group rounded-lg border px-3 py-2.5 text-xs font-semibold transition-colors"
              :class="buttonClass(item.key)"
            >
              <span class="flex items-center justify-center gap-1.5">
                <component :is="item.icon" class="h-3.5 w-3.5" />
                <span>{{ t(item.label) }}</span>
              </span>
            </button>
          </div>
        </div>
      </header>

      <section class="space-y-6">
        <div
          v-if="globalAlert"
          class="rounded-2xl border px-4 py-3 text-sm shadow-sm"
          :class="pageAlertClass(globalAlert.level)"
        >
          {{ globalAlert.message }}
        </div>

        <OrdersPanel v-if="currentSection === 'overview'" />
        <ProfilePanel v-else-if="currentSection === 'profile'" />
        <SecurityPanel v-else-if="currentSection === 'security'" />
        <OrdersPanel v-else-if="currentSection === 'orders'" />
        <WalletPanel v-else-if="currentSection === 'wallet'" />
        <AffiliatePanel v-else-if="currentSection === 'affiliate'" />
        <GiftCardPanel v-else-if="currentSection === 'giftCard'" />
        <ApiPanel v-else-if="currentSection === 'api'" />
        <OrdersPanel v-else />
      </section>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, type Component } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { HomeIcon, ShoppingBagIcon, WalletIcon, GiftIcon, ShieldCheckIcon, UserCircleIcon, MegaphoneIcon, KeyIcon } from '@heroicons/vue/24/outline'
import { pageAlertClass, type PageAlert } from '../utils/alerts'
import { useUserProfileStore } from '../stores/userProfile'
import ProfilePanel from './personal/ProfilePanel.vue'
import SecurityPanel from './personal/SecurityPanel.vue'
import OrdersPanel from './personal/OrdersPanel.vue'
import WalletPanel from './personal/WalletPanel.vue'
import GiftCardPanel from './personal/GiftCardPanel.vue'
import AffiliatePanel from './personal/AffiliatePanel.vue'
import ApiPanel from './personal/ApiPanel.vue'

type PersonalSection = 'overview' | 'profile' | 'security' | 'orders' | 'wallet' | 'giftCard' | 'affiliate' | 'api'

const props = withDefaults(defineProps<{ section?: PersonalSection }>(), {
  section: 'overview',
})

const router = useRouter()
const { t } = useI18n()
const userProfileStore = useUserProfileStore()

const sectionItems: Array<{ key: PersonalSection; label: string; icon: Component }> = [
  { key: 'orders', label: 'personalCenter.tabs.orders', icon: ShoppingBagIcon },
  { key: 'affiliate', label: 'personalCenter.tabs.affiliate', icon: MegaphoneIcon },
  { key: 'overview', label: 'personalCenter.tabs.overview', icon: HomeIcon },
  { key: 'wallet', label: 'personalCenter.tabs.wallet', icon: WalletIcon },
  { key: 'giftCard', label: 'personalCenter.tabs.giftCard', icon: GiftIcon },
  { key: 'security', label: 'personalCenter.tabs.security', icon: ShieldCheckIcon },
  { key: 'api', label: 'personalCenter.tabs.api', icon: KeyIcon },
  { key: 'profile', label: 'personalCenter.tabs.profile', icon: UserCircleIcon },
]

const sectionRouteMap: Record<PersonalSection, string> = {
  overview: '/me',
  profile: '/me/profile',
  security: '/me/security',
  orders: '/me/orders',
  wallet: '/me/wallet',
  affiliate: '/me/affiliate',
  giftCard: '/me/gift-cards',
  api: '/me/api',
}

const currentSection = computed(() => props.section)
const globalAlert = ref<PageAlert | null>(null)

const switchSection = (section: PersonalSection) => {
  router.push(sectionRouteMap[section])
}

const buttonClass = (key: PersonalSection) => {
  if (currentSection.value === key) return 'theme-selected-surface theme-text-primary'
  if (key === 'affiliate') return 'border border-emerald-300/70 bg-emerald-50 text-emerald-700 dark:border-emerald-700 dark:bg-emerald-900/30 dark:text-emerald-200'
  return 'border theme-btn-secondary'
}

const emailVerifiedLabel = computed(() => {
  if (userProfileStore.profile?.email_verified_at) {
    return t('personalCenter.overview.emailVerified')
  }
  return t('personalCenter.overview.emailUnverified')
})

const emailVerifiedClass = computed(() => {
  if (userProfileStore.profile?.email_verified_at) {
    return 'theme-badge-success'
  }
  return 'theme-badge-warning'
})

const initialize = async () => {
  globalAlert.value = null
  const [profileOk] = await Promise.all([
    userProfileStore.loadProfile(),
    userProfileStore.loadRecentOrders(5),
  ])
  if (!profileOk) {
    globalAlert.value = {
      level: 'error',
      message: userProfileStore.profileError || t('personalCenter.common.loadFailed'),
    }
  }
}

onMounted(() => {
  initialize()
})
</script>
