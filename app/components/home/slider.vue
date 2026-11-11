<template>
  <section
    v-if="sliders && sliders.length"
    class="w-full flex flex-col lg:flex-row gap-6 mt-6 lg:mt-10 px-4 md:px-8 lg:px-12 xl:px-28 items-start"
  >
    <div class="w-full">
      <!-- حالت بارگذاری -->
      <div
        v-if="pending"
        class="w-full h-[520px] md:h-[550px] flex items-center justify-center rounded-3xl jb-card"
      >
        <div class="flex flex-col items-center gap-3">
          <span
            class="w-10 h-10 rounded-full border-4 border-[color:var(--jb-pink-200)] border-t-[color:var(--jb-fuchsia)] animate-spin"
          ></span>
          <span class="jb-text-soft font-bold text-sm">در حال بارگذاری...</span>
        </div>
      </div>

      <!-- حالت خطا -->
      <div
        v-else-if="error"
        class="w-full h-[420px] md:h-[450px] flex items-center justify-center rounded-3xl jb-card"
      >
        <div class="flex flex-col items-center gap-3 text-center">
          <span class="text-4xl">😔</span>
          <span class="font-bold text-[color:var(--jb-pink-700)]">
            خطا در دریافت اسلایدر
          </span>
          <span class="text-xs jb-text-muted">
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