<template>
  <div class="space-y-6">
    <div v-if="panelAlert" class="rounded-xl border px-4 py-3 text-sm shadow-sm" :class="pageAlertClass(panelAlert.level)">
      {{ panelAlert.message }}
    </div>

    <div class="rounded-2xl border theme-panel-soft p-6 shadow-sm">
      <div class="flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
        <div>
          <h2 class="text-xl font-bold theme-text-primary">{{ t('personalCenter.siteOwner.title') }}</h2>
          <p class="mt-1 text-sm theme-text-muted">{{ t('personalCenter.siteOwner.subtitle') }}</p>
        </div>
        <span class="theme-badge px-3 py-1 text-xs font-semibold" :class="dashboard?.has_site ? 'theme-badge-success' : 'theme-badge-warning'">
          {{ dashboard?.has_site ? t('personalCenter.siteOwner.hasSite') : t('personalCenter.siteOwner.noSite') }}
        </span>
      </div>
    </div>

    <OpenSiteSection
      v-if="!dashboard?.has_site"
      :suffixes="dashboard?.supported_domain_suffixes || []"
      :opening-price="dashboard?.opening_price"
      :opening-currency="dashboard?.opening_currency"
      :submitting="opening"
      @submit="handleOpenSite"
    />

    <template v-else>
      <div class="flex flex-wrap gap-2">
        <button
          v-for="item in sections"
          :key="item.key"
          type="button"
          class="rounded-lg border px-3 py-2 text-xs font-semibold"
          :class="activeSection === item.key ? 'theme-selected-surface theme-text-primary' : 'theme-btn-secondary'"
          @click="activeSection = item.key"
        >
          {{ t(item.label) }}
        </button>
      </div>

      <MySiteSummarySection v-if="activeSection === 'summary'" :site="dashboard.site || null" :stats="dashboard.stats" />
      <SiteBasicSettingsSection v-else-if="activeSection === 'settings'" :site="dashboard.site || null" :saving="savingSettings" @save="handleSaveSettings" />
      <ProductPricingSection v-else-if="activeSection === 'pricing'" :items="pricingItems" :saving="savingPricing" @reload="loadPricing" @save="handleSavePricing" />
      <SiteOrderSummarySection v-else-if="activeSection === 'orders'" :items="orderItems" @reload="loadOrders" />
      <SiteProfitLedgerSection v-else-if="activeSection === 'ledger'" :items="ledgerItems" @reload="loadLedger" />
      <SiteWithdrawApplySection v-else-if="activeSection === 'withdrawApply'" :submitting="submittingWithdraw" @submit="handleApplyWithdraw" />
      <SiteWithdrawRecordsSection v-else :items="withdrawItems" @reload="loadWithdraws" />
    </template>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { useI18n } from 'vue-i18n'
import {
  siteOwnerAPI,
  type SiteOwnerDashboardData,
  type SiteOwnerProductPricingData,
} from '../../api'
import { pageAlertClass, type PageAlert } from '../../utils/alerts'
import { toast } from '../../composables/useToast'
import OpenSiteSection from './siteOwner/OpenSiteSection.vue'
import MySiteSummarySection from './siteOwner/MySiteSummarySection.vue'
import SiteBasicSettingsSection from './siteOwner/SiteBasicSettingsSection.vue'
import ProductPricingSection from './siteOwner/ProductPricingSection.vue'
import SiteOrderSummarySection from './siteOwner/SiteOrderSummarySection.vue'
import SiteProfitLedgerSection from './siteOwner/SiteProfitLedgerSection.vue'
import SiteWithdrawApplySection from './siteOwner/SiteWithdrawApplySection.vue'
import SiteWithdrawRecordsSection from './siteOwner/SiteWithdrawRecordsSection.vue'

const { t } = useI18n()
const dashboard = ref<SiteOwnerDashboardData | null>(null)
const pricingItems = ref<SiteOwnerProductPricingData[]>([])
const orderItems = ref<any[]>([])
const ledgerItems = ref<any[]>([])
const withdrawItems = ref<any[]>([])
const panelAlert = ref<PageAlert | null>(null)
const opening = ref(false)
const savingSettings = ref(false)
const savingPricing = ref(false)
const submittingWithdraw = ref(false)
const activeSection = ref<'summary' | 'settings' | 'pricing' | 'orders' | 'ledger' | 'withdrawApply' | 'withdrawRecords'>('summary')

const sections = [
  { key: 'summary', label: 'personalCenter.siteOwner.sections.summary' },
  { key: 'settings', label: 'personalCenter.siteOwner.sections.settings' },
  { key: 'pricing', label: 'personalCenter.siteOwner.sections.pricing' },
  { key: 'orders', label: 'personalCenter.siteOwner.sections.orders' },
  { key: 'ledger', label: 'personalCenter.siteOwner.sections.ledger' },
  { key: 'withdrawApply', label: 'personalCenter.siteOwner.sections.withdrawApply' },
  { key: 'withdrawRecords', label: 'personalCenter.siteOwner.sections.withdrawRecords' },
] as const

const showSuccess = (message: string) => {
  toast.success(message)
}

const loadDashboard = async () => {
  panelAlert.value = null
  try {
    const response = await siteOwnerAPI.dashboard()
    dashboard.value = response.data.data
  } catch (error: any) {
    panelAlert.value = { level: 'error', message: error?.message || t('personalCenter.siteOwner.errors.loadFailed') }
  }
}

const loadPricing = async () => {
  const response = await siteOwnerAPI.productPricingList({ page: 1, page_size: 100 })
  pricingItems.value = response.data.data || []
}

const loadOrders = async () => {
  const response = await siteOwnerAPI.orderSummaries({ page: 1, page_size: 20 })
  orderItems.value = response.data.data || []
}

const loadLedger = async () => {
  const response = await siteOwnerAPI.profitLedger({ page: 1, page_size: 20 })
  ledgerItems.value = response.data.data || []
}

const loadWithdraws = async () => {
  const response = await siteOwnerAPI.withdrawRecords({ page: 1, page_size: 20 })
  withdrawItems.value = response.data.data || []
}

const loadAllSiteData = async () => {
  if (!dashboard.value?.has_site) return
  await Promise.all([loadPricing(), loadOrders(), loadLedger(), loadWithdraws()])
}

const handleOpenSite = async (payload: { site_name: string; subdomain_prefix: string; domain_suffix: string }) => {
  opening.value = true
  try {
    await siteOwnerAPI.open(payload)
    showSuccess(t('personalCenter.siteOwner.open.success'))
    await loadDashboard()
    await loadAllSiteData()
  } catch (error: any) {
    panelAlert.value = { level: 'error', message: error?.message || t('personalCenter.siteOwner.errors.openFailed') }
  } finally {
    opening.value = false
  }
}

const handleSaveSettings = async (payload: { site_name?: string; logo_url?: string; description?: string }) => {
  savingSettings.value = true
  try {
    await siteOwnerAPI.updateBasicSettings(payload)
    showSuccess(t('personalCenter.siteOwner.settings.saveSuccess'))
    await loadDashboard()
  } catch (error: any) {
    panelAlert.value = { level: 'error', message: error?.message || t('personalCenter.siteOwner.errors.saveFailed') }
  } finally {
    savingSettings.value = false
  }
}

const handleSavePricing = async (items: SiteOwnerProductPricingData[]) => {
  const invalid = items.find((item) => Number(item.site_price) < Number(item.base_price))
  if (invalid) {
    panelAlert.value = { level: 'warning', message: t('personalCenter.siteOwner.pricing.baseRule') }
    return
  }
  savingPricing.value = true
  try {
    await siteOwnerAPI.batchUpdateProductPricing({
      items: items.map((item) => ({ product_id: item.product_id, site_price: item.site_price })),
    })
    showSuccess(t('personalCenter.siteOwner.pricing.saveSuccess'))
    await loadPricing()
  } catch (error: any) {
    panelAlert.value = { level: 'error', message: error?.message || t('personalCenter.siteOwner.errors.saveFailed') }
  } finally {
    savingPricing.value = false
  }
}

const handleApplyWithdraw = async (payload: { amount: string; channel: string; account: string }) => {
  submittingWithdraw.value = true
  try {
    await siteOwnerAPI.applyWithdraw(payload)
    showSuccess(t('personalCenter.siteOwner.withdrawApply.success'))
    await loadWithdraws()
    await loadLedger()
    await loadDashboard()
  } catch (error: any) {
    panelAlert.value = { level: 'error', message: error?.message || t('personalCenter.siteOwner.errors.withdrawFailed') }
  } finally {
    submittingWithdraw.value = false
  }
}

onMounted(async () => {
  await loadDashboard()
  await loadAllSiteData()
})
</script>
