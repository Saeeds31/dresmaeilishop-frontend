<template>
  <nuxt-link @click="saveLocation()" :to="`/product/${product.slug ?? product.id}`" class="jb-product-card group"
    :class="{ 'is-unavailable': product.status === 'unpublished' }">
    <!-- ===== بخش تصویر ===== -->
    <div class="jb-product-image-wrap">
      <div class="jb-product-image">
        <img v-if="imageSrc" :src="imageSrc" :alt="product.title ?? 'محصول'" class="jb-product-img" loading="lazy" />
        <div v-else class="jb-product-img jb-product-img-placeholder">
          <svg class="w-14 h-14 opacity-40" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
              d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
          </svg>
        </div>
      </div>

      <!-- بج تخفیف - گوشه بالا راست -->
      <div v-if="hasDiscount" class="jb-discount-badge">
        <span class="jb-discount-percent">{{ discountPercent }}٪</span>
        <span class="jb-discount-text">تخفیف</span>
      </div>

      <!-- قلب علاقه‌مندی - گوشه بالا چپ -->
      <button @click.stop.prevent="toggleFavorite" class="jb-fav-btn" :class="{ 'is-active': isFavoriteComputed }"
        aria-label="افزودن به علاقه‌مندی‌ها">
        <svg v-if="isFavoriteComputed" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
          <path
            d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z" />
        </svg>
        <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="w-5 h-5">
          <path stroke-linecap="round" stroke-linejoin="round"
            d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" />
        </svg>
      </button>

      <!-- بج ناموجود (شیشه‌ای) -->
      <div v-if="product.status === 'unpublished'" class="jb-unavailable-badge">
        <span>ناموجود</span>
      </div>
    </div>

    <!-- ===== بدنه کارت ===== -->
    <div class="jb-product-body">
      <!-- دسته‌بندی کوچک (اختیاری - برای حس داروخانه) -->
      <span v-if="product.category?.title" class="jb-product-category">
        {{ product.category.title }}
      </span>

      <!-- عنوان -->
      <h2 class="jb-product-title">
        {{ product?.title ?? "محصول" }}
      </h2>

      <!-- ردیف پایین: قیمت + دکمه -->
      <div class="jb-product-footer">
        <!-- بخش قیمت -->
        <div class="jb-product-pricing">
          <template v-if="product.status !== 'unpublished'">
            <div v-if="hasDiscount" class="jb-price-old">
              {{ Number(product?.price).toLocaleString("fa") ?? "۰" }}
            </div>
            <div class="jb-price-final">
              <span class="jb-price-amount">
                {{ Number(product?.final_price).toLocaleString("fa") ?? "۰" }}
              </span>
              <span class="jb-price-unit">تومان</span>
            </div>
          </template>
          <template v-else>
            <div class="jb-price-unavailable">ناموجود</div>
          </template>
        </div>

        <!-- دکمه -->
        <button class="jb-product-btn" :disabled="product.status === 'unpublished'">
          {{ product.status === 'unpublished' ? 'اطلاع' : 'خرید' }}
        </button>
      </div>
    </div>
  </nuxt-link>
</template>
<script setup>
import { computed, onMounted } from "vue";
import { useWishlist } from "@/composables/useWishlist";
import { useRouter } from "vue-router";

const { $getImageUrl } = useNuxtApp();
const router = useRouter();
const { isInWishlist, toggleWishlist, fetchWishlist } = useWishlist();

const props = defineProps({
  product: { type: Object, required: true },
  isFavorite: { type: Boolean, default: false },
  saveLoc: { type: Boolean, default: false },
  activeTab: { type: String, default: "1" },
});

const emit = defineEmits(["favorite-removed"]);

const isFavoriteComputed = computed(() => {
  if (props.isFavorite) return true;
  return isInWishlist(props.product.id);
});

const toggleFavorite = async () => {
  const token = process.client ? localStorage.getItem("auth-token") : null;
  if (!token) return;
  const wasAdded = await toggleWishlist(props.product.id);
  if (!wasAdded) emit("favorite-removed", props.product.id);
};



function saveLocation() {
  if (props.saveLoc) {
    localStorage.setItem("activeTab", props.activeTab);
    localStorage.setItem("lastPosition", window.scrollY);
  }
}

const imageSrc = computed(() => {
  const img = props.product?.main_image;
  if (!img) return "";
  if (img.startsWith("http://") || img.startsWith("https://")) return img;
  return $getImageUrl(img);
});

const hasDiscount = computed(() => {
  const p = props.product;
  return (
    p?.price &&
    p?.final_price &&
    Number(p.price) > Number(p.final_price) &&
    p.status !== "unpublished"
  );
});

const discountPercent = computed(() => {
  if (!hasDiscount.value) return 0;
  const p = Number(props.product.price);
  const f = Number(props.product.final_price);
  return Math.round(((p - f) / p) * 100);
});

onMounted(() => {
  fetchWishlist();
});
</script>
<style scoped>
/* =========================================================
   کارت محصول - طراحی داروخانه لوکس (تم سبز)
   ========================================================= */

.jb-product-card {
  display: flex;
  flex-direction: column;
  background: linear-gradient(180deg, #ffffff 0%, #f8fcf9 100%);
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 20px;
  overflow: hidden;
  position: relative;
  height: 100%;
  min-height: 400px;
  transition: all 0.35s cubic-bezier(0.25, 0.8, 0.25, 1);
  box-shadow: 0 4px 20px rgba(14, 90, 58, 0.04);
  text-decoration: none;
  color: inherit;
  /* پترن نقطه‌ای بسیار ملایم در پس‌زمینه */
  background-image:
    radial-gradient(rgba(14, 90, 58, 0.03) 1px, transparent 1px),
    linear-gradient(180deg, #ffffff 0%, #f8fcf9 100%);
  background-size: 20px 20px, 100% 100%;
}

.jb-product-card:hover {
  transform: translateY(-6px);
  border-color: rgba(14, 90, 58, 0.2);
  box-shadow: 0 20px 40px -10px rgba(14, 90, 58, 0.15);
}

.jb-product-card.is-unavailable {
  opacity: 0.85;
  filter: grayscale(0.3);
}

/* =========================================================
   تصویر
   ========================================================= */
.jb-product-image-wrap {
  position: relative;
  padding: 12px 12px 0;
}

.jb-product-image {
  position: relative;
  width: 100%;
  aspect-ratio: 1 / 1;
  background: #f1f7f3;
  border-radius: 14px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.jb-product-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.6s cubic-bezier(0.25, 0.8, 0.25, 1);
}

.jb-product-card:hover .jb-product-img {
  transform: scale(1.08);
}

.jb-product-card.is-unavailable .jb-product-img {
  filter: grayscale(100%);
}

.jb-product-img-placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #8cc63f;
}

/* =========================================================
   بج تخفیف (سبز لوکس)
   ========================================================= */
.jb-discount-badge {
  position: absolute;
  top: 20px;
  right: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
  border-radius: 12px;
  padding: 6px 10px;
  min-width: 46px;
  box-shadow: 0 6px 16px rgba(14, 90, 58, 0.35);
  z-index: 5;
  animation: popIn 0.35s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  border: 1.5px solid rgba(255, 255, 255, 0.2);
}

.jb-discount-percent {
  font-size: 0.9rem;
  font-weight: 900;
  line-height: 1.1;
  letter-spacing: -0.5px;
}

.jb-discount-text {
  font-size: 0.55rem;
  font-weight: 700;
  opacity: 0.9;
  margin-top: 1px;
}

@keyframes popIn {
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
   قلب علاقه‌مندی (شیشه‌ای)
   ========================================================= */
.jb-fav-btn {
  position: absolute;
  top: 20px;
  left: 20px;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  color: #4a5568;
  border: 1.5px solid rgba(255, 255, 255, 0.6);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 5;
}

.jb-fav-btn:hover {
  background: #fff;
  transform: scale(1.1);
  color: #0e5a3a;
  border-color: rgba(14, 90, 58, 0.2);
}

.jb-fav-btn.is-active {
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  border-color: rgba(255, 255, 255, 0.4);
  box-shadow: 0 6px 18px rgba(14, 90, 58, 0.4);
  animation: heartBeat 0.5s ease;
}

.jb-fav-btn:active {
  transform: scale(0.92);
}

@keyframes heartBeat {
  0% {
    transform: scale(1);
  }

  30% {
    transform: scale(1.2);
  }

  60% {
    transform: scale(0.95);
  }

  100% {
    transform: scale(1);
  }
}

/* =========================================================
   بج ناموجود (شیشه‌ای)
   ========================================================= */
.jb-unavailable-badge {
  position: absolute;
  inset: 12px 12px 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.6);
  backdrop-filter: blur(3px);
  -webkit-backdrop-filter: blur(3px);
  z-index: 4;
  border-radius: 14px;
}

.jb-unavailable-badge span {
  background: linear-gradient(135deg, #4a5568, #2d3748);
  color: #fff;
  font-weight: 900;
  font-size: 0.85rem;
  padding: 7px 22px;
  border-radius: 999px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
  border: 2px solid rgba(255, 255, 255, 0.7);
}

/* =========================================================
   بدنه کارت
   ========================================================= */
.jb-product-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 12px 16px 16px;
  flex: 1;
  position: relative;
}

/* دسته‌بندی کوچک */
.jb-product-category {
  font-size: 0.65rem;
  font-weight: 700;
  color: #8cc63f;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  text-align: right;
  opacity: 0.9;
}

/* عنوان */
.jb-product-title {
  font-size: 0.95rem;
  font-weight: 700;
  color: #1a202c;
  line-height: 1.7;
  text-align: right;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  min-height: 3.2em;
  transition: color 0.25s ease;
}

.jb-product-card:hover .jb-product-title {
  color: #0e5a3a;
}

/* خط جداکننده بریده‌بریده */
.jb-dashed-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 4px 0 8px;
}

/* =========================================================
   ردیف پایین: قیمت + دکمه
   ========================================================= */
.jb-product-footer {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 10px;
  margin-top: auto;
  padding-top: 4px;
}

/* ----- بخش قیمت ----- */
.jb-product-pricing {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
  flex-shrink: 0;
}

.jb-price-final {
  display: flex;
  align-items: baseline;
  gap: 4px;
  flex-wrap: nowrap;
}

.jb-price-amount {
  font-size: 1.15rem;
  font-weight: 800;
  color: #0e5a3a;
  line-height: 1.1;
  letter-spacing: -0.5px;
}

.jb-price-unit {
  font-size: 0.7rem;
  color: #718096;
  font-weight: 500;
}

.jb-price-old {
  font-size: 0.85rem;
  color: #a0aec0;
  text-decoration: line-through;
  text-decoration-color: #8cc63f;
  text-decoration-thickness: 1.5px;
  font-weight: 500;
}

.jb-price-unavailable {
  font-size: 0.85rem;
  font-weight: 800;
  color: #718096;
}

/* ----- دکمه (سبز لوکس) ----- */
.jb-product-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 10px 20px;
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
  border: none;
  border-radius: 12px;
  font-size: 0.82rem;
  font-weight: 800;
  font-family: 'Vazirmatn', sans-serif;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 14px rgba(14, 90, 58, 0.25);
  white-space: nowrap;
  flex-shrink: 0;
  min-width: 80px;
  position: relative;
  overflow: hidden;
}

/* افکت درخشش روی دکمه */
.jb-product-btn::before {
  content: "";
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s ease;
}

.jb-product-btn:hover::before {
  left: 100%;
}

.jb-product-btn:hover {
  background: linear-gradient(135deg, #0b472e, #0e5a3a);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(14, 90, 58, 0.4);
}

.jb-product-btn:active {
  transform: translateY(0);
}

.jb-product-btn:disabled {
  background: #e2e8f0;
  color: #a0aec0;
  box-shadow: none;
  cursor: not-allowed;
  opacity: 0.8;
}

.jb-product-btn:disabled:hover {
  transform: none;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 1280px) {
  .jb-price-amount {
    font-size: 1.05rem;
  }

  .jb-product-btn {
    padding: 9px 16px;
    font-size: 0.78rem;
    min-width: 74px;
  }
}

@media (max-width: 992px) {
  .jb-product-card {
    min-height: 380px;
  }
}

@media (max-width: 768px) {
  .jb-product-card {
    min-height: 340px;
    border-radius: 16px;
  }

  .jb-product-image-wrap {
    padding: 10px 10px 0;
  }

  .jb-product-image {
    border-radius: 12px;
  }

  .jb-fav-btn {
    width: 36px;
    height: 36px;
    top: 16px;
    left: 16px;
  }

  .jb-discount-badge {
    top: 16px;
    right: 16px;
    padding: 5px 8px;
    min-width: 42px;
  }

  .jb-discount-percent {
    font-size: 0.8rem;
  }

  .jb-discount-text {
    font-size: 0.5rem;
  }

  .jb-product-title {
    font-size: 0.88rem;
    min-height: 2.9em;
  }

  .jb-price-amount {
    font-size: 1rem;
  }

  .jb-product-btn {
    padding: 9px 14px;
    font-size: 0.75rem;
    min-width: 70px;
  }

  .jb-product-body {
    padding: 10px 14px 14px;
    gap: 6px;
  }
}

@media (max-width: 480px) {
  .jb-product-card {
    min-height: 320px;
  }

  .jb-product-footer {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
  }

  .jb-product-pricing {
    align-items: center;
  }

  .jb-product-btn {
    width: 100%;
    padding: 10px 12px;
  }

  .jb-price-amount {
    font-size: 0.95rem;
  }

  .jb-unavailable-badge {
    inset: 10px 10px 0;
  }
}
</style>