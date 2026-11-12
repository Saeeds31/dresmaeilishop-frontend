<template>
  <div class="jb-wallet-result-page">
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
      <div v-if="isSuccess && walletTransactionId" class="jb-result-info">
        <div class="jb-result-info-row">
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

        <NuxtLink v-if="!isSuccess" to="/" class="jb-result-btn-secondary">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M3 12l9-9 9 9M5 10v10a1 1 0 001 1h3a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1h3a1 1 0 001-1V10" />
          </svg>
          بازگشت به صفحه اصلی
        </NuxtLink>
      </div>
    </div>
  </div>
</template>

<script setup>
const route = useRoute();
const status = computed(() =>
  route.query.status === "success" ? "success" : "failed"
);
const walletTransactionId = computed(
  () => route.query.wallet_transaction_id || null
);
const amount = computed(() => route.query.amount || null);
const balance = computed(() => route.query.balance || null);
const isSuccess = computed(() => status.value === "success");

const title = computed(() =>
  isSuccess.value
    ? "شارژ کیف پول با موفقیت انجام شد"
    : "شارژ کیف پول ناموفق بود"
);

const description = computed(() => {
  if (isSuccess.value) {
    let text = "کیف پول شما با موفقیت شارژ شد.";
    if (amount.value) {
      text += ` مبلغ ${new Intl.NumberFormat("fa-IR").format(
        Number(amount.value)
      )} تومان به حساب شما واریز شد.`;
    }
    if (balance.value) {
      text += ` موجودی فعلی شما: ${new Intl.NumberFormat("fa-IR").format(
        Number(balance.value)
      )} تومان`;
    }
    return text;
  }
  return "متاسفانه در فرایند شارژ کیف پول مشکلی پیش آمد. در صورت کسر وجه، مبلغ تا ۷۲ ساعت آینده به حساب شما بازخواهد گشت.";
});

const primaryAction = computed(() => {
  if (isSuccess.value) {
    return {
      label: "مشاهده کیف پول",
      to: "/panel/wallet",
    };
  }
  return {
    label: "تلاش مجدد",
    to: "/panel/wallet",
  };
});
useSeoMeta({
  title: () => 'کیف پول پنل کاربری | فروشگاه دکتر اسماعیلی',
  description: ' کیف پول کاربر در پنل اختصاصی',
  robots: 'noindex, nofollow',
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/panel/result' }],
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه نتیجه شارژ کیف پول - مینیمال سبز
   ========================================================= */

.jb-wallet-result-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 32px 16px;
  background: #f7fafc;
}

/* کارت */
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

/* آیکون */
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

/* تیتر */
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

/* توضیحات */
.jb-result-desc {
  font-size: 0.85rem;
  line-height: 1.9;
  color: #4a5568;
  font-weight: 500;
  padding: 0 6px;
  margin-bottom: 20px;
}

/* اطلاعات تراکنش */
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
  word-break: break-all;
}

/* دکمه‌ها */
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

/* ریسپانسیو */
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
}
</style>