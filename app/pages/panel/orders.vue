<template>
  <div class="jb-orders-card">
    <!-- هدر -->
    <div class="jb-orders-header">
      <h3 class="jb-orders-header-title">
        <span class="jb-orders-header-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
          </svg>
        </span>
        سفارش‌ها
      </h3>
    </div>

    <!-- خط جداکننده -->
    <div class="jb-divider"></div>

    <!-- لودینگ -->
    <div v-if="!ordersData" class="jb-state-block">
      <SharedLoadingSpinner size="lg" />
    </div>

    <!-- خالی -->
    <div v-else-if="orders.length === 0" class="jb-state-block jb-state-empty">
      <div class="jb-state-icon">
        <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
        </svg>
      </div>
      <span>هیچ سفارشی یافت نشد</span>
    </div>

    <!-- لیست سفارشات -->
    <div v-else class="jb-orders-list">
      <div v-for="order in orders" :key="order.id" class="jb-order-card"
        :class="`status-${getStatusClass(order.status)}`">
        <div class="jb-order-card-inner">
          <!-- بالای کارت: وضعیت + مشاهده -->
          <div class="jb-order-top">
            <div class="jb-order-status">
              <span class="jb-order-status-icon" :class="`status-icon-${getStatusClass(order.status)}`">
                <component :is="getStatusIcon(order.status)" class="w-3.5 h-3.5" />
              </span>
              <span class="jb-order-status-text" :class="`status-text-${getStatusClass(order.status)}`">
                {{ getStatusText(order.status) }}
              </span>
            </div>

            <NuxtLink :to="`order/${order.id}`" class="jb-order-view-btn" aria-label="مشاهده جزئیات">
              <ChevronLeft class="w-4 h-4" />
            </NuxtLink>
          </div>

          <!-- اطلاعات سفارش -->
          <div class="jb-order-meta">
            <span class="jb-order-meta-item">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
              </svg>
              {{ order.date }}
            </span>

            <span class="jb-order-meta-item">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M7 20l4-16m2 16l4-16M6 9h14M4 15h14" />
              </svg>
              کد سفارش:
              <strong class="jb-order-meta-strong">#{{ order.id }}</strong>
            </span>

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

          <!-- محصول -->
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
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه لیست سفارشات - مینیمال سبز
   ========================================================= */

.jb-orders-card {
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

/* ===== هدر ===== */
.jb-orders-header {
  margin-bottom: 4px;
}

.jb-orders-header-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-orders-header-icon {
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

/* ===== حالت‌ها ===== */
.jb-state-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 32px 20px;
  text-align: center;
  gap: 10px;
  color: #718096;
  font-size: 0.85rem;
  font-weight: 500;
}

.jb-state-icon {
  width: 64px;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  color: #a0aec0;
  margin-bottom: 4px;
}

/* =========================================================
   لیست سفارشات
   ========================================================= */
.jb-orders-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

/* ===== کارت سفارش ===== */
.jb-order-card {
  position: relative;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 8px;
  padding: 14px;
  transition: all 0.2s ease;
  overflow: hidden;
}

/* خط رنگی کنار کارت بر اساس وضعیت */
.jb-order-card::before {
  content: "";
  position: absolute;
  top: 10px;
  bottom: 10px;
  right: 0;
  width: 3px;
  border-radius: 2px;
  background: #0e5a3a;
}

.jb-order-card.status-cancelled::before {
  background: #e53e3e;
}

.jb-order-card.status-pending::before {
  background: #d97706;
}

.jb-order-card.status-success::before {
  background: #16a34a;
}

.jb-order-card.status-muted::before {
  background: #a0aec0;
}

.jb-order-card:hover {
  border-color: rgba(14, 90, 58, 0.2);
}

.jb-order-card-inner {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

/* ===== بالای کارت ===== */
.jb-order-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
}

.jb-order-status {
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.jb-order-status-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 6px;
  flex-shrink: 0;
}

.status-icon-cancelled {
  background: #fff5f5;
  color: #e53e3e;
}

.status-icon-pending {
  background: #fffbea;
  color: #d97706;
}

.status-icon-success {
  background: #f0fff4;
  color: #16a34a;
}

.status-icon-muted {
  background: #f7fafc;
  color: #718096;
}

.jb-order-status-text {
  font-size: 0.8rem;
  font-weight: 700;
}

.status-text-cancelled {
  color: #e53e3e;
}

.status-text-pending {
  color: #d97706;
}

.status-text-success {
  color: #16a34a;
}

.status-text-muted {
  color: #718096;
}

/* دکمه مشاهده */
.jb-order-view-btn {
  flex-shrink: 0;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  color: #0e5a3a;
  transition: all 0.2s ease;
  text-decoration: none;
}

.jb-order-view-btn:hover {
  background: #0e5a3a;
  color: #fff;
  border-color: transparent;
}

/* ===== اطلاعات سفارش ===== */
.jb-order-meta {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
  padding: 8px 10px;
  background: #f7fafc;
  border-radius: 6px;
  font-size: 0.75rem;
}

.jb-order-meta-item {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  color: #4a5568;
  font-weight: 500;
}

.jb-order-meta-item svg {
  color: #0e5a3a;
  flex-shrink: 0;
}

.jb-order-meta-strong {
  color: #1a202c;
  font-weight: 700;
}

.jb-order-meta-unit {
  font-size: 0.65rem;
  color: #a0aec0;
  font-weight: 500;
  margin-right: 2px;
}

/* ===== محصول ===== */
.jb-order-product {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-top: 10px;
  border-top: 1px dashed rgba(14, 90, 58, 0.1);
}

.jb-order-product-img-wrap {
  flex-shrink: 0;
  width: 56px;
  height: 56px;
  border-radius: 6px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.06);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 4px;
  overflow: hidden;
}

.jb-order-product-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 4px;
}

.jb-order-product-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 3px;
  min-width: 0;
}

.jb-order-product-title {
  font-size: 0.82rem;
  font-weight: 700;
  color: #1a202c;
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.jb-order-product-more {
  font-size: 0.68rem;
  color: #0e5a3a;
  font-weight: 600;
  padding: 2px 8px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 4px;
  width: fit-content;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-orders-card {
    padding: 14px;
    border-radius: 8px;
  }

  .jb-orders-header-title {
    font-size: 0.9rem;
  }

  .jb-orders-header-icon {
    width: 28px;
    height: 28px;
  }

  .jb-order-card {
    padding: 12px;
  }

  .jb-order-meta {
    font-size: 0.7rem;
    gap: 8px;
  }

  .jb-order-product-img-wrap {
    width: 48px;
    height: 48px;
  }

  .jb-order-product-title {
    font-size: 0.78rem;
  }
}

@media (max-width: 400px) {
  .jb-order-meta {
    flex-direction: column;
    align-items: flex-start;
    gap: 5px;
  }
}
</style>