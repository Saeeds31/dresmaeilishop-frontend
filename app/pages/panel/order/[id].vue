<template>
  <!-- لودینگ -->
  <div v-if="loading" class="jb-order-detail-loading">
    <div class="jb-order-detail-spinner"></div>
    <span>در حال بارگذاری...</span>
  </div>

  <!-- خطا -->
  <div v-else-if="error" class="jb-order-detail-error">
    <div class="jb-error-icon">
      <svg class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8"
          d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
      </svg>
    </div>
    <h3 class="jb-error-title">خطا در بارگذاری اطلاعات</h3>
    <p class="jb-error-desc">{{ error }}</p>
    <button @click="fetchOrder" class="jb-btn-linear">تلاش مجدد</button>
  </div>

  <div v-else class="jb-order-detail-card">
    <!-- ============================================================
         هدر
    ============================================================ -->
    <div class="jb-detail-header">
      <div class="jb-detail-header-title">
        <span class="jb-detail-header-icon">
          <MoveRight class="w-5 h-5" />
        </span>
        <h2>جزئیات سفارش</h2>
        <span class="jb-detail-header-code">#{{ orderData?.id || "---" }}</span>
      </div>

      <!-- بج سفارش فرزند -->
      <span v-if="orderData?.parent_order_id" class="jb-child-badge">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2" d="M13 5l7 7-7 7M5 5l7 7-7 7" />
        </svg>
        سفارش فرزند
      </span>
    </div>

    <!-- خط بریده‌بریده -->
    <div class="jb-divider"></div>

    <!-- ============================================================
         وضعیت سفارش و پرداخت
    ============================================================ -->
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

    <!-- ============================================================
         بخش کارت به کارت
    ============================================================ -->
    <div v-if="orderData?.payment_method === 'card_transfer'" class="jb-card-transfer-box"
      :class="getCardTransferClass()">
      <!-- در انتظار آپلود رسید -->
      <div v-if="
        orderData.status === 'card_transfer_pending' ||
        orderData.status === 'card_transfer_send_again'
      ">
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
            <!-- شماره کارت -->
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

            <!-- شماره شبا -->
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

            <!-- صاحب حساب -->
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
        <button v-if="
          remainingSeconds > 0 ||
          orderData.status === 'card_transfer_send_again'
        " @click="openUploadReceiptModal" class="jb-btn-linear jb-upload-btn">
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
      <div v-else-if="
        orderData.status === 'cancelled' &&
        orderData.cardTransferReceipt?.status === 'rejected'
      ">
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

    <!-- ============================================================
         تصویر رسید
    ============================================================ -->
    <div v-if="orderData?.cardTransferReceipt?.image_url" class="jb-receipt-box">
      <h4 class="jb-receipt-title">
        <span class="jb-title-dot"></span>
        رسید پرداخت
      </h4>
      <img :src="$getImageUrl(orderData.cardTransferReceipt.image_url)" alt="رسید پرداخت" class="jb-receipt-img"
        @click="openImagePreview($getImageUrl(orderData.cardTransferReceipt.image_url))" />
      <div v-if="orderData.cardTransferReceipt.tracking_code" class="jb-receipt-tracking">
        <span class="jb-receipt-tracking-label">کد پیگیری:</span>
        <span class="jb-receipt-tracking-value" dir="ltr">
          {{ orderData.cardTransferReceipt.tracking_code }}
        </span>
      </div>
    </div>

    <!-- ============================================================
         اطلاعات سفارش
    ============================================================ -->
    <div class="jb-info-section">
      <h3 class="jb-section-title">
        <span class="jb-title-dot"></span>
        اطلاعات سفارش
      </h3>

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

    <!-- ============================================================
         اطلاعات حمل و نقل
    ============================================================ -->
    <div class="jb-info-section">
      <h3 class="jb-section-title">
        <span class="jb-title-dot"></span>
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

        <div v-if="
          orderData.card_transfer_receipt?.status == 'rejected' &&
          orderData.payment_method == 'card_transfer'
        " class="jb-info-item">
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

    <!-- ============================================================
         محصولات سفارش اصلی
    ============================================================ -->
    <div class="jb-info-section" v-if="orderData?.items?.length">
      <h3 class="jb-section-title">
        <span class="jb-title-dot"></span>
        محصولات سفارش
      </h3>

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

    <!-- ============================================================
         سفارش‌های فرزند
    ============================================================ -->
    <div v-if="orderData?.child_orders?.length" class="jb-info-section">
      <h3 class="jb-section-title">
        <span class="jb-title-dot"></span>
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
          <!-- هدر سفارش فرزند -->
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

          <!-- آیتم‌های سفارش فرزند -->
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

          <!-- جمع سفارش فرزند -->
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

    <!-- ============================================================
         جمع کل
    ============================================================ -->
    <div class="jb-totals-section">
      <h3 class="jb-section-title">
        <span class="jb-title-dot"></span>
        خلاصه پرداخت
      </h3>

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

    <!-- ============================================================
         دکمه‌های اقدام
    ============================================================ -->
    <div class="jb-actions">
      <button @click="goBack" class="jb-action-btn jb-action-back">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
        </svg>
        بازگشت
      </button>

      <!-- <button v-if="
        orderData?.status === 'paid' ||
        orderData?.status === 'delivered' ||
        orderData?.status === 'reserved'
      " @click="printOrder" class="jb-btn-linear jb-action-print">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M17 17h2a2 2 0 002-2v-4a2 2 0 00-2-2H5a2 2 0 00-2 2v4a2 2 0 002 2h2m2 4h6a2 2 0 002-2v-4H7v4a2 2 0 002 2zm8-12V5a2 2 0 00-2-2H9a2 2 0 00-2 2v4h10z" />
        </svg>
        چاپ فاکتور
      </button> -->
    </div>
  </div>

  <UploadReceiptModal :show="showUploadModal" :order-id="orderData?.id" @close="showUploadModal = false"
    @uploaded="onReceiptUploaded" />
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { MoveRight } from 'lucide-vue-next';
import { useApi } from '~/composables/api/useApi';
import UploadReceiptModal from '~/components/card-transfer/UploadReceiptModal.vue';
import { toast } from 'vue3-toastify';

const { $getImageUrl } = useNuxtApp();
const showUploadModal = ref(false);

const openUploadReceiptModal = () => {
  showUploadModal.value = true;
};

const onReceiptUploaded = (receipt) => {
  showUploadModal.value = false;
  // رفرش کردن اطلاعات سفارش
  fetchOrder();
  toast.success('رسید با موفقیت آپلود شد');
};
definePageMeta({
  layout: 'panel',
  middleware: 'auth'
});

const route = useRoute();
const router = useRouter();
const { getOrder } = useApi();

const orderData = ref(null);
const loading = ref(true);
const error = ref(null);

const orderId = computed(() => route.params.id);

// محاسبه جمع سفارش‌های فرزند
const childOrdersSubtotal = computed(() => {
  if (!orderData.value?.child_orders?.length) return 0;
  return orderData.value.child_orders.reduce((sum, child) => sum + (child.total || 0), 0);
});

// محاسبه جمع کل با احتساب سفارش‌های فرزند
const totalWithChildren = computed(() => {
  if (!orderData.value) return 0;
  const mainTotal = orderData.value.total || 0;
  return mainTotal + childOrdersSubtotal.value;
});

// تابع دریافت نام ویژگی
const getAttributeName = (id) => {
  const names = { 1: 'سایز', 2: 'رنگ' };
  return names[id] || 'ویژگی';
};

const fetchOrder = async () => {
  loading.value = true;
  error.value = null;

  try {
    const response = await getOrder(orderId.value);
    if (response && response.order) {
      orderData.value = response.order;
    } else {
      error.value = 'سفارش مورد نظر یافت نشد';
    }
    if (orderData.value.status === 'card_transfer_pending') {
      startTimer();
    }
  } catch (err) {
    console.error('Error fetching order:', err);
    error.value = err.response?.data?.message || 'خطا در ارتباط با سرور';
  } finally {
    loading.value = false;
  }
};

const formatPrice = (price) => {
  if (!price && price !== 0) return '۰';
  return price.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
};
const bankCardNumber = ref("6104-3310-4719-3157");
const bankAccountNumber = ref("4006051640119");
const bankShebaNumber = ref("IR190120010000002917515931");
const bankAccountOwner = ref("جاوید رحیمی نژاد - بانک ملت");
const formatDate = (date) => {
  if (!date) return '---';
  try {
    const dateObj = new Date(date);
    return new Intl.DateTimeFormat('fa-IR', {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    }).format(dateObj);
  } catch {
    return date;
  }
};

const getStatusText = (status) => {
  const statusMap = {
    pending: 'در انتظار پرداخت',
    paid: 'پرداخت شده',
    reserved: 'رزرو شده',
    completed: 'تکمیل شده',
    card_transfer_pending: "در انتظار آپلود رسید",
    card_transfer_review: "در حال پردازش رسید",
    card_transfer_send_again: "در انتظار آپلود مجدد رسید",
    processing: 'در حال پردازش',
    shipped: 'ارسال شده',
    delivered: 'تحویل داده شده',
    cancelled: 'لغو شده',
    failed: 'ناموفق'
  };
  return statusMap[status] || status || 'نامشخص';
};

const getStatusClass = (status) => {
  const classMap = {
    pending: 'bg-yellow-100 text-yellow-800',
    paid: 'bg-green-100 text-green-800',
    reserved: 'bg-blue-100 text-blue-800',
    processing: 'bg-indigo-100 text-indigo-800',
    shipped: 'bg-purple-100 text-purple-800',
    delivered: 'bg-emerald-100 text-emerald-800',
    cancelled: 'bg-red-100 text-red-800',
    failed: 'bg-rose-100 text-rose-800'
  };
  return classMap[status] || 'bg-gray-100 text-gray-800';
};

const getPaymentStatusText = (status) => {
  const statusMap = {
    pending: 'در انتظار پرداخت',
    paid: 'پرداخت شده',
    unpaid: 'پرداخت نشده',
    failed: 'ناموفق',
    refunded: 'بازگشت وجه'
  };
  return statusMap[status] || status || 'نامشخص';
};

const getPaymentStatusClass = (status) => {
  const classMap = {
    paid: 'bg-green-100 text-green-800',
    pending: 'bg-yellow-100 text-yellow-800',
    unpaid: 'bg-yellow-100 text-yellow-800',
    failed: 'bg-red-100 text-red-800',
    refunded: 'bg-blue-100 text-blue-800'
  };
  return classMap[status] || 'bg-gray-100 text-gray-800';
};

const getPaymentMethodText = (method) => {
  const methodMap = {
    wallet: 'کیف پول',
    online: 'درگاه اینترنتی',
    cash: 'پرداخت نقدی',
    card_transfer: 'کارت به کارت'
  };
  return methodMap[method] || method || 'نامشخص';
};

const getReservationTypeText = (type) => {
  const typeMap = {
    three_days: 'رزرو ۳ روزه',
    seven_days: 'رزرو ۷ روزه',
    none: 'بدون رزرو'
  };
  return typeMap[type] || type || 'نامشخص';
};

const goBack = () => {
  router.back();
};

const printOrder = () => {
  window.print();
};
const getCardTransferClass = () => {
  if (orderData.value?.status === 'card_transfer_pending') {
    return 'border-yellow-300 bg-yellow-50';
  }
  if (orderData.value?.status === 'card_transfer_review') {
    return 'border-blue-300 bg-blue-50';
  }
  if (orderData.value?.status === 'cancelled') {
    return 'border-red-300 bg-red-50';
  }
  return 'border-green-300 bg-green-50';
};
const copyText = (text) => {
  if (navigator.clipboard) {
    navigator.clipboard.writeText(text)
      .then(() => {
        toast.success('متن با موفقیت کپی شد');
      })
      .catch(() => {
        // fallback
        copyTextFallback(text);
      });
  } else {
    copyTextFallback(text);
  }
};

// روش جایگزین برای کپی
const copyTextFallback = (text) => {
  const textarea = document.createElement('textarea');
  textarea.value = text;
  textarea.style.position = 'fixed';
  textarea.style.opacity = '0';
  document.body.appendChild(textarea);
  textarea.select();
  try {
    document.execCommand('copy');
    toast.success('متن با موفقیت کپی شد');
  } catch (err) {
    toast.error('خطا در کپی متن');
  }
  document.body.removeChild(textarea);
};
const remainingSeconds = ref(0);
let timerInterval = null;

// تابع تایمر
const startTimer = () => {
  if (!orderData.value?.receipt_remaining_seconds) return;

  remainingSeconds.value = Math.ceil(Math.max(0, orderData.value.receipt_remaining_seconds));

  if (timerInterval) clearInterval(timerInterval);

  timerInterval = setInterval(() => {
    if (remainingSeconds.value > 0) {
      remainingSeconds.value--;
    } else {
      clearInterval(timerInterval);
      // زمان تمام شد، سفارش رو دوباره fetch کن
      fetchOrder();
    }
  }, 1000);
};
// باز کردن پیش‌نمایش تصویر
const openImagePreview = (imageUrl) => {
  // باز کردن تصویر در مودال یا new window
  window.open(imageUrl, '_blank');
};
// فرمت تایمر
const formatTimer = (seconds) => {
  const mins = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
};
onMounted(() => {
  fetchOrder();
});
useSeoMeta({
  title: () => 'جزئیات سفارش پنل کاربری | فروشگاه دکتر اسماعیلی',
  description: ' جزئیات سفارش کاربر در پنل اختصاصی',
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
   صفحه جزئیات سفارش - تم پودری
   ========================================================= */

/* ===== لودینگ ===== */
.jb-order-detail-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 60px 20px;
  color: var(--jb-text-muted);
  font-weight: 700;
  font-size: 0.9rem;
}

.jb-order-detail-spinner {
  width: 28px;
  height: 28px;
  border: 3px solid var(--jb-pink-200);
  border-top-color: var(--jb-fuchsia);
  border-radius: 50%;
  animation: jbSpin 0.9s linear infinite;
}

@keyframes jbSpin {
  to {
    transform: rotate(360deg);
  }
}

/* ===== خطا ===== */
.jb-order-detail-error {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 48px 24px;
  background: #fff;
  border: 2px solid rgba(201, 24, 74, 0.3);
  border-radius: 24px;
  text-align: center;
}

.jb-error-icon {
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, #ffe0e8, #ffc2d6);
  color: #c9184a;
  border: 3px dashed #ff9db8;
  margin-bottom: 6px;
}

.jb-error-title {
  font-size: 1.1rem;
  font-weight: 900;
  color: #c9184a;
}

.jb-error-desc {
  font-size: 0.85rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  line-height: 1.8;
}

/* ===== کارت اصلی ===== */
.jb-order-detail-card {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 24px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(214, 51, 132, 0.1);
}

/* =========================================================
   هدر
   ========================================================= */
.jb-detail-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
}

.jb-detail-header-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.jb-detail-header-icon {
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

.jb-detail-header-title h2 {
  font-size: 1.15rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-detail-header-code {
  font-size: 0.8rem;
  font-weight: 800;
  color: var(--jb-fuchsia);
  background: var(--jb-pink-50);
  padding: 4px 12px;
  border-radius: 999px;
  border: 1.5px solid var(--jb-pink-200);
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
}

.jb-child-badge {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 6px 12px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  font-size: 0.72rem;
  font-weight: 800;
  border-radius: 999px;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
}

/* خط بریده‌بریده */
.jb-divider {
  height: 0;
  border-top: 2px dashed var(--jb-pink-200);
  margin: 18px 0 24px;
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

/* =========================================================
   نوار وضعیت
   ========================================================= */
.jb-status-bar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 18px;
  padding: 16px 20px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px dashed var(--jb-pink-300);
  border-radius: 18px;
  margin-bottom: 20px;
}

.jb-status-item {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.jb-status-label {
  font-size: 0.78rem;
  color: var(--jb-text-muted);
  font-weight: 700;
}

.jb-status-badge {
  display: inline-flex;
  align-items: center;
  padding: 5px 14px;
  border-radius: 999px;
  font-size: 0.75rem;
  font-weight: 800;
  border: 1.5px solid transparent;
}

.jb-status-sm {
  padding: 3px 10px;
  font-size: 0.68rem;
}

.jb-badge-reserved {
  background: linear-gradient(135deg, #e0e7ff, #c7d2fe);
  color: #4338ca;
  border-color: rgba(67, 56, 202, 0.2);
}

/* رنگ‌های وضعیت - از کلاس‌های تابع getStatusClass میان، اما با ظاهر بهتر */
.jb-status-badge.bg-yellow-100 {
  background: linear-gradient(135deg, #fef3c7, #fde68a) !important;
  color: #b45309 !important;
  border-color: rgba(180, 83, 9, 0.2);
}

.jb-status-badge.bg-green-100 {
  background: linear-gradient(135deg, #dcfce7, #bbf7d0) !important;
  color: #16a34a !important;
  border-color: rgba(22, 163, 74, 0.2);
}

.jb-status-badge.bg-blue-100 {
  background: linear-gradient(135deg, #dbeafe, #bfdbfe) !important;
  color: #1d4ed8 !important;
  border-color: rgba(29, 78, 216, 0.2);
}

.jb-status-badge.bg-indigo-100 {
  background: linear-gradient(135deg, #e0e7ff, #c7d2fe) !important;
  color: #4338ca !important;
}

.jb-status-badge.bg-purple-100 {
  background: linear-gradient(135deg, #f3e8ff, #e9d5ff) !important;
  color: #7e22ce !important;
}

.jb-status-badge.bg-emerald-100 {
  background: linear-gradient(135deg, #d1fae5, #a7f3d0) !important;
  color: #047857 !important;
}

.jb-status-badge.bg-red-100,
.jb-status-badge.bg-rose-100 {
  background: linear-gradient(135deg, #ffe0e8, #ffc2d6) !important;
  color: #c9184a !important;
  border-color: rgba(201, 24, 74, 0.2);
}

.jb-status-badge.bg-gray-100 {
  background: linear-gradient(135deg, #f1f5f9, #e2e8f0) !important;
  color: #475569 !important;
}

/* =========================================================
   کارت به کارت
   ========================================================= */
.jb-card-transfer-box {
  margin-bottom: 20px;
  padding: 20px;
  border-radius: 18px;
  border: 2px dashed;
  transition: all 0.3s ease;
}

.jb-card-transfer-box.border-yellow-300 {
  background: linear-gradient(135deg, #fffbea, #fff3c4) !important;
  border-color: #ffd88a !important;
}

.jb-card-transfer-box.border-blue-300 {
  background: linear-gradient(135deg, #eff6ff, #dbeafe) !important;
  border-color: #93c5fd !important;
}

.jb-card-transfer-box.border-red-300 {
  background: linear-gradient(135deg, #ffe0e8, #ffc2d6) !important;
  border-color: #ff9db8 !important;
}

.jb-card-transfer-box.border-green-300 {
  background: linear-gradient(135deg, #dcfce7, #bbf7d0) !important;
  border-color: #86efac !important;
}

/* هشدار */
.jb-card-transfer-alert {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 14px;
}

.jb-alert-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 10px;
  border: 2px solid #fff;
  flex-shrink: 0;
}

.jb-alert-yellow {
  background: linear-gradient(135deg, #ffb84d, #ffa31a);
  color: #fff;
  box-shadow: 0 6px 16px rgba(255, 163, 26, 0.35);
}

.jb-alert-blue {
  background: linear-gradient(135deg, #60a5fa, #2563eb);
  color: #fff;
  box-shadow: 0 6px 16px rgba(37, 99, 235, 0.35);
}

.jb-alert-red {
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  color: #fff;
  box-shadow: 0 6px 16px rgba(201, 24, 74, 0.35);
}

.jb-alert-green {
  background: linear-gradient(135deg, #4ade80, #16a34a);
  color: #fff;
  box-shadow: 0 6px 16px rgba(34, 197, 94, 0.35);
}

.jb-alert-text {
  font-size: 0.88rem;
  font-weight: 900;
  color: #7a4a00;
}

.jb-alert-text-blue {
  color: #1d4ed8;
}

.jb-alert-text-red {
  color: #c9184a;
}

.jb-alert-text-green {
  color: #16a34a;
}

.jb-alert-desc {
  font-size: 0.8rem;
  color: var(--jb-text-soft);
  font-weight: 600;
  line-height: 1.8;
  margin-top: 6px;
  padding-right: 42px;
}

/* راهنما */
.jb-card-transfer-guide {
  padding: 16px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 14px;
  border: 1.5px solid rgba(255, 216, 138, 0.5);
  margin-bottom: 14px;
}

.jb-guide-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  font-weight: 900;
  color: #7a4a00;
  margin-bottom: 12px;
}

.jb-guide-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  list-style: none;
  padding: 0;
  margin: 0;
}

.jb-guide-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding: 10px 14px;
  background: #fff;
  border-radius: 12px;
  border: 1px solid rgba(255, 216, 138, 0.6);
}

.jb-guide-info-content {
  display: flex;
  flex-direction: column;
  gap: 3px;
  min-width: 0;
}

.jb-guide-info-label {
  font-size: 0.7rem;
  color: #b07500;
  font-weight: 700;
}

.jb-guide-info-value {
  font-family: 'Courier New', monospace;
  font-size: 0.85rem;
  font-weight: 900;
  color: #7a4a00;
  letter-spacing: 0.5px;
  word-break: break-all;
}

.jb-copy-btn {
  flex-shrink: 0;
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 6px 12px;
  background: linear-gradient(135deg, #ffb84d, #ffa31a);
  color: #fff;
  border: none;
  border-radius: 9px;
  font-size: 0.72rem;
  font-weight: 800;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 4px 12px rgba(255, 163, 26, 0.35);
}

.jb-copy-btn:hover {
  background: linear-gradient(135deg, #ffa31a, #cc7a00);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(255, 163, 26, 0.5);
}

.jb-guide-owner {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 12px;
  font-size: 0.82rem;
}

.jb-guide-owner-label {
  color: #b07500;
  font-weight: 700;
}

.jb-guide-owner-name {
  font-weight: 900;
  color: #7a4a00;
}

/* تایمر */
.jb-timer-box {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: #fff;
  border-radius: 14px;
  border: 1.5px solid rgba(255, 216, 138, 0.6);
  margin-bottom: 14px;
  flex-wrap: wrap;
}

.jb-timer-label {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.82rem;
  color: var(--jb-text-soft);
  font-weight: 700;
}

.jb-timer-label svg {
  color: #b45309;
}

.jb-timer-value {
  font-family: 'Courier New', monospace;
  font-size: 1.15rem;
  font-weight: 900;
  color: #b45309;
  letter-spacing: 1px;
}

.jb-timer-error {
  font-size: 0.82rem;
  color: #c9184a;
  font-weight: 800;
  line-height: 1.7;
}

/* دکمه آپلود */
.jb-upload-btn {
  width: 100%;
  justify-content: center;
  padding: 13px 24px !important;
  font-size: 0.9rem !important;
  background: linear-gradient(135deg, #ffb84d, #ffa31a) !important;
  box-shadow: 0 8px 22px rgba(255, 163, 26, 0.4) !important;
}

.jb-upload-btn:hover {
  background: linear-gradient(135deg, #ffa31a, #cc7a00) !important;
  box-shadow: 0 14px 32px rgba(255, 163, 26, 0.55) !important;
}

/* =========================================================
   تصویر رسید
   ========================================================= */
.jb-receipt-box {
  margin-bottom: 20px;
  padding: 18px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
}

.jb-receipt-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.92rem;
  font-weight: 900;
  color: var(--jb-text);
  margin-bottom: 14px;
}

.jb-title-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
  flex-shrink: 0;
}

.jb-receipt-img {
  max-width: 280px;
  max-height: 220px;
  object-fit: contain;
  border-radius: 14px;
  border: 3px solid #fff;
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.2);
  cursor: pointer;
  transition: all 0.3s ease;
}

.jb-receipt-img:hover {
  transform: scale(1.03);
  box-shadow: 0 15px 35px rgba(214, 51, 132, 0.3);
}

.jb-receipt-tracking {
  margin-top: 12px;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 14px;
  background: #fff;
  border: 1.5px dashed var(--jb-pink-300);
  border-radius: 10px;
  width: fit-content;
}

.jb-receipt-tracking-label {
  font-size: 0.78rem;
  color: var(--jb-text-muted);
  font-weight: 700;
}

.jb-receipt-tracking-value {
  font-family: 'Courier New', monospace;
  font-size: 0.85rem;
  color: var(--jb-fuchsia);
  font-weight: 900;
  letter-spacing: 0.5px;
}

/* =========================================================
   بخش اطلاعات
   ========================================================= */
.jb-info-section {
  padding: 22px 0;
  border-bottom: 1.5px dashed var(--jb-pink-200);
}

.jb-info-section:last-of-type {
  border-bottom: none;
}

.jb-section-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 0.95rem;
  font-weight: 900;
  color: var(--jb-text);
  margin-bottom: 18px;
  flex-wrap: wrap;
}

.jb-ship-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.jb-ship-title svg {
  color: var(--jb-fuchsia);
}

.jb-ship-count {
  font-size: 0.72rem;
  color: var(--jb-text-muted);
  font-weight: 700;
}

/* گرید اطلاعات */
.jb-info-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 14px;
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
  gap: 6px;
  padding: 12px 14px;
  background: #fff;
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 14px;
  transition: all 0.25s ease;
}

.jb-info-item:hover {
  border-color: var(--jb-pink-400);
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.1);
}

.jb-info-item-full {
  grid-column: 1 / -1;
}

.jb-info-label {
  font-size: 0.72rem;
  color: var(--jb-text-muted);
  font-weight: 700;
}

.jb-info-value {
  font-size: 0.85rem;
  color: var(--jb-text);
  font-weight: 800;
  word-break: break-word;
}

.jb-info-value-mono {
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
}

.jb-info-value-danger {
  color: #c9184a;
}

.jb-info-unit {
  font-size: 0.68rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  margin-right: 3px;
}

/* =========================================================
   آیتم‌های سفارش
   ========================================================= */
.jb-items-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.jb-item-card {
  padding: 16px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  display: flex;
  flex-direction: column;
  gap: 14px;
  transition: all 0.3s ease;
}

.jb-item-card:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.14);
  transform: translateY(-2px);
}

.jb-item-main {
  display: flex;
  align-items: flex-start;
  gap: 14px;
}

.jb-item-img-wrap {
  flex-shrink: 0;
  width: 80px;
  height: 80px;
  border-radius: 14px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 6px;
  overflow: hidden;
}

.jb-item-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 8px;
}

.jb-item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
  min-width: 0;
}

.jb-item-title {
  font-size: 0.9rem;
  font-weight: 900;
  color: var(--jb-text);
  line-height: 1.5;
}

.jb-item-attrs {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.jb-item-attr-chip {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 3px 10px;
  background: #fff;
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 999px;
  font-size: 0.7rem;
  font-weight: 700;
}

.jb-item-attr-name {
  color: var(--jb-text-muted);
}

.jb-item-attr-value {
  color: var(--jb-fuchsia);
  font-weight: 900;
}

.jb-item-sku {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.72rem;
}

.jb-item-sku-label {
  color: var(--jb-text-muted);
  font-weight: 700;
}

.jb-item-sku-value {
  font-family: 'Courier New', monospace;
  color: var(--jb-text-soft);
  font-weight: 800;
  letter-spacing: 0.5px;
}

/* آمار آیتم */
.jb-item-stats {
  display: flex;
  align-items: center;
  gap: 14px;
  flex-wrap: wrap;
  padding-top: 12px;
  border-top: 1.5px dashed var(--jb-pink-200);
}

.jb-item-stat {
  display: flex;
  flex-direction: column;
  gap: 3px;
  min-width: 72px;
}

.jb-item-stat-label {
  font-size: 0.68rem;
  color: var(--jb-text-muted);
  font-weight: 700;
}

.jb-item-stat-value {
  font-size: 0.85rem;
  font-weight: 900;
  color: var(--jb-text);
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-item-stat-total {
  color: var(--jb-pink-700);
}

.jb-item-stat-unit {
  font-size: 0.65rem;
  color: var(--jb-text-muted);
  font-weight: 600;
}

/* =========================================================
   سفارش‌های فرزند
   ========================================================= */
.jb-child-orders {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.jb-child-order-card {
  padding: 16px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  transition: all 0.3s ease;
}

.jb-child-order-card:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.14);
}

.jb-child-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
  padding-bottom: 12px;
  margin-bottom: 14px;
  border-bottom: 1.5px dashed var(--jb-pink-200);
}

.jb-child-header-info {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.jb-child-id {
  font-size: 0.85rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-child-date {
  font-size: 0.72rem;
  color: var(--jb-text-muted);
  font-weight: 700;
}

.jb-child-items {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.jb-child-item {
  padding: 14px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 14px;
}

.jb-child-item-main {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.jb-child-item-img-wrap {
  flex-shrink: 0;
  width: 64px;
  height: 64px;
  border-radius: 12px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px;
  overflow: hidden;
}

.jb-child-item-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 8px;
}

.jb-child-item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 6px;
  min-width: 0;
}

.jb-child-item-title {
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
  line-height: 1.5;
}

.jb-child-stats {
  margin-top: 10px;
}

/* جمع سفارش فرزند */
.jb-child-summary {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
  margin-top: 14px;
  padding-top: 12px;
  border-top: 1.5px dashed var(--jb-pink-200);
}

.jb-child-summary-info {
  display: inline-flex;
  align-items: center;
  gap: 16px;
  flex-wrap: wrap;
  font-size: 0.78rem;
  color: var(--jb-text-soft);
  font-weight: 600;
}

.jb-child-summary-info strong {
  color: var(--jb-text);
  font-weight: 900;
}

.jb-child-summary-total {
  font-size: 0.9rem;
  font-weight: 900;
  color: var(--jb-pink-700);
}

/* =========================================================
   جمع کل
   ========================================================= */
.jb-totals-section {
  padding-top: 22px;
}

.jb-totals-box {
  max-width: 480px;
  margin-right: auto;
  padding: 20px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px dashed var(--jb-pink-300);
  border-radius: 20px;
}

.jb-total-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 8px 0;
  font-size: 0.85rem;
}

.jb-total-label {
  color: var(--jb-text-soft);
  font-weight: 700;
}

.jb-total-value {
  color: var(--jb-text);
  font-weight: 900;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-total-unit {
  font-size: 0.68rem;
  color: var(--jb-text-muted);
  font-weight: 600;
}

.jb-total-row-dashed {
  padding-top: 12px;
  border-top: 1.5px dashed var(--jb-pink-200);
}

.jb-total-row-discount .jb-total-label,
.jb-total-row-discount .jb-total-value {
  color: #16a34a;
}

.jb-total-divider {
  height: 0;
  border-top: 1.5px solid var(--jb-pink-300);
  margin: 12px 0;
}

.jb-total-row-final {
  padding: 6px 0;
}

.jb-total-label-final {
  font-size: 1.05rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-total-value-final {
  font-size: 1.25rem;
  font-weight: 900;
  color: var(--jb-pink-700);
  display: inline-flex;
  align-items: baseline;
  gap: 4px;
}

.jb-total-note {
  margin-top: 10px;
  font-size: 0.72rem;
  color: var(--jb-text-muted);
  font-weight: 700;
  text-align: left;
}

/* =========================================================
   دکمه‌های اقدام
   ========================================================= */
.jb-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  padding-top: 22px;
  border-top: 1.5px dashed var(--jb-pink-200);
  margin-top: 12px;
}

.jb-action-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 12px 24px;
  border-radius: 14px;
  font-size: 0.85rem;
  font-weight: 800;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.25s ease;
  border: 2px solid transparent;
}

.jb-action-back {
  background: #fff;
  color: var(--jb-text-soft);
  border-color: var(--jb-pink-200);
}

.jb-action-back:hover {
  background: var(--jb-pink-50);
  color: var(--jb-fuchsia);
  border-color: var(--jb-pink-400);
  transform: translateY(-2px);
}

.jb-action-print {
  padding: 12px 26px !important;
  font-size: 0.85rem !important;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-order-detail-card {
    padding: 16px;
    border-radius: 20px;
  }

  .jb-detail-header-title h2 {
    font-size: 1rem;
  }

  .jb-detail-header-icon {
    width: 32px;
    height: 32px;
  }

  .jb-info-section {
    padding: 18px 0;
  }

  .jb-item-main {
    flex-direction: column;
    align-items: stretch;
  }

  .jb-item-img-wrap {
    width: 68px;
    height: 68px;
  }

  .jb-item-stats {
    justify-content: space-between;
  }

  .jb-totals-box {
    padding: 16px;
    max-width: none;
  }

  .jb-actions {
    flex-direction: column;
  }

  .jb-action-btn,
  .jb-action-print {
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