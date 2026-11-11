<!-- app/components/home/Categories/index.vue -->
<template>
  <section class="w-full mt-16 mb-24 px-4 md:px-8 lg:px-12 xl:px-28">
    <HomeTitleSection
      titleBlack="دسته‌بندی‌های"
      titleStrong="ما"
      description="راحتی، تنوع، پوشش زیبا، و تجربه خریدی فراموش‌نشدنی"
    />

    <!-- گرید دسته‌بندی‌ها: موبایل ۲، تبلت ۳، دسکتاپ ۳ (۹ تا در ۳ ردیف) -->
    <div
      class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-3 gap-4 md:gap-5 mt-8"
    >
      <HomeCategoriesCard
        v-for="(item, index) in mappedCategories"
        :key="item.id ?? index"
        :item="item"
      />
    </div>
  </section>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  categories: {
    type: Array,
    default: () => [],
  },
});

/* فقط ۹ دسته اول رو نگه می‌داریم، بدون اضافه کردن «همه محصولات» */
const mappedCategories = computed(() => {
  return [...props.categories]
    .slice(0, 9)
    .map((item, index) => mapCategory(item, index));
});

const mapCategory = (item, index) => {
  return {
    id: item.id,
    slug: item.slug,
    title: item.title ?? "دسته‌بندی",
    description:
      item.meta_description || item.description || "محصولات این دسته را ببینید",
    image: item.main_image || item.icon || "",
    link: item.slug ? `/category/${item.slug}` : `/category/${item.slug}`,
    buttonText: "مشاهده محصولات",
    big: index === 0,
    gradient: gradientByIndex(index),
  };
};

/* گرادیان‌های تم پودری برای حالت بدون تصویر */
const gradientByIndex = (index) => {
  const gradients = [
    "bg-gradient-to-br from-[#ffb6c1] to-[#ff8fa3]",
    "bg-gradient-to-br from-[#ffe0eb] to-[#ffc2d6]",
    "bg-gradient-to-br from-[#f8b4d9] to-[#d63384]",
    "bg-gradient-to-br from-[#ffc2d6] to-[#ff4d80]",
    "bg-gradient-to-br from-[#ff9db8] to-[#c91f52]",
    "bg-gradient-to-br from-[#ffd0e0] to-[#ff7aa0]",
  ];
  return gradients[index % gradients.length];
};
</script>