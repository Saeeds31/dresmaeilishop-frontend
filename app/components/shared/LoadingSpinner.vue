<template>
  <div class="jb-loader" :class="sizeClass" role="status" aria-label="در حال بارگذاری">
    <!-- حالت حلقه‌ای برای سایزهای کوچک -->
    <svg v-if="isSmall" class="jb-loader-ring" viewBox="0 0 50 50">
      <circle class="jb-loader-ring-bg" cx="25" cy="25" r="20" fill="none" stroke-width="4" />
      <circle class="jb-loader-ring-fg" cx="25" cy="25" r="20" fill="none" stroke-width="4" />
    </svg>

    <!-- حالت خطی برای سایزهای بزرگ -->
    <div v-else class="jb-loader-bar">
      <div class="jb-loader-bar-fill"></div>
    </div>
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
const isSmall = computed(() => ["xs", "sm"].includes(props.size));
</script>

<style scoped>
/* =========================================================
   لودر - مینیمال سبز
   ========================================================= */

.jb-loader {
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

/* ===== حالت حلقه‌ای (xs, sm) ===== */
.jb-loader-ring {
  animation: jbRotate 1.2s linear infinite;
}

.jb-loader-ring-bg {
  stroke: rgba(14, 90, 58, 0.12);
}

.jb-loader-ring-fg {
  stroke: #0e5a3a;
  stroke-linecap: round;
  stroke-dasharray: 90, 150;
  stroke-dashoffset: 0;
  animation: jbDash 1.4s ease-in-out infinite;
  transform-origin: center;
}

@keyframes jbRotate {
  100% { transform: rotate(360deg); }
}

@keyframes jbDash {
  0% {
    stroke-dasharray: 1, 150;
    stroke-dashoffset: 0;
  }
  50% {
    stroke-dasharray: 90, 150;
    stroke-dashoffset: -35;
  }
  100% {
    stroke-dasharray: 90, 150;
    stroke-dashoffset: -124;
  }
}

/* ===== حالت خطی (md, lg, xl) ===== */
.jb-loader-bar {
  width: 100%;
  height: 4px;
  background: rgba(14, 90, 58, 0.1);
  border-radius: 2px;
  overflow: hidden;
}

.jb-loader-bar-fill {
  width: 40%;
  height: 100%;
  background: linear-gradient(90deg, #0e5a3a, #1a7a50);
  border-radius: 2px;
  animation: jbBarMove 1.4s ease-in-out infinite;
}

@keyframes jbBarMove {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(350%);
  }
}

/* =========================================================
   سایزها
   ========================================================= */

/* xs */
.size-xs .jb-loader-ring {
  width: 18px;
  height: 18px;
}

/* sm */
.size-sm .jb-loader-ring {
  width: 24px;
  height: 24px;
}

/* md */
.size-md {
  width: 80px;
}
.size-md .jb-loader-bar {
  height: 3px;
}

/* lg */
.size-lg {
  width: 120px;
}
.size-lg .jb-loader-bar {
  height: 4px;
}

/* xl */
.size-xl {
  width: 180px;
}
.size-xl .jb-loader-bar {
  height: 5px;
}
</style>