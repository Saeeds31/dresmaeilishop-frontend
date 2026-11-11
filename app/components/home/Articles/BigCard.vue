<template>
  <NuxtLink
    v-if="card"
    :to="`/blog/${card.slug}`"
    class="jb-article-card"
  >
    <!-- ===== تصویر ===== -->
    <div class="jb-article-media">
      <img
        v-if="card.image"
        :src="$getImageUrl(card.image)"
        :alt="card.title"
        class="jb-article-img"
        loading="lazy"
      />

      <div v-else class="jb-article-placeholder">
        <svg class="w-12 h-12" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
        </svg>
      </div>

      <!-- اورلی -->
      <div class="jb-article-overlay"></div>
    </div>

    <!-- ===== محتوا ===== -->
    <div class="jb-article-content">
      <!-- تاریخ -->
      <div class="jb-article-date">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
        </svg>
        {{ formatDate(card.created_at) }}
      </div>

      <!-- عنوان -->
      <h3 class="jb-article-title">
        {{ card.title }}
      </h3>

      <!-- توضیحات -->
      <p class="jb-article-desc">
        {{ card.short_description || "توضیحات مقاله" }}
      </p>

      <!-- دکمه -->
      <div class="jb-article-cta">
        <span>مشاهده بیشتر</span>
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
        </svg>
      </div>
    </div>
  </NuxtLink>
</template>

<script setup>
const props = defineProps({
  card: {
    type: Object,
    required: true,
    default: () => ({}),
  },
});

const formatDate = (date) => {
  if (!date) return "";
  try {
    return new Date(date).toLocaleDateString("fa-IR", {
      year: "numeric",
      month: "long",
      day: "numeric",
    });
  } catch {
    return "";
  }
};
</script>

<style scoped>
/* =========================================================
   کارت مقاله (BigCard) - تم پودری
   ========================================================= */

.jb-article-card {
  display: flex;
  flex-direction: column;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 24px;
  overflow: hidden;
  text-decoration: none;
  color: inherit;
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.1);
  height: 100%;
}

.jb-article-card:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 16px 40px rgba(214, 51, 132, 0.22);
  transform: translateY(-6px);
}

/* ===== تصویر ===== */
.jb-article-media {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 10;
  overflow: hidden;
  background: linear-gradient(135deg, #fff5f8, #ffe0eb);
  flex-shrink: 0;
}

.jb-article-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.6s ease;
}

.jb-article-card:hover .jb-article-img {
  transform: scale(1.08);
}

.jb-article-placeholder {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--jb-pink-400);
}

.jb-article-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    to top,
    rgba(110, 15, 46, 0.25) 0%,
    transparent 50%
  );
  pointer-events: none;
}

/* ===== محتوا ===== */
.jb-article-content {
  padding: 18px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  flex: 1;
}

/* تاریخ */
.jb-article-date {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 4px 10px;
  background: var(--jb-pink-50);
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 999px;
  color: var(--jb-fuchsia);
  font-size: 0.68rem;
  font-weight: 800;
  width: fit-content;
}

/* عنوان */
.jb-article-title {
  font-size: 0.95rem;
  font-weight: 900;
  color: var(--jb-text);
  line-height: 1.65;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  transition: color 0.25s ease;
  min-height: 3.2em;
}

.jb-article-card:hover .jb-article-title {
  color: var(--jb-fuchsia);
}

/* توضیحات */
.jb-article-desc {
  font-size: 0.78rem;
  line-height: 1.9;
  color: var(--jb-text-soft);
  font-weight: 600;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* دکمه */
.jb-article-cta {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  margin-top: auto;
  padding-top: 12px;
  border-top: 1.5px dashed var(--jb-pink-200);
  color: var(--jb-fuchsia);
  font-size: 0.78rem;
  font-weight: 900;
  transition: all 0.25s ease;
}

.jb-article-cta svg {
  transition: transform 0.3s ease;
}

.jb-article-card:hover .jb-article-cta {
  color: var(--jb-pink-700);
}

.jb-article-card:hover .jb-article-cta svg {
  transform: translateX(-4px);
}

/* ===== ریسپانسیو ===== */
@media (max-width: 640px) {
  .jb-article-card {
    border-radius: 20px;
  }

  .jb-article-content {
    padding: 14px;
    gap: 8px;
  }

  .jb-article-title {
    font-size: 0.85rem;
    min-height: 3em;
  }

  .jb-article-desc {
    font-size: 0.72rem;
  }

  .jb-article-cta {
    font-size: 0.72rem;
  }
}
</style>