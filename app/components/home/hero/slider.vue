<template>
  <div class="w-full h-[420px] md:h-[450px] overflow-hidden rounded-3xl jb-shadow">
    <Swiper
      :modules="[Autoplay, Pagination, Navigation]"
      :slides-per-view="1"
      :loop="false"
      :autoplay="{ delay: 3500, disableOnInteraction: false }"
      :pagination="true"
      :navigation="{
        nextEl: '.hero-next',
        prevEl: '.hero-prev',
      }"
      class="hero-swiper"
      dir="rtl"
    >
      <SwiperSlide
        v-for="slide in slides"
        :key="slide.id"
        class="w-full h-full"
      >
        <nuxt-link
          :to="slide.link"
          class="relative w-full h-full rounded-3xl px-8 py-9 flex justify-between overflow-hidden"
          style="background: linear-gradient(135deg, #fff0f5 0%, #ffe0eb 50%, #ffd0e0 100%);"
        >
          <div :class="slide.type" class="absolute inset-0 flex z-0">
            <img
              :src="$getImageUrl(slide.image)"
              class="h-full w-full object-cover"
            />
          </div>
        </nuxt-link>
      </SwiperSlide>

      <!-- دکمه بعدی (چپ) -->
      <div class="hero-next custom-nav-btn">
        <img src="/icons/arrow.svg" class="rotate-180" />
      </div>

      <!-- دکمه قبلی (راست) -->
      <div class="hero-prev custom-nav-btn">
        <img src="/icons/arrow.svg" />
      </div>
    </Swiper>
  </div>
</template>

<script setup lang="ts">
import "swiper/css";
import "swiper/css/navigation";
import "swiper/css/pagination";

import { Swiper, SwiperSlide } from "swiper/vue";
import { Autoplay, Navigation, Pagination } from "swiper/modules";

const { $getImageUrl } = useNuxtApp();

defineProps<{ slides: any[] }>();
</script>

<style scoped>
/* ===== Swiper ===== */
:deep(.hero-swiper) {
  height: 100%;
  border-radius: 24px;
  overflow: hidden;
}

/* ===== نقطه‌های Pagination ===== */
:deep(.swiper-pagination) {
  bottom: 16px !important;
}

:deep(.swiper-pagination-bullet) {
  width: 10px;
  height: 10px;
  background: #ffffff;
  opacity: 0.85;
  border-radius: 50%;
  transition: all 0.3s ease;
  box-shadow: 0 2px 6px rgba(214, 51, 132, 0.35);
}

:deep(.swiper-pagination-bullet-active) {
  width: 26px;
  height: 10px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  border-radius: 999px;
  opacity: 1;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.55);
}

/* ===== دکمه‌های ناوبری ===== */
:deep(.custom-nav-btn) {
  width: 42px;
  height: 42px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  border-radius: 14px;
  box-shadow: 0 6px 18px rgba(214, 51, 132, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  top: 60%;
  transform: translateY(-50%);
  cursor: pointer;
  z-index: 20;
  position: absolute;
  transition: all 0.25s ease;
  border: 2px solid rgba(255, 255, 255, 0.6);
}

:deep(.custom-nav-btn:hover) {
  background: linear-gradient(135deg, var(--jb-pink-600), var(--jb-pink-800));
  transform: translateY(-50%) scale(1.08);
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.6);
}

:deep(.custom-nav-btn img) {
  width: 18px;
  height: 18px;
  filter: brightness(0) invert(1);
}

:deep(.hero-prev) {
  right: 14px !important;
}

:deep(.hero-next) {
  left: 14px !important;
}

/* ===== حالت غیرفعال ===== */
:deep(.custom-nav-btn.swiper-button-disabled) {
  background: rgba(255, 255, 255, 0.9) !important;
  border: 2px solid var(--jb-pink-300) !important;
  opacity: 0.75;
  cursor: not-allowed;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.15);
  transform: translateY(-50%) scale(1);
}

:deep(.custom-nav-btn.swiper-button-disabled img) {
  filter: none;
  opacity: 0.7;
}
</style>

<style>
/* ===== ریسپانسیو اسلایدها ===== */
.swiper-slide:has(.mobile) {
  display: none;
}

@media (max-width: 580px) {
  .swiper-slide:has(.desktop) {
    display: none;
  }
  .swiper-slide:has(.mobile) {
    display: block;
  }
}
</style>