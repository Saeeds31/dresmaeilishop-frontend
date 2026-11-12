<!-- app/components/home/newProducts.vue -->
<template>
  <section class="w-full py-8">
    <!-- دیباگ: تعداد دسته‌های دریافتی -->
    <div v-if="!categoriesWithProducts.length" class="mx-auto px-4 py-10 text-center text-gray-400">
      <p>محصولی برای نمایش وجود ندارد</p>
    </div>

    <!-- هر دسته‌بندی = یک سکشن با اسلایدر مستقل -->
    <div v-for="(category, idx) in categoriesWithProducts" :key="category.id"
      class="mx-auto px-4 md:px-12 mb-14 last:mb-8">

      <!-- ===== هدر سکشن: تیتر + دکمه ===== -->
      <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-4">
        <div class="relative z-10 flex items-center md:justify-between justify-start gap-3 ">

          <div class="w-[6px] h-[64px] rounded-full bg-gradient-to-b from-[#0e5a3a] to-[#8cc63f]"></div>

          <h2 class="text-xl md:text-2xl lg:text-3xl font-black text-gray-800 text-right">
            محصولات
            <span class="text-[#0e5a3a]">{{ category.title }}</span>
          </h2>
        </div>

        <NuxtLink :to="`/products?category_ids=${category.id}`"
          class="hidden group md:inline-flex items-center gap-2 px-6 py-2.5 rounded-full font-bold text-sm text-white bg-gradient-to-l from-[#0e5a3a] to-[#1a7a50] shadow-lg shadow-[#0e5a3a]/20 hover:shadow-xl hover:shadow-[#0e5a3a]/30 transition-all duration-300 hover:-translate-y-1 relative overflow-hidden self-start sm:self-auto">
          <span class="relative z-10">مشاهده همه</span>
          <svg class="w-4 h-4 relative z-10 transition-transform duration-300 group-hover:-translate-x-1" fill="none"
            stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
          </svg>
          <!-- افکت درخشش -->
          <span
            class="absolute inset-0 bg-gradient-to-r from-transparent via-white/30 to-transparent -translate-x-full group-hover:translate-x-full transition-transform duration-700"></span>
        </NuxtLink>
      </div>



      <!-- ===== اسلایدر محصولات این دسته ===== -->
      <div class="relative mt-6">
        <Swiper :id="`cat-swiper-${category.id}-${idx}`" :modules="[Navigation, Pagination]" :slides-per-view="1.15"
          :space-between="12" :breakpoints="{
            480: { slidesPerView: 1.3, spaceBetween: 14 },
            640: { slidesPerView: 2.2, spaceBetween: 16 },
            768: { slidesPerView: 2.6, spaceBetween: 18 },
            1024: { slidesPerView: 3, spaceBetween: 20 },
            1280: { slidesPerView: 4, spaceBetween: 22 },
          }" :navigation="{
            nextEl: `.hero-next-${category.id}-${idx}`,
            prevEl: `.hero-prev-${category.id}-${idx}`,
          }" :pagination="{ clickable: true, dynamicBullets: true }" class="jb-cat-swiper" dir="rtl">
          <SwiperSlide v-for="item in category.products" :key="item.id">
            <shared-product-card :product="item" :saveLoc="true" :activeTab="category.id" />
          </SwiperSlide>
        </Swiper>
        <div class="flex md:hidden items-center justify-center mt-4">
          <NuxtLink :to="`/products/${category.slug ?? category.id}`"
            class="inline-flex group  items-center gap-2 px-6 py-2.5 rounded-full font-bold text-sm text-white bg-gradient-to-l from-[#0e5a3a] to-[#1a7a50] shadow-lg shadow-[#0e5a3a]/20 hover:shadow-xl hover:shadow-[#0e5a3a]/30 transition-all duration-300 hover:-translate-y-1 relative overflow-hidden self-start sm:self-auto">
            <span class="relative z-10">مشاهده همه</span>
            <svg class="w-4 h-4 relative z-10 transition-transform duration-300 group-hover:-translate-x-1" fill="none"
              stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
            </svg>
            <!-- افکت درخشش -->
            <span
              class="absolute inset-0 bg-gradient-to-r from-transparent via-white/30 to-transparent -translate-x-full group-hover:translate-x-full transition-transform duration-700"></span>
          </NuxtLink>
        </div>
        <!-- دکمه‌های ناوبری اسلایدر -->
        <div class="hidden md:flex items-center justify-center gap-2 mt-5">
          <div :class="`hero-prev-${category.id}-${idx}`" class="jb-swiper-nav-btn">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M9 5l7 7-7 7" />
            </svg>
          </div>
          <div :class="`hero-next-${category.id}-${idx}`" class="jb-swiper-nav-btn">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
            </svg>
          </div>
        </div>
        <HomeBanners class="block md:hidden" v-if="idx == 0 && banners" :banners="banners" />

      </div>
    </div>
  </section>
</template>
<script setup>
import { computed } from "vue";
import { Swiper, SwiperSlide } from "swiper/vue";
import { Navigation, Pagination } from "swiper/modules";

import "swiper/css";
import "swiper/css/navigation";
import "swiper/css/pagination";

const props = defineProps({
  selectedCategoryList: { type: Array, default: () => [] },
  banners: { type: Array, default: () => [] },
});

/* ============================================================
   تشخیص ساختار داده:
   ۱. هر آیتم می‌تونه {id, title, list: [...] } باشه
   ۲. یا {id, title, products: [...] } باشه
   ۳. یا {id, title, items: [...] } باشه
   هر کدوم که بود، محصولاتش رو استخراج می‌کنیم.
============================================================ */
const extractProducts = (cat) => {
  if (!cat) return [];
  if (Array.isArray(cat.list) && cat.list.length) return cat.list;
  if (Array.isArray(cat.products) && cat.products.length) return cat.products;
  if (Array.isArray(cat.items) && cat.items.length) return cat.items;
  if (Array.isArray(cat.data) && cat.data.length) return cat.data;
  return [];
};

const categoriesWithProducts = computed(() => {
  if (!Array.isArray(props.selectedCategoryList)) return [];

  return props.selectedCategoryList
    .map((cat) => ({
      id: cat.id ?? cat.category_id ?? cat.title,
      slug: cat.slug,
      title: cat.title ?? cat.name ?? "دسته‌بندی",
      products: extractProducts(cat),
    }))
    .filter((cat) => cat.products.length > 0);
});
</script>
<style scoped>
/* ===== دکمه‌های ناوبری اسلایدر (سبز) ===== */
.jb-swiper-nav-btn {
  width: 42px;
  height: 42px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 6px 18px rgba(14, 90, 58, 0.4);
  border: 2px solid rgba(255, 255, 255, 0.6);
}

.jb-swiper-nav-btn:hover {
  background: linear-gradient(135deg, #0b472e, #0e5a3a);
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(14, 90, 58, 0.55);
}

/* حالت غیرفعال */
:deep(.swiper-button-disabled) {
  background: rgba(255, 255, 255, 0.9) !important;
  border: 2px solid #8cc63f !important;
  color: #8cc63f !important;
  opacity: 0.7;
  cursor: not-allowed;
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.15) !important;
  transform: none !important;
}

/* ===== Pagination ===== */
:deep(.jb-cat-swiper .swiper-pagination) {
  position: static;
  margin-top: 14px;
}

:deep(.jb-cat-swiper .swiper-pagination-bullet) {
  width: 8px;
  height: 8px;
  background: #8cc63f;
  opacity: 0.4;
  transition: all 0.3s ease;
}

:deep(.jb-cat-swiper .swiper-pagination-bullet-active) {
  width: 22px;
  height: 8px;
  border-radius: 999px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  opacity: 1;
}

/* ===== موبایل ===== */
@media (max-width: 640px) {
  .jb-swiper-nav-btn {
    width: 38px;
    height: 38px;
  }
}
</style>