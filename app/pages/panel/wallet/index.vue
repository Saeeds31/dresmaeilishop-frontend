<template>
  <div class="jb-wallet-card">
    <!-- ===== هدر ===== -->
    <div class="jb-wallet-header">
      <h3 class="jb-wallet-header-title">
        <span class="jb-wallet-header-icon">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M3 10h18M5 6h14a2 2 0 012 2v10a2 2 0 01-2 2H5a2 2 0 01-2-2V8a2 2 0 012-2z" />
          </svg>
        </span>
        کیف پول
      </h3>
    </div>

    <!-- خط بریده‌بریده -->
    <div class="jb-divider"></div>

    <!-- ===== حالت لودینگ ===== -->
    <div v-if="!walletData" class="jb-state-block">
      <SharedLoadingSpinner size="lg" />
    </div>

    <div v-else class="jb-wallet-body">
      <!-- ============================================================
           کارت موجودی
      ============================================================ -->
      <div class="jb-balance-card">
        <div class="jb-balance-inner">
          <div>
            <p class="jb-balance-label">
              <span class="jb-balance-label-icon"></span>
              موجودی کیف پول
            </p>
            <p class="jb-balance-value">
              {{ wallet?.balance?.toLocaleString() || "0" }}
              <span class="jb-balance-unit">تومان</span>
            </p>
          </div>

          <div class="jb-balance-icon">
            <Wallet class="w-7 h-7" />
          </div>
        </div>

        <!-- حباب تزئینی -->
        <span class="jb-balance-bubble jb-balance-bubble-1"></span>
        <span class="jb-balance-bubble jb-balance-bubble-2"></span>
      </div>

      <!-- ============================================================
           فرم شارژ کیف پول
      ============================================================ -->
      <div class="jb-charge-box">
        <h4 class="jb-section-title">
          <span class="jb-title-dot"></span>
          شارژ کیف پول
        </h4>

        <form @submit.prevent="handleCharge" class="jb-charge-form">
          <div class="jb-form-field">
            <label for="amount" class="jb-form-label">
              <span class="jb-form-dot"></span>
              مبلغ (تومان)
            </label>
            <div class="jb-input-wrap">
              <span class="jb-input-icon">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
              </span>
              <input id="amount" v-model.number="chargeAmount" type="number" min="10000" step="1000"
                class="jb-form-input" placeholder="حداقل 10,000 تومان" required />
            </div>
            <p class="jb-form-hint">
              مبلغ را به تومان وارد کنید (حداقل ۱۰,۰۰۰ تومان)
            </p>
          </div>

          <button type="submit" :disabled="isCharging || !chargeAmount || chargeAmount < 10000"
            class="jb-btn-linear jb-charge-btn">
            <span v-if="!isCharging" class="jb-charge-btn-inner">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 4v16m8-8H4" />
              </svg>
              شارژ کیف پول
            </span>
            <span v-else class="jb-charge-btn-inner">
              <svg class="jb-spinner" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
                <path class="opacity-75" fill="currentColor"
                  d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
              </svg>
              در حال انتقال به درگاه...
            </span>
          </button>
        </form>
      </div>

      <!-- ============================================================
           تراکنش‌های اخیر
      ============================================================ -->
      <div class="jb-transactions-box">
        <h4 class="jb-section-title">
          <span class="jb-title-dot"></span>
          تراکنش‌های اخیر
        </h4>

        <!-- لودینگ -->
        <div v-if="!transactionsData" class="jb-state-block">
          <SharedLoadingSpinner size="md" />
        </div>

        <!-- خالی -->
        <div v-else-if="transactions.length === 0" class="jb-state-empty">
          <div class="jb-empty-icon">
            <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8"
                d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" />
            </svg>
          </div>
          <span>هیچ تراکنشی یافت نشد</span>
        </div>

        <!-- لیست تراکنش‌ها -->
        <div v-else class="jb-transactions-list">
          <div v-for="transaction in transactions" :key="transaction.id" class="jb-transaction-item" :class="transaction.type === 'credit'
              ? 'is-credit'
              : transaction.type === 'purchase'
                ? 'is-purchase'
                : 'is-debit'
            ">
            <div class="jb-transaction-inner">
              <div class="jb-transaction-icon-wrap">
                <component :is="getTransactionIcon(transaction.type)" class="w-4 h-4" />
              </div>

              <div class="jb-transaction-info">
                <p class="jb-transaction-title">
                  {{ getTransactionTitle(transaction.type) }}
                </p>
                <p class="jb-transaction-date">{{ transaction.date }}</p>
                <p v-if="transaction.description" class="jb-transaction-desc">
                  {{ transaction.description }}
                </p>
              </div>
            </div>

            <div class="jb-transaction-amount-wrap">
              <p class="jb-transaction-amount" :class="transaction.type === 'credit'
                  ? 'is-credit'
                  : 'is-debit'
                ">
                {{ transaction.type === "credit" ? "+" : "-" }}
                {{ transaction.amount?.toLocaleString() }}
                <span class="jb-transaction-unit">تومان</span>
              </p>
              <span class="jb-transaction-status" :class="`status-${transaction.status}`">
                {{ getStatusText(transaction.status) }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useApi } from "@/composables/api/useApi";
import {
  Wallet,
  ArrowUpCircle,
  ArrowDownCircle,
  ShoppingCart,
} from "lucide-vue-next";
import { ref, computed } from "vue";
import { toast } from "vue3-toastify";

definePageMeta({
  layout: "panel",
  middleware: "auth",
});

const { getWallet, getWalletTransactions, chargeWallet } = useApi();

const { data: walletData, refresh: refreshWallet } = await useAsyncData(
  "userWallet",
  async () => {
    try {
      return await getWallet();
    } catch (error) {
      console.error("❌ Failed to fetch wallet:", error);
      return { data: null };
    }
  },
  {
    lazy: true,
    server: false,
  }
);

const { data: transactionsData, refresh: refreshTransactions } =
  await useAsyncData(
    "walletTransactions",
    async () => {
      try {
        return await getWalletTransactions();
      } catch (error) {
        console.error("❌ Failed to fetch transactions:", error);
        return { data: [] };
      }
    },
    {
      lazy: true,
      server: false,
    }
  );

const wallet = computed(() => walletData.value?.data || null);
const transactions = computed(
  () => transactionsData.value?.transactions || []
);

const chargeAmount = ref(null);
const isCharging = ref(false);

async function handleCharge() {
  if (!chargeAmount.value || chargeAmount.value < 10000) {
    return;
  }

  isCharging.value = true;
  try {
    const response = await chargeWallet(chargeAmount.value);

    if (response.data?.payment_url) {
      window.location.href = response.data.payment_url;
    } else {
      await refreshWallet();
      await refreshTransactions();
      chargeAmount.value = null;
    }
  } catch (error) {
    console.error("❌ Charge failed:", error);
    toast.error("شارژ کیف پول با خطا مواجه شد. لطفا دوباره تلاش کنید.");
  } finally {
    isCharging.value = false;
  }
}

function getTransactionIcon(type) {
  switch (type) {
    case "credit":
      return ArrowUpCircle;
    case "debit":
      return ArrowDownCircle;
    case "purchase":
      return ShoppingCart;
    default:
      return Wallet;
  }
}

function getTransactionTitle(type) {
  switch (type) {
    case "credit":
      return "واریز به کیف پول";
    case "debit":
      return "برداشت از کیف پول";
    case "purchase":
      return "خرید";
    default:
      return "تراکنش";
  }
}

function getStatusText(status) {
  switch (status) {
    case "completed":
    case "success":
      return "موفق";
    case "pending":
      return "در انتظار";
    case "failed":
    case "cancelled":
      return "ناموفق";
    default:
      return status;
  }
}

useSeoMeta({
  title: () => 'کیف پول پنل کاربری | فروشگاه دکتر اسماعیلی',
  description: ' کیف پول کاربر در پنل اختصاصی',
  robots: 'noindex, nofollow',
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/panel/wallet' }],
  // 🔹 جلوگیری از ایندکس شدن با متا تگ اضافی برای اطمینان بیشتر
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه کیف پول - تم پودری
   ========================================================= */

.jb-wallet-card {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 24px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(214, 51, 132, 0.1);
}

/* ===== هدر ===== */
.jb-wallet-header {
  margin-bottom: 4px;
}

.jb-wallet-header-title {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  font-size: 1.15rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-wallet-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  border-radius: 12px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
}

/* خط بریده‌بریده */
.jb-divider {
  height: 0;
  border-top: 2px dashed var(--jb-pink-200);
  margin: 16px 0 24px;
  position: relative;
}

.jb-divider::before,
.jb-divider::after {
  content: "";
  position: absolute;
  top: -5px;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: var(--jb-pink-400);
  box-shadow: 0 0 0 3px #fff;
}

.jb-divider::before {
  right: -4px;
}

.jb-divider::after {
  left: -4px;
}

/* ===== بدنه ===== */
.jb-wallet-body {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* =========================================================
   کارت موجودی
   ========================================================= */
.jb-balance-card {
  position: relative;
  padding: 24px;
  border-radius: 22px;
  background: linear-gradient(135deg, var(--jb-pink-500) 0%, var(--jb-fuchsia) 100%);
  color: #fff;
  overflow: hidden;
  box-shadow: 0 15px 35px rgba(214, 51, 132, 0.35);
}

.jb-balance-inner {
  position: relative;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

.jb-balance-label {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 10px;
}

.jb-balance-label-icon {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.3);
}

.jb-balance-value {
  font-size: 1.8rem;
  font-weight: 900;
  line-height: 1.1;
  display: inline-flex;
  align-items: baseline;
  gap: 6px;
  letter-spacing: -0.5px;
}

.jb-balance-unit {
  font-size: 0.85rem;
  font-weight: 600;
  opacity: 0.9;
}

.jb-balance-icon {
  width: 58px;
  height: 58px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.2);
  border: 2px solid rgba(255, 255, 255, 0.4);
  color: #fff;
  flex-shrink: 0;
  animation: jbWalletFloat 3s ease-in-out infinite;
}

@keyframes jbWalletFloat {

  0%,
  100% {
    transform: translateY(0);
  }

  50% {
    transform: translateY(-6px);
  }
}

/* حباب‌های تزئینی */
.jb-balance-bubble {
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.12);
  pointer-events: none;
}

.jb-balance-bubble-1 {
  width: 140px;
  height: 140px;
  top: -50px;
  right: -40px;
}

.jb-balance-bubble-2 {
  width: 90px;
  height: 90px;
  bottom: -30px;
  left: -20px;
  background: rgba(255, 255, 255, 0.08);
}

/* =========================================================
   فرم شارژ
   ========================================================= */
.jb-charge-box {
  padding: 20px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px solid var(--jb-pink-200);
  border-radius: 20px;
}

.jb-section-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 0.95rem;
  font-weight: 900;
  color: var(--jb-text);
  margin-bottom: 16px;
}

.jb-title-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
}

.jb-charge-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.jb-form-field {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.jb-form-label {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
}

.jb-form-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 2px rgba(214, 51, 132, 0.18);
}

.jb-input-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.jb-input-icon {
  position: absolute;
  right: 14px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  align-items: center;
  justify-content: center;
  width: 30px;
  height: 30px;
  border-radius: 9px;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  color: var(--jb-fuchsia);
  pointer-events: none;
  z-index: 1;
  transition: all 0.25s ease;
}

.jb-form-input {
  width: 100%;
  padding: 13px 54px 13px 16px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 14px;
  font-size: 0.92rem;
  font-weight: 800;
  color: var(--jb-text);
  outline: none;
  transition: all 0.25s ease;
  font-family: inherit;
  text-align: right;
  direction: ltr;
}

.jb-form-input::placeholder {
  color: var(--jb-text-muted);
  font-weight: 500;
  direction: rtl;
}

.jb-form-input:focus {
  border-color: var(--jb-fuchsia);
  box-shadow: 0 0 0 4px rgba(214, 51, 132, 0.15);
}

.jb-input-wrap:focus-within .jb-input-icon {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.35);
}

/* حذف فلش‌های اینپوت عدد */
.jb-form-input::-webkit-outer-spin-button,
.jb-form-input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.jb-form-input[type="number"] {
  -moz-appearance: textfield;
}

.jb-form-hint {
  font-size: 0.72rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  padding-right: 4px;
}

/* دکمه شارژ */
.jb-charge-btn {
  width: 100%;
  justify-content: center;
  padding: 14px 24px !important;
  font-size: 0.92rem !important;
}

.jb-charge-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none !important;
}

.jb-charge-btn-inner {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.jb-spinner {
  width: 18px;
  height: 18px;
  animation: jbSpin 0.9s linear infinite;
}

@keyframes jbSpin {
  to {
    transform: rotate(360deg);
  }
}

/* =========================================================
   تراکنش‌ها
   ========================================================= */
.jb-transactions-box {
  padding: 20px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 20px;
}

.jb-state-block {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 32px 20px;
}

.jb-state-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 32px 20px;
  color: var(--jb-text-muted);
  font-size: 0.85rem;
  font-weight: 700;
}

.jb-empty-icon {
  width: 72px;
  height: 72px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border: 2.5px dashed var(--jb-pink-300);
  color: var(--jb-pink-400);
}

/* لیست */
.jb-transactions-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.jb-transaction-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 14px 16px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 16px;
  transition: all 0.3s ease;
}

.jb-transaction-item:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.12);
  transform: translateY(-2px);
}

.jb-transaction-item.is-credit {
  border-color: rgba(34, 197, 94, 0.3);
}

.jb-transaction-item.is-credit:hover {
  border-color: rgba(34, 197, 94, 0.6);
  box-shadow: 0 8px 22px rgba(34, 197, 94, 0.15);
}

.jb-transaction-item.is-purchase,
.jb-transaction-item.is-debit {
  border-color: rgba(201, 24, 74, 0.25);
}

.jb-transaction-item.is-purchase:hover,
.jb-transaction-item.is-debit:hover {
  border-color: rgba(201, 24, 74, 0.55);
  box-shadow: 0 8px 22px rgba(201, 24, 74, 0.12);
}

.jb-transaction-inner {
  display: flex;
  align-items: center;
  gap: 12px;
  flex: 1;
  min-width: 0;
}

.jb-transaction-icon-wrap {
  flex-shrink: 0;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  border: 2px solid #fff;
}

.is-credit .jb-transaction-icon-wrap {
  background: linear-gradient(135deg, #4ade80, #16a34a);
  color: #fff;
  box-shadow: 0 6px 16px rgba(34, 197, 94, 0.35);
}

.is-purchase .jb-transaction-icon-wrap,
.is-debit .jb-transaction-icon-wrap {
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  color: #fff;
  box-shadow: 0 6px 16px rgba(201, 24, 74, 0.35);
}

.jb-transaction-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
  flex: 1;
}

.jb-transaction-title {
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
}

.jb-transaction-date {
  font-size: 0.72rem;
  color: var(--jb-text-muted);
  font-weight: 600;
}

.jb-transaction-desc {
  font-size: 0.72rem;
  color: var(--jb-text-soft);
  font-weight: 600;
  margin-top: 2px;
}

.jb-transaction-amount-wrap {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 4px;
  flex-shrink: 0;
}

.jb-transaction-amount {
  font-size: 0.92rem;
  font-weight: 900;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-transaction-amount.is-credit {
  color: #16a34a;
}

.jb-transaction-amount.is-debit {
  color: #c9184a;
}

.jb-transaction-unit {
  font-size: 0.65rem;
  font-weight: 600;
  opacity: 0.8;
}

/* بج وضعیت */
.jb-transaction-status {
  display: inline-flex;
  align-items: center;
  padding: 3px 10px;
  border-radius: 999px;
  font-size: 0.68rem;
  font-weight: 800;
}

.status-completed,
.status-success {
  background: linear-gradient(135deg, #dcfce7, #bbf7d0);
  color: #16a34a;
}

.status-pending {
  background: linear-gradient(135deg, #fef3c7, #fde68a);
  color: #b45309;
}

.status-failed,
.status-cancelled {
  background: linear-gradient(135deg, #ffe0e8, #ffc2d6);
  color: #c9184a;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-wallet-card {
    padding: 16px;
    border-radius: 20px;
  }

  .jb-balance-card {
    padding: 20px;
    border-radius: 18px;
  }

  .jb-balance-value {
    font-size: 1.5rem;
  }

  .jb-balance-icon {
    width: 50px;
    height: 50px;
  }

  .jb-charge-box,
  .jb-transactions-box {
    padding: 16px;
    border-radius: 16px;
  }

  .jb-transaction-item {
    padding: 12px;
    gap: 10px;
  }

  .jb-transaction-icon-wrap {
    width: 36px;
    height: 36px;
  }

  .jb-transaction-title {
    font-size: 0.8rem;
  }

  .jb-transaction-amount {
    font-size: 0.85rem;
  }
}

@media (max-width: 400px) {
  .jb-transaction-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .jb-transaction-amount-wrap {
    width: 100%;
    align-items: flex-start;
    flex-direction: row;
    justify-content: space-between;
    padding-top: 10px;
    border-top: 1.5px dashed var(--jb-pink-200);
  }
}
</style>