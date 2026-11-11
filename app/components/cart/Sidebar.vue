<!-- app/components/cart/Sidebar.vue -->
<template>
  <Transition name="overlay">
    <div
      v-if="show"
      class="jb-cart-overlay"
      @click="emit('close')"
    ></div>
  </Transition>

  <Transition name="slide">
    <div v-if="show" class="jb-cart-drawer">
      <!-- ============================================================
           هدر
      ============================================================ -->
      <div class="jb-cart-header">
        <div class="jb-cart-header-title">
          <span class="jb-cart-header-icon">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
            </svg>
          </span>
          <div class="jb-cart-header-text">
            <h2>سبد خرید</h2>
            <span v-if="normalizedItems.length" class="jb-cart-header-count">
              {{ toPersianNum(normalizedItems.length) }} کالا
            </span>
          </div>
        </div>

        <button @click="emit('close')" class="jb-cart-close-btn" aria-label="بستن">
          <X :size="20" />
        </button>
      </div>

      <!-- خط جداکننده بریده‌بریده -->
      <div class="jb-cart-dashed"></div>

      <!-- ============================================================
           محتوا
      ============================================================ -->
      <div class="jb-cart-body">
        <div v-if="normalizedItems.length > 0" class="jb-cart-items">
          <div
            v-for="item in normalizedItems"
            :key="item.id || item.variant_id"
            class="jb-cart-item"
          >
            <!-- تصویر -->
            <div class="jb-cart-item-img-wrap">
              <img
                :src="resolveImage(item.image)"
                :alt="item.title"
                class="jb-cart-item-img"
              />
            </div>

            <!-- اطلاعات -->
            <div class="jb-cart-item-info">
              <!-- عنوان -->
              <div class="jb-cart-item-title-row">
                <h3 class="jb-cart-item-title">
                  <nuxt-link :to="`/product/${item.product_id}`">
                    {{ item.title }}
                  </nuxt-link>
                </h3>

                <button
                  @click="removeItem(item)"
                  class="jb-cart-item-remove"
                  aria-label="حذف"
                >
                  <Trash2 :size="16" />
                </button>
              </div>

              <!-- ویژگی‌ها -->
              <div
                v-if="item.variant && item.variant.attributes"
                class="jb-cart-item-attrs"
              >
                <template
                  v-for="(attr, idx) in item.variant.attributes"
                  :key="idx"
                >
                  <span class="jb-cart-attr-chip">
                    <span class="jb-cart-attr-name">{{ attr.name }}:</span>
                    <span class="jb-cart-attr-value">{{ attr.value }}</span>
                  </span>
                </template>
              </div>

              <!-- ردیف: تعداد + قیمت -->
              <div class="jb-cart-item-bottom">
                <!-- تعداد -->
                <div class="jb-cart-qty">
                  <button
                    @click="decreaseQuantity(item)"
                    class="jb-cart-qty-btn"
                    aria-label="کاهش"
                  >
                    <Minus :size="13" />
                  </button>
                  <span class="jb-cart-qty-value">{{ item.quantity }}</span>
                  <button
                    @click="increaseQuantity(item)"
                    class="jb-cart-qty-btn"
                    aria-label="افزایش"
                  >
                    <Plus :size="13" />
                  </button>
                </div>

                <!-- قیمت -->
                <div class="jb-cart-prices">
                  <div class="jb-cart-price-final">
                    {{
                      (
                        item.line_final_total ||
                        item.price_final * item.quantity
                      ).toLocaleString("fa-IR")
                    }}
                    <span class="jb-cart-price-unit">تومان</span>
                  </div>
                  <div
                    v-if="item.price_original != item.price_final"
                    class="jb-cart-price-old"
                  >
                    {{
                      (
                        item.line_original_total ||
                        item.price_original * item.quantity
                      ).toLocaleString("fa-IR")
                    }}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- حالت خالی -->
        <div v-else class="jb-cart-empty">
          <div class="jb-cart-empty-icon">
            <svg class="w-12 h-12" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
                d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
            </svg>
          </div>
          <h3 class="jb-cart-empty-title">سبد خرید خالی است</h3>
          <p class="jb-cart-empty-desc">
            هنوز محصولی به سبد خرید اضافه نکرده‌اید
          </p>
        </div>
      </div>

      <!-- ============================================================
           فوتر
      ============================================================ -->
      <div v-if="normalizedItems.length > 0" class="jb-cart-footer">
        <!-- جمع‌بندی قیمت -->
        <div class="jb-cart-summary">
          <!-- جمع کل -->
          <div class="jb-summary-row">
            <span class="jb-summary-label">جمع کل:</span>
            <span class="jb-summary-value">
              {{ totalPrice.toLocaleString("fa-IR") }}
              <span class="jb-summary-unit">تومان</span>
            </span>
          </div>

          <!-- تخفیف -->
          <div
            v-if="totalDiscount > 0"
            class="jb-summary-row jb-summary-row-discount"
          >
            <span class="jb-summary-label">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a2 2 0 01-2.828 0l-7-7A1.994 1.994 0 013 12V7a4 4 0 014-4z" />
              </svg>
              تخفیف محصولات:
            </span>
            <span class="jb-summary-value jb-summary-value-discount">
              {{ totalDiscount.toLocaleString("fa-IR") }}
              <span class="jb-summary-unit">تومان</span>
            </span>
          </div>

          <!-- خط بریده‌بریده -->
          <div class="jb-summary-divider"></div>

          <!-- مبلغ قابل پرداخت -->
          <div class="jb-summary-row jb-summary-row-total">
            <span class="jb-summary-label">مبلغ قابل پرداخت:</span>
            <span class="jb-summary-value-total">
              {{ payableAmount.toLocaleString("fa-IR") }}
              <span class="jb-summary-unit">تومان</span>
            </span>
          </div>
        </div>

        <!-- دکمه‌ها -->
        <div class="jb-cart-actions">
          <nuxt-link
            to="/cart"
            @click="emit('close')"
            class="jb-btn-linear jb-cart-btn-primary"
          >
            <span>مشاهده سبد خرید</span>
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
            </svg>
          </nuxt-link>

          <button @click="emit('close')" class="jb-cart-btn-secondary">
            ادامه خرید
          </button>
        </div>
      </div>
    </div>
  </Transition>
</template>

<script setup>
import { computed, watch, onMounted, onBeforeUnmount } from "vue";
import { X, Plus, Minus, Trash2 } from "lucide-vue-next";
import { useApi } from "@/composables/api/useApi";
import { useCartStore } from "~/store/cart";
import { toast } from "vue3-toastify";

const { $getImageUrl } = useNuxtApp();

const props = defineProps({
  show: Boolean,
  cartItems: { type: Array, default: () => [] },
});

const emit = defineEmits(["close", "cart-updated", "open"]);

const openSidebar = () => emit("open");

onMounted(() => {
  if (typeof window === "undefined") return;
  window.addEventListener("open-cart-sidebar", openSidebar);
});

onBeforeUnmount(() => {
  if (typeof window === "undefined") return;
  window.removeEventListener("open-cart-sidebar", openSidebar);
});

const { increaseCartItem, decreaseCartItem, deleteCartItem } = useApi();
const cartStore = useCartStore();

/* تبدیل عدد به فارسی */
const toPersianNum = (num) => {
  const d = ["۰", "۱", "۲", "۳", "۴", "۵", "۶", "۷", "۸", "۹"];
  return num.toString().split("").map((x) => d[x] || x).join("");
};

const normalizedItems = computed(() => props.cartItems || []);

const totalPrice = computed(() => {
  if (!normalizedItems.value.length) return 0;
  return normalizedItems.value.reduce((sum, item) => {
    const itemTotal = item.line_original_total || item.price * item.quantity;
    return sum + itemTotal;
  }, 0);
});

const totalDiscount = computed(() => {
  if (!normalizedItems.value.length) return 0;
  return normalizedItems.value.reduce((sum, item) => {
    return sum + (item.line_discount || 0);
  }, 0);
});

const payableAmount = computed(() => {
  if (!normalizedItems.value.length) return 0;
  return normalizedItems.value.reduce((sum, item) => {
    const itemTotal = item.line_final_total || item.price_final * item.quantity;
    return sum + itemTotal;
  }, 0);
});

const resolveImage = (src) => {
  if (!src) return "";
  if (src.startsWith("http")) return src;
  return $getImageUrl(src);
};

const increaseQuantity = async (item) => {
  const token = process.client ? localStorage.getItem("auth-token") : null;

  if (!token) {
    cartStore.increaseQuantity(item.variant_id);
    emit("cart-updated");
    return;
  }

  try {
    await increaseCartItem(item.id);
    emit("cart-updated");
  } catch (error) {
    console.error("❌ Failed to increase quantity:", error);
    toast.error("خطا در افزایش تعداد محصول");
  }
};

const decreaseQuantity = async (item) => {
  const token = process.client ? localStorage.getItem("auth-token") : null;

  if (!token) {
    cartStore.decreaseQuantity(item.variant_id);
    emit("cart-updated");
    return;
  }

  try {
    await decreaseCartItem(item.id);
    emit("cart-updated");
  } catch (error) {
    console.error("❌ Failed to decrease quantity:", error);
    toast.error("خطا در کاهش تعداد محصول");
  }
};

const removeItem = async (item) => {
  const token = process.client ? localStorage.getItem("auth-token") : null;

  if (!token) {
    cartStore.removeItem(item.variant_id);
    emit("cart-updated");
    return;
  }

  try {
    await deleteCartItem(item.id);
    emit("cart-updated");
  } catch (error) {
    console.error("❌ Failed to remove item:", error);
    toast.error("خطا در حذف محصول");
  }
};

watch(() => props.cartItems, () => {}, { deep: true });
</script>

<style scoped>
/* =========================================================
   سایدبار سبد خرید - تم پودری
   ========================================================= */

/* ===== Overlay ===== */
.jb-cart-overlay {
  position: fixed;
  inset: 0;
  background: rgba(45, 20, 32, 0.55);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  z-index: 40;
}

/* ===== Drawer ===== */
.jb-cart-drawer {
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100%;
  max-width: 420px;
  background: linear-gradient(180deg, #fff8fb 0%, #ffeaf2 100%);
  border-right: 2px solid var(--jb-pink-300);
  box-shadow: 8px 0 40px rgba(214, 51, 132, 0.35);
  z-index: 50;
  display: flex;
  flex-direction: column;
}

@media (max-width: 480px) {
  .jb-cart-drawer {
    max-width: 100%;
  }
}

/* =========================================================
   هدر
   ========================================================= */
.jb-cart-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 18px 20px;
  background: rgba(255, 255, 255, 0.75);
  backdrop-filter: blur(10px);
  flex-shrink: 0;
}

.jb-cart-header-title {
  display: flex;
  align-items: center;
  gap: 12px;
}

.jb-cart-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 42px;
  height: 42px;
  border-radius: 14px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.4);
  border: 2px solid #fff;
}

.jb-cart-header-text {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.jb-cart-header-text h2 {
  font-size: 1.05rem;
  font-weight: 900;
  color: var(--jb-text);
  line-height: 1.2;
}

.jb-cart-header-count {
  font-size: 0.72rem;
  font-weight: 700;
  color: var(--jb-fuchsia);
}

.jb-cart-close-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  border-radius: 12px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  color: var(--jb-text-soft);
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-cart-close-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  transform: rotate(90deg);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
}

/* خط بریده‌بریده زیر هدر */
.jb-cart-dashed {
  height: 0;
  border-top: 1.5px dashed var(--jb-pink-300);
  margin: 0 16px;
  position: relative;
  flex-shrink: 0;
}

.jb-cart-dashed::before,
.jb-cart-dashed::after {
  content: "";
  position: absolute;
  top: -4px;
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--jb-pink-400);
  box-shadow: 0 0 0 2px #fff8fb;
}

.jb-cart-dashed::before { left: -3px; }
.jb-cart-dashed::after  { right: -3px; }

/* =========================================================
   بدنه
   ========================================================= */
.jb-cart-body {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  scrollbar-width: thin;
}

.jb-cart-body::-webkit-scrollbar { width: 6px; }
.jb-cart-body::-webkit-scrollbar-track { background: transparent; }
.jb-cart-body::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, var(--jb-pink-400), var(--jb-fuchsia));
  border-radius: 6px;
}

.jb-cart-items {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

/* =========================================================
   آیتم سبد
   ========================================================= */
.jb-cart-item {
  display: flex;
  gap: 12px;
  padding: 12px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.06);
}

.jb-cart-item:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.15);
  transform: translateY(-2px);
}

/* تصویر */
.jb-cart-item-img-wrap {
  flex-shrink: 0;
  width: 78px;
  height: 78px;
  border-radius: 14px;
  overflow: hidden;
  background: linear-gradient(135deg, #fff5f8, #ffe0eb);
  border: 2px solid var(--jb-pink-100);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 4px;
}

.jb-cart-item-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 10px;
}

/* اطلاعات */
.jb-cart-item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
  min-width: 0;
}

/* ردیف عنوان + حذف */
.jb-cart-item-title-row {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 8px;
}

.jb-cart-item-title {
  flex: 1;
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.jb-cart-item-title a {
  color: inherit;
  text-decoration: none;
  transition: color 0.2s ease;
}

.jb-cart-item-title a:hover {
  color: var(--jb-fuchsia);
}

.jb-cart-item-remove {
  flex-shrink: 0;
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 9px;
  background: #fff5f8;
  border: 1.5px solid #ffb8c8;
  color: #c9184a;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-cart-item-remove:hover {
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  color: #fff;
  border-color: transparent;
  transform: scale(1.08);
  box-shadow: 0 4px 12px rgba(201, 24, 74, 0.35);
}

/* ویژگی‌ها */
.jb-cart-item-attrs {
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

/* ردیف پایین: تعداد + قیمت */
.jb-cart-item-bottom {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 10px;
  margin-top: auto;
}

/* ===== تعداد ===== */
.jb-cart-qty {
  display: inline-flex;
  align-items: center;
  background: #fff;
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 10px;
  overflow: hidden;
  height: 32px;
}

.jb-cart-qty-btn {
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

.jb-cart-qty-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
}

.jb-cart-qty-btn:active {
  transform: scale(0.92);
}

.jb-cart-qty-value {
  min-width: 32px;
  text-align: center;
  font-size: 0.8rem;
  font-weight: 900;
  color: var(--jb-text);
  border-left: 1.5px dashed var(--jb-pink-200);
  border-right: 1.5px dashed var(--jb-pink-200);
  line-height: 32px;
}

/* ===== قیمت ===== */
.jb-cart-prices {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 2px;
}

.jb-cart-price-final {
  font-size: 0.9rem;
  font-weight: 900;
  color: var(--jb-pink-700);
  line-height: 1.1;
  display: inline-flex;
  align-items: baseline;
  gap: 4px;
}

.jb-cart-price-unit {
  font-size: 0.65rem;
  font-weight: 600;
  color: var(--jb-text-muted);
}

.jb-cart-price-old {
  font-size: 0.7rem;
  font-weight: 600;
  color: var(--jb-text-muted);
  text-decoration: line-through;
  text-decoration-color: #ff4d6d;
  text-decoration-thickness: 1.5px;
}

/* =========================================================
   حالت خالی
   ========================================================= */
.jb-cart-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 24px;
  text-align: center;
  gap: 14px;
}

.jb-cart-empty-icon {
  width: 96px;
  height: 96px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border: 3px dashed var(--jb-pink-300);
  color: var(--jb-pink-400);
  margin-bottom: 6px;
  animation: jbEmptyFloat 3s ease-in-out infinite;
}

@keyframes jbEmptyFloat {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-8px); }
}

.jb-cart-empty-title {
  font-size: 1rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-cart-empty-desc {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--jb-text-muted);
  line-height: 1.8;
}

/* =========================================================
   فوتر
   ========================================================= */
.jb-cart-footer {
  flex-shrink: 0;
  padding: 16px 20px 20px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  border-top: 1.5px dashed var(--jb-pink-300);
}

/* جمع‌بندی */
.jb-cart-summary {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 16px;
  padding: 14px 16px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.06);
}

.jb-summary-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  font-size: 0.82rem;
}

.jb-summary-label {
  color: var(--jb-text-soft);
  font-weight: 700;
  display: inline-flex;
  align-items: center;
  gap: 6px;
}

.jb-summary-label svg {
  color: var(--jb-fuchsia);
}

.jb-summary-value {
  color: var(--jb-text);
  font-weight: 900;
  font-size: 0.88rem;
}

.jb-summary-unit {
  font-size: 0.65rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  margin-right: 2px;
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
  padding-top: 4px;
}

.jb-summary-row-total .jb-summary-label {
  font-size: 0.9rem;
  color: var(--jb-text);
  font-weight: 900;
}

.jb-summary-value-total {
  font-size: 1.15rem;
  font-weight: 900;
  color: var(--jb-pink-700);
  display: inline-flex;
  align-items: baseline;
  gap: 4px;
}

/* ===== دکمه‌ها ===== */
.jb-cart-actions {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.jb-cart-btn-primary {
  width: 100%;
  justify-content: center;
  padding: 13px 24px !important;
  font-size: 0.9rem !important;
}

.jb-cart-btn-secondary {
  width: 100%;
  padding: 12px 24px;
  background: #fff;
  border: 2px solid var(--jb-pink-300);
  border-radius: 14px;
  color: var(--jb-fuchsia);
  font-size: 0.88rem;
  font-weight: 800;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-cart-btn-secondary:hover {
  background: var(--jb-pink-100);
  border-color: var(--jb-fuchsia);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.2);
}

/* =========================================================
   انیمیشن‌ها
   ========================================================= */
.overlay-enter-active,
.overlay-leave-active {
  transition: opacity 0.3s ease;
}

.overlay-enter-from,
.overlay-leave-to {
  opacity: 0;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(-100%);
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 480px) {
  .jb-cart-header {
    padding: 14px 16px;
  }

  .jb-cart-header-icon {
    width: 38px;
    height: 38px;
  }

  .jb-cart-body {
    padding: 12px;
  }

  .jb-cart-item {
    padding: 10px;
    gap: 10px;
  }

  .jb-cart-item-img-wrap {
    width: 68px;
    height: 68px;
  }

  .jb-cart-footer {
    padding: 14px 16px 18px;
  }

  .jb-cart-summary {
    padding: 12px 14px;
  }

  .jb-cart-empty {
    padding: 40px 20px;
  }

  .jb-cart-empty-icon {
    width: 80px;
    height: 80px;
  }
}
</style>