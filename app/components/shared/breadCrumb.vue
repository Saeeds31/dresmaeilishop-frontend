<template>
  <nav class="jb-breadcrumb" aria-label="مسیر صفحه">
    <!-- صفحه اصلی -->
    <NuxtLink to="/" class="jb-breadcrumb-home">
      <span class="jb-breadcrumb-home-icon">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M3 12l9-9 9 9M5 10v10a1 1 0 001 1h3a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1h3a1 1 0 001-1V10" />
        </svg>
      </span>
      <span>صفحه اصلی</span>
    </NuxtLink>

    <!-- جداکننده -->
    <span v-if="items.length > 0" class="jb-breadcrumb-sep">
      <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
      </svg>
    </span>

    <!-- آیتم‌ها -->
    <template v-for="(item, index) in items" :key="index">
      <NuxtLink
        v-if="index !== items.length - 1"
        :to="item.href"
        class="jb-breadcrumb-link"
      >
        {{ item.label }}
      </NuxtLink>

      <span v-if="index !== items.length - 1" class="jb-breadcrumb-sep">
        <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
        </svg>
      </span>

      <!-- آخرین آیتم -->
      <span v-else class="jb-breadcrumb-current">
        <span class="jb-breadcrumb-current-dot"></span>
        {{ item.label }}
      </span>
    </template>
  </nav>
</template>

<script setup>
defineProps({
  items: {
    type: Array,
    default: () => [{ label: "درباره ما", href: "/about" }],
  },
});
</script>

<style scoped>
/* =========================================================
   Breadcrumb - تم پودری
   ========================================================= */

.jb-breadcrumb {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
  padding: 10px 16px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 2px solid var(--jb-pink-200);
  border-radius: 14px;
  box-shadow: 0 4px 14px rgba(214, 51, 132, 0.08);
  font-size: 0.78rem;
  font-weight: 700;
  transition: all 0.3s ease;
  margin: 20px 0;
}

.jb-breadcrumb:hover {
  border-color: var(--jb-pink-300);
  box-shadow: 0 6px 18px rgba(214, 51, 132, 0.12);
}

/* ===== آیتم خانه ===== */
.jb-breadcrumb-home {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 5px 10px;
  border-radius: 10px;
  color: var(--jb-text-soft);
  transition: all 0.25s ease;
  text-decoration: none;
  font-weight: 700;
}

.jb-breadcrumb-home:hover {
  background: var(--jb-pink-100);
  color: var(--jb-fuchsia);
  transform: translateY(-1px);
}

.jb-breadcrumb-home-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  border-radius: 8px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 4px 10px rgba(214, 51, 132, 0.35);
  transition: all 0.25s ease;
}

.jb-breadcrumb-home:hover .jb-breadcrumb-home-icon {
  transform: scale(1.08) rotate(-4deg);
  box-shadow: 0 6px 14px rgba(214, 51, 132, 0.5);
}

/* ===== جداکننده ===== */
.jb-breadcrumb-sep {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  color: var(--jb-pink-400);
  opacity: 0.8;
}

/* ===== لینک‌های میانی ===== */
.jb-breadcrumb-link {
  padding: 5px 10px;
  border-radius: 10px;
  color: var(--jb-text-soft);
  transition: all 0.25s ease;
  text-decoration: none;
  font-weight: 700;
}

.jb-breadcrumb-link:hover {
  background: var(--jb-pink-100);
  color: var(--jb-fuchsia);
  transform: translateY(-1px);
}

/* ===== آیتم فعال (آخرین) ===== */
.jb-breadcrumb-current {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 6px 14px;
  border-radius: 10px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  font-weight: 800;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.4);
  animation: breadcrumbPulse 2.5s ease-in-out infinite;
}

.jb-breadcrumb-current-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.35);
}

@keyframes breadcrumbPulse {
  0%, 100% {
    box-shadow: 0 6px 16px rgba(214, 51, 132, 0.4);
  }
  50% {
    box-shadow: 0 8px 22px rgba(214, 51, 132, 0.6);
  }
}

/* ===== ریسپانسیو ===== */
@media (min-width: 640px) {
  .jb-breadcrumb {
    font-size: 0.85rem;
    padding: 11px 18px;
    gap: 10px;
  }
}

@media (min-width: 1024px) {
  .jb-breadcrumb {
    font-size: 0.9rem;
    padding: 12px 20px;
  }
}

@media (max-width: 480px) {
  .jb-breadcrumb {
    padding: 8px 12px;
    gap: 6px;
    font-size: 0.72rem;
  }
  .jb-breadcrumb-home span:not(.jb-breadcrumb-home-icon) {
    display: none;
  }
  .jb-breadcrumb-current-dot {
    display: none;
  }
}
</style>