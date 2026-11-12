<!-- app/components/home/sliders/HomeOfferSlider.client.vue -->
<template>
  <div class="relative w-full h-full mb-4 newProductHome">
    <!-- هدر: فقط دکمه‌های ناوبری -->
    <div class="flex flex-col md:flex-row items-center justify-between gap-2 mb-4">
      <div  class="relative z-10 flex items-center justify-center gap-3 ">
        <div class="w-[6px] h-[64px] rounded-full bg-gradient-to-b from-[#0e5a3a] to-[#8cc63f]"></div>
        <h2 class="text-lg md:text-2xl font-black text-[#0e5a3a] text-center">
          {{ mainTitle }}
        </h2>
        <!-- خط تزئینی زیر عنوان -->
      </div>
      <div class="hero-nav flex gap-2">
        <div class="hero-prev custom-nav-btn">
          <img src="/icons/arrow.svg" alt="قبلی" />
        </div>
        <div class="hero-next custom-nav-btn">
          <img src="/icons/arrow.svg" class="rotate-180" alt="بعدی" />
        </div>
      </div>
    </div>

    <Swiper dir="rtl" :modules="[Navigation, Pagination]" :slides-per-view="3" :space-between="16" :breakpoints="{
      320: { slidesPerView: 1.2, spaceBetween: 16 },
      480: { slidesPerView: 2, spaceBetween: 20 },
      768: { slidesPerView: 2.5, spaceBetween: 24 },
      1024: { slidesPerView: 3, spaceBetween: 24 },
      1280: { slidesPerView: 3, spaceBetween: 24 },
    }" :navigation="{
      nextEl: '.newProductHome .hero-next',
      prevEl: '.newProductHome .hero-prev',
    }" class="hero-swiper">
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
  mainTitle: {
    type: String,
    default: "جدیدترین‌های فروشگاه دکتر اسماعیلی",
  },
});
</script>

<style>
/* ===== دکمه‌های ناوبری اسلایدر محصولات ===== */
.newProductHome .custom-nav-btn {
  background: linear-gradient(135deg, #0e5a3a, #1a7a50) !important;
  width: 42px;
  height: 42px;
  border-radius: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 6px 18px rgba(14, 90, 58, 0.4);
  border: 2px solid rgba(255, 255, 255, 0.6);
}

.newProductHome .custom-nav-btn:hover {
  background: linear-gradient(135deg, #0b472e, #0e5a3a) !important;
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(14, 90, 58, 0.55);
}

.newProductHome .custom-nav-btn img {
  width: 18px;
  height: 18px;
  filter: brightness(0) invert(1);
}

.newProductHome .custom-nav-btn.swiper-button-disabled {
  background: rgba(255, 255, 255, 0.9) !important;
  border: 2px solid #8cc63f !important;
  opacity: 0.7;
  cursor: not-allowed;
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.15);
}

.newProductHome .custom-nav-btn.swiper-button-disabled img {
  filter: none;
  opacity: 0.6;
}
</style>