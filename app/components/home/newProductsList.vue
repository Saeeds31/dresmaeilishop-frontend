<!-- app/components/home/OffersSection.vue -->
<template>
  <section class="w-full mt-16 mb-24 px-4 md:px-8 lg:px-12 xl:px-28">

    <!-- گرید اصلی: سمت راست اسلایدر (بدون قاب) + سمت چپ بنر -->
    <div class="flex flex-col lg:flex-row gap-6 items-stretch">
      
      <!-- ===== بخش اسلایدر (سمت راست - ۳ از ۴) ===== -->
      <!-- بدون هیچ باکس، حاشیه یا سایه اضافی. فقط خود اسلایدر -->
      <div class="w-full lg:w-3/4">
        <div class="relative z-10 w-full">
          <ClientOnly>
            <!-- اگر کامپوننت اسلایدر امکان تنظیم slidesPerView را دارد، اینجا 3 تنظیم شود -->
            <HomeSlidersHomeOfferSlider :products="new_products" :slides-per-view="3" />
          </ClientOnly>
        </div>

        <!-- دکمه مشاهده همه -->
        <div class="relative z-10 flex justify-center mt-8">
          <NuxtLink to="/products" class="group inline-flex items-center gap-2.5 px-8 py-3 rounded-full font-bold text-sm text-white bg-gradient-to-l from-[#0e5a3a] to-[#1a7a50] shadow-lg shadow-[#0e5a3a]/20 hover:shadow-xl hover:shadow-[#0e5a3a]/30 transition-all duration-300 hover:-translate-y-1 relative overflow-hidden">
            <span class="relative z-10">مشاهده همه محصولات</span>
            <svg class="w-5 h-5 relative z-10 transition-transform duration-300 group-hover:-translate-x-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
            </svg>
            <!-- افکت درخشش روی دکمه -->
            <span class="absolute inset-0 bg-gradient-to-r from-transparent via-white/30 to-transparent -translate-x-full group-hover:translate-x-full transition-transform duration-700"></span>
          </NuxtLink>
        </div>
      </div>

      <!-- ===== بخش بنر پوستی (سمت چپ - ۱ از ۴) ===== -->
      <NuxtLink 
        to="/products/محصولات-پوستی" 
        class="relative w-full lg:w-1/4 aspect-[1/2] md:aspect-auto lg:min-h-full rounded-3xl overflow-hidden group shadow-md hover:shadow-2xl transition-all duration-500"
      >
        <!-- تصویر بنر (شامل متن و دکمه خودش) -->
        <img 
          src="/images/new_products.jpg" 
          alt="محصولات پوستی" 
          class="absolute inset-0 w-full h-full object-cover transition-transform duration-700 group-hover:scale-110"
        />

        <!-- انیمیشن درخشش (Shine) لوکس از بالا به پایین -->
        <div class="absolute inset-0 z-20 pointer-events-none overflow-hidden">
          <div class="shine-effect"></div>
        </div>
      </NuxtLink>
    </div>
  </section>
</template>

<script setup lang="ts">
defineProps(["new_products"]);
</script>

<style scoped>
/* ===== انیمیشن درخشش لوکس از بالا به پایین ===== */
.shine-effect {
  position: absolute;
  top: -100%; /* شروع از بالای تصویر */
  left: 0;
  width: 100%;
  height: 50%; /* ارتفاع نوار درخشش */
  background: linear-gradient(
    to bottom,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 0.15) 30%,
    rgba(255, 255, 255, 0.4) 50%,
    rgba(255, 255, 255, 0.15) 70%,
    rgba(255, 255, 255, 0) 100%
  );
  transform: skewY(-15deg) translateX(-20%); /* زاویه برای حس لوکس */
  animation: shineFall 4s infinite ease-in-out;
  opacity: 0;
}

@keyframes shineFall {
  0% {
    top: -100%;
    opacity: 0;
  }
  10% {
    opacity: 1; /* ظاهر شدن سریع */
  }
  35% {
    top: 100%; /* عبور از کل ارتفاع به پایین */
    opacity: 0; /* محو شدن در انتها */
  }
  100% {
    top: 100%;
    opacity: 0;
  }
}
</style>