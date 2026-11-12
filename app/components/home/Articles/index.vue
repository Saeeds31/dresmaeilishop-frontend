<template>
  <section class="container gap mb-10">
    <homeTitleSection
      titleBlack="مجله   "
      titleStrong="دکتر اسماعیلی"
      description="جدیدترین مقالات و نکات مفید درباره سلامت، زیبایی و سبک زندگی سالم"
    />

    <!-- گرید ۴ تایی کارت‌های بزرگ -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mt-8">
      <BigCard
        v-for="(item, index) in list.slice(0, 4)"
        :key="index"
        :card="item"
      />
    </div>
  </section>
</template>

<script setup>
import BigCard from "./BigCard.vue";

defineProps({
  list: {
    type: Array,
    default: () => [],
  },
});
</script>

<style scoped>
.swiper-slide {
  height: auto !important;
}
</style>