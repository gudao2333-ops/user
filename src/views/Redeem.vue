<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { redeemCodeAPI, type RedeemCodePreviewData, type RedeemCodeRedeemResult } from '../api'
import { orderStatusLabel } from '../utils/status'
import { fulfillmentStatusLabel } from '../utils/fulfillment'
import { useUserAuthStore } from '../stores/userAuth'

const route = useRoute()
const router = useRouter()
const userAuthStore = useUserAuthStore()
const { t } = useI18n()
const codeInput = ref('')
const loadingPreview = ref(false)
const loadingRedeem = ref(false)
const previewError = ref('')
const redeemError = ref('')
const preview = ref<RedeemCodePreviewData | null>(null)
const redeemResult = ref<RedeemCodeRedeemResult | null>(null)
const isAuthenticated = computed(() => userAuthStore.isAuthenticated)

const loginUrl = computed(() => `/auth/login?redirect=${encodeURIComponent(route.fullPath)}`)
const registerUrl = computed(() => `/auth/register?redirect=${encodeURIComponent(route.fullPath)}`)

const getLocalizedText = (jsonData: any) => {
  if (!jsonData || typeof jsonData !== 'object') return ''
  const locale = String((route.query.locale as string) || '')
  if (locale && typeof jsonData[locale] === 'string' && jsonData[locale].trim()) {
    return jsonData[locale].trim()
  }
  if (typeof jsonData['zh-CN'] === 'string' && jsonData['zh-CN'].trim()) return jsonData['zh-CN'].trim()
  if (typeof jsonData['zh-TW'] === 'string' && jsonData['zh-TW'].trim()) return jsonData['zh-TW'].trim()
  if (typeof jsonData['en-US'] === 'string' && jsonData['en-US'].trim()) return jsonData['en-US'].trim()
  const first = Object.values(jsonData).find((v) => typeof v === 'string' && String(v).trim())
  return first ? String(first).trim() : ''
}

const skuText = computed(() => {
  const spec = preview.value?.sku?.spec_values
  if (!spec || typeof spec !== 'object') return preview.value?.sku?.sku_code || '-'

  const name = (spec as any)?.name
  if (name && typeof name === 'object') {
    const localized = getLocalizedText(name)
    if (localized) return localized
  }
  if (typeof name === 'string' && name.trim()) return name.trim()

  const firstValue = Object.values(spec).find((v: any) => {
    if (typeof v === 'string' && v.trim()) return true
    if (v && typeof v === 'object') return Boolean(getLocalizedText(v))
    return false
  })
  if (typeof firstValue === 'string' && firstValue.trim()) return firstValue.trim()
  if (firstValue && typeof firstValue === 'object') {
    const localized = getLocalizedText(firstValue)
    if (localized) return localized
  }

  return preview.value?.sku?.sku_code || '-'
})

const statusText = computed(() => {
  if (!preview.value) return '-'
  if (preview.value.status === 'unused' && !preview.value.can_redeem) return '库存不足'
  if (preview.value.status === 'expired') return '已过期'
  if (preview.value.status === 'used') return '已使用'
  if (preview.value.status === 'frozen') return '已冻结'
  if (preview.value.status === 'unused') return '可兑换'
  return preview.value.status
})

const redeemOrderStatusText = computed(() => {
  const status = String(redeemResult.value?.order?.status || '').trim()
  if (!status) return '-'
  return orderStatusLabel(t, status)
})

const fulfillmentStatusText = computed(() => {
  const childFulfillmentStatus = String(redeemResult.value?.order?.children?.[0]?.fulfillment?.status || '').trim()
  if (childFulfillmentStatus) return fulfillmentStatusLabel(t, childFulfillmentStatus, 'orderDetail')
  const ownFulfillmentStatus = String(redeemResult.value?.order?.fulfillment?.status || '').trim()
  if (ownFulfillmentStatus) return fulfillmentStatusLabel(t, ownFulfillmentStatus, 'orderDetail')
  return '-'
})

const redeemFulfillmentLines = computed(() => {
  const fulfillment = redeemResult.value?.order?.children?.[0]?.fulfillment || redeemResult.value?.order?.fulfillment
  const deliveryData = fulfillment?.delivery_data || fulfillment?.logistics
  const lines: string[] = []
  if (deliveryData && typeof deliveryData === 'object') {
    const note = String(deliveryData.note || '').trim()
    if (note) lines.push(note)
    const entries = Array.isArray(deliveryData.entries) ? deliveryData.entries : []
    entries.forEach((entry: any) => {
      const value = String(entry?.value || '').trim()
      if (value) lines.push(value)
    })
  }
  if (lines.length) return lines

  const payload = String(fulfillment?.payload || '').trim()
  return payload ? [payload] : []
})

const resolveCodeFromQuery = () => {
  const code = String(route.query.code || '').trim()
  if (code) {
    codeInput.value = code
  }
}

const previewRedeem = async () => {
  previewError.value = ''
  redeemError.value = ''
  redeemResult.value = null
  preview.value = null
  if (!isAuthenticated.value) {
    previewError.value = '请先登录后再兑换礼品卡，没有账号可先注册。'
    return
  }
  const code = codeInput.value.trim()
  if (!code) {
    previewError.value = '请输入兑换码'
    return
  }
  loadingPreview.value = true
  try {
    const response = await redeemCodeAPI.preview({ code })
    preview.value = response.data.data.preview
    const errorCode = String(response.data.data.error_code || '')
    if (errorCode) {
      if (errorCode === 'redeem_code_used') previewError.value = '该兑换码已使用'
      else if (errorCode === 'redeem_code_frozen') previewError.value = '该兑换码已冻结'
      else if (errorCode === 'redeem_code_expired') previewError.value = '该兑换码已过期'
      else if (errorCode === 'redeem_code_out_of_stock') previewError.value = '库存不足，暂不可兑换'
      else previewError.value = '该兑换码当前不可兑换'
    }
  } catch (err: any) {
    previewError.value = err?.message || '预览失败'
  } finally {
    loadingPreview.value = false
  }
}

const redeem = async () => {
  redeemError.value = ''
  if (!isAuthenticated.value) {
    redeemError.value = '请先登录后再兑换礼品卡，没有账号可先注册。'
    return
  }
  const code = codeInput.value.trim()
  if (!code) {
    redeemError.value = '请输入兑换码'
    return
  }
  loadingRedeem.value = true
  try {
    const response = await redeemCodeAPI.redeem({ code })
    redeemResult.value = {
      ...response.data.data,
      code: {
        ...response.data.data.code,
        status: 'used',
      },
    }
    if (preview.value) {
      preview.value = {
        ...preview.value,
        status: 'used',
        can_redeem: false,
      }
    }
    previewError.value = ''
  } catch (err: any) {
    const message = String(err?.message || '').trim()
    if (message === 'error.redeem_code_out_of_stock') redeemError.value = '库存不足，暂不可兑换'
    else redeemError.value = message || '兑换失败'
  } finally {
    loadingRedeem.value = false
  }
}

const goOrder = () => {
  const orderNo = String(redeemResult.value?.order?.order_no || '').trim()
  if (!orderNo) return
  router.push(`/orders/${encodeURIComponent(orderNo)}`)
}

onMounted(() => {
  resolveCodeFromQuery()
  if (codeInput.value && isAuthenticated.value) {
    previewRedeem()
  }
})

watch(() => route.query.code, () => {
  resolveCodeFromQuery()
})
</script>

<template>
  <main class="min-h-screen pt-24 pb-10">
    <div class="container mx-auto px-4 max-w-3xl space-y-6">
      <section v-if="!isAuthenticated" class="theme-panel-strong rounded-2xl border theme-border p-6 space-y-4">
        <div class="space-y-2 text-center sm:text-left">
          <p class="text-xs font-semibold uppercase tracking-[0.2em] theme-text-accent">Redeem Access</p>
          <h2 class="text-2xl font-bold theme-text-primary">登录后即可兑换礼品卡</h2>
          <p class="text-sm theme-text-muted">为保障订单和发货信息安全，请先登录账号后再兑换。</p>
              <p class="text-sm theme-text-muted">没有账号也没关系，先注册，完成后会自动回到本页继续兑换。</p>
        </div>
        <div class="flex flex-col sm:flex-row gap-3 sm:justify-start">
          <router-link :to="loginUrl" class="inline-flex items-center justify-center px-4 py-2 rounded-lg theme-btn-primary">
            立即登录
          </router-link>
          <router-link :to="registerUrl" class="inline-flex items-center justify-center px-4 py-2 rounded-lg theme-btn-neutral border theme-border">
            免费注册
          </router-link>
        </div>
      </section>

      <section class="theme-panel-strong rounded-2xl border theme-border p-6 space-y-4">
        <h1 class="text-2xl font-bold theme-text-primary">兑换礼品卡</h1>
        <p class="text-sm theme-text-muted">{{ isAuthenticated ? '输入兑换码并预览商品信息，确认后将自动创建订单并展示发货结果。' : '登录后可预览兑换信息并完成兑换。' }}</p>

        <div class="flex flex-col sm:flex-row gap-3">
          <input v-model="codeInput" :disabled="!isAuthenticated" placeholder="请输入兑换码" class="flex-1 rounded-lg border theme-border bg-transparent px-3 py-2 text-sm disabled:opacity-60" />
          <button :disabled="loadingPreview || !isAuthenticated" class="px-4 py-2 rounded-lg theme-btn-primary disabled:opacity-60" @click="previewRedeem">{{ loadingPreview ? '预览中...' : '预览兑换信息' }}</button>
        </div>

        <p v-if="previewError" class="text-sm text-destructive">{{ previewError }}</p>

        <div v-if="preview" class="rounded-xl border theme-border p-4 space-y-2 text-sm">
          <div class="flex justify-between"><span class="theme-text-muted">商品</span><span>{{ getLocalizedText(preview.product?.title) || '-' }}</span></div>
          <div class="flex justify-between"><span class="theme-text-muted">SKU/规格</span><span>{{ skuText }}</span></div>
          <div class="flex justify-between"><span class="theme-text-muted">数量</span><span>{{ preview.quantity }}</span></div>
          <div class="flex justify-between"><span class="theme-text-muted">状态</span><span>{{ statusText }}</span></div>
          <div class="flex justify-between"><span class="theme-text-muted">有效期</span><span>{{ preview.expires_at || '长期有效' }}</span></div>
          <div class="pt-2 text-center">
            <button :disabled="!preview.can_redeem || loadingRedeem" class="px-4 py-2 rounded-lg theme-btn-primary disabled:opacity-60" @click="redeem">
              {{ loadingRedeem ? '兑换中...' : '确认兑换' }}
            </button>
          </div>
        </div>

        <p v-if="redeemError" class="text-sm text-destructive">{{ redeemError }}</p>
      </section>

      <section v-if="redeemResult" class="theme-panel-strong rounded-2xl border theme-border p-6 space-y-3">
        <h2 class="text-xl font-semibold theme-text-primary">兑换成功</h2>
        <div class="text-sm space-y-2">
          <div class="flex justify-between"><span class="theme-text-muted">商品</span><span>{{ getLocalizedText(redeemResult.product?.title) || '-' }}</span></div>
          <div class="flex justify-between"><span class="theme-text-muted">订单号</span><span>{{ redeemResult.order?.order_no || '-' }}</span></div>
          <div class="flex justify-between"><span class="theme-text-muted">订单状态</span><span>{{ redeemOrderStatusText }}</span></div>
          <div class="flex justify-between"><span class="theme-text-muted">发货状态</span><span>{{ fulfillmentStatusText }}</span></div>
          <div v-if="redeemFulfillmentLines.length" class="space-y-1 pt-1">
            <div class="theme-text-muted">发货内容</div>
            <div class="rounded-lg border theme-border p-2">
              <div v-for="(line, index) in redeemFulfillmentLines" :key="`redeem-fulfillment-${index}`">{{ line }}</div>
            </div>
          </div>
        </div>
        <button class="px-4 py-2 rounded-lg theme-btn-neutral border theme-border" @click="goOrder">查看订单详情</button>
      </section>
    </div>
  </main>
</template>
