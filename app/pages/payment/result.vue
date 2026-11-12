<template>
  <div class="jb-result-page">
    <div class="jb-result-card" :class="isSuccess ? 'is-success' : 'is-failed'">
      <!-- آیکون نتیجه -->
      <div class="jb-result-icon-wrap">
        <div class="jb-result-icon" :class="isSuccess ? 'jb-icon-success' : 'jb-icon-failed'">
          <svg v-if="isSuccess" class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7" />
          </svg>
          <svg v-else class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </div>
      </div>

      <!-- تیتر -->
      <h1 class="jb-result-title" :class="isSuccess ? 'jb-title-success' : 'jb-title-failed'">
        {{ title }}
      </h1>

      <!-- خط جداکننده -->
      <div class="jb-result-divider" :class="isSuccess ? 'jb-divider-success' : 'jb-divider-failed'"></div>

      <!-- توضیحات -->
      <p class="jb-result-desc">{{ description }}</p>

      <!-- اطلاعات تراکنش -->
      <div v-if="isSuccess && (orderId || walletTransactionId)" class="jb-result-info">
        <div v-if="orderId" class="jb-result-info-row">
          <span class="jb-result-info-label">شماره سفارش</span>
          <span class="jb-result-info-value" dir="ltr">
            #{{ orderId }}
          </span>
        </div>
        <div v-if="walletTransactionId" class="jb-result-info-row">
          <span class="jb-result-info-label">شماره تراکنش</span>
          <span class="jb-result-info-value" dir="ltr">
            {{ walletTransactionId }}
          </span>
        </div>
      </div>

      <!-- دکمه‌ها -->
      <div class="jb-result-actions">
        <NuxtLink :to="primaryAction.to" class="jb-result-btn-primary">
          <span>{{ primaryAction.label }}</span>
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
          </svg>
        </NuxtLink>

        <NuxtLink v-if="isSuccess && orderId" to="/" class="jb-result-btn-secondary">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M3 12l9-9 9 9M5 10v10a1 1 0 001 1h3a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1h3a1 1 0 001-1V10" />
          </svg>
          بازگشت به صفحه اصلی
        </NuxtLink>
      </div>

      <!-- پیام کمکی برای شکست -->
      <div v-if="!isSuccess" class="jb-result-help">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M18.364 5.636l-3.536 3.536m0 5.656l3.536 3.536M9.172 9.172L5.636 5.636m3.536 9.192l-3.536 3.536M21 12a9 9 0 11-18 0 9 9 0 0118 0zm-5 0a4 4 0 11-8 0 4 4 0 018 0z" />
        </svg>
        <span>در صورت وجود مشکل با پشتیبانی تماس بگیرید</span>
      </div>
    </div>
  </div>
</template>

<script setup>
const route = useRoute();

const status = computed(() =>
  route.query.status === "success" ? "success" : "failed"
);
const orderId = computed(() => route.query.order_id || null);
const walletTransactionId = computed(
  () => route.query.wallet_transaction_id || null
);

const isSuccess = computed(() => status.value === "success");

const title = computed(() =>
  isSuccess.value ? "پرداخت با موفقیت انجام شد" : "پرداخت ناموفق بود"
);

const description = computed(() => {
  if (isSuccess.value) {
    if (orderId.value) return "سفارش شما با موفقیت ثبت و پرداخت شد.";
    if (walletTransactionId.value) return "کیف پول شما با موفقیت شارژ شد.";
    return "تراکنش شما با موفقیت انجام شد.";
  }
  return "متاسفانه در فرایند پرداخت مشکلی پیش آمد. در صورت کسر وجه، مبلغ تا ۷۲ ساعت آینده به حساب شما بازخواهد گشت.";
});

const primaryAction = computed(() => {
  if (isSuccess.value && orderId.value) {
    return {
      label: "مشاهده جزئیات سفارش",
      to: `/panel/order/${orderId.value}`,
    };
  }
  return { label: "بازگشت به صفحه اصلی", to: "/" };
});
useSeoMeta({
  title: () => (isSuccess.value ? 'پرداخت موفق | فروشگاه دکتر اسماعیلی' : 'پرداخت ناموفق | فروشگاه دکتر اسماعیلی'),
  description: 'نتیجه تراکنش پرداخت در فروشگاه دکتر اسماعیلی.',
  robots: 'noindex, nofollow',
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/payment/result' }],
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه نتیجه پرداخت - مینیمال سبز
   ========================================================= */

.jb-result-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 32px 16px;
  background: #f7fafc;
}

/* ===== کارت ===== */
.jb-result-card {
  width: 100%;
  max-width: 440px;
  padding: 36px 24px 28px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  text-align: center;
  box-shadow: 0 4px 16px rgba(14, 90, 58, 0.06);
}

.jb-result-card.is-success {
  border-color: rgba(34, 197, 94, 0.25);
}

.jb-result-card.is-failed {
  border-color: rgba(229, 62, 62, 0.25);
}

/* =========================================================
   آیکون نتیجه
   ========================================================= */
.jb-result-icon-wrap {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
}

.jb-result-icon {
  width: 72px;
  height: 72px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  color: #fff;
}

.jb-icon-success {
  background: #16a34a;
}

.jb-icon-failed {
  background: #e53e3e;
}

/* =========================================================
   تیتر
   ========================================================= */
.jb-result-title {
  font-size: 1.15rem;
  font-weight: 800;
  line-height: 1.5;
  letter-spacing: -0.3px;
  margin-bottom: 12px;
}

.jb-title-success {
  color: #16a34a;
}

.jb-title-failed {
  color: #e53e3e;
}

/* خط جداکننده */
.jb-result-divider {
  width: 60px;
  height: 2px;
  border-radius: 1px;
  margin: 0 auto 16px;
}

.jb-divider-success {
  background: #16a34a;
}

.jb-divider-failed {
  background: #e53e3e;
}

/* =========================================================
   توضیحات
   ========================================================= */
.jb-result-desc {
  font-size: 0.85rem;
  line-height: 1.9;
  color: #4a5568;
  font-weight: 500;
  padding: 0 6px;
  margin-bottom: 20px;
}

/* =========================================================
   اطلاعات تراکنش
   ========================================================= */
.jb-result-info {
  display: flex;
  flex-direction: column;
  gap: 6px;
  padding: 12px 14px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 6px;
  margin-bottom: 20px;
}

.jb-result-info-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  font-size: 0.8rem;
}

.jb-result-info-label {
  color: #718096;
  font-weight: 500;
}

.jb-result-info-value {
  color: #0e5a3a;
  font-weight: 700;
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
}

/* =========================================================
   دکمه‌ها
   ========================================================= */
.jb-result-actions {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.jb-result-btn-primary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
  padding: 11px 20px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 700;
  font-family: inherit;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-result-btn-primary:hover {
  background: #0b472e;
}

.jb-result-btn-secondary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
  padding: 10px 20px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.15);
  border-radius: 6px;
  color: #0e5a3a;
  font-size: 0.82rem;
  font-weight: 600;
  font-family: inherit;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-result-btn-secondary:hover {
  background: #f7fafc;
  border-color: rgba(14, 90, 58, 0.3);
}

/* =========================================================
   پیام کمکی
   ========================================================= */
.jb-result-help {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  margin-top: 16px;
  padding: 8px 14px;
  background: #f7fafc;
  border-radius: 6px;
  color: #718096;
  font-size: 0.72rem;
  font-weight: 500;
  width: 100%;
}

.jb-result-help svg {
  color: #0e5a3a;
  flex-shrink: 0;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 480px) {
  .jb-result-card {
    padding: 28px 18px 22px;
  }

  .jb-result-icon {
    width: 64px;
    height: 64px;
  }

  .jb-result-title {
    font-size: 1.05rem;
  }

  .jb-result-desc {
    font-size: 0.8rem;
    line-height: 1.85;
  }

  .jb-result-info-row {
    font-size: 0.75rem;
  }
}
</style>