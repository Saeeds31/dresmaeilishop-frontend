<template>
  <section
    v-if="sliders && sliders.length"
    class="w-full flex flex-col lg:flex-row gap-6 mt-6 lg:mt-10 px-4 md:px-8 lg:px-12 xl:px-28 items-start"
  >
    <div class="w-full">
      <!-- حالت بارگذاری -->
      <div
        v-if="pending"
        class="w-full h-[520px] md:h-[550px] flex items-center justify-center rounded-3xl bg-white border border-gray-100 shadow-sm"
      >
        <div class="flex flex-col items-center gap-3">
          <span
            class="w-10 h-10 rounded-full border-4 border-[#e8f3ec] border-t-[#0e5a3a] animate-spin"
          ></span>
          <span class="text-gray-500 font-bold text-sm">در حال بارگذاری...</span>
        </div>
      </div>

      <!-- حالت خطا -->
      <div
        v-else-if="error"
        class="w-full h-[420px] md:h-[450px] flex items-center justify-center rounded-3xl bg-white border border-gray-100 shadow-sm"
      >
        <div class="flex flex-col items-center gap-3 text-center">
          <span class="text-4xl">😔</span>
          <span class="font-bold text-[#0e5a3a]">
            خطا در دریافت اسلایدر
          </span>
          <span class="text-xs text-gray-400">
            لطفاً صفحه را دوباره بارگذاری کنید
          </span>
        </div>
      </div>

      <!-- اسلایدر -->
      <HomeHeroSlider v-else :slides="sliders" />
    </div>
  </section>
</template>

<script setup>
const props = defineProps(["sliders", "pending", "error"]);
</script>