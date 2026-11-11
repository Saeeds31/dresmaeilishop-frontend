<!-- app/components/product/verity/count.vue -->
<template>
  <div class="jb-count-block">
    <!-- لیبل -->
    <div class="jb-count-label-row">
      <span class="jb-count-label-dot"></span>
      <span class="jb-count-label">تعداد</span>
    </div>

    <!-- کانتینر شمارنده -->
    <div class="jb-count-wrap">
      <!-- دکمه منفی -->
      <button
        type="button"
        class="jb-count-btn"
        :disabled="quantity <= min"
        @click="decrease"
        aria-label="کاهش"
      >
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 12h14" />
        </svg>
      </button>

      <!-- مقدار -->
      <div class="jb-count-value">{{ quantity }}</div>

      <!-- دکمه مثبت -->
      <button
        type="button"
        class="jb-count-btn"
        :disabled="quantity >= max"
        @click="increase"
        aria-label="افزایش"
      >
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M12 5v14M5 12h14" />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, defineProps, defineEmits } from "vue";

const props = defineProps({
  modelValue: { type: Number, default: 1 },
  min: { type: Number, default: 1 },
  max: { type: Number, default: 99 },
});

const emit = defineEmits(["update:modelValue"]);
const quantity = ref(props.modelValue);

watch(quantity, (val) => {
  emit("update:modelValue", val);
});

watch(
  () => props.modelValue,
  (val) => {
    if (val !== quantity.value) quantity.value = val;
  }
);

function increase() {
  if (quantity.value < props.max) quantity.value++;
}

function decrease() {
  if (quantity.value > props.min) quantity.value--;
}
</script>

<style scoped>
/* =========================================================
   شمارنده تعداد - تم پودری
   ========================================================= */

.jb-count-block {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

/* لیبل */
.jb-count-label-row {
  display: flex;
  align-items: center;
  gap: 10px;
}

.jb-count-label-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
}

.jb-count-label {
  font-size: 0.9rem;
  font-weight: 800;
  color: var(--jb-text);
}

/* کانتینر شمارنده */
.jb-count-wrap {
  display: inline-flex;
  align-items: center;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 14px;
  overflow: hidden;
  transition: all 0.25s ease;
  width: fit-content;
}

.jb-count-wrap:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.12);
}

/* دکمه‌های + و - */
.jb-count-btn {
  width: 44px;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  border: none;
  color: var(--jb-fuchsia);
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-count-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
}

.jb-count-btn:active:not(:disabled) {
  transform: scale(0.92);
}

.jb-count-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

/* مقدار وسط */
.jb-count-value {
  min-width: 52px;
  padding: 0 16px;
  text-align: center;
  font-size: 1rem;
  font-weight: 900;
  color: var(--jb-text);
  line-height: 44px;
  border-left: 2px dashed var(--jb-pink-200);
  border-right: 2px dashed var(--jb-pink-200);
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
}

/* ریسپانسیو */
@media (max-width: 480px) {
  .jb-count-btn {
    width: 40px;
    height: 40px;
  }
  .jb-count-value {
    min-width: 44px;
    padding: 0 12px;
    line-height: 40px;
    font-size: 0.92rem;
  }
}
</style>