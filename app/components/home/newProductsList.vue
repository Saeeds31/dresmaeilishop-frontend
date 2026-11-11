<!-- app/components/home/OffersSection.vue -->
<template>
  <section class="w-full py-10 mb-10 md:mb-16">
    <div class=" mx-auto px-4 md:px-0 ">
      <!-- هدر سکشن: وکتور وسط + عنوان + دکمه -->
      <div class="relative w-full jb-offers-wrapper py-8 md:py-10  overflow-hidden">
        <div class="relative z-10 flex flex-col items-center justify-center gap-3 mb-6 md:mb-8">
          <h2 class="text-lg md:text-2xl font-black jb-text-pink text-center">
            جدیدترین‌های فروشگاه دکتر اسماعیلی
          </h2>

          <!-- خط تزئینی زیر عنوان -->
          <div class="jb-divider w-24 md:w-32"></div>
        </div>

        <!-- اسلایدر محصولات -->
        <div class="relative z-10 w-full">
          <ClientOnly>
            <HomeSlidersHomeOfferSlider :products="new_products" />
          </ClientOnly>
        </div>

        <!-- دکمه مشاهده همه -->
        <div class="relative z-10 flex justify-center mt-8">
          <NuxtLink to="/products" class="jb-btn-linear">
            <span>مشاهده همه محصولات</span>
            <svg
              class="w-5 h-5"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2.5"
                d="M15 19l-7-7 7-7"
              />
            </svg>
          </NuxtLink>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
defineProps(["new_products"]);
</script>

<style scoped>
/* ===== wrapper با پس‌زمینه پودری ===== */
.jb-offers-wrapper {
  background: linear-gradient(135deg, #fff0f5 0%, #ffe0eb 50%, #ffd0e0 100%);
  border: 2px solid var(--jb-pink-200);
  box-shadow: 0 20px 45px rgba(214, 51, 132, 0.15);
}

/* ===== بج وکتور ===== */
.jb-vector-badge {
  width: 64px;
  height: 64px;
  border-radius: 20px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.45);
  border: 3px solid #fff;
  animation: floatBadge 3s ease-in-out infinite;
}

@keyframes floatBadge {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-6px); }
}

/* ===== دکمه Linear ===== */
.jb-btn-linear {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 12px 32px;
  border-radius: 999px;
  font-weight: 800;
  font-size: 0.95rem;
  color: #fff;
  background: linear-gradient(
    90deg,
    var(--jb-pink-500) 0%,
    var(--jb-fuchsia) 50%,
    var(--jb-pink-600) 100%
  );
  background-size: 200% auto;
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.45);
  transition: all 0.4s ease;
  border: 2px solid rgba(255, 255, 255, 0.5);
  position: relative;
  overflow: hidden;
}

.jb-btn-linear::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(
    90deg,
    transparent 0%,
    rgba(255, 255, 255, 0.35) 50%,
    transparent 100%
  );
  transform: translateX(-100%);
  transition: transform 0.6s ease;
}

.jb-btn-linear:hover {
  background-position: 100% center;
  transform: translateY(-3px);
  box-shadow: 0 14px 32px rgba(214, 51, 132, 0.6);
}

.jb-btn-linear:hover::before {
  transform: translateX(100%);
}

.jb-btn-linear svg {
  transition: transform 0.3s ease;
}

.jb-btn-linear:hover svg {
  transform: translateX(-5px);
}
</style>