<!-- app/components/home/Categories/Card.vue -->
<template>
  <NuxtLink
    :to="item.id == 0 ? '/products' : `/products/${item.slug??item.id}`"
    class="jb-cat-card group"
  >
    <!-- تصویر یا گرادیان پس‌زمینه -->
    <div class="jb-cat-media">
      <img
        v-if="imageSrc"
        :src="imageSrc"
        :alt="item.title"
        class="jb-cat-img"
        loading="lazy"
      />
      <div v-else :class="['jb-cat-gradient', item.gradient]"></div>

      <!-- لایه بلور مشکی ملایم -->
      <div class="jb-cat-blur-overlay"></div>

      <!-- عنوان وسط کارت -->
      <div class="jb-cat-title-wrap">
        <h3 class="jb-cat-title">{{ item.title }}</h3>
        <span class="jb-cat-line"></span>
      </div>
    </div>
  </NuxtLink>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  item: { type: Object, required: true },
});

const { $getImageUrl } = useNuxtApp();

const imageSrc = computed(() => {
  const img = props.item?.image;
  if (!img) return "";
  if (img.startsWith("http://") || img.startsWith("https://")) return img;
  return $getImageUrl(img);
});
</script>

<style scoped>
/* =========================================================
   کارت دسته‌بندی - تم پودری با بلور تیره
   ========================================================= */

.jb-cat-card {
  display: block;
  position: relative;
  width: 100%;
  aspect-ratio: 1 / 1;
  border-radius: 24px;
  overflow: hidden;
  border: 2px solid var(--jb-pink-200);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.12);
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  text-decoration: none;
  background: #fff;
}

.jb-cat-card:hover {
  transform: translateY(-6px);
  border-color: var(--jb-pink-400);
  box-shadow: 0 20px 45px rgba(214, 51, 132, 0.28);
}

/* ===== مدیا (تصویر یا گرادیان) ===== */
.jb-cat-media {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.jb-cat-img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.6s ease;
}

.jb-cat-card:hover .jb-cat-img {
  transform: scale(1.08);
}

.jb-cat-gradient {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  transition: transform 0.6s ease;
}

.jb-cat-card:hover .jb-cat-gradient {
  transform: scale(1.08);
}

/* ===== لایه بلور مشکی ملایم ===== */
.jb-cat-blur-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    180deg,
    rgba(0, 0, 0, 0.05) 0%,
    rgba(0, 0, 0, 0.15) 40%,
    rgba(0, 0, 0, 0.45) 100%
  );
  backdrop-filter: blur(0.5px);
  -webkit-backdrop-filter: blur(2px);
  z-index: 2;
  transition: all 0.35s ease;
}

.jb-cat-card:hover .jb-cat-blur-overlay {
  background: linear-gradient(
    180deg,
    rgba(0, 0, 0, 0.1) 0%,
    rgba(0, 0, 0, 0.25) 40%,
    rgba(157, 24, 66, 0.247) 100%
  );
  backdrop-filter: blur(0px);
  -webkit-backdrop-filter: blur(1px);
}

/* ===== عنوان وسط کارت ===== */
.jb-cat-title-wrap {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
  z-index: 3;
  padding: 16px;
  text-align: center;
}

.jb-cat-title {
  color: #fff;
  font-size: 1.05rem;
  font-weight: 900;
  line-height: 1.4;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.45);
  transition: all 0.35s ease;
  letter-spacing: 0.3px;
}

.jb-cat-card:hover .jb-cat-title {
  transform: translateY(-4px);
  text-shadow: 0 4px 14px rgba(0, 0, 0, 0.6);
}

.jb-cat-line {
  display: block;
  width: 32px;
  height: 3px;
  border-radius: 999px;
  background: linear-gradient(90deg, var(--jb-pink-300), #fff, var(--jb-pink-300));
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
  transition: all 0.35s ease;
}

.jb-cat-card:hover .jb-cat-line {
  width: 52px;
  background: linear-gradient(90deg, #fff, var(--jb-pink-300), #fff);
}

/* ===== ریسپانسیو ===== */
@media (max-width: 768px) {
  .jb-cat-title {
    font-size: 0.95rem;
  }
  .jb-cat-title-wrap {
    gap: 8px;
  }
  .jb-cat-line {
    width: 26px;
    height: 2.5px;
  }
}

@media (max-width: 480px) {
  .jb-cat-card {
    border-radius: 18px;
  }
  .jb-cat-title {
    font-size: 0.85rem;
  }
}
</style>