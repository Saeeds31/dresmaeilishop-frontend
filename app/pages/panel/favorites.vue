<template>
  <div class="jb-favorites-card">
    <!-- هدر -->
    <div class="jb-favorites-header">
      <h3 class="jb-favorites-header-title">
        <span class="jb-favorites-header-icon">
          <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
            <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z" />
          </svg>
        </span>
        علاقه‌مندی‌ها
      </h3>

      <span v-if="!isLoading && wishlistItems.length > 0" class="jb-favorites-count">
        {{ toPersianNum(wishlistItems.length) }} محصول
      </span>
    </div>

    <!-- خط جداکننده -->
    <div class="jb-divider"></div>

    <!-- لودینگ -->
    <div v-if="isLoading" class="jb-state-block">
      <SharedLoadingSpinner size="lg" />
    </div>

    <!-- خالی -->
    <div v-else-if="wishlistItems.length === 0" class="jb-state-block jb-state-empty">
      <div class="jb-state-icon">
        <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
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

    <!-- گرید محصولات -->
    <div v-else class="jb-favorites-grid">
      <SharedProductCard v-for="item in wishlistItems" :key="item.id" :product="item.product" :is-favorite="true"
        @favorite-removed="handleFavoriteRemoved" />
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
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه علاقه‌مندی‌ها - مینیمال سبز
   ========================================================= */

.jb-favorites-card {
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

/* ===== هدر ===== */
.jb-favorites-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
}

.jb-favorites-header-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-favorites-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 6px;
  background: #0e5a3a;
  color: #fff;
}

/* شمارنده */
.jb-favorites-count {
  display: inline-flex;
  align-items: center;
  padding: 4px 10px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 4px;
  color: #0e5a3a;
  font-size: 0.72rem;
  font-weight: 600;
}

/* خط جداکننده */
.jb-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 16px 0 20px;
}

/* ===== حالت‌ها ===== */
.jb-state-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px 20px;
  text-align: center;
  gap: 10px;
  color: #718096;
  font-size: 0.85rem;
  font-weight: 500;
}

.jb-state-empty {
  gap: 8px;
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

.jb-empty-title {
  font-size: 0.95rem;
  font-weight: 700;
  color: #1a202c;
}

.jb-empty-desc {
  font-size: 0.8rem;
  color: #718096;
  font-weight: 500;
  line-height: 1.8;
  max-width: 300px;
}

/* دکمه CTA */
.jb-empty-cta {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  margin-top: 10px;
  padding: 10px 20px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.82rem;
  font-weight: 700;
  text-decoration: none;
  transition: all 0.2s ease;
}

.jb-empty-cta:hover {
  background: #0b472e;
}

/* =========================================================
   گرید محصولات
   ========================================================= */
.jb-favorites-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

@media (min-width: 1280px) {
  .jb-favorites-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 16px;
  }
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-favorites-card {
    padding: 14px;
    border-radius: 8px;
  }

  .jb-favorites-header-title {
    font-size: 0.9rem;
  }

  .jb-favorites-header-icon {
    width: 28px;
    height: 28px;
  }

  .jb-state-block {
    padding: 32px 14px;
  }

  .jb-state-icon {
    width: 56px;
    height: 56px;
  }

  .jb-empty-title {
    font-size: 0.85rem;
  }

  .jb-empty-desc {
    font-size: 0.75rem;
  }
}
</style>