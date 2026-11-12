<template>
  <section class="jb-about-hero">
    <!-- ستون راست: متن -->
    <div class="jb-about-hero-content">
      <!-- بج کوچک بالا -->
      <span class="jb-about-hero-badge">
        <span class="jb-about-hero-badge-dot"></span>
        از سال ۱۳۹۵ در کنار شما
      </span>

      <!-- تیتر -->
      <h1 class="jb-about-hero-title">
        دنیای سلامت و زیبایی
        <span class="jb-about-hero-title-strong">فروشگاه دکتر اسماعیلی</span>
      </h1>

      <!-- خط تزئینی -->
      <div class="jb-about-hero-divider"></div>

      <!-- متن -->
      <p class="jb-about-hero-desc">
        از سال ۱۳۹۵ با عشق و علاقه در کنار شما هستیم؛
        فضایی از بهترین محصولات مراقبت از پوست، مو، مکمل‌های غذایی و لوازم بهداشتی
        که با دقت و وسواس انتخاب شده‌اند تا سلامت و زیبایی شما را تضمین کنند.
      </p>

      <!-- ویژگی‌ها -->
      <ul class="jb-about-hero-features">
        <li class="jb-feature-item">
          <span class="jb-feature-icon">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
            </svg>
          </span>
          اصالت و کیفیت تضمین‌شده
        </li>
        <li class="jb-feature-item">
          <span class="jb-feature-icon">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
            </svg>
          </span>
          تنوع بی‌پایان محصولات سلامت
        </li>
        <li class="jb-feature-item">
          <span class="jb-feature-icon">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
            </svg>
          </span>
          ارسال به سراسر کشور
        </li>
      </ul>
    </div>

    <!-- ستون چپ: تصویر -->
    <div class="jb-about-hero-image">
      <div class="jb-about-hero-image-wrap">
        <img
          :src="imageSrc"
          :alt="alt"
          @error="onImageError"
          class="jb-about-hero-img"
        />
        <!-- حلقه‌های تزئینی -->
        <span class="jb-about-ring jb-ring-1"></span>
        <span class="jb-about-ring jb-ring-2"></span>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps({
  imageSrc: { type: String, required: true },
  alt: { type: String, default: "" },
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
   Hero درباره ما - تم سبز لوکس
   ========================================================= */

.jb-about-hero {
  display: flex;
  flex-direction: column;
  gap: 32px;
  padding: 40px 32px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid rgba(14, 90, 58, 0.1);
  border-radius: 32px;
  box-shadow: 0 15px 40px rgba(14, 90, 58, 0.08);
  position: relative;
  overflow: hidden;
}

/* حباب‌های تزئینی پس‌زمینه (سبز ملایم) */
.jb-about-hero::before,
.jb-about-hero::after {
  content: "";
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  z-index: 0;
}

.jb-about-hero::before {
  width: 320px;
  height: 320px;
  background: radial-gradient(circle, rgba(140, 198, 63, 0.12), transparent 70%);
  top: -120px;
  right: -80px;
}

.jb-about-hero::after {
  width: 220px;
  height: 220px;
  background: radial-gradient(circle, rgba(14, 90, 58, 0.08), transparent 70%);
  bottom: -60px;
  left: -40px;
}

@media (min-width: 1024px) {
  .jb-about-hero {
    flex-direction: row;
    align-items: center;
    gap: 48px;
    padding: 56px 48px;
  }
}

/* ===== محتوای متنی ===== */
.jb-about-hero-content {
  position: relative;
  z-index: 1;
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* بج بالا */
.jb-about-hero-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: linear-gradient(135deg, #f4f9f6, #e8f3ec);
  border: 1.5px solid rgba(140, 198, 63, 0.5);
  border-radius: 999px;
  color: #0e5a3a;
  font-size: 0.78rem;
  font-weight: 900;
  width: fit-content;
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.08);
}

.jb-about-hero-badge-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  box-shadow: 0 0 0 3px rgba(140, 198, 63, 0.2);
  animation: jbBadgePulse 2s ease-in-out infinite;
}

@keyframes jbBadgePulse {
  0%, 100% { transform: scale(1); }
  50%      { transform: scale(1.2); }
}

/* تیتر */
.jb-about-hero-title {
  font-size: 1.8rem;
  font-weight: 900;
  color: #1a202c;
  line-height: 1.4;
  letter-spacing: -0.5px;
}

.jb-about-hero-title-strong {
  display: block;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

@media (min-width: 768px) {
  .jb-about-hero-title {
    font-size: 2.2rem;
  }
}

/* خط تزئینی */
.jb-about-hero-divider {
  width: 90px;
  height: 3px;
  border-radius: 999px;
  background: linear-gradient(90deg, #0e5a3a, #8cc63f, transparent);
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.2);
}

/* متن */
.jb-about-hero-desc {
  font-size: 0.95rem;
  line-height: 2.1;
  color: #4a5568;
  font-weight: 600;
  max-width: 520px;
}

/* ویژگی‌ها */
.jb-about-hero-features {
  display: flex;
  flex-direction: column;
  gap: 12px;
  list-style: none;
  padding: 0;
  margin: 4px 0 0;
}

.jb-feature-item {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 0.88rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-feature-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 26px;
  height: 26px;
  border-radius: 8px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.25);
  border: 2px solid #fff;
  flex-shrink: 0;
}

/* ===== تصویر ===== */
.jb-about-hero-image {
  position: relative;
  z-index: 1;
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.jb-about-hero-image-wrap {
  position: relative;
  width: 100%;
  max-width: 340px;
  padding: 24px;
  border-radius: 40px;
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
  border: 2px dashed rgba(140, 198, 63, 0.5);
  box-shadow: 0 20px 45px rgba(14, 90, 58, 0.12);
  animation: jbHeroFloat 4s ease-in-out infinite;
}

@keyframes jbHeroFloat {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-10px); }
}

.jb-about-hero-img {
  width: 100%;
  height: auto;
  object-fit: contain;
  mix-blend-mode: multiply;
  position: relative;
  z-index: 2;
}

/* حلقه‌های تزئینی دور تصویر */
.jb-about-ring {
  position: absolute;
  top: 50%;
  left: 50%;
  border-radius: 50%;
  border: 2px dashed;
  pointer-events: none;
  z-index: 1;
}

.jb-ring-1 {
  width: 90%;
  height: 90%;
  transform: translate(-50%, -50%);
  border-color: rgba(140, 198, 63, 0.3);
  animation: jbRingRotate 25s linear infinite;
}

.jb-ring-2 {
  width: 105%;
  height: 105%;
  transform: translate(-50%, -50%);
  border-color: rgba(14, 90, 58, 0.15);
  animation: jbRingRotate 35s linear infinite reverse;
}

@keyframes jbRingRotate {
  to { transform: translate(-50%, -50%) rotate(360deg); }
}

/* ===== ریسپانسیو ===== */
@media (max-width: 640px) {
  .jb-about-hero {
    padding: 24px 20px;
    border-radius: 24px;
    gap: 24px;
  }

  .jb-about-hero-title {
    font-size: 1.4rem;
  }

  .jb-about-hero-desc {
    font-size: 0.85rem;
    line-height: 1.95;
  }

  .jb-about-hero-image-wrap {
    max-width: 260px;
    padding: 16px;
  }
}
</style>