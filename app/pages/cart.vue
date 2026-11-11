<template>
  <div class="container jb-cart-page">
    <SharedBreadCrumb :items="[{ label: 'سبد خرید', href: '/cart' }]" />

    <CartProgressSteps :current-step="1" />

    <!-- ============ لودینگ ============ -->
    <div v-if="showLoader" class="jb-cart-card jb-cart-state">
      <div class="jb-state-icon-wrap">
        <img class="jb-state-img" src="@/assets/images/shopping.png" alt="" />
      </div>
      <h2 class="jb-state-title">در حال دریافت اطلاعات سبد خرید</h2>
      <SharedLoadingSpinner size="lg" />
    </div>

    <template v-else>
      <!-- ============ سبد خالی ============ -->
      <div v-if="cartItems.length === 0" class="jb-cart-card jb-cart-state">
        <div class="jb-state-icon-wrap jb-state-icon-empty">
          <svg class="w-14 h-14" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
              d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
          </svg>
        </div>
        <h2 class="jb-state-title">سبد خرید شما خالی است</h2>
        <p class="jb-state-desc">هنوز محصولی به سبد خرید اضافه نکرده‌اید</p>
        <NuxtLink to="/products" class="jb-btn-linear mt-4">
          <span>مشاهده محصولات</span>
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
          </svg>
        </NuxtLink>
      </div>

      <!-- ============ سبد خرید ============ -->
      <div v-else class="jb-cart-card">
        <!-- جدول (دسکتاپ) / کارت (موبایل) -->
        <div class="jb-cart-list">
          <!-- هدر (فقط دسکتاپ) -->
          <div class="jb-cart-list-header">
            <span class="text-right">سبد خرید شما</span>
            <span>قیمت</span>
            <span>تخفیف</span>
            <span>تعداد</span>
            <span>قیمت نهایی</span>
            <span>حذف</span>
          </div>

          <!-- آیتم‌ها -->
          <div v-for="item in cartItems" :key="item.id || item.variant_id" class="jb-cart-row">
            <!-- تصویر + عنوان -->
            <div class="jb-cart-row-product">
              <div class="jb-cart-row-img-wrap">
                <img v-if="item.image" :src="resolveImage(item.image)" :alt="item.title" class="jb-cart-row-img" />
                <span v-else class="jb-cart-row-img-placeholder">📦</span>
              </div>

              <div class="jb-cart-row-info">
                <nuxt-link :to="`/product/${item.product_id}`" class="jb-cart-row-title">
                  {{ item.title }}
                </nuxt-link>

                <div v-if="item.size || item.color" class="jb-cart-row-attrs">
                  <span v-if="item.size" class="jb-cart-attr-chip">
                    <span class="jb-cart-attr-name">سایز:</span>
                    <span class="jb-cart-attr-value">{{ item.size }}</span>
                  </span>
                  <span v-if="item.color" class="jb-cart-attr-chip">
                    <span class="jb-cart-attr-name">رنگ:</span>
                    <span class="jb-cart-attr-value">{{ item.color }}</span>
                  </span>
                </div>

                <p v-if="item.alert_message" class="jb-cart-alert">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
                      d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
                  </svg>
                  {{ item.alert_message }}
                </p>
              </div>
            </div>

            <!-- قیمت اصلی -->
            <div class="jb-cart-row-cell">
              <span class="jb-cell-label">قیمت اصلی:</span>
              <span class="jb-cell-value">
                {{
                  (
                    item.line_original_total ||
                    item.price * item.quantity
                  ).toLocaleString("fa-IR")
                }}
                <span class="jb-cell-unit">تومان</span>
              </span>
            </div>

            <!-- تخفیف -->
            <div class="jb-cart-row-cell jb-cell-discount">
              <span class="jb-cell-label">تخفیف:</span>
              <span class="jb-cell-value">
                {{
                  (
                    item.line_discount ||
                    item.price * item.quantity
                  ).toLocaleString("fa-IR")
                }}
                <span class="jb-cell-unit">تومان</span>
              </span>
            </div>

            <!-- تعداد -->
            <div class="jb-cart-row-cell">
              <span class="jb-cell-label">تعداد:</span>
              <div class="jb-qty">
                <button @click="increaseQuantity(item.id || item.variant_id)" class="jb-qty-btn" aria-label="افزایش">
                  <Plus :size="14" />
                </button>
                <span class="jb-qty-value">{{ item.quantity }}</span>
                <button @click="decreaseQuantity(item.id || item.variant_id)" class="jb-qty-btn" aria-label="کاهش">
                  <Minus :size="14" />
                </button>
              </div>
            </div>

            <!-- قیمت نهایی -->
            <div class="jb-cart-row-cell jb-cell-final">
              <span class="jb-cell-label">قیمت نهایی:</span>
              <span class="jb-cell-value-final">
                {{
                  (
                    item.line_final_total ||
                    item.price * item.quantity
                  ).toLocaleString("fa-IR")
                }}
                <span class="jb-cell-unit">تومان</span>
              </span>
            </div>

            <!-- حذف -->
            <div class="jb-cart-row-cell jb-cell-remove">
              <span class="jb-cell-label">حذف آیتم:</span>
              <button @click="removeItem(item.id || item.variant_id)" class="jb-cart-remove-btn" aria-label="حذف">
                <Trash2 :size="18" />
              </button>
            </div>
          </div>
        </div>

        <!-- ============ جمع‌بندی + کوپن ============ -->
        <div class="jb-cart-summary-box">
          <!-- ستون راست: کوپن -->
          <form @submit="applyCoupon" class="jb-coupon-form">
            <div class="jb-coupon-title">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a2 2 0 01-2.828 0l-7-7A1.994 1.994 0 013 12V7a4 4 0 014-4z" />
              </svg>
              <span>کد تخفیف دارید؟</span>
            </div>

            <div class="jb-coupon-input-row">
              <input v-model="couponCode" class="jb-coupon-input" type="text"
                placeholder="کد تخفیف خود را وارد کنید ..." :disabled="isCheckingCoupon" />
              <button type="submit" class="jb-btn-linear jb-coupon-btn" :disabled="isCheckingCoupon">
                {{ isCheckingCoupon ? "بررسی..." : "اعمال" }}
              </button>
            </div>

            <p v-if="couponError" class="jb-coupon-error">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
                  d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
              {{ couponError }}
            </p>

            <p v-if="couponDiscount > 0" class="jb-coupon-success">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
              </svg>
              تخفیف کوپن: {{ couponDiscount.toLocaleString("fa-IR") }} تومان
            </p>
          </form>

          <!-- ستون چپ: جمع‌بندی -->
          <div class="jb-summary-list">
            <div class="jb-summary-row">
              <span class="jb-summary-label">مجموع قیمت:</span>
              <span class="jb-summary-value">
                {{ subtotal.toLocaleString("fa-IR") }}
                <span class="jb-summary-unit">تومان</span>
              </span>
            </div>

            <div class="jb-summary-row">
              <span class="jb-summary-label">تخفیف محصولات:</span>
              <span class="jb-summary-value jb-summary-value-discount">
                {{ discount.toLocaleString("fa-IR") }}
                <span class="jb-summary-unit">تومان</span>
              </span>
            </div>

            <div v-if="couponDiscount > 0" class="jb-summary-row">
              <span class="jb-summary-label">تخفیف کوپن:</span>
              <span class="jb-summary-value jb-summary-value-discount">
                {{ couponDiscount.toLocaleString("fa-IR") }}
                <span class="jb-summary-unit">تومان</span>
              </span>
            </div>

            <div class="jb-summary-divider"></div>

            <div class="jb-summary-row jb-summary-row-total">
              <span class="jb-summary-label-total">پرداخت:</span>
              <span class="jb-summary-value-total">
                {{ totalPrice.toLocaleString("fa-IR") }}
                <span class="jb-summary-unit">تومان</span>
              </span>
            </div>
          </div>
        </div>

        <!-- ============ رزروهای فعال ============ -->
        <div v-if="isLoggedIn && (isLoadingReservations || activeReservations.length > 0)" class="jb-reservations-box">
          <!-- Loading -->
          <div v-if="isLoadingReservations" class="jb-res-loading">
            <div class="jb-res-spinner"></div>
            <span>در حال بررسی رزروهای شما...</span>
          </div>

          <template v-else>
            <div class="jb-res-header">
              <div>
                <h3 class="jb-res-title">
                  <span class="jb-res-title-dot"></span>
                  رزروهای فعال شما
                </h3>
                <p class="jb-res-desc">
                  در صورت تمایل می‌توانید یکی از رزروهای خود را برای ادامه سفارش انتخاب کنید.
                </p>
              </div>

              <div class="jb-res-count">
                {{ activeReservations.length }} رزرو فعال
              </div>
            </div>

            <div class="jb-res-grid">
              <button v-for="reservation in activeReservations" :key="reservation.id" type="button"
                @click="selectReservation(reservation)" class="jb-res-card"
                :class="{ 'is-selected': selectedReservation === reservation.id }">
                <div class="jb-res-card-top">
                  <div class="jb-res-card-info">
                    <!-- رادیو -->
                    <div class="jb-res-radio">
                      <div v-if="selectedReservation === reservation.id" class="jb-res-radio-dot"></div>
                    </div>

                    <div>
                      <div class="jb-res-card-title">
                        سفارش #{{ reservation.id }}
                      </div>
                      <div class="jb-res-card-sub">
                        {{ reservation.reservation_type_label || "رزرو فعال" }}
                      </div>
                    </div>
                  </div>

                  <div class="jb-res-card-price">
                    <div class="jb-res-card-price-amount">
                      {{ Number(reservation.total || 0).toLocaleString("fa-IR") }}
                    </div>
                    <div class="jb-res-card-price-unit">تومان</div>
                  </div>
                </div>

                <div class="jb-res-card-bottom">
                  <span class="jb-res-card-bottom-label">مهلت رزرو</span>
                  <span class="jb-res-card-bottom-value">
                    {{ formatReservationDate(reservation.reserved_until) }}
                  </span>
                </div>
              </button>
            </div>

            <!-- بدون رزرو -->
            <div class="jb-res-no-reservation">
              <button type="button" @click="selectReservation(null)" class="jb-res-no-btn"
                :class="{ 'is-active': selectedReservation === null }">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7" />
                </svg>
                بدون استفاده از رزرو ادامه می‌دهم
              </button>
            </div>
          </template>
        </div>

        <!-- ============ دکمه ادامه ============ -->
        <div class="jb-cart-continue">
          <button @click="goToShipping" class="jb-btn-linear jb-cart-continue-btn">
            <span>ادامه فرایند</span>
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
            </svg>
          </button>
        </div>
      </div>
    </template>

    <AuthLoginModal v-if="loginModal" />
  </div>
</template>

<script setup>
import { Plus, Minus, Trash2 } from "lucide-vue-next";
import { useApi } from "@/composables/api/useApi";
import { computed, ref, onMounted, watch } from "vue";
import { useSettingStore } from "~/store/settings";
import { useAuth } from "~/composables/useAuth";
import { useCartStore } from "~/store/cart";

const route = useRoute();
const { $getImageUrl } = useNuxtApp();

const {
  getCart,
  increaseCartItem,
  decreaseCartItem,
  deleteCartItem,
  checkCoupon,
  getActiveReservations,
} = useApi();
const { loginModal } = useAuth();
const cartStore = useCartStore();
const settingStore = useSettingStore();

const isLoggedIn = computed(() => !!settingStore.user);
const activeReservations = ref([]);
const selectedReservation = ref(null);
const isLoadingReservations = ref(false);

const reservationStorageKey = "selected-reservation";
const token = process.client ? localStorage.getItem("auth-token") : null;
const cacheKey = token ? `cart-${token.substring(0, 10)}` : "cart-guest";
let loading = ref(true);
const showLoader = ref(true);

const { data: cartData, refresh: refreshCart } = await useAsyncData(
  cacheKey,
  async () => {
    loading.value = true;
    const token = process.client ? localStorage.getItem("auth-token") : null;
    if (!token) {
      cartStore.loadFromLocalStorage();
      loading.value = false;
      return {
        items: cartStore.items,
        summary: {
          subtotal: cartStore.subtotal,
          product_discount_total: cartStore.product_discount_total,
        },
      };
    }

    try {
      return await getCart();
    } catch (error) {
      console.error("❌ Failed to fetch cart:", error);
      return { items: [], summary: { subtotal: 0, product_discount_total: 0 } };
    } finally {
      loading.value = false;
    }
  }
);

watch(
  () => [cartData.value, loading.value, isLoadingReservations.value],
  ([data, isLoading, isResLoading]) => {
    if (!isLoading && !isResLoading && data) {
      setTimeout(() => {
        showLoader.value = false;
      }, 300);
    }
  },
  { immediate: true }
);

const formatReservationDate = (date) => {
  if (!date) return "-";
  try {
    return new Date(date).toLocaleDateString("fa-IR");
  } catch {
    return date;
  }
};

watch(() => cartData.value?.items, () => { }, { deep: true });

onMounted(async () => {
  await loadActiveReservations();
  if (process.client) {
    localStorage.removeItem("selected-reservation");
    const token = process.client ? localStorage.getItem("auth-token") : null;
    if (!token) {
      setTimeout(() => {
        refreshCart();
      }, 500);
    }
  }
  showLoader.value = false;
});

const loadActiveReservations = async () => {
  if (!isLoggedIn.value) {
    activeReservations.value = [];
    selectedReservation.value = null;
    if (process.client) {
      localStorage.removeItem(reservationStorageKey);
    }
    return;
  }

  isLoadingReservations.value = true;

  try {
    const result = await getActiveReservations();

    if (result?.success) {
      activeReservations.value = result.data || [];

      if (activeReservations.value.length === 0) {
        selectedReservation.value = null;
        if (process.client) {
          localStorage.removeItem(reservationStorageKey);
        }
        return;
      }
    } else {
      activeReservations.value = [];
      selectedReservation.value = null;
      if (process.client) {
        localStorage.removeItem(reservationStorageKey);
      }
      return;
    }

    if (process.client) {
      const saved = localStorage.getItem(reservationStorageKey);
      if (saved) {
        try {
          const parsed = JSON.parse(saved);
          const exists = activeReservations.value.some(
            (reservation) => reservation.id === parsed.order_id
          );
          if (exists) {
            selectedReservation.value = parsed.order_id;
          } else {
            selectedReservation.value = null;
            localStorage.removeItem(reservationStorageKey);
          }
        } catch (error) {
          console.error("Invalid saved reservation:", error);
          selectedReservation.value = null;
          localStorage.removeItem(reservationStorageKey);
        }
      }
    }
  } catch (error) {
    console.error("❌ Failed to fetch active reservations:", error);
    activeReservations.value = [];
  } finally {
    isLoadingReservations.value = false;
  }
};

const selectReservation = (reservation) => {
  if (!reservation) {
    selectedReservation.value = null;
    if (process.client) {
      localStorage.removeItem(reservationStorageKey);
    }
    return;
  }

  selectedReservation.value = reservation.id;

  if (process.client) {
    localStorage.setItem(
      reservationStorageKey,
      JSON.stringify({
        order_id: reservation.id,
        address_id: reservation.address_id,
        shipping_id: reservation.shipping_id,
        reservation_type: reservation.reservation_type,
        reserved_until: reservation.reserved_until,
        total: reservation.total,
        selected_at: new Date().toISOString(),
      })
    );
  }
};

const couponCode = ref("");
const couponDiscount = ref(0);
const couponError = ref("");
const isCheckingCoupon = ref(false);

const broadcastCartUpdate = () => {
  if (typeof window === "undefined") return;
  window.dispatchEvent(new Event("cart-updated"));
};

const extractSizeColor = (item) => {
  let size = item?.size || null;
  let color = item?.color || null;

  const attrs = item?.variant?.attributes || [];

  if (Array.isArray(attrs)) {
    for (const a of attrs) {
      const name = String(a?.name || "").trim();
      const value = a?.value ?? null;
      if (!value) continue;

      if (!size && name.includes("سایز")) size = value;
      if (!color && name.includes("رنگ")) color = value;
    }
  }

  return { size, color };
};

const normalizeCartItem = (item) => {
  const { size, color } = extractSizeColor(item);

  const title =
    item?.title ??
    item?.name ??
    item?.product?.title ??
    item?.product?.name ??
    "";

  const image =
    item?.image ??
    item?.main_image ??
    item?.product?.main_image ??
    item?.variant?.image ??
    "";

  const priceRaw =
    item?.final_price ??
    item?.price ??
    item?.variant?.final_price ??
    item?.variant?.price ??
    item?.product?.final_price ??
    item?.product?.price ??
    item?.product_price ??
    item?.variant_price ??
    item?.unit_price ??
    0;

  const price = Number(priceRaw) || 0;

  return {
    ...item,
    title,
    image,
    price,
    price_final: price,
    size,
    color,
  };
};

const cartItems = computed(() => {
  const items = cartData.value?.items || [];
  const normalized = items.map(normalizeCartItem);
  return normalized;
});

watch(cartItems, () => { }, { deep: true, immediate: true });

const subtotal = computed(() => {
  return cartData.value?.summary?.subtotal || 0;
});

const discount = computed(() => {
  return cartData.value?.summary?.product_discount_total || 0;
});

const totalDiscount = computed(() => {
  return discount.value + couponDiscount.value;
});

const totalPrice = computed(() => {
  return subtotal.value - totalDiscount.value;
});

const resolveImage = (src) => {
  if (!src) return "";
  if (src.startsWith("http")) return src;
  return $getImageUrl(src);
};

const updateCartSummaryStorage = () => {
  if (typeof window === "undefined") return;

  const totalQuantity = cartItems.value.reduce(
    (sum, item) => sum + (Number(item.quantity) || 0),
    0
  );

  const cartSummary = {
    items: cartItems.value,
    totalQuantity,
    subtotal: subtotal.value,
    product_discount_total: discount.value,
    couponDiscount: couponDiscount.value,
    totalPrice: totalPrice.value,
    timestamp: new Date().toISOString(),
  };

  localStorage.setItem("cart-summary", JSON.stringify(cartSummary));
  broadcastCartUpdate();
};

watch(
  [cartItems, subtotal, discount, couponDiscount, totalPrice],
  () => {
    updateCartSummaryStorage();
  },
  { deep: true, immediate: true }
);

const increaseQuantity = async (id) => {
  const token = process.client ? localStorage.getItem("auth-token") : null;

  if (!token) {
    const item = cartStore.items.find((i) => i.id === id || i.variant_id === id);
    if (item) {
      cartStore.increaseQuantity(item.variant_id);
      await refreshCart();
      updateCartSummaryStorage();
    }
    return;
  }

  try {
    await increaseCartItem(id);
    await refreshCart();
    updateCartSummaryStorage();
  } catch (error) {
    alert("خطا در افزایش تعداد محصول");
  }
};

const decreaseQuantity = async (id) => {
  const token = process.client ? localStorage.getItem("auth-token") : null;

  if (!token) {
    const item = cartStore.items.find((i) => i.id === id || i.variant_id === id);
    if (item) {
      cartStore.decreaseQuantity(item.variant_id);
      await refreshCart();
      updateCartSummaryStorage();
    }
    return;
  }

  try {
    await decreaseCartItem(id);
    await refreshCart();
    updateCartSummaryStorage();
  } catch (error) {
    console.error("❌ Failed to decrease quantity:", error);
    alert("خطا در کاهش تعداد محصول");
  }
};

const removeItem = async (id) => {
  const token = process.client ? localStorage.getItem("auth-token") : null;

  if (!token) {
    const item = cartStore.items.find((i) => i.id === id || i.variant_id === id);
    if (item) {
      cartStore.removeItem(item.variant_id);
      await refreshCart();
      updateCartSummaryStorage();
    }
    return;
  }

  try {
    await deleteCartItem(id);
    await refreshCart();
    updateCartSummaryStorage();
  } catch (error) {
    console.error("❌ Failed to remove item:", error);
    alert("خطا در حذف محصول");
  }
};

const applyCoupon = async (event) => {
  event.preventDefault();

  if (!couponCode.value) {
    couponError.value = "لطفاً کد تخفیف را وارد کنید";
    return;
  }

  isCheckingCoupon.value = true;
  couponError.value = "";

  try {
    const result = await checkCoupon(couponCode.value, subtotal.value);

    if (result.success) {
      couponDiscount.value = result.discount || 0;
      alert("کد تخفیف با موفقیت اعمال شد");
    }
  } catch (error) {
    console.error("❌ Coupon check failed:", error);
    couponError.value = error.response?.data?.message || "کد تخفیف نامعتبر است";
    couponDiscount.value = 0;
  } finally {
    isCheckingCoupon.value = false;
  }
};

const goToShipping = () => {
  const totalQuantity = cartItems.value.reduce(
    (sum, item) => sum + (Number(item.quantity) || 0),
    0
  );

  const cartSummary = {
    items: cartItems.value,
    totalQuantity,
    subtotal: subtotal.value,
    discount: discount.value,
    couponDiscount: couponDiscount.value,
    couponCode: couponCode.value,
    totalPrice: totalPrice.value,
    timestamp: new Date().toISOString(),
  };

  localStorage.setItem("cart-summary", JSON.stringify(cartSummary));
  broadcastCartUpdate();

  const token = localStorage.getItem("auth-token");
  if (!token) {
    const loginModal = useState("loginModal");
    localStorage.setItem("lastPage", route.fullPath);
    loginModal.value = true;
  } else {
    navigateTo("/shipping");
  }
};

useSeoMeta({
  title: 'سبد خرید | فروشگاه دکتر اسماعیلی',
  description: 'مشاهده و مدیریت سبد خرید در فروشگاه فروشگاه دکتر اسماعیلی.',
  robots: 'noindex, nofollow', // 🔴 مهم‌ترین خط
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/cart' }],
})
</script>

<style scoped>
/* =========================================================
   صفحه سبد خرید - تم پودری
   ========================================================= */

.jb-cart-page {
  padding-bottom: 40px;
}

/* ===== کارت اصلی ===== */
.jb-cart-card {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 26px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(214, 51, 132, 0.1);
  margin-bottom: 24px;
}

/* ===== حالت خالی/لودینگ ===== */
.jb-cart-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 24px;
  text-align: center;
  gap: 14px;
}

.jb-state-icon-wrap {
  margin-bottom: 8px;
}

.jb-state-img {
  width: 140px;
  height: auto;
  opacity: 0.9;
}

.jb-state-icon-empty {
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border: 3px dashed var(--jb-pink-300);
  color: var(--jb-pink-400);
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

.jb-state-title {
  font-size: 1.05rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-state-desc {
  font-size: 0.85rem;
  color: var(--jb-text-muted);
  font-weight: 600;
}

/* =========================================================
   لیست سبد
   ========================================================= */
.jb-cart-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

/* هدر (فقط دسکتاپ) */
.jb-cart-list-header {
  display: none;
  grid-template-columns: minmax(0, 3fr) 1fr 1fr 1fr 1.1fr 60px;
  gap: 10px;
  padding: 12px 16px;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border-radius: 14px;
  font-size: 0.8rem;
  font-weight: 900;
  color: var(--jb-text);
  text-align: center;
}

.jb-cart-list-header span:first-child {
  text-align: right;
}

@media (min-width: 768px) {
  .jb-cart-list-header {
    display: grid;
  }
}

/* ردیف */
.jb-cart-row {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
  padding: 14px 16px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  transition: all 0.25s ease;
}

.jb-cart-row:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.14);
  transform: translateY(-2px);
}

@media (min-width: 768px) {
  .jb-cart-row {
    grid-template-columns: minmax(0, 3fr) 1fr 1fr 1fr 1.1fr 60px;
    align-items: center;
    gap: 10px;
  }
}

/* ===== محصول ===== */
.jb-cart-row-product {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  min-width: 0;
}

.jb-cart-row-img-wrap {
  flex-shrink: 0;
  width: 84px;
  height: 84px;
  border-radius: 14px;
  background: linear-gradient(135deg, #fff5f8, #ffe0eb);
  border: 2px solid var(--jb-pink-100);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px;
  overflow: hidden;
}

.jb-cart-row-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 10px;
}

.jb-cart-row-img-placeholder {
  font-size: 2rem;
}

.jb-cart-row-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
  min-width: 0;
}

.jb-cart-row-title {
  font-size: 0.88rem;
  font-weight: 800;
  color: var(--jb-text);
  line-height: 1.6;
  text-decoration: none;
  transition: color 0.2s ease;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.jb-cart-row-title:hover {
  color: var(--jb-fuchsia);
}

/* چیپ ویژگی‌ها */
.jb-cart-row-attrs {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}

.jb-cart-attr-chip {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 3px 9px;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 999px;
  font-size: 0.68rem;
  font-weight: 700;
}

.jb-cart-attr-name {
  color: var(--jb-text-muted);
}

.jb-cart-attr-value {
  color: var(--jb-fuchsia);
  font-weight: 800;
}

/* هشدار */
.jb-cart-alert {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  background: linear-gradient(135deg, #ffe0e8, #ffc2d6);
  border: 1.5px solid #ff9db8;
  border-radius: 10px;
  color: #c9184a;
  font-size: 0.75rem;
  font-weight: 700;
  width: fit-content;
  max-width: 100%;
}

/* ===== سلول‌ها ===== */
.jb-cart-row-cell {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 0.82rem;
  padding: 8px 0;
  border-top: 1.5px dashed var(--jb-pink-100);
}

@media (min-width: 768px) {
  .jb-cart-row-cell {
    justify-content: center;
    border-top: none;
    padding: 0;
    text-align: center;
    flex-direction: column;
    gap: 2px;
  }
}

/* لیبل (فقط موبایل) */
.jb-cell-label {
  color: var(--jb-text-muted);
  font-weight: 700;
  font-size: 0.75rem;
}

@media (min-width: 768px) {
  .jb-cell-label {
    display: none;
  }
}

.jb-cell-value {
  color: var(--jb-text);
  font-weight: 800;
}

.jb-cell-unit {
  font-size: 0.65rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  margin-right: 2px;
}

/* تخفیف */
.jb-cell-discount .jb-cell-value {
  color: #16a34a;
}

/* قیمت نهایی */
.jb-cell-final .jb-cell-value-final {
  color: var(--jb-pink-700);
  font-weight: 900;
  font-size: 0.9rem;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

/* ===== تعداد ===== */
.jb-qty {
  display: inline-flex;
  align-items: center;
  background: #fff;
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 10px;
  overflow: hidden;
  height: 32px;
}

.jb-qty-btn {
  width: 30px;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  border: none;
  color: var(--jb-fuchsia);
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-qty-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
}

.jb-qty-value {
  min-width: 30px;
  text-align: center;
  font-size: 0.82rem;
  font-weight: 900;
  color: var(--jb-text);
  border-left: 1.5px dashed var(--jb-pink-200);
  border-right: 1.5px dashed var(--jb-pink-200);
  line-height: 32px;
}

/* ===== حذف ===== */
.jb-cell-remove {
  justify-content: space-between;
}

@media (min-width: 768px) {
  .jb-cell-remove {
    justify-content: center;
  }
}

.jb-cart-remove-btn {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  background: #fff5f8;
  border: 1.5px solid #ffb8c8;
  color: #c9184a;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-cart-remove-btn:hover {
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  color: #fff;
  border-color: transparent;
  transform: scale(1.08);
  box-shadow: 0 6px 16px rgba(201, 24, 74, 0.35);
}

/* =========================================================
   جمع‌بندی + کوپن
   ========================================================= */
.jb-cart-summary-box {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-top: 24px;
  padding: 20px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px dashed var(--jb-pink-300);
  border-radius: 22px;
}

@media (min-width: 768px) {
  .jb-cart-summary-box {
    flex-direction: row;
    align-items: flex-start;
    justify-content: space-between;
    gap: 30px;
  }
}

/* ===== فرم کوپن ===== */
.jb-coupon-form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  flex: 1;
}

.jb-coupon-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.9rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-coupon-title svg {
  color: var(--jb-fuchsia);
}

.jb-coupon-input-row {
  display: flex;
  gap: 8px;
  flex-direction: column;
}

@media (min-width: 480px) {
  .jb-coupon-input-row {
    flex-direction: row;
  }
}

.jb-coupon-input {
  flex: 1;
  padding: 11px 14px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 12px;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--jb-text);
  outline: none;
  transition: all 0.25s ease;
  font-family: inherit;
}

.jb-coupon-input::placeholder {
  color: var(--jb-text-muted);
  font-weight: 500;
}

.jb-coupon-input:focus {
  border-color: var(--jb-fuchsia);
  box-shadow: 0 0 0 4px rgba(214, 51, 132, 0.15);
}

.jb-coupon-input:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.jb-coupon-btn {
  justify-content: center;
  padding: 11px 24px !important;
  font-size: 0.85rem !important;
  white-space: nowrap;
}

.jb-coupon-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.jb-coupon-error {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  color: #c9184a;
  font-size: 0.8rem;
  font-weight: 700;
  padding: 8px 12px;
  background: #ffe0e8;
  border-radius: 10px;
  width: fit-content;
}

.jb-coupon-success {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  color: #16a34a;
  font-size: 0.8rem;
  font-weight: 700;
  padding: 8px 12px;
  background: #dcfce7;
  border-radius: 10px;
  width: fit-content;
}

/* ===== جمع‌بندی ===== */
.jb-summary-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  min-width: 260px;
  flex: 1;
}

.jb-summary-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 0.85rem;
  gap: 8px;
}

.jb-summary-label {
  color: var(--jb-text-soft);
  font-weight: 700;
}

.jb-summary-value {
  color: var(--jb-text);
  font-weight: 900;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-summary-unit {
  font-size: 0.68rem;
  color: var(--jb-text-muted);
  font-weight: 600;
}

.jb-summary-value-discount {
  color: #16a34a;
}

.jb-summary-divider {
  height: 0;
  border-top: 1.5px dashed var(--jb-pink-300);
  margin: 4px 0;
}

.jb-summary-row-total {
  padding-top: 6px;
}

.jb-summary-label-total {
  font-size: 0.95rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-summary-value-total {
  font-size: 1.2rem;
  font-weight: 900;
  color: var(--jb-pink-700);
  display: inline-flex;
  align-items: baseline;
  gap: 4px;
}

/* =========================================================
   رزروهای فعال
   ========================================================= */
.jb-reservations-box {
  margin-top: 24px;
  padding: 20px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px solid var(--jb-pink-200);
  border-radius: 22px;
}

.jb-res-loading {
  display: flex;
  align-items: center;
  gap: 12px;
  color: var(--jb-text-muted);
  font-size: 0.85rem;
  font-weight: 700;
}

.jb-res-spinner {
  width: 20px;
  height: 20px;
  border: 3px solid var(--jb-pink-200);
  border-top-color: var(--jb-fuchsia);
  border-radius: 50%;
  animation: spin 0.9s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* هدر */
.jb-res-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 18px;
  flex-wrap: wrap;
}

.jb-res-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 1rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-res-title-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
}

.jb-res-desc {
  font-size: 0.8rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  line-height: 1.7;
  margin-top: 4px;
}

.jb-res-count {
  display: inline-flex;
  align-items: center;
  padding: 6px 14px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  font-size: 0.75rem;
  font-weight: 900;
  border-radius: 999px;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
  white-space: nowrap;
}

/* گرید */
.jb-res-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
}

@media (min-width: 768px) {
  .jb-res-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* کارت رزرو */
.jb-res-card {
  padding: 16px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-family: inherit;
  text-align: right;
  width: 100%;
}

.jb-res-card:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.15);
  transform: translateY(-2px);
}

.jb-res-card.is-selected {
  border-color: var(--jb-fuchsia);
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.2);
}

.jb-res-card-top {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
}

.jb-res-card-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

/* رادیو */
.jb-res-radio {
  flex-shrink: 0;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  border: 2.5px solid var(--jb-pink-300);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.25s ease;
}

.jb-res-card.is-selected .jb-res-radio {
  border-color: var(--jb-fuchsia);
  background: rgba(214, 51, 132, 0.08);
}

.jb-res-radio-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 2px 6px rgba(214, 51, 132, 0.4);
}

.jb-res-card-title {
  font-size: 0.9rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-res-card-sub {
  font-size: 0.75rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  margin-top: 3px;
}

.jb-res-card-price {
  text-align: left;
  flex-shrink: 0;
}

.jb-res-card-price-amount {
  font-size: 1rem;
  font-weight: 900;
  color: var(--jb-pink-700);
  line-height: 1.1;
}

.jb-res-card-price-unit {
  font-size: 0.65rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  text-align: left;
}

.jb-res-card-bottom {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 14px;
  padding-top: 12px;
  border-top: 1.5px dashed var(--jb-pink-200);
  font-size: 0.78rem;
}

.jb-res-card-bottom-label {
  color: var(--jb-text-muted);
  font-weight: 600;
}

.jb-res-card-bottom-value {
  color: var(--jb-text);
  font-weight: 800;
}

/* بدون رزرو */
.jb-res-no-reservation {
  margin-top: 16px;
  padding-top: 16px;
  border-top: 1.5px dashed var(--jb-pink-300);
}

.jb-res-no-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 18px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 12px;
  font-size: 0.82rem;
  font-weight: 700;
  color: var(--jb-text-soft);
  cursor: pointer;
  transition: all 0.25s ease;
  font-family: inherit;
}

.jb-res-no-btn:hover {
  border-color: var(--jb-pink-400);
  background: var(--jb-pink-50);
  color: var(--jb-fuchsia);
}

.jb-res-no-btn.is-active {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.4);
}

/* =========================================================
   دکمه ادامه
   ========================================================= */
.jb-cart-continue {
  display: flex;
  justify-content: flex-end;
  margin-top: 24px;
}

.jb-cart-continue-btn {
  padding: 14px 32px !important;
  font-size: 0.92rem !important;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 768px) {
  .jb-cart-card {
    padding: 16px;
    border-radius: 20px;
  }

  .jb-cart-row {
    padding: 12px;
  }

  .jb-cart-row-img-wrap {
    width: 72px;
    height: 72px;
  }

  .jb-cart-summary-box {
    padding: 16px;
    border-radius: 18px;
  }

  .jb-summary-list {
    min-width: auto;
  }

  .jb-reservations-box {
    padding: 16px;
  }

  .jb-res-header {
    flex-direction: column;
    align-items: stretch;
  }

  .jb-res-count {
    align-self: flex-start;
  }

  .jb-cart-continue {
    justify-content: stretch;
  }

  .jb-cart-continue-btn {
    width: 100%;
    justify-content: center;
  }
}
</style>