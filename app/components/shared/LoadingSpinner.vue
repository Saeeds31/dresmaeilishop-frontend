<template>
  <div class="jb-loader" :class="sizeClass">
    <!-- هاله‌ی تزئینی پس‌زمینه -->
    <span class="jb-loader-glow"></span>

    <!-- سه حباب پودری -->
    <span class="jb-loader-dot"></span>
    <span class="jb-loader-dot"></span>
    <span class="jb-loader-dot"></span>
  </div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  size: {
    type: String,
    default: "md",
    validator: (v) => ["xs", "sm", "md", "lg", "xl"].includes(v),
  },
});

const sizeClass = computed(() => `size-${props.size}`);
</script>

<style scoped>
/* =========================================================
   لودر - سه حباب پودری با هاله‌ی گرادیانی
   ========================================================= */

.jb-loader {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 6px 10px;
}

/* هاله‌ی نرم پشت حباب‌ها */
.jb-loader-glow {
  position: absolute;
  inset: 0;
  border-radius: 999px;
  background: radial-gradient(
    circle,
    rgba(255, 122, 160, 0.25) 0%,
    rgba(255, 122, 160, 0.08) 50%,
    transparent 80%
  );
  animation: jbGlowPulse 1.6s ease-in-out infinite;
  z-index: 0;
}

@keyframes jbGlowPulse {
  0%, 100% {
    opacity: 0.6;
    transform: scale(0.95);
  }
  50% {
    opacity: 1;
    transform: scale(1.08);
  }
}

/* ===== حباب‌ها ===== */
.jb-loader-dot {
  position: relative;
  z-index: 1;
  display: block;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-400), var(--jb-fuchsia));
  box-shadow:
    0 4px 12px rgba(214, 51, 132, 0.4),
    inset 0 -2px 4px rgba(110, 15, 46, 0.25),
    inset 0 2px 3px rgba(255, 255, 255, 0.5);
  animation: jbBounce 1.1s ease-in-out infinite both;
}

.jb-loader-dot:nth-child(2) { animation-delay: -0.2s; }
.jb-loader-dot:nth-child(3) { animation-delay: -0.4s; }

@keyframes jbBounce {
  0%, 80%, 100% {
    transform: translateY(0) scale(1);
    opacity: 0.7;
  }
  40% {
    transform: translateY(-45%) scale(1.15);
    opacity: 1;
  }
}

/* =========================================================
   سایزها
   ========================================================= */

/* xs */
.size-xs {
  gap: 4px;
  padding: 4px 6px;
}
.size-xs .jb-loader-dot {
  width: 6px;
  height: 6px;
}

/* sm */
.size-sm {
  gap: 5px;
  padding: 5px 8px;
}
.size-sm .jb-loader-dot {
  width: 8px;
  height: 8px;
}

/* md */
.size-md {
  gap: 6px;
  padding: 6px 10px;
}
.size-md .jb-loader-dot {
  width: 11px;
  height: 11px;
}

/* lg */
.size-lg {
  gap: 8px;
  padding: 8px 14px;
}
.size-lg .jb-loader-dot {
  width: 15px;
  height: 15px;
}

/* xl */
.size-xl {
  gap: 10px;
  padding: 10px 18px;
}
.size-xl .jb-loader-dot {
  width: 20px;
  height: 20px;
}
</style>