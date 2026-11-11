<!-- app/components/sections/home/AboutStoreSection.vue -->
<template>
  <section class="jb-about-section">
    <div class="container mx-auto px-4 md:px-8 lg:px-12 xl:px-28">
      <div class="jb-about-card">
        <!-- آیکون تزئینی بالا -->
        <div class="jb-about-badge">
          <span>🍼</span>
        </div>

        <!-- تایتل -->
        <h1 class="jb-about-title">
          فروشگاه
          <span class="jb-about-title-strong">فروشگاه دکتر اسماعیلی</span>
          ، انتخابی شایسته
        </h1>

        <!-- خط تزئینی -->
        <div class="jb-about-line"></div>

        <!-- متن -->
        <p class="jb-about-desc">
          {{ aboutData.description }}
        </p>

        <!-- دکمه -->
        <NuxtLink :to="aboutData.ctaLink" class="jb-btn-linear jb-about-cta">
          <span>{{ aboutData.ctaText }}</span>
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
          </svg>
        </NuxtLink>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
// ⛳️ بعداً فقط این data میاد از API / store
const aboutData = {
  title: "فروشگاه فروشگاه دکتر اسماعیلی انتخابی شایسته",
  description: `در فروشگاه دکتر اسماعیلی، هر وسیله‌ای که برای کوچولوی شما لازم است، با دقت و وسواس انتخاب شده؛
  از لباس‌های نرم و راحت گرفته تا اسباب‌بازی‌های رنگارنگ و لوازم بهداشتی.
  هدف ما این است که تجربه‌ی خرید سیسمونی را برای شما ساده، دلنشین و به‌یادماندنی کنیم.`,
  ctaText: "بیشتر بدانید",
  ctaLink: "/about-us",
};
</script>

<style scoped>
/* =========================================================
   سکشن درباره فروشگاه - تم پودری
   ========================================================= */

.jb-about-section {
  padding: 64px 0;
  background: linear-gradient(
    135deg,
    #fff8fb 0%,
    #ffeaf2 50%,
    #ffe0eb 100%
  );
  position: relative;
  overflow: hidden;
}

/* حباب‌های تزئینی پس‌زمینه */
.jb-about-section::before,
.jb-about-section::after {
  content: "";
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  z-index: 0;
}

.jb-about-section::before {
  width: 280px;
  height: 280px;
  background: radial-gradient(circle, rgba(255, 122, 160, 0.18), transparent 70%);
  top: -80px;
  right: -60px;
}

.jb-about-section::after {
  width: 220px;
  height: 220px;
  background: radial-gradient(circle, rgba(214, 51, 132, 0.12), transparent 70%);
  bottom: -60px;
  left: -40px;
}

/* ===== کارت شیشه‌ای ===== */
.jb-about-card {
  position: relative;
  z-index: 1;
  max-width: 900px;
  margin: 0 auto;
  padding: 56px 40px 48px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 2px solid rgba(255, 184, 207, 0.7);
  border-radius: 32px;
  box-shadow:
    0 25px 55px rgba(214, 51, 132, 0.15),
    0 10px 25px rgba(255, 122, 160, 0.18),
    inset 0 1px 0 rgba(255, 255, 255, 0.9);
  text-align: center;
  transition: all 0.35s ease;
}

.jb-about-card:hover {
  box-shadow:
    0 30px 65px rgba(214, 51, 132, 0.22),
    0 12px 30px rgba(255, 122, 160, 0.22),
    inset 0 1px 0 rgba(255, 255, 255, 0.9);
  transform: translateY(-3px);
}

/* ===== بج تزئینی ===== */
.jb-about-badge {
  position: absolute;
  top: -28px;
  left: 50%;
  transform: translateX(-50%);
  width: 62px;
  height: 62px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 20px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.45);
  border: 3px solid #fff;
  animation: floatBadge 3s ease-in-out infinite;
  font-size: 1.5rem;
}

@keyframes floatBadge {
  0%, 100% { transform: translateX(-50%) translateY(0); }
  50%      { transform: translateX(-50%) translateY(-6px); }
}

/* ===== تایتل ===== */
.jb-about-title {
  font-size: 1.2rem;
  font-weight: 900;
  line-height: 1.7;
  color: var(--jb-text);
  margin-bottom: 16px;
  display: inline-flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 6px;
  letter-spacing: -0.3px;
}

.jb-about-title-strong {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 900;
}

/* ===== خط تزئینی ===== */
.jb-about-line {
  width: 90px;
  height: 3px;
  margin: 0 auto 24px;
  border-radius: 999px;
  background: linear-gradient(
    90deg,
    transparent,
    var(--jb-pink-400),
    var(--jb-fuchsia),
    var(--jb-pink-400),
    transparent
  );
  box-shadow: 0 2px 8px rgba(214, 51, 132, 0.25);
}

/* ===== متن ===== */
.jb-about-desc {
  font-size: 0.95rem;
  line-height: 2.1;
  color: var(--jb-text-soft);
  font-weight: 500;
  max-width: 680px;
  margin: 0 auto 32px;
  text-align: center;
  padding: 0 8px;
}

/* ===== دکمه ===== */
.jb-about-cta {
  /* فقط وسط‌چین */
  display: inline-flex;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 768px) {
  .jb-about-section {
    padding: 48px 0;
  }

  .jb-about-card {
    padding: 44px 22px 36px;
    border-radius: 26px;
  }

  .jb-about-badge {
    width: 54px;
    height: 54px;
    top: -24px;
    font-size: 1.3rem;
    border-radius: 16px;
  }

  .jb-about-title {
    font-size: 1.05rem;
    gap: 4px;
  }

  .jb-about-line {
    width: 70px;
    margin-bottom: 20px;
  }

  .jb-about-desc {
    font-size: 0.88rem;
    line-height: 2;
    margin-bottom: 26px;
  }
}

@media (max-width: 480px) {
  .jb-about-card {
    padding: 38px 18px 32px;
  }
  .jb-about-title {
    font-size: 0.98rem;
  }
  .jb-about-desc {
    font-size: 0.84rem;
    line-height: 1.95;
  }
}

@media (min-width: 1024px) {
  .jb-about-title {
    font-size: 1.45rem;
  }
  .jb-about-desc {
    font-size: 1rem;
  }
}
</style>