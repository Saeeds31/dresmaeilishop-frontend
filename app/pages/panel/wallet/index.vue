<template>
  <div class="jb-wallet-card">
    <!-- هدر -->
    <div class="jb-wallet-header">
      <h3 class="jb-wallet-header-title">
        <span class="jb-wallet-header-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M3 10h18M5 6h14a2 2 0 012 2v10a2 2 0 01-2 2H5a2 2 0 01-2-2V8a2 2 0 012-2z" />
          </svg>
        </span>
        کیف پول
      </h3>
    </div>

    <!-- خط جداکننده -->
    <div class="jb-divider"></div>

    <!-- حالت لودینگ -->
    <div v-if="!walletData" class="jb-state-block">
      <SharedLoadingSpinner size="lg" />
    </div>

    <div v-else class="jb-wallet-body">
      <!-- کارت موجودی -->
      <div class="jb-balance-card">
        <div class="jb-balance-inner">
          <div>
            <p class="jb-balance-label">موجودی کیف پول</p>
            <p class="jb-balance-value">
              {{ wallet?.balance?.toLocaleString() || "0" }}
              <span class="jb-balance-unit">تومان</span>
            </p>
          </div>

          <div class="jb-balance-icon">
            <Wallet class="w-6 h-6" />
          </div>
        </div>
      </div>

      <!-- فرم شارژ کیف پول -->
      <div class="jb-charge-box">
        <h4 class="jb-section-title">شارژ کیف پول</h4>

        <form @submit.prevent="handleCharge" class="jb-charge-form">
          <div class="jb-form-field">
            <label for="amount" class="jb-form-label">مبلغ (تومان)</label>
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
            class="jb-charge-btn">
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

      <!-- تراکنش‌های اخیر -->
      <div class="jb-transactions-box">
        <h4 class="jb-section-title">تراکنش‌های اخیر</h4>

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
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه کیف پول - مینیمال سبز
   ========================================================= */

.jb-wallet-card {
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

/* ===== هدر ===== */
.jb-wallet-header {
  margin-bottom: 4px;
}

.jb-wallet-header-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-wallet-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 6px;
  background: #0e5a3a;
  color: #fff;
}

/* خط جداکننده */
.jb-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 14px 0 20px;
}

/* ===== بدنه ===== */
.jb-wallet-body {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* =========================================================
   کارت موجودی
   ========================================================= */
.jb-balance-card {
  position: relative;
  padding: 20px;
  border-radius: 8px;
  background: #0e5a3a;
  color: #fff;
  overflow: hidden;
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
  font-size: 0.8rem;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 8px;
}

.jb-balance-value {
  font-size: 1.6rem;
  font-weight: 800;
  line-height: 1.1;
  display: inline-flex;
  align-items: baseline;
  gap: 6px;
  letter-spacing: -0.5px;
}

.jb-balance-unit {
  font-size: 0.8rem;
  font-weight: 500;
  opacity: 0.8;
}

.jb-balance-icon {
  width: 48px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.15);
  color: #fff;
  flex-shrink: 0;
}

/* =========================================================
   فرم شارژ
   ========================================================= */
.jb-charge-box {
  padding: 16px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 8px;
}

.jb-section-title {
  font-size: 0.9rem;
  font-weight: 800;
  color: #1a202c;
  margin-bottom: 14px;
}

.jb-charge-form {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.jb-form-field {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.jb-form-label {
  font-size: 0.8rem;
  font-weight: 600;
  color: #1a202c;
}

.jb-input-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.jb-input-icon {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 6px;
  background: #f7fafc;
  color: #0e5a3a;
  pointer-events: none;
  z-index: 1;
}

.jb-form-input {
  width: 100%;
  padding: 10px 48px 10px 14px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.12);
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 600;
  color: #1a202c;
  outline: none;
  transition: all 0.2s ease;
  font-family: inherit;
  text-align: right;
  direction: ltr;
}

.jb-form-input::placeholder {
  color: #a0aec0;
  font-weight: 500;
  direction: rtl;
}

.jb-form-input:focus {
  border-color: #0e5a3a;
  box-shadow: 0 0 0 3px rgba(14, 90, 58, 0.1);
}

.jb-input-wrap:focus-within .jb-input-icon {
  background: #0e5a3a;
  color: #fff;
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
  font-size: 0.7rem;
  color: #a0aec0;
  font-weight: 500;
  padding-right: 2px;
}

/* دکمه شارژ */
.jb-charge-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 10px 20px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-charge-btn:hover:not(:disabled) {
  background: #0b472e;
}

.jb-charge-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.jb-charge-btn-inner {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.jb-spinner {
  width: 16px;
  height: 16px;
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
  padding: 16px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 8px;
}

.jb-state-block {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px 20px;
}

.jb-state-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 24px 20px;
  color: #718096;
  font-size: 0.82rem;
  font-weight: 500;
}

.jb-empty-icon {
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  color: #a0aec0;
}

/* لیست */
.jb-transactions-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.jb-transaction-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding: 12px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 6px;
  transition: all 0.2s ease;
}

.jb-transaction-item:hover {
  border-color: rgba(14, 90, 58, 0.2);
}

.jb-transaction-item.is-credit {
  border-color: rgba(34, 197, 94, 0.2);
}

.jb-transaction-item.is-purchase,
.jb-transaction-item.is-debit {
  border-color: rgba(229, 62, 62, 0.15);
}

.jb-transaction-inner {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 1;
  min-width: 0;
}

.jb-transaction-icon-wrap {
  flex-shrink: 0;
  width: 34px;
  height: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
}

.is-credit .jb-transaction-icon-wrap {
  background: #dcfce7;
  color: #16a34a;
}

.is-purchase .jb-transaction-icon-wrap,
.is-debit .jb-transaction-icon-wrap {
  background: #fff5f5;
  color: #e53e3e;
}

.jb-transaction-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
  flex: 1;
}

.jb-transaction-title {
  font-size: 0.82rem;
  font-weight: 700;
  color: #1a202c;
}

.jb-transaction-date {
  font-size: 0.68rem;
  color: #a0aec0;
  font-weight: 500;
}

.jb-transaction-desc {
  font-size: 0.68rem;
  color: #718096;
  font-weight: 500;
  margin-top: 2px;
}

.jb-transaction-amount-wrap {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 3px;
  flex-shrink: 0;
}

.jb-transaction-amount {
  font-size: 0.85rem;
  font-weight: 800;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-transaction-amount.is-credit {
  color: #16a34a;
}

.jb-transaction-amount.is-debit {
  color: #e53e3e;
}

.jb-transaction-unit {
  font-size: 0.62rem;
  font-weight: 500;
  opacity: 0.8;
}

/* بج وضعیت */
.jb-transaction-status {
  display: inline-flex;
  align-items: center;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 0.65rem;
  font-weight: 600;
}

.status-completed,
.status-success {
  background: #dcfce7;
  color: #16a34a;
}

.status-pending {
  background: #fef3c7;
  color: #b45309;
}

.status-failed,
.status-cancelled {
  background: #fff5f5;
  color: #e53e3e;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-wallet-card {
    padding: 14px;
    border-radius: 8px;
  }

  .jb-balance-card {
    padding: 16px;
  }

  .jb-balance-value {
    font-size: 1.35rem;
  }

  .jb-balance-icon {
    width: 42px;
    height: 42px;
  }

  .jb-charge-box,
  .jb-transactions-box {
    padding: 14px;
  }

  .jb-transaction-item {
    padding: 10px;
    gap: 8px;
  }

  .jb-transaction-icon-wrap {
    width: 30px;
    height: 30px;
  }

  .jb-transaction-title {
    font-size: 0.78rem;
  }

  .jb-transaction-amount {
    font-size: 0.8rem;
  }
}

@media (max-width: 400px) {
  .jb-transaction-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }

  .jb-transaction-amount-wrap {
    width: 100%;
    align-items: flex-start;
    flex-direction: row;
    justify-content: space-between;
    padding-top: 8px;
    border-top: 1px dashed rgba(14, 90, 58, 0.1);
  }
}
</style>