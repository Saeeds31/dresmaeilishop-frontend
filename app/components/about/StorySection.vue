<template>
  <section class="jb-about-story">
    <!-- ===== ستون تصویر ===== -->
    <div class="jb-about-story-image">
      <div class="jb-about-story-image-wrap">
        <img
          :src="imageSrc"
          :alt="imageAlt"
          @error="onImageError"
          class="jb-about-story-img"
        />

        <!-- بج تزئینی روی تصویر -->
        <span class="jb-about-story-badge">
          <span class="jb-about-story-badge-icon">
            <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
              <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z" />
            </svg>
          </span>
          <span>با عشق برای کودکان</span>
        </span>
      </div>
    </div>

    <!-- ===== ستون متن ===== -->
    <div class="jb-about-story-content">
      <!-- تیتر -->
      <div class="jb-about-story-title-wrap">
        <span class="jb-about-story-title-icon">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253" />
          </svg>
        </span>
        <h2 class="jb-about-story-title">
          <span class="jb-about-story-title-strong">داستان ما</span>
        </h2>
      </div>

      <!-- خط تزئینی -->
      <div class="jb-about-story-divider"></div>

      <!-- پاراگراف‌ها -->
      <div class="jb-about-story-paragraphs">
        <p
          v-for="(paragraph, index) in paragraphs"
          :key="index"
          class="jb-about-story-paragraph"
        >
          {{ paragraph }}
        </p>
      </div>

      <!-- هایلایت -->
      <div v-if="highlight" class="jb-about-story-highlight">
        <span class="jb-highlight-quote">❞</span>
        <p>{{ highlight }}</p>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps({
  title: { type: String, default: "" },
  paragraphs: { type: Array, default: () => [] },
  highlight: { type: String, default: "" },
  imageSrc: { type: String, required: true },
  imageAlt: { type: String, default: "" },
  fallbackSrc: { type: String, default: "" },
});

const hasError = ref(false);

const onImageError = (e) => {
  if (!hasError.value && props.fallbackSrc) {
    hasError.value = true;
    e.target.src = props.fallbackSrc;
  }
};
</script>

<style scoped>
/* =========================================================
   Story بخش داستان ما - تم پودری
   ========================================================= */

.jb-about-story {
  display: flex;
  flex-direction: column;
  gap: 32px;
  margin-top: 60px;
  padding: 40px 32px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 32px;
  box-shadow: 0 15px 40px rgba(214, 51, 132, 0.12);
  position: relative;
  overflow: hidden;
}

@media (min-width: 1024px) {
  .jb-about-story {
    flex-direction: row;
    align-items: center;
    gap: 48px;
    padding: 56px 48px;
  }
}

/* ===== تصویر ===== */
.jb-about-story-image {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.jb-about-story-image-wrap {
  position: relative;
  width: 100%;
  max-width: 320px;
  padding: 20px;
  border-radius: 32px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px dashed var(--jb-pink-300);
  box-shadow: 0 20px 45px rgba(214, 51, 132, 0.15);
  animation: jbStoryFloat 4.5s ease-in-out infinite;
}

@keyframes jbStoryFloat {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  50%      { transform: translateY(-8px) rotate(-2deg); }
}

.jb-about-story-img {
  width: 100%;
  height: auto;
  object-fit: contain;
  mix-blend-mode: multiply;
}

/* بج روی تصویر */
.jb-about-story-badge {
  position: absolute;
  bottom: -12px;
  right: 16px;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  font-size: 0.75rem;
  font-weight: 900;
  border-radius: 999px;
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.45);
  border: 3px solid #fff;
  z-index: 3;
}

.jb-about-story-badge-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
}

/* ===== محتوای متنی ===== */
.jb-about-story-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 18px;
}

/* تیتر */
.jb-about-story-title-wrap {
  display: inline-flex;
  align-items: center;
  gap: 12px;
}

.jb-about-story-title-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 42px;
  height: 42px;
  border-radius: 14px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
  flex-shrink: 0;
}

.jb-about-story-title {
  font-size: 1.4rem;
  font-weight: 900;
  color: var(--jb-text);
  letter-spacing: -0.5px;
}

.jb-about-story-title-strong {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

@media (min-width: 768px) {
  .jb-about-story-title {
    font-size: 1.6rem;
  }
}

/* خط تزئینی */
.jb-about-story-divider {
  width: 90px;
  height: 3px;
  border-radius: 999px;
  background: linear-gradient(90deg, var(--jb-pink-500), var(--jb-fuchsia), transparent);
  box-shadow: 0 2px 8px rgba(214, 51, 132, 0.25);
  margin-bottom: 8px;
}

/* پاراگراف‌ها */
.jb-about-story-paragraphs {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.jb-about-story-paragraph {
  font-size: 0.92rem;
  line-height: 2.15;
  color: var(--jb-text-soft);
  font-weight: 600;
  text-align: justify;
  padding-right: 16px;
  position: relative;
}

.jb-about-story-paragraph::before {
  content: "";
  position: absolute;
  right: 0;
  top: 0.9em;
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.15);
}

/* هایلایت */
.jb-about-story-highlight {
  position: relative;
  margin-top: 8px;
  padding: 20px 22px 20px 20px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border-radius: 18px;
  border-right: 4px solid transparent;
  border-image: linear-gradient(180deg, var(--jb-pink-500), var(--jb-fuchsia)) 1;
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.1);
}

.jb-about-story-highlight p {
  font-size: 0.88rem;
  line-height: 2.1;
  color: var(--jb-text);
  font-weight: 700;
  font-style: italic;
}

.jb-highlight-quote {
  position: absolute;
  top: -14px;
  right: 18px;
  font-size: 2.2rem;
  color: var(--jb-pink-400);
  line-height: 1;
  background: #fff;
  padding: 0 8px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.15);
}

/* ===== ریسپانسیو ===== */
@media (max-width: 640px) {
  .jb-about-story {
    padding: 24px 20px;
    border-radius: 24px;
    gap: 24px;
    margin-top: 40px;
  }

  .jb-about-story-title {
    font-size: 1.15rem;
  }

  .jb-about-story-title-icon {
    width: 36px;
    height: 36px;
  }

  .jb-about-story-paragraph {
    font-size: 0.85rem;
    line-height: 2;
  }

  .jb-about-story-highlight p {
    font-size: 0.82rem;
  }

  .jb-about-story-image-wrap {
    max-width: 240px;
    padding: 14px;
  }
}
</style>