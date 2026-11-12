<template>
  <section class="jb-contact-hero">
    <!-- ===== ستون راست: محتوا ===== -->
    <div class="jb-contact-hero-content">
      <!-- تیتر -->
      <div class="jb-hero-title-wrap">
        <span class="jb-hero-title-icon">
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
          </svg>
        </span>
        <h1 class="jb-hero-title">
          تماس <span class="jb-hero-title-strong">با ما</span>
        </h1>
      </div>

      <!-- خط تزئینی -->
      <div class="jb-hero-divider"></div>

      <!-- متن معرفی -->
      <ul class="jb-hero-list">
        <li class="jb-hero-list-item">
          <span class="jb-hero-list-dot"></span>
          <span>
            در فروشگاه دکتر اسماعیلی، همراه شما هستیم تا برای سلامت و زیبایی خود بهترین
            انتخاب‌ها را با تجربه‌ای صادقانه و پشتیبانی سریع فراهم کنیم.
          </span>
        </li>
        <li class="jb-hero-list-item">
          <span class="jb-hero-list-dot"></span>
          <span>
            ارتباط آسان، پاسخگویی بی‌وقفه. تیم پشتیبانی ما همیشه آماده است تا
            در انتخاب محصولات مراقبتی، مکمل‌ها و لوازم بهداشتی به شما مشاوره بدهد.
          </span>
        </li>
      </ul>

      <!-- ===== کارت‌های تماس ===== -->
      <div class="jb-contact-cards">
        <!-- آدرس -->
        <div class="jb-contact-card">
          <div class="jb-contact-card-header">
            <span class="jb-contact-card-icon">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M17.657 16.657L13.414 20.9a2 2 0 01-2.828 0l-4.243-4.243a8 8 0 1111.314 0z" />
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
              </svg>
            </span>
            <span class="jb-contact-card-label">آدرس داروخانه</span>
          </div>
          <p class="jb-contact-card-value">{{ contactInfo.address }}</p>
        </div>

        <!-- تماس -->
        <div class="jb-contact-card">
          <div class="jb-contact-card-header">
            <span class="jb-contact-card-icon">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z" />
              </svg>
            </span>
            <span class="jb-contact-card-label">شماره تماس</span>
          </div>
          <div class="jb-contact-card-phones">
            <a :href="`tel:${contactInfo.mobile}`" class="jb-phone-link" dir="ltr">
              {{ contactInfo.mobile }}
            </a>
            <span class="jb-phone-sep">|</span>
            <a :href="`tel:${contactInfo.phone}`" class="jb-phone-link" dir="ltr">
              {{ contactInfo.phone }}
            </a>
          </div>
        </div>

        <!-- پشتیبانی -->
        <div class="jb-contact-card">
          <div class="jb-contact-card-header">
            <span class="jb-contact-card-icon">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M18.364 5.636l-3.536 3.536m0 5.656l3.536 3.536M9.172 9.172L5.636 5.636m3.536 9.192l-3.536 3.536M21 12a9 9 0 11-18 0 9 9 0 0118 0zm-5 0a4 4 0 11-8 0 4 4 0 018 0z" />
              </svg>
            </span>
            <span class="jb-contact-card-label">پشتیبانی</span>
          </div>
          <div class="jb-contact-card-row">
            <span class="jb-contact-card-value">{{ contactInfo.supportHours }}</span>
            <span class="jb-contact-card-sub">شنبه تا پنج‌شنبه</span>
          </div>
        </div>

        <!-- ساعت کاری -->
        <div class="jb-contact-card">
          <div class="jb-contact-card-header">
            <span class="jb-contact-card-icon">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
            </span>
            <span class="jb-contact-card-label">ساعت کاری داروخانه</span>
          </div>
          <div class="jb-contact-card-row">
            <span class="jb-contact-card-value">{{ contactInfo.storeHours }}</span>
            <span class="jb-contact-card-sub">شنبه تا پنج‌شنبه</span>
          </div>
        </div>

        <!-- تلگرام -->
        <a
          :href="contactInfo.telegram"
          target="_blank"
          rel="noopener"
          class="jb-contact-social jb-social-telegram"
        >
          <span class="jb-social-icon">
            <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
              <path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.447 1.394c-.16.16-.295.295-.605.295l.213-3.053 5.56-5.023c.24-.213-.054-.333-.373-.12L7.24 13.51l-2.96-.924c-.643-.204-.657-.643.136-.953l11.57-4.461c.537-.194 1.006.131.908.95z"/>
            </svg>
          </span>
          <div class="jb-social-info">
            <span class="jb-social-name">تلگرام</span>
            <span class="jb-social-id">@{{ contactInfo.telegramId }}</span>
          </div>
          <svg class="jb-social-arrow" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
          </svg>
        </a>

        <!-- اینستاگرام -->
        <a
          :href="contactInfo.instagram"
          target="_blank"
          rel="noopener"
          class="jb-contact-social jb-social-instagram"
        >
          <span class="jb-social-icon">
            <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
              <path d="M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.069 1.645.069 4.849 0 3.205-.012 3.584-.069 4.849-.149 3.225-1.664 4.771-4.919 4.919-1.266.058-1.644.07-4.85.07-3.204 0-3.584-.012-4.849-.07-3.26-.149-4.771-1.699-4.919-4.92-.058-1.265-.07-1.644-.07-4.849 0-3.204.013-3.583.07-4.849.149-3.227 1.664-4.771 4.919-4.919 1.266-.057 1.645-.069 4.849-.069zm0-2.163C8.741 0 8.333.014 7.053.072 2.695.272.273 2.69.073 7.052.014 8.333 0 8.741 0 12c0 3.259.014 3.668.072 4.948.2 4.358 2.618 6.78 6.98 6.98C8.333 23.986 8.741 24 12 24c3.259 0 3.668-.014 4.948-.072 4.354-.2 6.782-2.618 6.979-6.98.059-1.28.073-1.689.073-4.948 0-3.259-.014-3.667-.072-4.947-.196-4.354-2.617-6.78-6.979-6.98C15.668.014 15.259 0 12 0zm0 5.838a6.162 6.162 0 100 12.324 6.162 6.162 0 000-12.324zM12 16a4 4 0 110-8 4 4 0 010 8zm6.406-11.845a1.44 1.44 0 100 2.881 1.44 1.44 0 000-2.881z"/>
            </svg>
          </span>
          <div class="jb-social-info">
            <span class="jb-social-name">اینستاگرام</span>
            <span class="jb-social-id">@{{ contactInfo.instagramId }}</span>
          </div>
          <svg class="jb-social-arrow" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
          </svg>
        </a>
      </div>
    </div>

    <!-- ===== ستون چپ: تصویر ===== -->
    <div class="jb-contact-hero-image">
      <div class="jb-hero-image-wrap">
        <img src="/contact/phone-hand.svg" alt="تماس با ما" />
      </div>
    </div>
  </section>
</template>

<script setup>
import { joinURL } from "ufo";

defineProps({
  contactInfo: { type: Object, required: true },
});

const config = useRuntimeConfig();

const asset = (file) => {
  return joinURL(config.app.baseURL || "/", "contact", file);
};
</script>

<style scoped>
/* =========================================================
   Hero تماس با ما - تم سبز لوکس
   ========================================================= */

.jb-contact-hero {
  display: flex;
  width: 100%;
  gap: 32px;
  align-items: flex-start;
}

.jb-contact-hero-content {
  flex: 1;
  min-width: 0;
}

.jb-contact-hero-image {
  flex-shrink: 0;
  width: 33%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.jb-hero-image-wrap {
  width: 100%;
  max-width: 320px;
  padding: 20px;
  border-radius: 32px;
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
  border: 2px dashed rgba(140, 198, 63, 0.5);
  box-shadow: 0 20px 45px rgba(14, 90, 58, 0.12);
  animation: jbHeroFloat 4s ease-in-out infinite;
}

@keyframes jbHeroFloat {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-10px); }
}

.jb-hero-image-wrap img {
  width: 100%;
  height: auto;
  object-fit: contain;
  mix-blend-mode: multiply;
}

/* ===== تیتر ===== */
.jb-hero-title-wrap {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}

.jb-hero-title-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 46px;
  height: 46px;
  border-radius: 15px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.3);
  border: 3px solid #fff;
}

.jb-hero-title {
  font-size: 1.6rem;
  font-weight: 900;
  color: #1a202c;
  letter-spacing: -0.5px;
}

.jb-hero-title-strong {
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 900;
}

/* خط تزئینی */
.jb-hero-divider {
  width: 90px;
  height: 3px;
  border-radius: 999px;
  background: linear-gradient(90deg, #0e5a3a, #8cc63f, transparent);
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.2);
  margin-bottom: 24px;
}

/* ===== لیست معرفی ===== */
.jb-hero-list {
  display: flex;
  flex-direction: column;
  gap: 14px;
  margin-bottom: 32px;
  list-style: none;
  padding: 0;
}

.jb-hero-list-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  font-size: 0.92rem;
  line-height: 2;
  color: #4a5568;
  font-weight: 600;
}

.jb-hero-list-dot {
  flex-shrink: 0;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  box-shadow: 0 0 0 3px rgba(140, 198, 63, 0.18);
  margin-top: 0.7em;
}

/* ===== کارت‌های تماس ===== */
.jb-contact-cards {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
}

@media (min-width: 640px) {
  .jb-contact-cards {
    grid-template-columns: repeat(2, 1fr);
  }
}

.jb-contact-card {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 16px 18px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid rgba(14, 90, 58, 0.1);
  border-radius: 18px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 14px rgba(14, 90, 58, 0.06);
}

.jb-contact-card:hover {
  border-color: rgba(14, 90, 58, 0.25);
  box-shadow: 0 10px 25px rgba(14, 90, 58, 0.12);
  transform: translateY(-3px);
}

.jb-contact-card-header {
  display: inline-flex;
  align-items: center;
  gap: 10px;
}

.jb-contact-card-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 11px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  box-shadow: 0 6px 14px rgba(14, 90, 58, 0.25);
  border: 2px solid #fff;
  flex-shrink: 0;
}

.jb-contact-card-label {
  font-size: 0.82rem;
  font-weight: 900;
  color: #0e5a3a;
}

.jb-contact-card-value {
  font-size: 0.85rem;
  font-weight: 800;
  color: #1a202c;
  line-height: 1.8;
}

.jb-contact-card-phones {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.jb-phone-link {
  font-family: 'Courier New', monospace;
  font-size: 0.85rem;
  font-weight: 900;
  color: #1a202c;
  text-decoration: none;
  transition: color 0.25s ease;
  letter-spacing: 0.3px;
}

.jb-phone-link:hover {
  color: #0e5a3a;
}

.jb-phone-sep {
  color: rgba(140, 198, 63, 0.5);
}

.jb-contact-card-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  flex-wrap: wrap;
}

.jb-contact-card-sub {
  font-size: 0.72rem;
  color: #718096;
  font-weight: 700;
}

/* ===== کارت‌های شبکه اجتماعی ===== */
.jb-contact-social {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 18px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid rgba(14, 90, 58, 0.1);
  border-radius: 18px;
  text-decoration: none;
  transition: all 0.3s ease;
  box-shadow: 0 4px 14px rgba(14, 90, 58, 0.06);
}

.jb-contact-social:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 28px rgba(14, 90, 58, 0.15);
}

.jb-social-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 42px;
  height: 42px;
  border-radius: 13px;
  color: #fff;
  flex-shrink: 0;
  border: 2px solid #fff;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
  transition: all 0.3s ease;
}

.jb-social-telegram .jb-social-icon {
  background: linear-gradient(135deg, #29a9eb, #0088cc);
}

.jb-social-instagram .jb-social-icon {
  background: linear-gradient(135deg, #f58529, #dd2a7b, #8134af);
}

.jb-contact-social:hover .jb-social-icon {
  transform: scale(1.08) rotate(-4deg);
}

.jb-social-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
  flex: 1;
  min-width: 0;
}

.jb-social-name {
  font-size: 0.82rem;
  font-weight: 900;
  color: #1a202c;
}

.jb-social-id {
  font-size: 0.72rem;
  color: #718096;
  font-weight: 700;
  font-family: 'Courier New', monospace;
  direction: ltr;
  text-align: left;
}

.jb-social-arrow {
  width: 16px;
  height: 16px;
  color: rgba(140, 198, 63, 0.7);
  flex-shrink: 0;
  transition: all 0.25s ease;
}

.jb-contact-social:hover .jb-social-arrow {
  color: #0e5a3a;
  transform: translateX(-4px);
}

/* ===== ریسپانسیو ===== */
@media (max-width: 1024px) {
  .jb-contact-hero-image {
    display: none;
  }
}

@media (max-width: 640px) {
  .jb-hero-title {
    font-size: 1.3rem;
  }

  .jb-hero-title-icon {
    width: 40px;
    height: 40px;
  }

  .jb-hero-list-item {
    font-size: 0.85rem;
  }

  .jb-contact-card {
    padding: 14px;
  }
}
</style>