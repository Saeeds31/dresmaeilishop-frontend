<template>
  <!-- لودینگ -->
  <div v-if="loading" class="jb-order-detail-loading">
    <SharedLoadingSpinner size="lg" />
    <span>در حال بارگذاری...</span>
  </div>

  <!-- خطا -->
  <div v-else-if="error" class="jb-order-detail-error">
    <div class="jb-error-icon">
      <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8"
          d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
      </svg>
    </div>
    <h3 class="jb-error-title">خطا در بارگذاری اطلاعات</h3>
    <p class="jb-error-desc">{{ error }}</p>
    <button @click="fetchOrder" class="jb-btn-retry">تلاش مجدد</button>
  </div>

  <div v-else class="jb-order-detail-card">
    <!-- هدر -->
    <div class="jb-detail-header">
      <div class="jb-detail-header-title">
        <span class="jb-detail-header-icon">
          <MoveRight class="w-4 h-4" />
        </span>
        <h2>جزئیات سفارش</h2>
        <span class="jb-detail-header-code">#{{ orderData?.id || "---" }}</span>
      </div>

      <span v-if="orderData?.parent_order_id" class="jb-child-badge">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2" d="M13 5l7 7-7 7M5 5l7 7-7 7" />
        </svg>
        سفارش فرزند
      </span>
    </div>

    <!-- خط جداکننده -->
    <div class="jb-divider"></div>

    <!-- وضعیت سفارش و پرداخت -->
    <div class="jb-status-bar">
      <div class="jb-status-item">
        <span class="jb-status-label">وضعیت سفارش</span>
        <span class="jb-status-badge" :class="getStatusClass(orderData?.status)">
          {{ getStatusText(orderData?.status) }}
        </span>
      </div>

      <div class="jb-status-item">
        <span class="jb-status-label">وضعیت پرداخت</span>
        <span class="jb-status-badge" :class="getPaymentStatusClass(orderData?.payment_status)">
          {{ getPaymentStatusText(orderData?.payment_status) }}
        </span>
      </div>

      <div v-if="orderData?.reservation_type !== 'none'" class="jb-status-item">
        <span class="jb-status-label">نوع رزرو</span>
        <span class="jb-status-badge jb-badge-reserved">
          {{ getReservationTypeText(orderData?.reservation_type) }}
        </span>
      </div>
    </div>

    <!-- بخش کارت به کارت -->
    <div v-if="orderData?.payment_method === 'card_transfer'" class="jb-card-transfer-box"
      :class="getCardTransferClass()">
      <!-- در انتظار آپلود رسید -->
      <div v-if="orderData.status === 'card_transfer_pending' || orderData.status === 'card_transfer_send_again'">
        <div class="jb-card-transfer-alert">
          <span class="jb-alert-icon jb-alert-yellow">
            <IconClock class="w-4 h-4" />
          </span>
          <span class="jb-alert-text">
            لطفاً رسید پرداخت کارت به کارت را آپلود کنید
          </span>
        </div>

        <!-- راهنما -->
        <div class="jb-card-transfer-guide">
          <p class="jb-guide-title">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
            راهنمای پرداخت کارت به کارت
          </p>

          <ul v-if="remainingSeconds > 0" class="jb-guide-list">
            <li class="jb-guide-info">
              <div class="jb-guide-info-content">
                <span class="jb-guide-info-label">شماره کارت</span>
                <span class="jb-guide-info-value" dir="ltr">
                  {{ bankCardNumber }}
                </span>
              </div>
              <button @click="copyText(bankCardNumber)" class="jb-copy-btn">
                <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
                </svg>
                کپی
              </button>
            </li>

            <li class="jb-guide-info">
              <div class="jb-guide-info-content">
                <span class="jb-guide-info-label">شماره شبا</span>
                <span class="jb-guide-info-value" dir="ltr">
                  {{ bankShebaNumber }}
                </span>
              </div>
              <button @click="copyText(bankShebaNumber)" class="jb-copy-btn">
                <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
                </svg>
                کپی
              </button>
            </li>

            <li class="jb-guide-owner">
              <span class="jb-guide-owner-label">صاحب حساب:</span>
              <span class="jb-guide-owner-name">{{ bankAccountOwner }}</span>
            </li>
          </ul>
        </div>

        <!-- تایمر -->
        <div class="jb-timer-box">
          <span class="jb-timer-label">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
            زمان باقی‌مانده برای آپلود:
          </span>

          <span v-if="remainingSeconds > 0" class="jb-timer-value">
            {{ formatTimer(remainingSeconds) }}
          </span>

          <span v-else-if="orderData.status === 'card_transfer_send_again'" class="jb-timer-error">
            در اسرع وقت رسید خود را بارگذاری کنید - دلیل رد رسید قبلی:
            {{ orderData.card_transfer_receipt?.description }}
          </span>

          <span v-else class="jb-timer-error"> زمان تمام شد! </span>
        </div>

        <!-- دکمه آپلود -->
        <button v-if="remainingSeconds > 0 || orderData.status === 'card_transfer_send_again'"
          @click="openUploadReceiptModal" class="jb-upload-btn">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
              d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12" />
          </svg>
          آپلود رسید پرداخت
        </button>
      </div>

      <!-- در انتظار بررسی ادمین -->
      <div v-else-if="orderData.status === 'card_transfer_review'">
        <div class="jb-card-transfer-alert">
          <span class="jb-alert-icon jb-alert-blue">
            <IconClock class="w-4 h-4" />
          </span>
          <span class="jb-alert-text jb-alert-text-blue">
            رسید شما در انتظار بررسی ادمین است
          </span>
        </div>
        <p class="jb-alert-desc">
          پس از تأیید رسید، وضعیت سفارش به‌روزرسانی می‌شود
        </p>
      </div>

      <!-- رسید رد شده -->
      <div v-else-if="orderData.status === 'cancelled' && orderData.cardTransferReceipt?.status === 'rejected'">
        <div class="jb-card-transfer-alert">
          <span class="jb-alert-icon jb-alert-red">
            <IconX class="w-4 h-4" />
          </span>
          <span class="jb-alert-text jb-alert-text-red">
            رسید شما رد شده است
          </span>
        </div>
        <p v-if="orderData.cardTransferReceipt?.description" class="jb-alert-desc">
          دلیل: {{ orderData.cardTransferReceipt.description }}
        </p>
      </div>

      <!-- رسید تأیید شده -->
      <div v-else-if="orderData.cardTransferReceipt?.status === 'approved'">
        <div class="jb-card-transfer-alert">
          <span class="jb-alert-icon jb-alert-green">
            <IconCheck class="w-4 h-4" />
          </span>
          <span class="jb-alert-text jb-alert-text-green">
            رسید شما تأیید شده است ✓
          </span>
        </div>
      </div>
    </div>

    <!-- تصویر رسید -->
    <div v-if="orderData?.cardTransferReceipt?.image_url" class="jb-receipt-box">
      <h4 class="jb-receipt-title">رسید پرداخت</h4>
      <img :src="$getImageUrl(orderData.cardTransferReceipt.image_url)" alt="رسید پرداخت" class="jb-receipt-img"
        @click="openImagePreview($getImageUrl(orderData.cardTransferReceipt.image_url))" />
      <div v-if="orderData.cardTransferReceipt.tracking_code" class="jb-receipt-tracking">
        <span class="jb-receipt-tracking-label">کد پیگیری:</span>
        <span class="jb-receipt-tracking-value" dir="ltr">
          {{ orderData.cardTransferReceipt.tracking_code }}
        </span>
      </div>
    </div>

    <!-- اطلاعات سفارش -->
    <div class="jb-info-section">
      <h3 class="jb-section-title">اطلاعات سفارش</h3>

      <div class="jb-info-grid">
        <div class="jb-info-item">
          <span class="jb-info-label">کد سفارش</span>
          <span class="jb-info-value jb-info-value-mono" dir="ltr">
            #{{ orderData?.id || "---" }}
          </span>
        </div>

        <div class="jb-info-item">
          <span class="jb-info-label">تحویل گیرنده</span>
          <span class="jb-info-value">
            {{ orderData?.address?.receiver_name || "---" }}
          </span>
        </div>

        <div class="jb-info-item">
          <span class="jb-info-label">شماره موبایل</span>
          <span class="jb-info-value jb-info-value-mono" dir="ltr">
            {{ orderData?.user?.mobile || "---" }}
          </span>
        </div>

        <div class="jb-info-item jb-info-item-full">
          <span class="jb-info-label">آدرس</span>
          <span class="jb-info-value">
            {{ orderData?.address?.address_line || "---" }}
          </span>
        </div>
      </div>
    </div>

    <!-- اطلاعات حمل و نقل -->
    <div class="jb-info-section">
      <h3 class="jb-section-title">
        <span class="jb-ship-title">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M9 17a2 2 0 11-4 0 2 2 0 014 0zM19 17a2 2 0 11-4 0 2 2 0 014 0z" />
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M13 16V6a1 1 0 00-1-1H4a1 1 0 00-1 1v10a1 1 0 001 1h1m8-1a1 1 0 01-1 1H9m4-1V8a1 1 0 011-1h2.586a1 1 0 01.707.293l3.414 3.414a1 1 0 01.293.707V16a1 1 0 01-1 1h-1m-6-1a1 1 0 001 1h1M5 17a2 2 0 104 0m-4 0a2 2 0 114 0m6 0a2 2 0 104 0m-4 0a2 2 0 114 0" />
          </svg>
          {{ orderData?.shipping?.title || "ارسال عادی" }}
        </span>
      </h3>

      <div class="jb-info-grid">
        <div class="jb-info-item">
          <span class="jb-info-label">زمان تحویل</span>
          <span class="jb-info-value">
            {{ formatDate(orderData?.delivery_date) || "---" }}
          </span>
        </div>

        <div class="jb-info-item">
          <span class="jb-info-label">هزینه ارسال</span>
          <span class="jb-info-value">
            {{ formatPrice(orderData?.shipping?.cost) }}
            <span class="jb-info-unit">تومان</span>
          </span>
        </div>

        <div class="jb-info-item">
          <span class="jb-info-label">روش پرداخت</span>
          <span class="jb-info-value">
            {{ getPaymentMethodText(orderData?.payment_method) }}
          </span>
        </div>

        <div v-if="orderData.card_transfer_receipt?.status == 'rejected' && orderData.payment_method == 'card_transfer'"
          class="jb-info-item">
          <span class="jb-info-label">دلیل رد رسید</span>
          <span class="jb-info-value jb-info-value-danger">
            {{ orderData.card_transfer_receipt?.description }}
          </span>
        </div>

        <div v-if="orderData?.reserved_until" class="jb-info-item">
          <span class="jb-info-label">زمان انقضای رزرو</span>
          <span class="jb-info-value">
            {{ formatDate(orderData?.reserved_until) }}
          </span>
        </div>
      </div>
    </div>

    <!-- محصولات سفارش اصلی -->
    <div class="jb-info-section" v-if="orderData?.items?.length">
      <h3 class="jb-section-title">محصولات سفارش</h3>

      <div class="jb-items-list">
        <div v-for="item in orderData.items" :key="item.id" class="jb-item-card">
          <div class="jb-item-main">
            <div v-if="item.product?.main_image" class="jb-item-img-wrap">
              <img :src="$getImageUrl(item.product.main_image)" :alt="item.product.title" class="jb-item-img" />
            </div>

            <div class="jb-item-info">
              <h4 class="jb-item-title">
                {{ item.product?.title || "نامشخص" }}
              </h4>

              <div class="jb-item-attrs">
                <span v-for="(value, index) in item.variant?.values || []" :key="index" class="jb-item-attr-chip">
                  <span class="jb-item-attr-name">
                    {{ getAttributeName(value.attribute_id) }}:
                  </span>
                  <span class="jb-item-attr-value">
                    {{ decodeURIComponent(value.value) }}
                  </span>
                </span>
              </div>

              <div class="jb-item-sku">
                <span class="jb-item-sku-label">کد:</span>
                <span class="jb-item-sku-value" dir="ltr">
                  {{ item.product?.sku || "---" }}
                </span>
              </div>
            </div>
          </div>

          <div class="jb-item-stats">
            <div class="jb-item-stat">
              <span class="jb-item-stat-label">تعداد</span>
              <span class="jb-item-stat-value">{{ item.quantity }}</span>
            </div>
            <div class="jb-item-stat">
              <span class="jb-item-stat-label">قیمت واحد</span>
              <span class="jb-item-stat-value">
                {{ formatPrice(item.price) }}
                <span class="jb-item-stat-unit">تومان</span>
              </span>
            </div>
            <div class="jb-item-stat">
              <span class="jb-item-stat-label">جمع</span>
              <span class="jb-item-stat-value jb-item-stat-total">
                {{ formatPrice(item.price * item.quantity) }}
                <span class="jb-item-stat-unit">تومان</span>
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- سفارش‌های فرزند -->
    <div v-if="orderData?.child_orders?.length" class="jb-info-section">
      <h3 class="jb-section-title">
        <span class="jb-ship-title">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4" />
          </svg>
          سفارش‌های مرتبط
        </span>
        <span class="jb-ship-count">
          ({{ orderData.child_orders.length }} سفارش)
        </span>
      </h3>

      <div class="jb-child-orders">
        <div v-for="child in orderData.child_orders" :key="child.id" class="jb-child-order-card">
          <div class="jb-child-header">
            <div class="jb-child-header-info">
              <span class="jb-child-id">سفارش #{{ child.id }}</span>

              <span class="jb-status-badge jb-status-sm" :class="getStatusClass(child.status)">
                {{ getStatusText(child.status) }}
              </span>

              <span class="jb-status-badge jb-status-sm" :class="getPaymentStatusClass(child.payment_status)">
                {{ getPaymentStatusText(child.payment_status) }}
              </span>
            </div>

            <span class="jb-child-date">{{ formatDate(child.created_at) }}</span>
          </div>

          <div class="jb-child-items">
            <div v-for="item in child.items" :key="item.id" class="jb-child-item">
              <div class="jb-child-item-main">
                <div v-if="item.product?.main_image" class="jb-child-item-img-wrap">
                  <img :src="$getImageUrl(item.product.main_image)" :alt="item.product.title"
                    class="jb-child-item-img" />
                </div>

                <div class="jb-child-item-info">
                  <h4 class="jb-child-item-title">
                    {{ item.product?.title || "نامشخص" }}
                  </h4>

                  <div class="jb-item-attrs">
                    <span v-for="(value, index) in item.variant?.values || []" :key="index" class="jb-item-attr-chip">
                      <span class="jb-item-attr-name">
                        {{ getAttributeName(value.attribute_id) }}:
                      </span>
                      <span class="jb-item-attr-value">
                        {{ decodeURIComponent(value.value) }}
                      </span>
                    </span>
                  </div>

                  <div class="jb-item-sku">
                    <span class="jb-item-sku-label">کد:</span>
                    <span class="jb-item-sku-value" dir="ltr">
                      {{ item.product?.sku || "---" }}
                    </span>
                  </div>
                </div>
              </div>

              <div class="jb-item-stats jb-child-stats">
                <div class="jb-item-stat">
                  <span class="jb-item-stat-label">تعداد</span>
                  <span class="jb-item-stat-value">{{ item.quantity }}</span>
                </div>
                <div class="jb-item-stat">
                  <span class="jb-item-stat-label">واحد</span>
                  <span class="jb-item-stat-value">
                    {{ formatPrice(item.price) }}
                  </span>
                </div>
                <div class="jb-item-stat">
                  <span class="jb-item-stat-label">جمع</span>
                  <span class="jb-item-stat-value jb-item-stat-total">
                    {{ formatPrice(item.price * item.quantity) }}
                  </span>
                </div>
              </div>
            </div>
          </div>

          <div class="jb-child-summary">
            <div class="jb-child-summary-info">
              <span>
                تعداد اقلام:
                <strong>{{ child.items?.length || 0 }}</strong>
              </span>
              <span>
                هزینه ارسال:
                <strong>{{ formatPrice(child.shipping?.cost) }} تومان</strong>
              </span>
            </div>
            <div class="jb-child-summary-total">
              جمع: {{ formatPrice(child.total) }} تومان
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- جمع کل -->
    <div class="jb-totals-section">
      <h3 class="jb-section-title">خلاصه پرداخت</h3>

      <div class="jb-totals-box">
        <div class="jb-total-row">
          <span class="jb-total-label">جمع سبد خرید</span>
          <span class="jb-total-value">
            {{ formatPrice(orderData?.subtotal) }}
            <span class="jb-total-unit">تومان</span>
          </span>
        </div>

        <div v-if="orderData?.child_orders?.length" class="jb-total-row jb-total-row-dashed">
          <span class="jb-total-label">جمع سفارش‌های مرتبط</span>
          <span class="jb-total-value">
            {{ formatPrice(childOrdersSubtotal) }}
            <span class="jb-total-unit">تومان</span>
          </span>
        </div>

        <div v-if="orderData?.discount_amount > 0" class="jb-total-row jb-total-row-discount">
          <span class="jb-total-label">تخفیف</span>
          <span class="jb-total-value">
            -{{ formatPrice(orderData.discount_amount) }}
            <span class="jb-total-unit">تومان</span>
          </span>
        </div>

        <div class="jb-total-row">
          <span class="jb-total-label">هزینه ارسال</span>
          <span class="jb-total-value">
            {{ formatPrice(orderData?.shipping?.cost) }}
            <span class="jb-total-unit">تومان</span>
          </span>
        </div>

        <div class="jb-total-divider"></div>

        <div class="jb-total-row jb-total-row-final">
          <span class="jb-total-label-final">جمع کل</span>
          <span class="jb-total-value-final">
            {{ formatPrice(totalWithChildren) }}
            <span class="jb-total-unit">تومان</span>
          </span>
        </div>

        <div v-if="orderData?.child_orders?.length" class="jb-total-note">
          * شامل {{ orderData.child_orders.length }} سفارش مرتبط
        </div>
      </div>
    </div>

    <!-- دکمه‌های اقدام -->
    <div class="jb-actions">
      <button @click="goBack" class="jb-action-btn jb-action-back">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
        </svg>
        بازگشت
      </button>
    </div>
  </div>

  <UploadReceiptModal :show="showUploadModal" :order-id="orderData?.id" @close="showUploadModal = false"
    @uploaded="onReceiptUploaded" />
</template>

<script setup>
// ... (بخش اسکریپت دست‌نخورده)
</script>

<style scoped>
/* =========================================================
   صفحه جزئیات سفارش - مینیمال سبز
   ========================================================= */

/* ===== لودینگ ===== */
.jb-order-detail-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 50px 20px;
  color: #718096;
  font-weight: 500;
  font-size: 0.85rem;
}

/* ===== خطا ===== */
.jb-order-detail-error {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 40px 24px;
  background: #fff;
  border: 1px solid rgba(229, 62, 62, 0.2);
  border-radius: 10px;
  text-align: center;
}

.jb-error-icon {
  width: 64px;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: #fff5f5;
  color: #e53e3e;
  margin-bottom: 4px;
}

.jb-error-title {
  font-size: 1rem;
  font-weight: 700;
  color: #e53e3e;
}

.jb-error-desc {
  font-size: 0.82rem;
  color: #718096;
  font-weight: 500;
  line-height: 1.8;
}

.jb-btn-retry {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 9px 20px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.82rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
  margin-top: 6px;
}

.jb-btn-retry:hover {
  background: #0b472e;
}

/* ===== کارت اصلی ===== */
.jb-order-detail-card {
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

/* =========================================================
   هدر
   ========================================================= */
.jb-detail-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
}

.jb-detail-header-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.jb-detail-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 6px;
  background: #0e5a3a;
  color: #fff;
}

.jb-detail-header-title h2 {
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-detail-header-code {
  font-size: 0.75rem;
  font-weight: 600;
  color: #0e5a3a;
  background: #f7fafc;
  padding: 3px 10px;
  border-radius: 4px;
  border: 1px solid rgba(14, 90, 58, 0.08);
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
}

.jb-child-badge {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 4px 10px;
  background: #0e5a3a;
  color: #fff;
  font-size: 0.68rem;
  font-weight: 600;
  border-radius: 4px;
}

/* خط جداکننده */
.jb-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 14px 0 20px;
}

/* =========================================================
   نوار وضعیت
   ========================================================= */
.jb-status-bar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 16px;
  padding: 14px 16px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 8px;
  margin-bottom: 16px;
}

.jb-status-item {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.jb-status-label {
  font-size: 0.75rem;
  color: #718096;
  font-weight: 500;
}

.jb-status-badge {
  display: inline-flex;
  align-items: center;
  padding: 4px 10px;
  border-radius: 4px;
  font-size: 0.72rem;
  font-weight: 600;
}

.jb-status-sm {
  padding: 2px 8px;
  font-size: 0.65rem;
}

.jb-badge-reserved {
  background: #e0e7ff;
  color: #4338ca;
}

/* رنگ‌های وضعیت */
.jb-status-badge.bg-yellow-100 {
  background: #fef3c7 !important;
  color: #b45309 !important;
}

.jb-status-badge.bg-green-100 {
  background: #dcfce7 !important;
  color: #16a34a !important;
}

.jb-status-badge.bg-blue-100 {
  background: #dbeafe !important;
  color: #1d4ed8 !important;
}

.jb-status-badge.bg-indigo-100 {
  background: #e0e7ff !important;
  color: #4338ca !important;
}

.jb-status-badge.bg-purple-100 {
  background: #f3e8ff !important;
  color: #7e22ce !important;
}

.jb-status-badge.bg-emerald-100 {
  background: #d1fae5 !important;
  color: #047857 !important;
}

.jb-status-badge.bg-red-100,
.jb-status-badge.bg-rose-100 {
  background: #fff5f5 !important;
  color: #e53e3e !important;
}

.jb-status-badge.bg-gray-100 {
  background: #f7fafc !important;
  color: #718096 !important;
}

/* =========================================================
   کارت به کارت
   ========================================================= */
.jb-card-transfer-box {
  margin-bottom: 16px;
  padding: 16px;
  border-radius: 8px;
  border: 1px dashed;
}

.jb-card-transfer-box.border-yellow-300 {
  background: #fffbea !important;
  border-color: #ffd88a !important;
}

.jb-card-transfer-box.border-blue-300 {
  background: #eff6ff !important;
  border-color: #93c5fd !important;
}

.jb-card-transfer-box.border-red-300 {
  background: #fff5f5 !important;
  border-color: #fed7d7 !important;
}

.jb-card-transfer-box.border-green-300 {
  background: #f0fff4 !important;
  border-color: #86efac !important;
}

/* هشدار */
.jb-card-transfer-alert {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 12px;
}

.jb-alert-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 6px;
  flex-shrink: 0;
}

.jb-alert-yellow {
  background: #d97706;
  color: #fff;
}

.jb-alert-blue {
  background: #2563eb;
  color: #fff;
}

.jb-alert-red {
  background: #e53e3e;
  color: #fff;
}

.jb-alert-green {
  background: #16a34a;
  color: #fff;
}

.jb-alert-text {
  font-size: 0.82rem;
  font-weight: 700;
  color: #7a4a00;
}

.jb-alert-text-blue {
  color: #1d4ed8;
}

.jb-alert-text-red {
  color: #e53e3e;
}

.jb-alert-text-green {
  color: #16a34a;
}

.jb-alert-desc {
  font-size: 0.78rem;
  color: #4a5568;
  font-weight: 500;
  line-height: 1.8;
  margin-top: 6px;
  padding-right: 38px;
}

/* راهنما */
.jb-card-transfer-guide {
  padding: 12px;
  background: #fff;
  border-radius: 6px;
  border: 1px solid rgba(255, 216, 138, 0.5);
  margin-bottom: 12px;
}

.jb-guide-title {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 0.82rem;
  font-weight: 700;
  color: #7a4a00;
  margin-bottom: 10px;
}

.jb-guide-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  list-style: none;
  padding: 0;
  margin: 0;
}

.jb-guide-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding: 8px 12px;
  background: #f7fafc;
  border-radius: 6px;
  border: 1px solid rgba(14, 90, 58, 0.06);
}

.jb-guide-info-content {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
}

.jb-guide-info-label {
  font-size: 0.68rem;
  color: #a0aec0;
  font-weight: 500;
}

.jb-guide-info-value {
  font-family: 'Courier New', monospace;
  font-size: 0.82rem;
  font-weight: 700;
  color: #1a202c;
  letter-spacing: 0.5px;
  word-break: break-all;
}

.jb-copy-btn {
  flex-shrink: 0;
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 5px 10px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 4px;
  font-size: 0.68rem;
  font-weight: 600;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-copy-btn:hover {
  background: #0b472e;
}

.jb-guide-owner {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  background: #f7fafc;
  border-radius: 6px;
  font-size: 0.78rem;
}

.jb-guide-owner-label {
  color: #a0aec0;
  font-weight: 500;
}

.jb-guide-owner-name {
  font-weight: 700;
  color: #1a202c;
}

/* تایمر */
.jb-timer-box {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 14px;
  background: #fff;
  border-radius: 6px;
  border: 1px solid rgba(255, 216, 138, 0.5);
  margin-bottom: 12px;
  flex-wrap: wrap;
}

.jb-timer-label {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 0.78rem;
  color: #4a5568;
  font-weight: 500;
}

.jb-timer-label svg {
  color: #d97706;
}

.jb-timer-value {
  font-family: 'Courier New', monospace;
  font-size: 1rem;
  font-weight: 800;
  color: #d97706;
  letter-spacing: 1px;
}

.jb-timer-error {
  font-size: 0.78rem;
  color: #e53e3e;
  font-weight: 600;
  line-height: 1.7;
}

/* دکمه آپلود */
.jb-upload-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  width: 100%;
  padding: 10px 20px;
  background: #d97706;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-upload-btn:hover {
  background: #b45309;
}

/* =========================================================
   تصویر رسید
   ========================================================= */
.jb-receipt-box {
  margin-bottom: 16px;
  padding: 14px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 8px;
}

.jb-receipt-title {
  font-size: 0.85rem;
  font-weight: 700;
  color: #1a202c;
  margin-bottom: 10px;
}

.jb-receipt-img {
  max-width: 240px;
  max-height: 200px;
  object-fit: contain;
  border-radius: 6px;
  border: 1px solid rgba(14, 90, 58, 0.1);
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-receipt-img:hover {
  border-color: rgba(14, 90, 58, 0.25);
}

.jb-receipt-tracking {
  margin-top: 10px;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 4px;
  width: fit-content;
}

.jb-receipt-tracking-label {
  font-size: 0.72rem;
  color: #a0aec0;
  font-weight: 500;
}

.jb-receipt-tracking-value {
  font-family: 'Courier New', monospace;
  font-size: 0.8rem;
  color: #0e5a3a;
  font-weight: 700;
  letter-spacing: 0.5px;
}

/* =========================================================
   بخش اطلاعات
   ========================================================= */
.jb-info-section {
  padding: 18px 0;
  border-bottom: 1px dashed rgba(14, 90, 58, 0.1);
}

.jb-info-section:last-of-type {
  border-bottom: none;
}

.jb-section-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.88rem;
  font-weight: 700;
  color: #1a202c;
  margin-bottom: 14px;
  flex-wrap: wrap;
}

.jb-ship-title {
  display: inline-flex;
  align-items: center;
  gap: 6px;
}

.jb-ship-title svg {
  color: #0e5a3a;
}

.jb-ship-count {
  font-size: 0.7rem;
  color: #a0aec0;
  font-weight: 500;
}

/* گرید اطلاعات */
.jb-info-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 10px;
}

@media (min-width: 640px) {
  .jb-info-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .jb-info-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.jb-info-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding: 10px 12px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.06);
  border-radius: 6px;
}

.jb-info-item-full {
  grid-column: 1 / -1;
}

.jb-info-label {
  font-size: 0.7rem;
  color: #a0aec0;
  font-weight: 500;
}

.jb-info-value {
  font-size: 0.82rem;
  color: #1a202c;
  font-weight: 600;
  word-break: break-word;
}

.jb-info-value-mono {
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
}

.jb-info-value-danger {
  color: #e53e3e;
}

.jb-info-unit {
  font-size: 0.62rem;
  color: #a0aec0;
  font-weight: 500;
  margin-right: 2px;
}

/* =========================================================
   آیتم‌های سفارش
   ========================================================= */
.jb-items-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.jb-item-card {
  padding: 12px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.06);
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.jb-item-main {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.jb-item-img-wrap {
  flex-shrink: 0;
  width: 64px;
  height: 64px;
  border-radius: 6px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.08);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 4px;
  overflow: hidden;
}

.jb-item-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 4px;
}

.jb-item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 6px;
  min-width: 0;
}

.jb-item-title {
  font-size: 0.85rem;
  font-weight: 700;
  color: #1a202c;
  line-height: 1.5;
}

.jb-item-attrs {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.jb-item-attr-chip {
  display: inline-flex;
  align-items: center;
  gap: 3px;
  padding: 2px 8px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 4px;
  font-size: 0.65rem;
  font-weight: 500;
}

.jb-item-attr-name {
  color: #a0aec0;
}

.jb-item-attr-value {
  color: #0e5a3a;
  font-weight: 600;
}

.jb-item-sku {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 0.68rem;
}

.jb-item-sku-label {
  color: #a0aec0;
  font-weight: 500;
}

.jb-item-sku-value {
  font-family: 'Courier New', monospace;
  color: #718096;
  font-weight: 600;
  letter-spacing: 0.5px;
}

/* آمار آیتم */
.jb-item-stats {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
  padding-top: 10px;
  border-top: 1px dashed rgba(14, 90, 58, 0.1);
}

.jb-item-stat {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 64px;
}

.jb-item-stat-label {
  font-size: 0.65rem;
  color: #a0aec0;
  font-weight: 500;
}

.jb-item-stat-value {
  font-size: 0.82rem;
  font-weight: 700;
  color: #1a202c;
  display: inline-flex;
  align-items: baseline;
  gap: 2px;
}

.jb-item-stat-total {
  color: #0e5a3a;
}

.jb-item-stat-unit {
  font-size: 0.6rem;
  color: #a0aec0;
  font-weight: 500;
}

/* =========================================================
   سفارش‌های فرزند
   ========================================================= */
.jb-child-orders {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.jb-child-order-card {
  padding: 12px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 8px;
}

.jb-child-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  flex-wrap: wrap;
  padding-bottom: 10px;
  margin-bottom: 10px;
  border-bottom: 1px dashed rgba(14, 90, 58, 0.1);
}

.jb-child-header-info {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  flex-wrap: wrap;
}

.jb-child-id {
  font-size: 0.8rem;
  font-weight: 700;
  color: #1a202c;
}

.jb-child-date {
  font-size: 0.68rem;
  color: #a0aec0;
  font-weight: 500;
}

.jb-child-items {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.jb-child-item {
  padding: 10px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.06);
  border-radius: 6px;
}

.jb-child-item-main {
  display: flex;
  align-items: flex-start;
  gap: 10px;
}

.jb-child-item-img-wrap {
  flex-shrink: 0;
  width: 56px;
  height: 56px;
  border-radius: 6px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.08);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 4px;
  overflow: hidden;
}

.jb-child-item-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 4px;
}

.jb-child-item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 5px;
  min-width: 0;
}

.jb-child-item-title {
  font-size: 0.8rem;
  font-weight: 600;
  color: #1a202c;
  line-height: 1.5;
}

.jb-child-stats {
  margin-top: 8px;
}

/* جمع سفارش فرزند */
.jb-child-summary {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  flex-wrap: wrap;
  margin-top: 10px;
  padding-top: 10px;
  border-top: 1px dashed rgba(14, 90, 58, 0.1);
}

.jb-child-summary-info {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
  font-size: 0.72rem;
  color: #718096;
  font-weight: 500;
}

.jb-child-summary-info strong {
  color: #1a202c;
  font-weight: 700;
}

.jb-child-summary-total {
  font-size: 0.82rem;
  font-weight: 700;
  color: #0e5a3a;
}

/* =========================================================
   جمع کل
   ========================================================= */
.jb-totals-section {
  padding-top: 18px;
}

.jb-totals-box {
  max-width: 440px;
  margin-right: auto;
  padding: 16px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 8px;
}

.jb-total-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding: 6px 0;
  font-size: 0.82rem;
}

.jb-total-label {
  color: #4a5568;
  font-weight: 500;
}

.jb-total-value {
  color: #1a202c;
  font-weight: 700;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-total-unit {
  font-size: 0.62rem;
  color: #a0aec0;
  font-weight: 500;
}

.jb-total-row-dashed {
  padding-top: 10px;
  border-top: 1px dashed rgba(14, 90, 58, 0.1);
}

.jb-total-row-discount .jb-total-label,
.jb-total-row-discount .jb-total-value {
  color: #16a34a;
}

.jb-total-divider {
  height: 0;
  border-top: 1px solid rgba(14, 90, 58, 0.12);
  margin: 10px 0;
}

.jb-total-row-final {
  padding: 4px 0;
}

.jb-total-label-final {
  font-size: 0.95rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-total-value-final {
  font-size: 1.1rem;
  font-weight: 800;
  color: #0e5a3a;
  display: inline-flex;
  align-items: baseline;
  gap: 4px;
}

.jb-total-note {
  margin-top: 8px;
  font-size: 0.68rem;
  color: #a0aec0;
  font-weight: 500;
  text-align: left;
}

/* =========================================================
   دکمه‌های اقدام
   ========================================================= */
.jb-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  padding-top: 18px;
  border-top: 1px dashed rgba(14, 90, 58, 0.12);
  margin-top: 10px;
}

.jb-action-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 10px 20px;
  border-radius: 6px;
  font-size: 0.82rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
  border: 1px solid transparent;
}

.jb-action-back {
  background: #fff;
  color: #4a5568;
  border-color: rgba(14, 90, 58, 0.15);
}

.jb-action-back:hover {
  background: #f7fafc;
  color: #0e5a3a;
  border-color: rgba(14, 90, 58, 0.3);
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-order-detail-card {
    padding: 14px;
    border-radius: 8px;
  }

  .jb-detail-header-title h2 {
    font-size: 0.9rem;
  }

  .jb-detail-header-icon {
    width: 28px;
    height: 28px;
  }

  .jb-info-section {
    padding: 14px 0;
  }

  .jb-item-main {
    flex-direction: column;
    align-items: stretch;
  }

  .jb-item-img-wrap {
    width: 56px;
    height: 56px;
  }

  .jb-item-stats {
    justify-content: space-between;
  }

  .jb-totals-box {
    padding: 12px;
    max-width: none;
  }

  .jb-actions {
    flex-direction: column;
  }

  .jb-action-btn {
    width: 100%;
  }
}

@media (max-width: 480px) {
  .jb-guide-info {
    flex-direction: column;
    align-items: stretch;
  }

  .jb-copy-btn {
    align-self: flex-end;
  }

  .jb-receipt-tracking {
    flex-direction: column;
    align-items: flex-start;
  }
}

/* =========================================================
   استایل چاپ
   ========================================================= */
@media print {
  header,
  footer,
  aside {
    display: none !important;
  }

  .jb-order-detail-card {
    border: none !important;
    box-shadow: none !important;
    border-radius: 0 !important;
    padding: 0 !important;
  }

  .jb-actions,
  .jb-copy-btn,
  .jb-upload-btn {
    display: none !important;
  }
}
</style>