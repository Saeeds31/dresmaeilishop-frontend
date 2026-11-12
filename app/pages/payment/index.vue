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
</script><template>
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
            <h3 class="jb-section-title">انتخاب روش پرداخت</h3>

            <div class="jb-payment-methods">
              <!-- پرداخت اینترنتی -->
              <label class="jb-pay-method" :class="{ 'is-selected': paymentMethod === 'online' }">
                <input type="radio" name="paymentMethod" value="online" v-model="paymentMethod"
                  class="jb-pay-radio-input" />
                <div class="jb-pay-method-inner">
                  <div class="jb-pay-radio" :class="{ 'is-checked': paymentMethod === 'online' }"></div>
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
                  <div class="jb-pay-radio" :class="{ 'is-checked': paymentMethod === 'wallet' }"></div>
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
            </div>
          </div>

          <!-- ============================================================
               انتخاب درگاه (فقط اینترنتی)
          ============================================================ -->
          <div v-if="paymentMethod === 'online'" class="jb-payment-section">
            <h3 class="jb-section-title">انتخاب درگاه پرداخت</h3>

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
                  <img width="48" height="48" class="rounded-full" src="/images/torobpaylogo.webp"
                    alt="پرداخت چهار قسطه با ترب پی فروشگاه دکتر اسماعیلی" />
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
              <label class="jb-form-label">توضیحات سفارش</label>
              <textarea v-model="user_note" class="jb-form-textarea"
                placeholder="توضیحات خود را در خصوص سفارش وارد کنید..." rows="3"></textarea>
            </div>
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
          <!-- اطلاعات آدرس -->
          <div v-if="isFetchingSummary" class="jb-summary-box jb-summary-loading">
            <SharedLoadingSpinner size="sm" />
          </div>

          <div v-else-if="selectedAddress" class="jb-summary-box">
            <p class="jb-summary-box-title">مشخصات تحویل گیرنده</p>

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
            <p class="jb-summary-box-title">خلاصه سفارش</p>

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
                    ? "0"
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
            class="jb-payment-btn">
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
            class="jb-payment-btn jb-payment-btn-yellow">
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


<style scoped>
/* =========================================================
   صفحه پرداخت - مینیمال سبز
   ========================================================= */

.jb-payment-page {
  padding-bottom: 40px;
}

.jb-payment-card {
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

/* ===== هدر ===== */
.jb-payment-header {
  margin-bottom: 4px;
}

.jb-payment-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-payment-title-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  border-radius: 6px;
  background: #0e5a3a;
  color: #fff;
}

.jb-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 16px 0 20px;
}

/* ===== گرید اصلی ===== */
.jb-payment-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 20px;
}

@media (min-width: 1024px) {
  .jb-payment-grid {
    grid-template-columns: 1fr 1fr;
    gap: 28px;
  }
}

.jb-payment-col {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* ===== تیتر بخش‌ها ===== */
.jb-payment-section {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.jb-section-title {
  font-size: 0.9rem;
  font-weight: 800;
  color: #1a202c;
}

/* =========================================================
   روش‌های پرداخت
   ========================================================= */
.jb-payment-methods {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.jb-pay-method {
  position: relative;
  display: block;
  padding: 14px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-pay-method:hover {
  border-color: rgba(14, 90, 58, 0.25);
}

.jb-pay-method.is-selected {
  border-color: #0e5a3a;
  background: #f7fafc;
}

.jb-pay-method-inner {
  display: flex;
  align-items: flex-start;
  gap: 10px;
}

.jb-pay-radio-input {
  position: absolute;
  opacity: 0;
  pointer-events: none;
}

.jb-pay-radio {
  flex-shrink: 0;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 2px solid rgba(14, 90, 58, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  background: #fff;
  margin-top: 1px;
}

.jb-pay-radio.is-checked {
  border-color: #0e5a3a;
  background: #0e5a3a;
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
  gap: 6px;
  font-size: 0.85rem;
  font-weight: 700;
  color: #1a202c;
}

.jb-pay-method-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  border-radius: 6px;
  background: #f7fafc;
  color: #0e5a3a;
}

.jb-pay-method-desc {
  font-size: 0.75rem;
  color: #718096;
  font-weight: 500;
  line-height: 1.7;
}

.jb-pay-wallet-amount {
  color: #0e5a3a;
  font-weight: 700;
}

/* =========================================================
   درگاه‌های پرداخت
   ========================================================= */
.jb-gateway-grid {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.jb-gateway-card {
  position: relative;
  width: 130px;
  height: 90px;
  padding: 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 6px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-family: inherit;
}

.jb-gateway-card:hover {
  border-color: rgba(14, 90, 58, 0.25);
}

.jb-gateway-card.is-selected {
  border-color: #0e5a3a;
  background: #f7fafc;
}

.jb-gateway-logo {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 30px;
  height: 30px;
  border-radius: 6px;
  background: #f7fafc;
  color: #0e5a3a;
}

.jb-gateway-card.is-selected .jb-gateway-logo {
  background: #0e5a3a;
  color: #fff;
}

.jb-gateway-name {
  font-size: 0.9rem;
  font-weight: 700;
  text-align: center;
  color: #1a202c;
}

.jb-gateway-sub-name {
  font-size: 0.72rem;
  font-weight: 600;
  text-align: center;
  color: #718096;
}

.jb-gateway-card.is-selected .jb-gateway-name,
.jb-gateway-card.is-selected .jb-gateway-sub-name {
  color: #0e5a3a;
}

.jb-gateway-check {
  position: absolute;
  top: 6px;
  left: 6px;
  width: 18px;
  height: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: #0e5a3a;
  color: #fff;
}

/* =========================================================
   پیام اطلاع‌رسانی
   ========================================================= */
.jb-info-box {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding: 12px 14px;
  background: #fffbea;
  border: 1px solid #ffd88a;
  border-radius: 6px;
  color: #7a4a00;
  font-size: 0.78rem;
  font-weight: 500;
  line-height: 1.8;
}

.jb-info-box-icon {
  flex-shrink: 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  border-radius: 4px;
  background: #ffa31a;
  color: #fff;
  margin-top: 1px;
}

/* =========================================================
   فرم توضیحات
   ========================================================= */
.jb-form-field {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.jb-form-label {
  font-size: 0.82rem;
  font-weight: 700;
  color: #1a202c;
}

.jb-form-textarea {
  width: 100%;
  padding: 10px 12px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.12);
  border-radius: 6px;
  font-size: 0.82rem;
  font-weight: 600;
  color: #1a202c;
  outline: none;
  transition: all 0.2s ease;
  font-family: inherit;
  resize: none;
  line-height: 1.8;
}

.jb-form-textarea::placeholder {
  color: #a0aec0;
  font-weight: 500;
}

.jb-form-textarea:focus {
  border-color: #0e5a3a;
  box-shadow: 0 0 0 3px rgba(14, 90, 58, 0.1);
}

/* =========================================================
   دکمه مرحله قبل
   ========================================================= */
.jb-shipping-btn-secondary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 10px 20px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.15);
  border-radius: 6px;
  color: #0e5a3a;
  font-size: 0.85rem;
  font-weight: 700;
  font-family: inherit;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-shipping-btn-secondary:hover {
  background: #f7fafc;
  border-color: rgba(14, 90, 58, 0.3);
}

.jb-payment-back-btn {
  width: 100%;
  margin-top: auto;
}

/* =========================================================
   خلاصه سفارش
   ========================================================= */
.jb-summary-box {
  padding: 16px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 8px;
}

.jb-summary-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 80px;
}

.jb-summary-empty {
  text-align: center;
  font-size: 0.8rem;
  color: #718096;
  font-weight: 500;
}

.jb-summary-box-title {
  font-size: 0.88rem;
  font-weight: 800;
  color: #1a202c;
  margin-bottom: 12px;
}

.jb-summary-line {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  font-size: 0.8rem;
  padding: 5px 0;
}

.jb-summary-label {
  color: #4a5568;
  font-weight: 500;
}

.jb-summary-value {
  color: #1a202c;
  font-weight: 700;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-summary-value-strong {
  color: #0e5a3a;
  font-weight: 700;
}

.jb-summary-unit {
  font-size: 0.62rem;
  color: #a0aec0;
  font-weight: 500;
}

.jb-summary-coupon {
  color: #0e5a3a;
  font-weight: 600;
}

.jb-line-discount .jb-summary-value {
  color: #38a169;
}

.jb-line-free .jb-summary-value {
  color: #38a169;
}

.jb-summary-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.12);
  margin: 8px 0;
}

.jb-line-total {
  padding-top: 6px;
}

.jb-summary-label-total {
  font-size: 0.88rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-summary-value-total {
  font-size: 1.05rem;
  font-weight: 800;
  color: #0e5a3a;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

/* =========================================================
   دکمه پرداخت
   ========================================================= */
.jb-payment-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 11px 20px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.88rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-payment-btn:hover:not(:disabled) {
  background: #0b472e;
}

.jb-payment-btn-inner {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.jb-payment-btn-yellow {
  background: #d97706 !important;
}

.jb-payment-btn-yellow:hover:not(:disabled) {
  background: #b45309 !important;
}

.jb-payment-btn:disabled {
  opacity: 0.6;
  cursor: wait;
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
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-payment-card {
    padding: 14px;
    border-radius: 8px;
  }

  .jb-payment-title {
    font-size: 0.9rem;
  }

  .jb-payment-title-icon {
    width: 30px;
    height: 30px;
  }

  .jb-pay-method {
    padding: 12px;
  }

  .jb-gateway-card {
    width: 100%;
    flex: 1;
    height: 100px;
  }
}
</style>