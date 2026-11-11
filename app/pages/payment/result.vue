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
  description: 'نتیجه تراکنش پرداخت در فروشگاه فروشگاه دکتر اسماعیلی.',
  robots: 'noindex, nofollow', // 🔴 مهم‌ترین خط
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/payment/result' }],
  // 🔹 جلوگیری از ایندکس شدن با متا تگ اضافی برای اطمینان بیشتر
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<template>
  <div class="jb-result-page">
    <!-- حباب‌های تزئینی پس‌زمینه -->
    <div class="jb-result-bubble jb-result-bubble-1"></div>
    <div class="jb-result-bubble jb-result-bubble-2"></div>

    <div class="jb-result-card" :class="isSuccess ? 'is-success' : 'is-failed'">
      <!-- ===== آیکون نتیجه ===== -->
      <div class="jb-result-icon-wrap">
        <div class="jb-result-icon" :class="isSuccess ? 'jb-icon-success' : 'jb-icon-failed'">
          <svg v-if="isSuccess" class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7" />
          </svg>
          <svg v-else class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </div>

        <!-- حلقه‌های تزئینی -->
        <span class="jb-result-ring jb-ring-1"></span>
        <span class="jb-result-ring jb-ring-2"></span>
      </div>

      <!-- ===== تیتر ===== -->
      <h1 class="jb-result-title" :class="isSuccess ? 'jb-title-success' : 'jb-title-failed'">
        {{ title }}
      </h1>

      <!-- خط تزئینی -->
      <div class="jb-result-divider" :class="isSuccess ? 'jb-divider-success' : 'jb-divider-failed'"></div>

      <!-- ===== توضیحات ===== -->
      <p class="jb-result-desc">{{ description }}</p>

      <!-- ===== اطلاعات تراکنش (در صورت موفقیت) ===== -->
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

      <!-- ===== دکمه‌ها ===== -->
      <div class="jb-result-actions">
        <NuxtLink :to="primaryAction.to" class="jb-btn-linear jb-result-btn-primary">
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

<style scoped>
/* =========================================================
   صفحه نتیجه پرداخت - تم پودری
   ========================================================= */

.jb-result-page {
  position: relative;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px 16px;
  background: linear-gradient(135deg, #fff0f5 0%, #ffe0eb 50%, #ffd0e0 100%);
  background-attachment: fixed;
  overflow: hidden;
}

/* حباب‌های تزئینی پس‌زمینه */
.jb-result-bubble {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  z-index: 0;
  animation: jbFloatBubble 8s ease-in-out infinite;
}

.jb-result-bubble-1 {
  width: 320px;
  height: 320px;
  background: radial-gradient(circle, rgba(255, 122, 160, 0.22), transparent 70%);
  top: -120px;
  right: -100px;
}

.jb-result-bubble-2 {
  width: 260px;
  height: 260px;
  background: radial-gradient(circle, rgba(214, 51, 132, 0.18), transparent 70%);
  bottom: -80px;
  left: -80px;
  animation-delay: 3s;
}

@keyframes jbFloatBubble {

  0%,
  100% {
    transform: translateY(0) scale(1);
  }

  50% {
    transform: translateY(-24px) scale(1.05);
  }
}

/* ===== کارت ===== */
.jb-result-card {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 480px;
  padding: 48px 32px 36px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 2px solid var(--jb-pink-200);
  border-radius: 32px;
  text-align: center;
  box-shadow:
    0 25px 60px rgba(214, 51, 132, 0.18),
    0 10px 25px rgba(255, 122, 160, 0.15),
    inset 0 1px 0 rgba(255, 255, 255, 0.9);
  animation: jbCardIn 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes jbCardIn {
  0% {
    opacity: 0;
    transform: translateY(40px) scale(0.95);
  }

  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.jb-result-card.is-success {
  border-color: rgba(34, 197, 94, 0.35);
}

.jb-result-card.is-failed {
  border-color: rgba(239, 68, 68, 0.35);
}

/* =========================================================
   آیکون نتیجه
   ========================================================= */
.jb-result-icon-wrap {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 110px;
  height: 110px;
  margin-bottom: 24px;
}

.jb-result-icon {
  position: relative;
  z-index: 2;
  width: 88px;
  height: 88px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  color: #fff;
  border: 4px solid #fff;
  animation: jbIconPop 0.6s cubic-bezier(0.4, 0, 0.2, 1) 0.15s backwards;
}

@keyframes jbIconPop {
  0% {
    transform: scale(0);
  }

  60% {
    transform: scale(1.15);
  }

  100% {
    transform: scale(1);
  }
}

.jb-icon-success {
  background: linear-gradient(135deg, #4ade80, #16a34a);
  box-shadow:
    0 12px 30px rgba(34, 197, 94, 0.45),
    0 4px 10px rgba(34, 197, 94, 0.25);
}

.jb-icon-failed {
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  box-shadow:
    0 12px 30px rgba(201, 24, 74, 0.45),
    0 4px 10px rgba(201, 24, 74, 0.25);
}

/* حلقه‌های تزئینی */
.jb-result-ring {
  position: absolute;
  top: 50%;
  left: 50%;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  pointer-events: none;
  z-index: 1;
}

.jb-ring-1 {
  width: 100px;
  height: 100px;
  border: 2px dashed;
  animation: jbRingPulse 2.5s ease-in-out infinite;
}

.jb-ring-2 {
  width: 120px;
  height: 120px;
  border: 2px dashed;
  animation: jbRingPulse 2.5s ease-in-out infinite 0.4s;
  opacity: 0.5;
}

.is-success .jb-ring-1,
.is-success .jb-ring-2 {
  border-color: rgba(34, 197, 94, 0.35);
}

.is-failed .jb-ring-1,
.is-failed .jb-ring-2 {
  border-color: rgba(201, 24, 74, 0.35);
}

@keyframes jbRingPulse {

  0%,
  100% {
    transform: translate(-50%, -50%) scale(1);
    opacity: 0.7;
  }

  50% {
    transform: translate(-50%, -50%) scale(1.08);
    opacity: 1;
  }
}

/* =========================================================
   تیتر
   ========================================================= */
.jb-result-title {
  font-size: 1.35rem;
  font-weight: 900;
  line-height: 1.5;
  letter-spacing: -0.3px;
  margin-bottom: 14px;
}

.jb-title-success {
  color: #16a34a;
}

.jb-title-failed {
  color: #c9184a;
}

/* خط تزئینی زیر تیتر */
.jb-result-divider {
  width: 80px;
  height: 3px;
  border-radius: 999px;
  margin: 0 auto 18px;
}

.jb-divider-success {
  background: linear-gradient(90deg, transparent, #4ade80, #16a34a, #4ade80, transparent);
  box-shadow: 0 2px 8px rgba(34, 197, 94, 0.35);
}

.jb-divider-failed {
  background: linear-gradient(90deg, transparent, #ff4d6d, #c9184a, #ff4d6d, transparent);
  box-shadow: 0 2px 8px rgba(201, 24, 74, 0.35);
}

/* =========================================================
   توضیحات
   ========================================================= */
.jb-result-desc {
  font-size: 0.88rem;
  line-height: 2;
  color: var(--jb-text-soft);
  font-weight: 600;
  padding: 0 8px;
  margin-bottom: 24px;
}

/* =========================================================
   اطلاعات تراکنش
   ========================================================= */
.jb-result-info {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 14px 16px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px dashed var(--jb-pink-300);
  border-radius: 16px;
  margin-bottom: 24px;
}

.jb-result-info-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  font-size: 0.82rem;
}

.jb-result-info-label {
  color: var(--jb-text-muted);
  font-weight: 700;
}

.jb-result-info-value {
  color: var(--jb-fuchsia);
  font-weight: 900;
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
}

/* =========================================================
   دکمه‌ها
   ========================================================= */
.jb-result-actions {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.jb-result-btn-primary {
  width: 100%;
  justify-content: center;
  padding: 14px 24px !important;
  font-size: 0.92rem !important;
}

.jb-result-btn-secondary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
  padding: 13px 24px;
  background: #fff;
  border: 2px solid var(--jb-pink-300);
  border-radius: 14px;
  color: var(--jb-fuchsia);
  font-size: 0.88rem;
  font-weight: 800;
  font-family: inherit;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-result-btn-secondary:hover {
  background: var(--jb-pink-100);
  border-color: var(--jb-fuchsia);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.2);
}

/* =========================================================
   پیام کمکی
   ========================================================= */
.jb-result-help {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  margin-top: 20px;
  padding: 10px 16px;
  background: var(--jb-pink-50);
  border-radius: 12px;
  color: var(--jb-text-muted);
  font-size: 0.75rem;
  font-weight: 700;
  width: 100%;
}

.jb-result-help svg {
  color: var(--jb-fuchsia);
  flex-shrink: 0;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 480px) {
  .jb-result-card {
    padding: 40px 20px 28px;
    border-radius: 26px;
  }

  .jb-result-icon-wrap {
    width: 96px;
    height: 96px;
  }

  .jb-result-icon {
    width: 76px;
    height: 76px;
  }

  .jb-ring-1 {
    width: 88px;
    height: 88px;
  }

  .jb-ring-2 {
    width: 106px;
    height: 106px;
  }

  .jb-result-title {
    font-size: 1.15rem;
  }

  .jb-result-desc {
    font-size: 0.82rem;
    line-height: 1.9;
  }

  .jb-result-info-row {
    font-size: 0.78rem;
  }
}
</style>