<!-- app/components/home/sliders/HomeOfferSlider.client.vue -->
<template>
  <div class="relative w-full h-full mb-4 newProductHome">
    <!-- هدر: فقط دکمه‌های ناوبری -->
    <div class="flex items-center gap-2 justify-end mb-4">
      <div class="hero-nav flex gap-2">
        <div class="hero-prev custom-nav-btn">
          <img src="/icons/arrow.svg" alt="قبلی" />
        </div>
        <div class="hero-next custom-nav-btn">
          <img src="/icons/arrow.svg" class="rotate-180" alt="بعدی" />
        </div>
      </div>
    </div>

    <Swiper
      dir="rtl"
      :modules="[Navigation, Pagination]"
      :slides-per-view="1.2"
      :space-between="12"
      :breakpoints="{
        480:  { slidesPerView: 1.2, spaceBetween: 16 },
        640:  { slidesPerView: 2.2, spaceBetween: 20 },
        768:  { slidesPerView: 2.4, spaceBetween: 24 },
        1024: { slidesPerView: 3.2, spaceBetween: 28 },
        1280: { slidesPerView: 4.2, spaceBetween: 32 },
      }"
      :navigation="{
        nextEl: '.hero-next',
        prevEl: '.hero-prev',
      }"
      class="hero-swiper"
    >
      <SwiperSlide v-for="item in products" :key="item.id">
        <SharedProductCard :product="item" />
      </SwiperSlide>
    </Swiper>
  </div>
</template>

<script setup>
import { Swiper, SwiperSlide } from "swiper/vue";
import { Navigation, Pagination } from "swiper/modules";

import "swiper/css";
import "swiper/css/navigation";
import "swiper/css/pagination";

defineProps({
  products: {
    type: Array,
    default: () => [],
  },
});
</script>

<style>
/* ===== دکمه‌های ناوبری اسلایدر محصولات ===== */
.newProductHome .custom-nav-btn {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia)) !important;
  width: 42px;
  height: 42px;
  border-radius: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 6px 18px rgba(214, 51, 132, 0.4);
  border: 2px solid rgba(255, 255, 255, 0.6);
}

.newProductHome .custom-nav-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-600), var(--jb-pink-800)) !important;
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.55);
}

.newProductHome .custom-nav-btn img {
  width: 18px;
  height: 18px;
  filter: brightness(0) invert(1);
}

.newProductHome .custom-nav-btn.swiper-button-disabled {
  background: rgba(255, 255, 255, 0.9) !important;
  border: 2px solid var(--jb-pink-300) !important;
  opacity: 0.7;
  cursor: not-allowed;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.15);
}

.newProductHome .custom-nav-btn.swiper-button-disabled img {
  filter: none;
  opacity: 0.6;
}
</style>