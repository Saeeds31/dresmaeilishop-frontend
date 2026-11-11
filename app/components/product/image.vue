<!-- app/components/product/image.vue -->
<script setup>
const { $getImageUrl } = useNuxtApp();

import { ref, onMounted, onUnmounted, watch, computed } from "vue";
import { Swiper, SwiperSlide } from "swiper/vue";
import { Thumbs, FreeMode, Navigation } from "swiper/modules";
import "swiper/css";
import "swiper/css/thumbs";
import "swiper/css/free-mode";
import "swiper/css/navigation";

const props = defineProps({
  images: { type: Array, required: true },
  video: { type: String, default: null },
  thumbnails: { type: Array, required: true },
});

const showVideo = ref(false);
const showZoom = ref(false);
const currentZoomIndex = ref(0);
const thumbsSwiper = ref(null);
const mainSwiper = ref(null);

const setThumbsSwiper = (swiper) => { thumbsSwiper.value = swiper; };
const setMainSwiper = (swiper) => { mainSwiper.value = swiper; };

const goNext = () => { if (mainSwiper.value) mainSwiper.value.slideNext(); };
const goPrev = () => { if (mainSwiper.value) mainSwiper.value.slidePrev(); };

const zoomNext = () => {
  if (currentZoomIndex.value < props.images.length - 1) currentZoomIndex.value++;
  else currentZoomIndex.value = 0;
};
const zoomPrev = () => {
  if (currentZoomIndex.value > 0) currentZoomIndex.value--;
  else currentZoomIndex.value = props.images.length - 1;
};

const openZoom = (index) => {
  currentZoomIndex.value = index;
  showZoom.value = true;
  document.body.style.overflow = "hidden";
};
const closeZoom = () => {
  showZoom.value = false;
  document.body.style.overflow = "";
};

const handleKeydown = (e) => {
  if (!showZoom.value) return;
  if (e.key === "Escape") closeZoom();
  else if (e.key === "ArrowRight") { e.preventDefault(); zoomNext(); }
  else if (e.key === "ArrowLeft") { e.preventDefault(); zoomPrev(); }
};

onMounted(() => window.addEventListener("keydown", handleKeydown));
onUnmounted(() => {
  window.removeEventListener("keydown", handleKeydown);
  document.body.style.overflow = "";
});

watch(showZoom, (newVal) => { if (!newVal) currentZoomIndex.value = 0; });

/* ============================================================
   🖱️ Hover Zoom - فقط در دسکتاپ
   ============================================================ */
const isDesktop = ref(false);
const hoverActive = ref(false);
const mouseX = ref(0);
const mouseY = ref(0);
const containerWidth = ref(0);
const containerHeight = ref(0);

// بررسی دسکتاپ بودن
const checkDesktop = () => {
  if (typeof window === "undefined") return;
  isDesktop.value =
    window.matchMedia("(pointer: fine)").matches &&
    window.innerWidth >= 1024;
};

onMounted(() => {
  checkDesktop();
  window.addEventListener("resize", checkDesktop);
});
onUnmounted(() => {
  window.removeEventListener("resize", checkDesktop);
});

// ورود موس به تصویر
const handleMouseEnter = (event) => {
  if (!isDesktop.value || showZoom.value) return;
  hoverActive.value = true;
  updateMousePosition(event);
  updateContainerSize(event);
};

// حرکت موس روی تصویر
const handleMouseMove = (event) => {
  if (!isDesktop.value || !hoverActive.value || showZoom.value) return;
  updateMousePosition(event);
};

// خروج موس از تصویر
const handleMouseLeave = () => {
  hoverActive.value = false;
};

// محاسبه موقعیت موس نسبت به کانتینر (درصد)
const updateMousePosition = (event) => {
  const target = event.currentTarget;
  if (!target) return;
  const rect = target.getBoundingClientRect();
  const x = ((event.clientX - rect.left) / rect.width) * 100;
  const y = ((event.clientY - rect.top) / rect.height) * 100;
  mouseX.value = Math.max(0, Math.min(100, x));
  mouseY.value = Math.max(0, Math.min(100, y));
};

// اندازه کانتینر برای زوم دقیق
const updateContainerSize = (event) => {
  const target = event.currentTarget;
  if (!target) return;
  const rect = target.getBoundingClientRect();
  containerWidth.value = rect.width;
  containerHeight.value = rect.height;
};

// استایل داینامیک برای زوم
const zoomImageStyle = computed(() => {
  if (!hoverActive.value) return {};
  return {
    transform: "scale(2)",
    transformOrigin: `${mouseX.value}% ${mouseY.value}%`,
  };
});

// فقط در دسکتاپ اجازه‌ی زوم می‌دیم
const allowHoverZoom = computed(() => isDesktop.value);
</script>

<template>
  <div class="jb-product-gallery">
    <!-- اسلایدر اصلی -->
    <Swiper
      @swiper="setMainSwiper"
      :modules="[Thumbs]"
      :thumbs="{ swiper: thumbsSwiper }"
      :loop="true"
      class="jb-main-swiper"
    >
      <SwiperSlide v-for="(image, index) in images" :key="index">
        <figure
          class="jb-main-slide"
          :class="{ 'jb-zoom-active': hoverActive && allowHoverZoom }"
          @mouseenter="handleMouseEnter"
          @mousemove="handleMouseMove"
          @mouseleave="handleMouseLeave"
        >
          <img
            :src="$getImageUrl(image)"
            alt="تصویر محصول"
            class="jb-main-img"
            :style="allowHoverZoom ? zoomImageStyle : {}"
            :class="{ 'jb-main-img-zooming': hoverActive && allowHoverZoom }"
            @click="openZoom(index)"
          />

          <!-- آیکون ذره‌بین (فقط دسکتاپ، قبل از هاور) -->
          <span
            v-if="allowHoverZoom && !hoverActive"
            class="jb-zoom-hint"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0zM10 7v3m0 0v3m0-3h3m-3 0H7" />
            </svg>
          </span>

          <!-- دکمه پلی ویدیو -->
          <button
            v-if="index === 0 && video && !hoverActive"
            @click.stop="showVideo = true"
            class="jb-video-play-btn"
            aria-label="پخش ویدیو"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none">
              <path d="M21.4086 9.35258C23.5305 10.5065 23.5305 13.4935 21.4086 14.6474L8.59662 21.6145C6.53435 22.736 4 21.2763 4 18.9671L4 5.0329C4 2.72368 6.53435 1.26402 8.59661 2.38548L21.4086 9.35258Z" fill="currentColor" />
            </svg>
          </button>
        </figure>
      </SwiperSlide>
    </Swiper>

    <!-- مودال ویدیو -->
    <div v-if="showVideo && video" @click.self="showVideo = false" class="jb-modal-overlay">
      <div class="jb-modal-content">
        <button @click="showVideo = false" class="jb-modal-close" aria-label="بستن">
          <svg class="w-7 h-7" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
        <video :src="$getImageUrl(video)" controls autoplay class="jb-modal-video">
          مرورگر شما از ویدیو پشتیبانی نمی‌کند.
        </video>
      </div>
    </div>

    <!-- مودال زوم -->
    <div v-if="showZoom" @click.self="closeZoom" class="jb-modal-overlay jb-zoom-overlay">
      <div class="jb-zoom-inner">
        <button @click="closeZoom" class="jb-modal-close jb-modal-close-top" aria-label="بستن">
          <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>

        <button @click.stop="zoomPrev" class="jb-zoom-nav jb-zoom-prev" aria-label="قبلی">
          <svg class="w-7 h-7" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
          </svg>
        </button>

        <button @click.stop="zoomNext" class="jb-zoom-nav jb-zoom-next" aria-label="بعدی">
          <svg class="w-7 h-7" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
          </svg>
        </button>

        <img
          :src="$getImageUrl(images[currentZoomIndex])"
          alt="تصویر زوم شده"
          class="jb-zoom-img"
          @click="closeZoom"
        />

        <div class="jb-zoom-counter">
          {{ currentZoomIndex + 1 }} / {{ images.length }}
        </div>
      </div>
    </div>

    <!-- thumbnail ها -->
    <div class="jb-thumbs-row">
      <button @click="goNext" class="jb-thumb-nav" aria-label="بعدی">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M9 5l7 7-7 7" />
        </svg>
      </button>

      <Swiper
        @swiper="setThumbsSwiper"
        :slides-per-view="2"
        :space-between="16"
        :free-mode="true"
        :watch-slides-progress="true"
        :loop="true"
        :modules="[FreeMode, Thumbs]"
        class="jb-thumbs-swiper"
        :breakpoints="{
          480: { slidesPerView: 3, spaceBetween: 10 },
          768: { slidesPerView: 4, spaceBetween: 12 },
          1024: { slidesPerView: 4, spaceBetween: 14 },
          1280: { slidesPerView: 4, spaceBetween: 16 },
        }"
      >
        <SwiperSlide v-for="(thumb, index) in thumbnails" :key="index" class="jb-thumb-slide">
          <figure class="jb-thumb-figure">
            <img :src="$getImageUrl(thumb)" class="jb-thumb-img" />
          </figure>
        </SwiperSlide>
      </Swiper>

      <button @click="goPrev" class="jb-thumb-nav" aria-label="قبلی">
        <svg class="w-5 h-5 rotate-180" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M9 5l7 7-7 7" />
        </svg>
      </button>
    </div>
  </div>
</template>

<style scoped>
/* =========================================================
   گالری تصاویر محصول - تم پودری
   ========================================================= */

.jb-product-gallery {
  width: 100%;
}

/* ===== اسلایدر اصلی ===== */
:deep(.jb-main-swiper) {
  border-radius: 24px;
  overflow: hidden;
  border: 2px solid var(--jb-pink-200);
  background: #fff;
  box-shadow: 0 12px 30px rgba(214, 51, 132, 0.12);
}

.jb-main-slide {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 32px;
  position: relative;
  aspect-ratio: 1 / 1;
  overflow: hidden;
  transition: box-shadow 0.3s ease;
}

/* وقتی زوم فعاله */
.jb-main-slide.jb-zoom-active {
  cursor: crosshair;
  box-shadow: inset 0 0 0 2px rgba(214, 51, 132, 0.3);
}

.jb-main-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  cursor: zoom-in;
  transition: transform 0.15s ease-out;
  aspect-ratio: 1 / 1;
  border-radius: 16px;
  will-change: transform;
}

/* هاور روی دسکتاپ (بدون زوم) */
.jb-main-img:hover:not(.jb-main-img-zooming) {
  transform: scale(1.06);
}

/* حالت زوم فعال */
.jb-main-img.jb-main-img-zooming {
  cursor: zoom-out;
  border-radius: 0;
  transition: transform 0.1s ease-out;
}

/* آیکون ذره‌بین (راهنمای زوم) */
.jb-zoom-hint {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  color: var(--jb-fuchsia);
  border: 2px solid var(--jb-pink-200);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.2);
  pointer-events: none;
  z-index: 5;
  animation: jbHintPulse 2.5s ease-in-out infinite;
}

@keyframes jbHintPulse {
  0%, 100% {
    transform: scale(1);
    opacity: 0.95;
  }
  50% {
    transform: scale(1.08);
    opacity: 1;
  }
}

/* دکمه پلی ویدیو */
.jb-video-play-btn {
  position: absolute;
  bottom: 20px;
  left: 20px;
  width: 70px;
  height: 70px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border: 3px solid #fff;
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.5);
  cursor: pointer;
  z-index: 10;
  transition: all 0.3s ease;
  animation: jbPulse 2s ease-in-out infinite;
}

.jb-video-play-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 14px 32px rgba(214, 51, 132, 0.7);
}

.jb-video-play-btn svg {
  width: 26px;
  height: 26px;
  margin-right: -2px;
}

@keyframes jbPulse {
  0%, 100% {
    box-shadow: 0 10px 25px rgba(214, 51, 132, 0.5), 0 0 0 0 rgba(214, 51, 132, 0.4);
  }
  50% {
    box-shadow: 0 10px 25px rgba(214, 51, 132, 0.5), 0 0 0 18px rgba(214, 51, 132, 0);
  }
}

/* ===== thumbnail ها ===== */
.jb-thumbs-row {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-top: 16px;
}

.jb-thumb-nav {
  flex-shrink: 0;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  color: var(--jb-fuchsia);
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-thumb-nav:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.35);
}

:deep(.jb-thumbs-swiper) {
  flex: 1;
  padding: 4px 0;
}

.jb-thumb-slide {
  cursor: pointer;
}

.jb-thumb-figure {
  background: #fff;
  border-radius: 14px;
  border: 2px solid var(--jb-pink-200);
  overflow: hidden;
  transition: all 0.25s ease;
  aspect-ratio: 3 / 4;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
}

.jb-thumb-figure:hover {
  border-color: var(--jb-pink-400);
  transform: scale(1.05);
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.2);
}

.jb-thumb-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

/* thumbnail فعال */
:deep(.swiper-slide-thumb-active .jb-thumb-figure) {
  border-color: var(--jb-fuchsia);
  box-shadow: 0 0 0 4px rgba(214, 51, 132, 0.18);
  transform: scale(1.05);
}

/* ===== مودال‌ها ===== */
.jb-modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 50;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(26, 13, 24, 0.9);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  padding: 16px;
}

.jb-modal-content {
  position: relative;
  width: 100%;
  max-width: 900px;
}

.jb-modal-close {
  position: absolute;
  top: -50px;
  right: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 42px;
  height: 42px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.15);
  border: 2px solid rgba(255, 255, 255, 0.25);
  color: #fff;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-modal-close:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  border-color: transparent;
  transform: rotate(90deg);
}

.jb-modal-close-top {
  top: 20px;
  right: 20px;
  z-index: 10;
}

.jb-modal-video {
  width: 100%;
  border-radius: 20px;
  box-shadow: 0 25px 60px rgba(0, 0, 0, 0.5);
}

/* ===== زوم مودال ===== */
.jb-zoom-overlay {
  background: rgba(0, 0, 0, 0.96);
}

.jb-zoom-inner {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.jb-zoom-img {
  max-width: 100%;
  max-height: 90vh;
  object-fit: contain;
  cursor: zoom-out;
}

.jb-zoom-nav {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 52px;
  height: 52px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.12);
  border: 2px solid rgba(255, 255, 255, 0.25);
  color: #fff;
  cursor: pointer;
  z-index: 10;
  transition: all 0.25s ease;
}

.jb-zoom-nav:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  border-color: transparent;
  transform: translateY(-50%) scale(1.08);
}

.jb-zoom-prev { left: 20px; }
.jb-zoom-next { right: 20px; }

.jb-zoom-counter {
  position: absolute;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  padding: 8px 20px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  font-size: 0.82rem;
  font-weight: 800;
  border-radius: 999px;
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.5);
}

/* ===== ریسپانسیو ===== */
@media (max-width: 768px) {
  .jb-main-slide { padding: 20px; }
  .jb-video-play-btn { width: 58px; height: 58px; }
  .jb-video-play-btn svg { width: 22px; height: 22px; }
  .jb-thumb-nav { width: 36px; height: 36px; }
  .jb-zoom-nav { width: 44px; height: 44px; }
  .jb-zoom-prev { left: 10px; }
  .jb-zoom-next { right: 10px; }
}

/* 🖱️ در موبایل/تبلت، هاور زوم غیرفعال */
@media (hover: none), (pointer: coarse) {
  .jb-main-img:hover {
    transform: none !important;
  }
  .jb-zoom-hint {
    display: none;
  }
}
</style>