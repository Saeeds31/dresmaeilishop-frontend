<template>
  <div class="jb-orders-card">
    <!-- ===== هدر ===== -->
    <div class="jb-orders-header">
      <h3 class="jb-orders-header-title">
        <span class="jb-orders-header-icon">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
          </svg>
        </span>
        سفارش‌ها
      </h3>
    </div>

    <!-- خط بریده‌بریده -->
    <div class="jb-divider"></div>

    <!-- ===== لودینگ ===== -->
    <div v-if="!ordersData" class="jb-state-block">
      <SharedLoadingSpinner size="lg" />
    </div>

    <!-- ===== خالی ===== -->
    <div v-else-if="orders.length === 0" class="jb-state-block jb-state-empty">
      <div class="jb-state-icon">
        <svg class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
        </svg>
      </div>
      <span>هیچ سفارشی یافت نشد</span>
    </div>

    <!-- ===== لیست سفارشات ===== -->
    <div v-else class="jb-orders-list">
      <div v-for="order in orders" :key="order.id" class="jb-order-card"
        :class="`status-${getStatusClass(order.status)}`">
        <div class="jb-order-card-inner">
          <!-- ===== بالای کارت: وضعیت + مشاهده ===== -->
          <div class="jb-order-top">
            <div class="jb-order-status">
              <span class="jb-order-status-icon" :class="`status-icon-${getStatusClass(order.status)}`">
                <component :is="getStatusIcon(order.status)" class="w-4 h-4" />
              </span>
              <span class="jb-order-status-text" :class="`status-text-${getStatusClass(order.status)}`">
                {{ getStatusText(order.status) }}
              </span>
            </div>

            <NuxtLink :to="`order/${order.id}`" class="jb-order-view-btn" aria-label="مشاهده جزئیات">
              <ChevronLeft class="w-4 h-4" />
            </NuxtLink>
          </div>

          <!-- ===== اطلاعات سفارش ===== -->
          <div class="jb-order-meta">
            <span class="jb-order-meta-item">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
              </svg>
              {{ order.date }}
            </span>

            <span class="jb-order-meta-dot"></span>

            <span class="jb-order-meta-item">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M7 20l4-16m2 16l4-16M6 9h14M4 15h14" />
              </svg>
              کد سفارش:
              <strong class="jb-order-meta-strong">#{{ order.id }}</strong>
            </span>

            <span class="jb-order-meta-dot"></span>

            <span class="jb-order-meta-item">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
              مبلغ:
              <strong class="jb-order-meta-strong">
                {{ order.total?.toLocaleString() }}
                <span class="jb-order-meta-unit">تومان</span>
              </strong>
            </span>
          </div>

          <!-- ===== محصول ===== -->
          <div v-if="order.items.length" class="jb-order-product">
            <div class="jb-order-product-img-wrap">
              <img :src="$getImageUrl(order.items[0].product.main_image)" :alt="order.items[0].product.title"
                class="jb-order-product-img" />
            </div>
            <div class="jb-order-product-info">
              <span class="jb-order-product-title">
                {{ order.items[0].product.title }}
              </span>
              <span v-if="order.items.length > 1" class="jb-order-product-more">
                و {{ order.items.length - 1 }} محصول دیگر
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
const { $getImageUrl } = useNuxtApp();

definePageMeta({
  layout: "panel",
  middleware: "auth",
});

import {
  CheckCircle,
  Clock,
  AlertCircle,
  ChevronLeft,
} from "lucide-vue-next";

const { getOrders } = useApi();

const { data: ordersData } = await useAsyncData(
  "userOrders",
  async () => {
    try {
      return await getOrders();
    } catch (error) {
      console.error("❌ Failed to fetch orders:", error);
      return { data: [] };
    }
  },
  {
    lazy: true,
    server: false,
  }
);

const orders = computed(() => ordersData.value?.orders || []);

function getStatusIcon(status) {
  switch (status) {
    case "cancelled":
      return AlertCircle;
    case "pending":
      return Clock;
    case "delivered":
      return CheckCircle;
    default:
      return Clock;
  }
}

function getStatusText(status) {
  switch (status) {
    case "card_transfer_pending":
      return "در انتظار آپلود رسید";
    case "card_transfer_review":
      return "در حال پردازش رسید";
    case "card_transfer_send_again":
      return "در انتظار آپلود مجدد رسید";
    case "cancelled":
      return "لغو شده";
    case "pending":
      return "در حال بررسی";
    case "delivered":
      return "تحویل داده شد";
    case "paid":
      return "پرداخت شده";
    case "completed":
      return "تکمیل شده";
    case "reserved":
      return "رزرو شده";
    default:
      return "نامشخص";
  }
}

/* دسته‌بندی وضعیت‌ها برای رنگ‌بندی تم */
function getStatusClass(status) {
  switch (status) {
    case "cancelled":
      return "cancelled";
    case "pending":
    case "card_transfer_pending":
    case "card_transfer_review":
    case "card_transfer_send_again":
    case "reserved":
      return "pending";
    case "delivered":
    case "paid":
    case "completed":
      return "success";
    default:
      return "muted";
  }
}
useSeoMeta({
  title: () => 'سفارشات پنل کاربری | فروشگاه دکتر اسماعیلی',
  description: 'لیست سفارشات کاربر در پنل اختصاصی',
  robots: 'noindex, nofollow',
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/panel/orders' }],
  // 🔹 جلوگیری از ایندکس شدن با متا تگ اضافی برای اطمینان بیشتر
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه لیست سفارشات - تم پودری
   ========================================================= */

.jb-orders-card {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 24px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(214, 51, 132, 0.1);
}

/* ===== هدر ===== */
.jb-orders-header {
  margin-bottom: 4px;
}

.jb-orders-header-title {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  font-size: 1.15rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-orders-header-icon {
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

/* ===== حالت‌ها ===== */
.jb-state-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px 20px;
  text-align: center;
  gap: 12px;
  color: var(--jb-text-muted);
  font-size: 0.88rem;
  font-weight: 600;
}

.jb-state-icon {
  width: 84px;
  height: 84px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border: 3px dashed var(--jb-pink-300);
  color: var(--jb-pink-400);
  margin-bottom: 4px;
  animation: jbEmptyFloat 3s ease-in-out infinite;
}

@keyframes jbEmptyFloat {

  0%,
  100% {
    transform: translateY(0);
  }

  50% {
    transform: translateY(-8px);
  }
}

/* =========================================================
   لیست سفارشات
   ========================================================= */
.jb-orders-list {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

/* ===== کارت سفارش ===== */
.jb-order-card {
  position: relative;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 20px;
  padding: 18px;
  transition: all 0.3s ease;
  overflow: hidden;
}

/* خط رنگی کنار کارت بر اساس وضعیت */
.jb-order-card::before {
  content: "";
  position: absolute;
  top: 12px;
  bottom: 12px;
  right: 0;
  width: 4px;
  border-radius: 999px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  transition: all 0.3s ease;
}

.jb-order-card.status-cancelled::before {
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
}

.jb-order-card.status-pending::before {
  background: linear-gradient(135deg, #ffb84d, #ffa31a);
}

.jb-order-card.status-success::before {
  background: linear-gradient(135deg, #4ade80, #16a34a);
}

.jb-order-card.status-muted::before {
  background: linear-gradient(135deg, #cbd5e1, #94a3b8);
}

.jb-order-card:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 12px 28px rgba(214, 51, 132, 0.16);
  transform: translateY(-3px);
}

.jb-order-card.status-cancelled:hover {
  border-color: rgba(201, 24, 74, 0.5);
  box-shadow: 0 12px 28px rgba(201, 24, 74, 0.16);
}

.jb-order-card.status-pending:hover {
  border-color: rgba(255, 163, 26, 0.5);
  box-shadow: 0 12px 28px rgba(255, 163, 26, 0.16);
}

.jb-order-card.status-success:hover {
  border-color: rgba(34, 197, 94, 0.5);
  box-shadow: 0 12px 28px rgba(34, 197, 94, 0.16);
}

.jb-order-card-inner {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

/* ===== بالای کارت ===== */
.jb-order-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.jb-order-status {
  display: inline-flex;
  align-items: center;
  gap: 10px;
}

.jb-order-status-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 10px;
  border: 2px solid #fff;
  flex-shrink: 0;
}

.status-icon-cancelled {
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  color: #fff;
  box-shadow: 0 4px 12px rgba(201, 24, 74, 0.35);
}

.status-icon-pending {
  background: linear-gradient(135deg, #ffb84d, #ffa31a);
  color: #fff;
  box-shadow: 0 4px 12px rgba(255, 163, 26, 0.35);
}

.status-icon-success {
  background: linear-gradient(135deg, #4ade80, #16a34a);
  color: #fff;
  box-shadow: 0 4px 12px rgba(34, 197, 94, 0.35);
}

.status-icon-muted {
  background: linear-gradient(135deg, #cbd5e1, #94a3b8);
  color: #fff;
}

.jb-order-status-text {
  font-size: 0.85rem;
  font-weight: 900;
}

.status-text-cancelled {
  color: #c9184a;
}

.status-text-pending {
  color: #b45309;
}

.status-text-success {
  color: #16a34a;
}

.status-text-muted {
  color: var(--jb-text-muted);
}

/* دکمه مشاهده */
.jb-order-view-btn {
  flex-shrink: 0;
  width: 34px;
  height: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  background: var(--jb-pink-50);
  border: 1.5px solid var(--jb-pink-200);
  color: var(--jb-fuchsia);
  transition: all 0.25s ease;
  text-decoration: none;
}

.jb-order-view-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  transform: translateX(-3px);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
}

/* ===== اطلاعات سفارش ===== */
.jb-order-meta {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
  padding: 10px 12px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border-radius: 12px;
  border: 1.5px dashed var(--jb-pink-200);
  font-size: 0.78rem;
}

.jb-order-meta-item {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  color: var(--jb-text-soft);
  font-weight: 700;
}

.jb-order-meta-item svg {
  color: var(--jb-fuchsia);
  flex-shrink: 0;
}

.jb-order-meta-strong {
  color: var(--jb-text);
  font-weight: 900;
}

.jb-order-meta-unit {
  font-size: 0.68rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  margin-right: 2px;
}

.jb-order-meta-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--jb-pink-300);
  box-shadow: 0 0 0 2px rgba(214, 51, 132, 0.12);
  flex-shrink: 0;
}

/* ===== محصول ===== */
.jb-order-product {
  display: flex;
  align-items: center;
  gap: 12px;
  padding-top: 12px;
  border-top: 1.5px dashed var(--jb-pink-200);
}

.jb-order-product-img-wrap {
  flex-shrink: 0;
  width: 64px;
  height: 64px;
  border-radius: 14px;
  background: linear-gradient(135deg, #fff5f8, #ffe0eb);
  border: 2px solid var(--jb-pink-200);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px;
  overflow: hidden;
}

.jb-order-product-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 9px;
}

.jb-order-product-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 0;
}

.jb-order-product-title {
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
  line-height: 1.6;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.jb-order-product-more {
  font-size: 0.72rem;
  color: var(--jb-fuchsia);
  font-weight: 800;
  padding: 3px 10px;
  background: var(--jb-pink-50);
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 999px;
  width: fit-content;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-orders-card {
    padding: 16px;
    border-radius: 20px;
  }

  .jb-orders-header-title {
    font-size: 1rem;
  }

  .jb-orders-header-icon {
    width: 32px;
    height: 32px;
  }

  .jb-order-card {
    padding: 14px;
    border-radius: 16px;
  }

  .jb-order-meta {
    font-size: 0.72rem;
    gap: 6px;
  }

  .jb-order-meta-dot {
    width: 5px;
    height: 5px;
  }

  .jb-order-product-img-wrap {
    width: 56px;
    height: 56px;
  }

  .jb-order-product-title {
    font-size: 0.8rem;
  }
}

@media (max-width: 400px) {
  .jb-order-meta {
    flex-direction: column;
    align-items: flex-start;
    gap: 6px;
  }

  .jb-order-meta-dot {
    display: none;
  }
}
</style>