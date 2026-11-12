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
      <!-- هدر -->
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
            <span v-else class="jb-cart-header-count-empty">
              خالی
            </span>
          </div>
        </div>

        <button @click="emit('close')" class="jb-cart-close-btn" aria-label="بستن">
          <X :size="18" />
        </button>
      </div>

      <!-- بدنه -->
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
                  <Trash2 :size="15" />
                </button>
              </div>

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

              <div class="jb-cart-item-bottom">
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
            هنوز محصولی به سبد خرید اضافه نکرده‌اید.
          </p>
          <button @click="emit('close')" class="jb-cart-empty-btn">
            شروع خرید
          </button>
        </div>
      </div>

      <!-- فوتر -->
      <div v-if="normalizedItems.length > 0" class="jb-cart-footer">
        <div class="jb-cart-summary">
          <div class="jb-summary-row">
            <span class="jb-summary-label">جمع کل:</span>
            <span class="jb-summary-value">
              {{ totalPrice.toLocaleString("fa-IR") }}
              <span class="jb-summary-unit">تومان</span>
            </span>
          </div>

          <div
            v-if="totalDiscount > 0"
            class="jb-summary-row"
          >
            <span class="jb-summary-label">تخفیف:</span>
            <span class="jb-summary-value jb-summary-value-discount">
              {{ totalDiscount.toLocaleString("fa-IR") }}
              <span class="jb-summary-unit">تومان</span>
            </span>
          </div>

          <div class="jb-summary-divider"></div>

          <div class="jb-summary-row jb-summary-row-total">
            <span class="jb-summary-label">مبلغ قابل پرداخت:</span>
            <span class="jb-summary-value-total">
              {{ payableAmount.toLocaleString("fa-IR") }}
              <span class="jb-summary-unit">تومان</span>
            </span>
          </div>
        </div>

        <div class="jb-cart-actions">
          <nuxt-link
            to="/cart"
            @click="emit('close')"
            class="jb-cart-btn-primary"
          >
            مشاهده سبد خرید
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
   سایدبار سبد خرید - مینیمال سبز
   ========================================================= */

.jb-cart-overlay {
  position: fixed;
  inset: 0;
  background: rgba(14, 90, 58, 0.35);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  z-index: 40;
}

.jb-cart-drawer {
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100%;
  max-width: 420px;
  background: #fff;
  border-right: 1px solid rgba(14, 90, 58, 0.1);
  box-shadow: 4px 0 24px rgba(14, 90, 58, 0.12);
  z-index: 50;
  display: flex;
  flex-direction: column;
}

@media (max-width: 480px) {
  .jb-cart-drawer {
    max-width: 100%;
  }
}

/* ===== هدر ===== */
.jb-cart-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid rgba(14, 90, 58, 0.08);
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
  width: 40px;
  height: 40px;
  border-radius: 8px;
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
  flex-shrink: 0;
}

.jb-cart-header-text {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.jb-cart-header-text h2 {
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
  line-height: 1.2;
}

.jb-cart-header-count {
  font-size: 0.7rem;
  font-weight: 600;
  color: #718096;
}

.jb-cart-header-count-empty {
  font-size: 0.7rem;
  font-weight: 600;
  color: #a0aec0;
}

.jb-cart-close-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 8px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.1);
  color: #4a5568;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-cart-close-btn:hover {
  background: #0e5a3a;
  color: #fff;
  border-color: transparent;
}

/* ===== بدنه ===== */
.jb-cart-body {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  scrollbar-width: thin;
}

.jb-cart-body::-webkit-scrollbar { width: 5px; }
.jb-cart-body::-webkit-scrollbar-track { background: transparent; }
.jb-cart-body::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 3px;
}

.jb-cart-items {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

/* ===== آیتم سبد ===== */
.jb-cart-item {
  display: flex;
  gap: 12px;
  padding: 12px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 10px;
  transition: all 0.2s ease;
}

.jb-cart-item:hover {
  border-color: rgba(14, 90, 58, 0.2);
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.06);
}

.jb-cart-item-img-wrap {
  flex-shrink: 0;
  width: 76px;
  height: 76px;
  border-radius: 8px;
  overflow: hidden;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.06);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 4px;
}

.jb-cart-item-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 4px;
}

.jb-cart-item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 6px;
  min-width: 0;
}

.jb-cart-item-title-row {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 8px;
}

.jb-cart-item-title {
  flex: 1;
  font-size: 0.82rem;
  font-weight: 700;
  color: #1a202c;
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.jb-cart-item-title a {
  color: inherit;
  text-decoration: none;
}

.jb-cart-item-title a:hover {
  color: #0e5a3a;
}

.jb-cart-item-remove {
  flex-shrink: 0;
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  background: #fff5f5;
  border: 1px solid #fed7d7;
  color: #e53e3e;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-cart-item-remove:hover {
  background: #e53e3e;
  color: #fff;
  border-color: transparent;
}

/* ویژگی‌ها */
.jb-cart-item-attrs {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.jb-cart-attr-chip {
  display: inline-flex;
  align-items: center;
  gap: 3px;
  padding: 2px 8px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 4px;
  font-size: 0.62rem;
  font-weight: 600;
}

.jb-cart-attr-name {
  color: #a0aec0;
}

.jb-cart-attr-value {
  color: #0e5a3a;
  font-weight: 700;
}

/* ردیف پایین */
.jb-cart-item-bottom {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 10px;
  margin-top: auto;
  padding-top: 4px;
}

/* تعداد */
.jb-cart-qty {
  display: inline-flex;
  align-items: center;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.12);
  border-radius: 6px;
  overflow: hidden;
  height: 30px;
}

.jb-cart-qty-btn {
  width: 28px;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  border: none;
  color: #0e5a3a;
  cursor: pointer;
  transition: all 0.15s ease;
}

.jb-cart-qty-btn:hover {
  background: #0e5a3a;
  color: #fff;
}

.jb-cart-qty-value {
  min-width: 28px;
  text-align: center;
  font-size: 0.8rem;
  font-weight: 800;
  color: #1a202c;
  line-height: 30px;
}

/* قیمت */
.jb-cart-prices {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 2px;
}

.jb-cart-price-final {
  font-size: 0.88rem;
  font-weight: 800;
  color: #0e5a3a;
  line-height: 1.1;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-cart-price-unit {
  font-size: 0.62rem;
  font-weight: 500;
  color: #a0aec0;
}

.jb-cart-price-old {
  font-size: 0.68rem;
  font-weight: 500;
  color: #cbd5e1;
  text-decoration: line-through;
  text-decoration-color: #a0aec0;
  text-decoration-thickness: 1px;
}

/* ===== حالت خالی ===== */
.jb-cart-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 24px;
  text-align: center;
  gap: 14px;
  height: 100%;
}

.jb-cart-empty-icon {
  width: 90px;
  height: 90px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  color: #a0aec0;
  margin-bottom: 4px;
}

.jb-cart-empty-title {
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-cart-empty-desc {
  font-size: 0.82rem;
  font-weight: 500;
  color: #718096;
  line-height: 1.8;
  max-width: 260px;
}

.jb-cart-empty-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  margin-top: 6px;
  padding: 10px 24px;
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 0.85rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-cart-empty-btn:hover {
  background: linear-gradient(135deg, #0b472e, #0e5a3a);
  transform: translateY(-1px);
}

/* ===== فوتر ===== */
.jb-cart-footer {
  flex-shrink: 0;
  padding: 16px 20px 20px;
  border-top: 1px solid rgba(14, 90, 58, 0.08);
}

.jb-cart-summary {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 14px;
  padding: 14px 16px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.06);
  border-radius: 10px;
}

.jb-summary-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  font-size: 0.8rem;
}

.jb-summary-label {
  color: #4a5568;
  font-weight: 600;
}

.jb-summary-value {
  color: #1a202c;
  font-weight: 700;
  font-size: 0.85rem;
}

.jb-summary-unit {
  font-size: 0.62rem;
  color: #a0aec0;
  font-weight: 500;
  margin-right: 2px;
}

.jb-summary-value-discount {
  color: #38a169;
}

.jb-summary-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.12);
  margin: 4px 0;
}

.jb-summary-row-total {
  padding-top: 2px;
}

.jb-summary-row-total .jb-summary-label {
  font-size: 0.85rem;
  color: #1a202c;
  font-weight: 800;
}

.jb-summary-value-total {
  font-size: 1.1rem;
  font-weight: 800;
  color: #0e5a3a;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

/* ===== دکمه‌ها ===== */
.jb-cart-actions {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.jb-cart-btn-primary {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 11px 20px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 0.85rem;
  font-weight: 700;
  font-family: inherit;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-cart-btn-primary:hover {
  background: #0b472e;
}

.jb-cart-btn-secondary {
  width: 100%;
  padding: 10px 20px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.15);
  border-radius: 8px;
  color: #0e5a3a;
  font-size: 0.82rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-cart-btn-secondary:hover {
  background: #f7fafc;
  border-color: rgba(14, 90, 58, 0.3);
}

/* ===== انیمیشن‌ها ===== */
.overlay-enter-active,
.overlay-leave-active {
  transition: opacity 0.25s ease;
}

.overlay-enter-from,
.overlay-leave-to {
  opacity: 0;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(-100%);
}

/* ===== ریسپانسیو ===== */
@media (max-width: 480px) {
  .jb-cart-header {
    padding: 14px 16px;
  }

  .jb-cart-header-icon {
    width: 36px;
    height: 36px;
    border-radius: 6px;
  }

  .jb-cart-body {
    padding: 12px;
  }

  .jb-cart-item {
    padding: 10px;
    gap: 10px;
  }

  .jb-cart-item-img-wrap {
    width: 66px;
    height: 66px;
  }

  .jb-cart-footer {
    padding: 14px 16px 18px;
  }

  .jb-cart-empty {
    padding: 40px 20px;
  }

  .jb-cart-empty-icon {
    width: 76px;
    height: 76px;
  }
}
</style>