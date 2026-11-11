<template>
  <div class="container jb-payment-page">
    <SharedBreadCrumb :items="[
      { label: 'سبد خرید', href: '/cart' },
      { label: 'انتخاب آدرس و روش ارسال', href: '/shipping' },
      { label: 'پرداخت', href: '/payment' },
    ]" />
    <CartProgressSteps :current-step="3" />

    <div class="jb-payment-card">
      <!-- هدر -->
      <div class="jb-payment-header">
        <h2 class="jb-payment-title">
          <span class="jb-payment-title-icon">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M3 10h18M7 15h1m4 0h1m-7 4h12a3 3 0 003-3V8a3 3 0 00-3-3H6a3 3 0 00-3 3v8a3 3 0 003 3z" />
            </svg>
          </span>
          انتخاب شیوه پرداخت
        </h2>
      </div>

      <div class="jb-divider"></div>

      <div class="jb-payment-grid">
        <!-- ============================================================
             ستون راست: روش‌های پرداخت
        ============================================================ -->
        <div class="jb-payment-col">
          <!-- روش‌های پرداخت -->
          <div class="jb-payment-section">
            <h3 class="jb-section-title">
              <span class="jb-title-dot"></span>
              انتخاب روش پرداخت
            </h3>

            <div class="jb-payment-methods">
              <!-- پرداخت اینترنتی -->
              <label class="jb-pay-method" :class="{ 'is-selected': paymentMethod === 'online' }">
                <input type="radio" name="paymentMethod" value="online" v-model="paymentMethod"
                  class="jb-pay-radio-input" />
                <div class="jb-pay-method-inner">
                  <div class="jb-pay-radio" :class="{ 'is-checked': paymentMethod === 'online' }">
                    <div v-if="paymentMethod === 'online'" class="jb-pay-radio-dot"></div>
                  </div>
                  <div class="jb-pay-method-info">
                    <div class="jb-pay-method-title">
                      <span class="jb-pay-method-icon">
                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                            d="M3 10h18M7 15h1m4 0h1m-7 4h12a3 3 0 003-3V8a3 3 0 00-3-3H6a3 3 0 00-3 3v8a3 3 0 003 3z" />
                        </svg>
                      </span>
                      پرداخت اینترنتی
                    </div>
                    <span class="jb-pay-method-desc">
                      پرداخت با کلیه کارت‌های عضو شتاب
                    </span>
                  </div>
                </div>
              </label>

              <!-- کیف پول -->
              <label v-if="wallet && wallet.balance" class="jb-pay-method"
                :class="{ 'is-selected': paymentMethod === 'wallet' }">
                <input type="radio" name="paymentMethod" value="wallet" v-model="paymentMethod"
                  class="jb-pay-radio-input" />
                <div class="jb-pay-method-inner">
                  <div class="jb-pay-radio" :class="{ 'is-checked': paymentMethod === 'wallet' }">
                    <div v-if="paymentMethod === 'wallet'" class="jb-pay-radio-dot"></div>
                  </div>
                  <div class="jb-pay-method-info">
                    <div class="jb-pay-method-title">
                      <span class="jb-pay-method-icon">
                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                            d="M3 10h18M7 15h1m4 0h1m-7 4h12a3 3 0 003-3V8a3 3 0 00-3-3H6a3 3 0 00-3 3v8a3 3 0 003 3z" />
                        </svg>
                      </span>
                      پرداخت از کیف پول
                    </div>
                    <span class="jb-pay-method-desc">
                      استفاده از موجودی کیف پول شما
                      <strong class="jb-pay-wallet-amount">
                        ({{ wallet.balance.toLocaleString("fa-IR") }} تومان)
                      </strong>
                    </span>
                  </div>
                </div>
              </label>

              <!-- کارت به کارت -->
              <label class="jb-pay-method" :class="{ 'is-selected': paymentMethod === 'card_transfer' }">
                <input type="radio" name="paymentMethod" value="card_transfer" v-model="paymentMethod"
                  class="jb-pay-radio-input" />
                <div class="jb-pay-method-inner">
                  <div class="jb-pay-radio" :class="{ 'is-checked': paymentMethod === 'card_transfer' }">
                    <div v-if="paymentMethod === 'card_transfer'" class="jb-pay-radio-dot"></div>
                  </div>
                  <div class="jb-pay-method-info">
                    <div class="jb-pay-method-title">
                      <span class="jb-pay-method-icon">
                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                            d="M8 7h12m0 0l-4-4m4 4l-4 4m0 6H4m0 0l4 4m-4-4l4-4" />
                        </svg>
                      </span>
                      کارت به کارت
                    </div>
                    <span class="jb-pay-method-desc">
                      پرداخت از طریق کارت به کارت و آپلود رسید
                    </span>
                  </div>
                </div>
              </label>
            </div>
          </div>

          <!-- ============================================================
               انتخاب درگاه (فقط اینترنتی)
          ============================================================ -->
          <div v-if="paymentMethod === 'online'" class="jb-payment-section">
            <h3 class="jb-section-title">
              <span class="jb-title-dot"></span>
              انتخاب درگاه پرداخت
            </h3>

            <div class="jb-gateway-grid">
              <button type="button" aria-label="انتخاب درگاه زرین‌پال" class="jb-gateway-card"
                :class="{ 'is-selected': selected === 2 }" @click="selected = 2">
                <span class="jb-gateway-logo">
                  <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M12 2l9 5v10l-9 5-9-5V7l9-5z" />
                  </svg>
                </span>
                <span class="jb-gateway-name">درگاه زیبال</span>
                <span class="jb-gateway-sub-name">پرداخت نقدی</span>
                <span v-if="selected === 2" class="jb-gateway-check">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7" />
                  </svg>
                </span>
              </button>
              <button type="button" aria-label="انتخاب درگاه ترب پی" class="jb-gateway-card"
                :class="{ 'is-selected': selected === 3 }" @click="selected = 3">
                <span class="jb-gateway-logo">
                  <img width="48" height="48" class="rounded-full" src="/images/torobpaylogo.webp" alt="پرداخت چهار قسطه با ترب پی فروشگاه دکتر اسماعیلی" />
                </span>
                <span class="jb-gateway-name">ترب پی</span>
                <span class="jb-gateway-sub-name">پرداخت 4 قسطه </span>
                <span v-if="selected === 3" class="jb-gateway-check">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7" />
                  </svg>
                </span>
              </button>
            </div>

            <!-- پیام توجه -->
            <div class="jb-info-box">
              <span class="jb-info-box-icon">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
              </span>
              <span>
                بسته به حجم سفارشات شما عزیزان متعاقبا بسته‌های ارسالی‌تان بین ۲۴ تا ۴۸ ساعت زمان برای بسته‌بندی و ارسال
                نیاز دارد.
              </span>
            </div>

            <!-- توضیحات سفارش -->
            <div class="jb-form-field">
              <label class="jb-form-label">
                <span class="jb-form-label-dot"></span>
                توضیحات سفارش
              </label>
              <textarea v-model="user_note" class="jb-form-textarea"
                placeholder="توضیحات خود را در خصوص سفارش وارد کنید..." rows="3"></textarea>
            </div>
          </div>

          <!-- ============================================================
               راهنمای کارت به کارت
          ============================================================ -->
          <div v-if="paymentMethod === 'card_transfer'" class="jb-card-transfer-box">
            <div class="jb-card-transfer-header">
              <span class="jb-card-transfer-icon">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
              </span>
              <span class="jb-card-transfer-title">راهنمای پرداخت کارت به کارت</span>
            </div>

            <ul class="jb-card-transfer-list">
              <li class="jb-card-transfer-item">
                پس از ثبت سفارش، مبلغ را به یکی از روش‌های زیر واریز کنید
              </li>

              <!-- شماره کارت -->
              <li class="jb-card-transfer-info">
                <div class="jb-card-transfer-info-content">
                  <span class="jb-card-transfer-label">شماره کارت</span>
                  <span class="jb-card-transfer-value" dir="ltr">
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
              <li class="jb-card-transfer-info">
                <div class="jb-card-transfer-info-content">
                  <span class="jb-card-transfer-label">شماره شبا</span>
                  <span class="jb-card-transfer-value" dir="ltr">
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
              <li class="jb-card-transfer-owner">
                <span class="jb-card-transfer-label">صاحب حساب:</span>
                <span class="jb-card-transfer-owner-name">{{ bankAccountOwner }}</span>
              </li>

              <!-- هشدار -->
              <li class="jb-card-transfer-warning">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
                </svg>
                <span>
                  توجه: پس از واریز، رسید را در صفحه سفارش آپلود کنید. برای ثبت رسید تا ۱۵ دقیقه بعد از سفارش زمان
                  دارید!
                </span>
              </li>
            </ul>
          </div>

          <!-- دکمه مرحله قبل -->
          <NuxtLink to="/shipping" class="jb-shipping-btn-secondary jb-payment-back-btn">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M9 5l7 7-7 7" />
            </svg>
            مرحله قبل
          </NuxtLink>
        </div>

        <!-- ============================================================
             ستون چپ: خلاصه سفارش
        ============================================================ -->
        <div class="jb-payment-col">
          <!-- نوع سفارش (رزرو) -->
          <!-- <div
            class="jb-reservation-grid"
            :class="[
              selectedReservation && selectedReservation.order_id
                ? 'is-single'
                : 'is-three',
            ]"
          >
            <label
              class="jb-res-card"
              :class="{ 'is-selected': reservationType === 'none' }"
            >
              <input
                v-model="reservationType"
                type="radio"
                value="none"
                name="reservation_type"
                class="jb-pay-radio-input"
              />
              <div class="jb-res-card-inner">
                <div class="jb-pay-radio" :class="{ 'is-checked': reservationType === 'none' }">
                  <div v-if="reservationType === 'none'" class="jb-pay-radio-dot"></div>
                </div>
                <div class="jb-res-card-info">
                  <div class="jb-res-card-title">سفارش عادی</div>
                  <div class="jb-res-card-desc">
                    بدون رزرو
                    {{
                      selectedReservation && selectedReservation.order_id
                        ? ` به همراه سفارش ${selectedReservation.order_id}`
                        : ""
                    }}
                  </div>
                </div>
              </div>
            </label>

            <label
              v-if="!selectedReservation || !selectedReservation.order_id"
              class="jb-res-card"
              :class="{ 'is-selected': reservationType === 'three_days' }"
            >
              <input
                v-model="reservationType"
                type="radio"
                value="three_days"
                name="reservation_type"
                class="jb-pay-radio-input"
              />
              <div class="jb-res-card-inner">
                <div class="jb-pay-radio" :class="{ 'is-checked': reservationType === 'three_days' }">
                  <div v-if="reservationType === 'three_days'" class="jb-pay-radio-dot"></div>
                </div>
                <div class="jb-res-card-info">
                  <div class="jb-res-card-title">رزرو 3 روزه</div>
                  <div class="jb-res-card-desc">نگه داری سفارش برای سه روز </div>
                </div>
              </div>
            </label>

            <label
              v-if="!selectedReservation || !selectedReservation.order_id"
              class="jb-res-card"
              :class="{ 'is-selected': reservationType === 'seven_days' }"
            >
              <input
                v-model="reservationType"
                type="radio"
                value="seven_days"
                name="reservation_type"
                class="jb-pay-radio-input"
              />
              <div class="jb-res-card-inner">
                <div class="jb-pay-radio" :class="{ 'is-checked': reservationType === 'seven_days' }">
                  <div v-if="reservationType === 'seven_days'" class="jb-pay-radio-dot"></div>
                </div>
                <div class="jb-res-card-info">
                  <div class="jb-res-card-title">رزرو ۷ روزه</div>
                  <div class="jb-res-card-desc">نگه داری سفارش برای ۷ روز</div>
                </div>
              </div>
            </label>
          </div> -->

          <!-- اطلاعات آدرس -->
          <div v-if="isFetchingSummary" class="jb-summary-box jb-summary-loading">
            <SharedLoadingSpinner size="sm" />
          </div>

          <div v-else-if="selectedAddress" class="jb-summary-box">
            <p class="jb-summary-box-title">
              <span class="jb-title-dot"></span>
              مشخصات تحویل گیرنده
            </p>

            <div class="jb-summary-line">
              <span class="jb-summary-label">نام:</span>
              <span class="jb-summary-value jb-summary-value-strong">
                {{ selectedAddress.receiver_name }}
              </span>
            </div>

            <div class="jb-summary-line">
              <span class="jb-summary-label">آدرس:</span>
              <span class="jb-summary-value">{{ selectedAddress.address_line }}</span>
            </div>

            <div class="jb-summary-line">
              <span class="jb-summary-label">شماره تماس:</span>
              <span class="jb-summary-value" dir="ltr">{{ selectedAddress.phone }}</span>
            </div>
          </div>

          <div v-else class="jb-summary-box jb-summary-empty">
            <span>اطلاعات آدرس یافت نشد</span>
          </div>

          <!-- خلاصه مالی -->
          <div class="jb-summary-box">
            <p class="jb-summary-box-title">
              <span class="jb-title-dot"></span>
              خلاصه سفارش
            </p>

            <div class="jb-summary-line">
              <span class="jb-summary-label">مجموع قیمت کالاها:</span>
              <span class="jb-summary-value">
                {{ subtotal.toLocaleString("fa-IR") }}
                <span class="jb-summary-unit">تومان</span>
              </span>
            </div>

            <div class="jb-summary-line" :class="discount > 0 ? 'jb-line-discount' : ''">
              <span class="jb-summary-label">تخفیف محصولات:</span>
              <span class="jb-summary-value">
                {{ discount > 0 ? "-" : "" }}
                {{ discount.toLocaleString("fa-IR") }}
                <span class="jb-summary-unit">تومان</span>
              </span>
            </div>

            <div v-if="apiCouponDiscount > 0" class="jb-summary-line jb-line-discount">
              <span class="jb-summary-label">
                تخفیف کوپن
                <span class="jb-summary-coupon">({{ couponCode }})</span>:
              </span>
              <span class="jb-summary-value">
                - {{ apiCouponDiscount.toLocaleString("fa-IR") }}
                <span class="jb-summary-unit">تومان</span>
              </span>
            </div>

            <div class="jb-summary-line">
              <span class="jb-summary-label">هزینه ارسال:</span>
              <span class="jb-summary-value" :class="shippingCost === 0 ? 'jb-line-free' : ''">
                {{
                  shippingCost === 0
                    ? "رایگان"
                    : `${shippingCost.toLocaleString("fa-IR")} تومان`
                }}
              </span>
            </div>

            <div class="jb-summary-divider"></div>

            <div class="jb-summary-line jb-line-total">
              <span class="jb-summary-label-total">پرداخت نهایی:</span>
              <span class="jb-summary-value-total">
                {{ total.toLocaleString("fa-IR") }}
                <span class="jb-summary-unit">تومان</span>
              </span>
            </div>
          </div>

          <!-- دکمه پرداخت -->
          <button v-if="paymentMethod !== 'card_transfer'" @click="handlePayment" :disabled="isProcessing"
            class="jb-btn-linear jb-payment-btn">
            <span v-if="isProcessing" class="jb-payment-btn-inner">
              <svg class="jb-spinner" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
                <path class="opacity-75" fill="currentColor"
                  d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
              </svg>
              در حال پردازش...
            </span>
            <span v-else class="jb-payment-btn-inner">
              <span>تکمیل و پرداخت</span>
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
              </svg>
            </span>
          </button>

          <button v-else @click="handleCardTransferOrder" :disabled="isProcessing"
            class="jb-btn-linear jb-payment-btn jb-payment-btn-yellow">
            <span v-if="isProcessing" class="jb-payment-btn-inner">
              <svg class="jb-spinner" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
                <path class="opacity-75" fill="currentColor"
                  d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
              </svg>
              در حال ثبت سفارش...
            </span>
            <span v-else class="jb-payment-btn-inner">
              <span>ثبت سفارش کارت به کارت</span>
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
              </svg>
            </span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, computed, onMounted } from "vue";
import { useApi } from "@/composables/api/useApi";
import { toast } from "vue3-toastify";

definePageMeta({
  middleware: "auth",
});

const { getCheckoutSummary } = useApi();
const { $api } = useNuxtApp();

const paymentMethod = ref("online"); // 'online' یا 'wallet'
const selected = ref(2); // برای انتخاب درگاه پرداخت
const cartData = ref(null);
const couponCode = ref(""); // کد تخفیف
const isProcessing = ref(false); // برای جلوگیری از کلیک مکرر
const checkoutSummary = ref(null); // داده‌های checkout summary از API
const isFetchingSummary = ref(false); // برای نمایش loading
const shippingInfo = ref(null); // اطلاعات shipping از API
const reservationStorageKey = "selected-reservation";
const selectedReservation = ref(null);
let user_note = ref('');

const bankCardNumber = ref("6104-3310-4719-3157");
const bankAccountNumber = ref("4006051640119");
const bankShebaNumber = ref("IR190120010000002917515931");
const bankAccountOwner = ref("جاوید رحیمی نژاد - بانک ملت");
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

// تابع ثبت سفارش کارت به کارت
async function handleCardTransferOrder() {
  if (isProcessing.value) return;

  if (!selectedAddress.value) {
    toast.error("آدرس تحویل یافت نشد");
    return;
  }

  if (!selectedShippingMethod.value) {
    toast.error("روش ارسال یافت نشد");
    return;
  }

  isProcessing.value = true;

  try {
    const payload = {
      address_id: selectedAddress.value.id,
      shipping_id: selectedShippingMethod.value.id,
      payment_method: 'card_transfer',
      coupon_code: couponCode.value.trim() || "",
    };

    const response = await $api.post("/front/card-transfer/order", payload);

    toast.success(response.data?.message || "سفارش با موفقیت ثبت شد");

    localStorage.removeItem("cart-summary");

    // هدایت به صفحه جزئیات سفارش
    navigateTo(`/panel/order/${response.data.order_id}`);

  } catch (err) {
    console.error("❌ Card transfer order error:", err);
    toast.error(err.response?.data?.message || "خطا در ثبت سفارش");
  } finally {
    isProcessing.value = false;
  }
}
const getSelectedReservation = () => {
  if (!process.client) {
    return null;
  }

  const saved = localStorage.getItem(reservationStorageKey);

  if (!saved) {
    return null;
  }

  try {
    return JSON.parse(saved);
  } catch (error) {
    console.error("Invalid reservation storage:", error);

    localStorage.removeItem(reservationStorageKey);

    return null;
  }
};
onMounted(async () => {
  selectedReservation.value = getSelectedReservation();
  // بارگذاری اطلاعات cart از localStorage
  const savedCart = localStorage.getItem("cart-summary");
  if (savedCart) {
    try {
      cartData.value = JSON.parse(savedCart);

      // بارگذاری کد تخفیف از localStorage
      if (cartData.value.couponCode) {
        couponCode.value = cartData.value.couponCode;
      }

    } catch (err) {
      console.error("❌ Failed to parse cart data:", err);
    }
  }

  // بارگذاری اطلاعات shipping از localStorage
  const savedShipping = localStorage.getItem("shipping-info");
  if (savedShipping) {
    try {
      shippingInfo.value = JSON.parse(savedShipping);

      // بارگذاری checkout summary با استفاده از address_id و shipping_method_id
      if (
        shippingInfo.value?.address_id &&
        shippingInfo.value?.shipping_method_id
      ) {
        await fetchCheckoutSummary();
      } else {
        toast.error(
          "اطلاعات آدرس و روش ارسال یافت نشد. لطفاً به مرحله قبل بازگردید.");
      }
    } catch (err) {
      console.error("❌ Failed to parse shipping data:", err);
      toast.error("خطا در بارگذاری اطلاعات. لطفاً به مرحله قبل بازگردید.");
    }
  } else {
    toast.error(
      "اطلاعات آدرس و روش ارسال یافت نشد. لطفاً به مرحله قبل بازگردید.");
  }
});
const reservationType = ref("none");
// تابع برای fetch کردن checkout summary
const fetchCheckoutSummary = async () => {
  if (
    !shippingInfo.value?.address_id ||
    !shippingInfo.value?.shipping_method_id
  ) {
    return;
  }

  isFetchingSummary.value = true;

  try {
    const response = await getCheckoutSummary({
      address_id: shippingInfo.value.address_id,
      shipping_id: shippingInfo.value.shipping_method_id,
      coupon_code: couponCode.value || "",
      reservation_order_id:
        selectedReservation.value?.order_id || null,
      // payment_method
      // reservation_type
      // parent_order_id
      // gateway
    });

    checkoutSummary.value = response.data || response;

    // اگر کوپن از API برگشت، کد تخفیف رو آپدیت کن
    if (checkoutSummary.value?.coupon) {
      couponCode.value = checkoutSummary.value.coupon.code || "";
    }
  } catch (err) {
    console.error("❌ Failed to fetch checkout summary:", err);
  } finally {
    isFetchingSummary.value = false;
  }
};

// محاسبه قیمت‌ها - از API اگر موجود باشه، وگرنه از localStorage
const subtotal = computed(() => {
  if (checkoutSummary.value?.summary?.subtotal !== undefined) {
    return checkoutSummary.value.summary.subtotal;
  }
  return cartData.value?.totalPrice || 0;
});

const shippingCost = computed(() => {
  if (checkoutSummary.value?.summary?.shipping_cost !== undefined) {
    return checkoutSummary.value.summary.shipping_cost;
  }
  return 0;
});

const discount = computed(() => {
  if (checkoutSummary.value?.summary?.product_discount !== undefined) {
    return checkoutSummary.value.summary.product_discount;
  }
  return cartData.value?.discount || 0;
});

const totalDiscount = computed(() => {
  if (checkoutSummary.value?.summary?.coupon_discount !== undefined) {
    return discount.value + checkoutSummary.value.summary.coupon_discount;
  }
  return discount.value;
});

// تخفیف کوپن از API
const apiCouponDiscount = computed(() => {
  if (checkoutSummary.value?.summary?.coupon_discount !== undefined) {
    return checkoutSummary.value.summary.coupon_discount;
  }
  return 0;
});

const total = computed(() => {
  if (checkoutSummary.value?.summary?.payable_amount !== undefined) {
    return checkoutSummary.value.summary.payable_amount;
  }
  return subtotal.value + shippingCost.value - totalDiscount.value;
});

// آدرس و روش ارسال انتخاب شده - مستقیماً از checkout summary
const selectedAddress = computed(() => {
  return checkoutSummary.value?.address || null;
});

const selectedShippingMethod = computed(() => {
  return checkoutSummary.value?.shipping_method || null;
});
const wallet = computed(() => {
  return checkoutSummary.value?.wallet || null;
});

// وقتی می‌خواهید درگاه را ارسال کنید
const getSelectedGateway = () => {
  const gatewayMap = {
    1: 'parsian',
    2: 'zibal',
    3: 'torobpay',
  };
  return gatewayMap[selected.value] || 'parsian';
};
// تابع ثبت سفارش
async function handlePayment() {
  const { $api } = useNuxtApp();

  if (isProcessing.value) return;

  // اعتبارسنجی
  if (!selectedAddress.value) {
    toast.error("آدرس تحویل یافت نشد");
    return;
  }

  if (!selectedShippingMethod.value) {
    toast.error("روش ارسال یافت نشد");
    return;
  }

  isProcessing.value = true;

  try {
    const payload = {
      address_id: selectedAddress.value.id,
      gateway: getSelectedGateway(),
      user_note: user_note.value,
      shipping_id: selectedShippingMethod.value.id,
      payment_method: paymentMethod.value === "wallet" ? "wallet" : "online",
      coupon_code: couponCode.value.trim() || "", // همیشه ارسال میشه، حتی اگه خالی باشه
    };

    if (reservationType.value && reservationType.value !== "none") {
      payload.reservation_type = reservationType.value;
    }

    if (selectedReservation.value?.order_id) {
      payload.parent_order_id = selectedReservation.value.order_id;
    }
    const response = await $api.post("/front/order", payload);
    if (process.client) {
      localStorage.removeItem("selected-reservation");
    }
    localStorage.removeItem("cart-summary");
    if (response.data?.gateway_url) {
      toast.error(response.data?.message || "سفارش شما با موفقیت ثبت شد , در حال انتقال به درگاه");
      window.location.href = response.data.gateway_url;
    } else {
      toast.error(response.data?.message || "سفارش شما با موفقیت ثبت شد ");
      navigateTo("/panel/orders");
    }
  } catch (err) {
    console.error("❌ Order error:", err);
    toast.error(err.response?.data?.message || "خطا در ثبت سفارش");
  } finally {
    isProcessing.value = false;
  }
}
useSeoMeta({
  title: 'نحوه پرداخت سفارش | فروشگاه دکتر اسماعیلی',
  description: 'مشاهده و مدیریت نحوه پرداخت سفارش در فروشگاه فروشگاه دکتر اسماعیلی.',
  robots: 'noindex, nofollow', // 🔴 مهم‌ترین خط
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/payment' }],
})
</script>
<style scoped>
/* =========================================================
   صفحه پرداخت - تم پودری
   ========================================================= */

.jb-payment-page {
  padding-bottom: 40px;
}

.jb-payment-card {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 26px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(214, 51, 132, 0.1);
}

/* ===== هدر ===== */
.jb-payment-header {
  margin-bottom: 4px;
}

.jb-payment-title {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  font-size: 1.1rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-payment-title-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 13px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
}

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

/* ===== گرید اصلی ===== */
.jb-payment-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 24px;
}

@media (min-width: 1024px) {
  .jb-payment-grid {
    grid-template-columns: 1fr 1fr;
    gap: 32px;
  }
}

.jb-payment-col {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* ===== تیتر بخش‌ها ===== */
.jb-payment-section {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.jb-section-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 0.95rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-title-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
  flex-shrink: 0;
}

/* =========================================================
   روش‌های پرداخت
   ========================================================= */
.jb-payment-methods {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.jb-pay-method {
  position: relative;
  display: block;
  padding: 16px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.jb-pay-method:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.14);
  transform: translateY(-2px);
}

.jb-pay-method.is-selected {
  border-color: var(--jb-fuchsia);
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.22);
}

.jb-pay-method-inner {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.jb-pay-radio-input {
  position: absolute;
  opacity: 0;
  pointer-events: none;
}

.jb-pay-radio {
  flex-shrink: 0;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  border: 2.5px solid var(--jb-pink-300);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.25s ease;
  background: #fff;
  margin-top: 1px;
}

.jb-pay-radio.is-checked {
  border-color: var(--jb-fuchsia);
  background: rgba(214, 51, 132, 0.08);
}

.jb-pay-radio-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 2px 6px rgba(214, 51, 132, 0.4);
}

.jb-pay-method-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.jb-pay-method-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.9rem;
  font-weight: 800;
  color: var(--jb-text);
}

.jb-pay-method-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 26px;
  height: 26px;
  border-radius: 8px;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  color: var(--jb-fuchsia);
}

.jb-pay-method-desc {
  font-size: 0.78rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  line-height: 1.7;
}

.jb-pay-wallet-amount {
  color: var(--jb-fuchsia);
  font-weight: 900;
}

/* =========================================================
   درگاه‌های پرداخت
   ========================================================= */
.jb-gateway-grid {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.jb-gateway-card {
  position: relative;
  width: 140px;
  height: 100px;
  padding: 14px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-family: inherit;
}

.jb-gateway-card:hover {
  border-color: var(--jb-pink-400);
  transform: translateY(-2px) scale(1.03);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.14);
}

.jb-gateway-card.is-selected {
  border-color: transparent;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.4);
}

.jb-gateway-logo {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 10px;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  color: var(--jb-fuchsia);
  transition: all 0.3s ease;
}

.jb-gateway-card.is-selected .jb-gateway-logo {
  background: rgba(255, 255, 255, 0.2);
  color: #fff;
}

.jb-gateway-name {
  font-size: 1rem;
  font-weight: bolder;
  text-align: center;
  color: var(--jb-text);
}
.jb-gateway-sub-name {
  font-size: 0.78rem;
  font-weight: 800;
  text-align: center;
  color: var(--jb-text);
}
.jb-gateway-card.is-selected .jb-gateway-name ,
.jb-gateway-card.is-selected .jb-gateway-sub-name {
  color: #fff;
}

.jb-gateway-check {
  position: absolute;
  top: 8px;
  left: 8px;
  width: 22px;
  height: 22px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: #fff;
  color: var(--jb-fuchsia);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  animation: jbCheckIn 0.3s ease;
}

@keyframes jbCheckIn {
  0% {
    transform: scale(0);
  }

  70% {
    transform: scale(1.15);
  }

  100% {
    transform: scale(1);
  }
}

/* =========================================================
   پیام اطلاع‌رسانی
   ========================================================= */
.jb-info-box {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  padding: 14px 16px;
  background: linear-gradient(135deg, #fff8e1, #fff3c4);
  border: 1.5px solid #ffd88a;
  border-radius: 14px;
  color: #7a4a00;
  font-size: 0.8rem;
  font-weight: 600;
  line-height: 1.8;
}

.jb-info-box-icon {
  flex-shrink: 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 26px;
  height: 26px;
  border-radius: 8px;
  background: linear-gradient(135deg, #ffb84d, #ffa31a);
  color: #fff;
  margin-top: 1px;
}

/* =========================================================
   فرم توضیحات
   ========================================================= */
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

.jb-form-label-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 2px rgba(214, 51, 132, 0.18);
}

.jb-form-textarea {
  width: 100%;
  padding: 12px 14px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 14px;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--jb-text);
  outline: none;
  transition: all 0.25s ease;
  font-family: inherit;
  resize: none;
  line-height: 1.9;
}

.jb-form-textarea::placeholder {
  color: var(--jb-text-muted);
  font-weight: 500;
}

.jb-form-textarea:focus {
  border-color: var(--jb-fuchsia);
  box-shadow: 0 0 0 4px rgba(214, 51, 132, 0.15);
}

/* =========================================================
   کارت به کارت
   ========================================================= */
.jb-card-transfer-box {
  padding: 18px;
  background: linear-gradient(135deg, #fffbea, #fff3c4);
  border: 2px dashed #ffd88a;
  border-radius: 18px;
}

.jb-card-transfer-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 14px;
  padding-bottom: 12px;
  border-bottom: 1.5px dashed #ffd88a;
}

.jb-card-transfer-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 30px;
  height: 30px;
  border-radius: 9px;
  background: linear-gradient(135deg, #ffb84d, #ffa31a);
  color: #fff;
  box-shadow: 0 6px 16px rgba(255, 163, 26, 0.35);
}

.jb-card-transfer-title {
  font-size: 0.9rem;
  font-weight: 900;
  color: #7a4a00;
}

.jb-card-transfer-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  list-style: none;
  padding: 0;
  margin: 0;
}

.jb-card-transfer-item {
  font-size: 0.8rem;
  color: #7a4a00;
  font-weight: 600;
  line-height: 1.8;
}

.jb-card-transfer-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding: 10px 14px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 12px;
  border: 1px solid rgba(255, 216, 138, 0.6);
}

.jb-card-transfer-info-content {
  display: flex;
  flex-direction: column;
  gap: 3px;
  min-width: 0;
}

.jb-card-transfer-label {
  font-size: 0.7rem;
  color: #b07500;
  font-weight: 700;
}

.jb-card-transfer-value {
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

.jb-card-transfer-owner {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  background: rgba(255, 255, 255, 0.6);
  border-radius: 12px;
  font-size: 0.82rem;
}

.jb-card-transfer-owner-name {
  font-weight: 900;
  color: #7a4a00;
}

.jb-card-transfer-warning {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding: 12px 14px;
  background: linear-gradient(135deg, #ffe0e8, #ffc2d6);
  border: 1.5px solid #ff9db8;
  border-radius: 12px;
  color: #c9184a;
  font-size: 0.78rem;
  font-weight: 800;
  line-height: 1.8;
}

.jb-card-transfer-warning svg {
  flex-shrink: 0;
  margin-top: 2px;
}

/* =========================================================
   دکمه مرحله قبل
   ========================================================= */
.jb-shipping-btn-secondary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
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

.jb-shipping-btn-secondary:hover {
  background: var(--jb-pink-100);
  border-color: var(--jb-fuchsia);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.2);
}

.jb-payment-back-btn {
  width: 100%;
  margin-top: auto;
}

/* =========================================================
   رزرو (نوع سفارش)
   ========================================================= */
.jb-reservation-grid {
  display: grid;
  gap: 10px;
}

.jb-reservation-grid.is-three {
  grid-template-columns: 1fr;
}

@media (min-width: 768px) {
  .jb-reservation-grid.is-three {
    grid-template-columns: repeat(3, 1fr);
  }
}

.jb-reservation-grid.is-single {
  grid-template-columns: 1fr;
}

.jb-res-card {
  position: relative;
  display: block;
  padding: 14px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.jb-res-card:hover {
  border-color: var(--jb-pink-400);
  transform: translateY(-2px);
}

.jb-res-card.is-selected {
  border-color: var(--jb-fuchsia);
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.18);
}

.jb-res-card-inner {
  display: flex;
  align-items: flex-start;
  gap: 10px;
}

.jb-res-card-info {
  flex: 1;
  min-width: 0;
}

.jb-res-card-title {
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
}

.jb-res-card-desc {
  font-size: 0.72rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  margin-top: 3px;
  line-height: 1.7;
}

/* =========================================================
   خلاصه سفارش
   ========================================================= */
.jb-summary-box {
  padding: 18px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px solid var(--jb-pink-200);
  border-radius: 20px;
}

.jb-summary-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 90px;
}

.jb-summary-empty {
  text-align: center;
  font-size: 0.82rem;
  color: var(--jb-text-muted);
  font-weight: 600;
}

.jb-summary-box-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.92rem;
  font-weight: 900;
  color: var(--jb-text);
  margin-bottom: 14px;
}

.jb-summary-line {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  font-size: 0.82rem;
  padding: 6px 0;
}

.jb-summary-label {
  color: var(--jb-text-soft);
  font-weight: 700;
}

.jb-summary-value {
  color: var(--jb-text);
  font-weight: 800;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-summary-value-strong {
  color: var(--jb-fuchsia);
  font-weight: 900;
}

.jb-summary-unit {
  font-size: 0.65rem;
  color: var(--jb-text-muted);
  font-weight: 600;
}

.jb-summary-coupon {
  color: var(--jb-fuchsia);
  font-weight: 800;
}

.jb-line-discount .jb-summary-value {
  color: #16a34a;
}

.jb-line-free .jb-summary-value {
  color: #16a34a;
}

.jb-summary-divider {
  height: 0;
  border-top: 1.5px dashed var(--jb-pink-300);
  margin: 8px 0;
}

.jb-line-total {
  padding-top: 6px;
}

.jb-summary-label-total {
  font-size: 0.92rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-summary-value-total {
  font-size: 1.1rem;
  font-weight: 900;
  color: var(--jb-pink-700);
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

/* =========================================================
   دکمه پرداخت
   ========================================================= */
.jb-payment-btn {
  width: 100%;
  justify-content: center;
  padding: 14px 24px !important;
  font-size: 0.92rem !important;
}

.jb-payment-btn-inner {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.jb-payment-btn-yellow {
  background: linear-gradient(135deg, #ffb84d, #ffa31a) !important;
  box-shadow: 0 8px 22px rgba(255, 163, 26, 0.4) !important;
}

.jb-payment-btn-yellow:hover {
  background: linear-gradient(135deg, #ffa31a, #cc7a00) !important;
  box-shadow: 0 14px 32px rgba(255, 163, 26, 0.55) !important;
}

.jb-payment-btn:disabled {
  opacity: 0.7;
  cursor: wait;
  transform: none !important;
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
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-payment-card {
    padding: 16px;
    border-radius: 20px;
  }

  .jb-payment-title {
    font-size: 0.95rem;
  }

  .jb-payment-title-icon {
    width: 34px;
    height: 34px;
  }

  .jb-pay-method {
    padding: 14px;
  }

  .jb-gateway-card {
    width: 100%;
    flex: 1;
    height: 118px;
  }

  .jb-card-transfer-info {
    flex-direction: column;
    align-items: stretch;
  }

  .jb-copy-btn {
    align-self: flex-end;
  }

  .jb-reservation-grid.is-three {
    grid-template-columns: 1fr;
  }
}
</style>