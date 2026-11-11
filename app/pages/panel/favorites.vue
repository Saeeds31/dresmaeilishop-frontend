<template>
  <div class="jb-favorites-card">
    <!-- ===== هدر ===== -->
    <div class="jb-favorites-header">
      <h3 class="jb-favorites-header-title">
        <span class="jb-favorites-header-icon">
          <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
            <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z" />
          </svg>
        </span>
        علاقه‌مندی‌ها
      </h3>

      <span v-if="!isLoading && wishlistItems.length > 0" class="jb-favorites-count">
        {{ toPersianNum(wishlistItems.length) }} محصول
      </span>
    </div>

    <!-- خط بریده‌بریده -->
    <div class="jb-divider"></div>

    <!-- ===== لودینگ ===== -->
    <div v-if="isLoading" class="jb-state-block">
      <SharedLoadingSpinner size="lg" />
    </div>

    <!-- ===== خالی ===== -->
    <div v-else-if="wishlistItems.length === 0" class="jb-state-block jb-state-empty">
      <div class="jb-state-icon">
        <svg class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" />
        </svg>
      </div>
      <span class="jb-empty-title">هنوز محصولی به علاقه‌مندی‌ها اضافه نکرده‌اید</span>
      <span class="jb-empty-desc">
        محصولات مورد علاقه‌تان را با کلیک روی قلب ذخیره کنید
      </span>
      <NuxtLink to="/products" class="jb-empty-cta">
        <span>مشاهده محصولات</span>
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
        </svg>
      </NuxtLink>
    </div>

    <!-- ===== گرید محصولات ===== -->
    <div v-else class="jb-favorites-grid">
      <SharedProductCard
        v-for="item in wishlistItems"
        :key="item.id"
        :product="item.product"
        :is-favorite="true"
        @favorite-removed="handleFavoriteRemoved"
      />
    </div>
  </div>
</template>

<script setup>
import { onMounted } from "vue";
import { useApi } from "@/composables/api/useApi";
import { useWishlist } from "@/composables/useWishlist";

definePageMeta({
  layout: "panel",
  middleware: "auth",
});

const { wishlistItems, isLoading, fetchWishlist } = useWishlist();

const handleFavoriteRemoved = async (productId) => {
  await fetchWishlist();
};

onMounted(async () => {
  await fetchWishlist();
});

/* تبدیل عدد به فارسی برای شمارنده */
const toPersianNum = (num) => {
  const digits = ["۰", "۱", "۲", "۳", "۴", "۵", "۶", "۷", "۸", "۹"];
  return num.toString().split("").map((d) => digits[d] || d).join("");
};
useSeoMeta({
  title: () => 'علاقه مندی پنل کاربری | فروشگاه دکتر اسماعیلی',
  description: 'لیست علاقه مندی کاربر در پنل اختصاصی',
  robots: 'noindex, nofollow',
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/panel/favorities' }],
  // 🔹 جلوگیری از ایندکس شدن با متا تگ اضافی برای اطمینان بیشتر
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه علاقه‌مندی‌ها - تم پودری
   ========================================================= */

.jb-favorites-card {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 24px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(214, 51, 132, 0.1);
}

/* ===== هدر ===== */
.jb-favorites-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
}

.jb-favorites-header-title {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  font-size: 1.15rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-favorites-header-icon {
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
  animation: jbHeartBeat 2.5s ease-in-out infinite;
}

@keyframes jbHeartBeat {
  0%, 100% { transform: scale(1); }
  50%      { transform: scale(1.1); }
}

/* شمارنده */
.jb-favorites-count {
  display: inline-flex;
  align-items: center;
  padding: 6px 14px;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 999px;
  color: var(--jb-fuchsia);
  font-size: 0.75rem;
  font-weight: 900;
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

.jb-divider::before { right: -4px; }
.jb-divider::after  { left: -4px; }

/* ===== حالت‌ها ===== */
.jb-state-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 48px 20px;
  text-align: center;
  gap: 12px;
  color: var(--jb-text-muted);
  font-size: 0.88rem;
  font-weight: 600;
}

.jb-state-empty {
  gap: 10px;
}

.jb-state-icon {
  width: 90px;
  height: 90px;
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

.jb-empty-title {
  font-size: 1rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-empty-desc {
  font-size: 0.82rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  line-height: 1.8;
  max-width: 320px;
}

/* دکمه CTA */
.jb-empty-cta {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  margin-top: 14px;
  padding: 12px 26px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border: none;
  border-radius: 14px;
  font-size: 0.85rem;
  font-weight: 800;
  text-decoration: none;
  transition: all 0.3s ease;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.4);
  position: relative;
  overflow: hidden;
}

.jb-empty-cta::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.35), transparent);
  transform: translateX(-100%);
  transition: transform 0.6s ease;
}

.jb-empty-cta:hover::before {
  transform: translateX(100%);
}

.jb-empty-cta:hover {
  background: linear-gradient(135deg, var(--jb-pink-600), var(--jb-pink-800));
  transform: translateY(-3px);
  box-shadow: 0 14px 32px rgba(214, 51, 132, 0.55);
}

.jb-empty-cta svg {
  transition: transform 0.3s ease;
}

.jb-empty-cta:hover svg {
  transform: translateX(-5px);
}

/* =========================================================
   گرید محصولات
   ========================================================= */
.jb-favorites-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 14px;
}


@media (min-width: 1280px) {
  .jb-favorites-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 18px;
  }
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-favorites-card {
    padding: 16px;
    border-radius: 20px;
  }

  .jb-favorites-header-title {
    font-size: 1rem;
  }

  .jb-favorites-header-icon {
    width: 32px;
    height: 32px;
  }

  .jb-state-block {
    padding: 40px 16px;
  }

  .jb-state-icon {
    width: 76px;
    height: 76px;
  }

  .jb-empty-title {
    font-size: 0.9rem;
  }

  .jb-empty-desc {
    font-size: 0.78rem;
  }
}
</style>