<template>
  <section class="flex flex-col md:flex-row gap-4 container gap py-18">
    <nuxt-link v-for="(item, index) in banners" :key="index" to="" class="block w-full "
      :class="`md:w-[${item.ratio}%]`">
      <!-- تصویر دسکتاپ (فقط در دسکتاپ نمایش داده می‌شود) -->
      <figure v-if="item.image_desktop" class="hidden md:block">
        <img class="w-full h-auto rounded-2xl object-cover" :src="$getImageUrl(item.image_desktop)" alt="" />
      </figure>

      <!-- تصویر موبایل (فقط در موبایل نمایش داده می‌شود) -->
      <figure v-if="item.image_mobile" class="flex md:hidden w-[100%]">
        <img class="w-[100%] h-auto rounded-2xl object-cover" :src="$getImageUrl(item.image_mobile)" alt="" />
      </figure>
    </nuxt-link>
  </section>
</template>

<script setup>
const { $getImageUrl } = useNuxtApp();

defineProps({
  banners: Array,
});
</script>