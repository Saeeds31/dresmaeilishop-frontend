

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
<template>
  <div class="jb-count-block">
    <!-- لیبل -->
    <div class="jb-count-label-row">
      <span class="jb-count-label-dot"></span>
      <span class="jb-count-label">تعداد</span>
    </div>

    <!-- کانتینر شمارنده -->
    <div class="jb-count-wrap">
      <button
        type="button"
        class="jb-count-btn"
        :disabled="quantity <= min"
        @click="decrease"
        aria-label="کاهش"
      >
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 12h14" />
        </svg>
      </button>

      <div class="jb-count-value">{{ quantity }}</div>

      <button
        type="button"
        class="jb-count-btn"
        :disabled="quantity >= max"
        @click="increase"
        aria-label="افزایش"
      >
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M12 5v14M5 12h14" />
        </svg>
      </button>
    </div>
  </div>
</template>

<style scoped>
/* =========================================================
   شمارنده تعداد - تم سبز لوکس
   ========================================================= */

.jb-count-block {
  display: flex;
  flex-direction: column;
  gap: 14px;
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
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  box-shadow: 0 0 0 3px rgba(140, 198, 63, 0.18);
}

.jb-count-label {
  font-size: 0.9rem;
  font-weight: 800;
  color: #1a202c;
}

/* کانتینر شمارنده */
.jb-count-wrap {
  display: inline-flex;
  align-items: center;
  background: #fff;
  border: 2px solid rgba(14, 90, 58, 0.12);
  border-radius: 16px;
  overflow: hidden;
  transition: all 0.25s ease;
  width: fit-content;
}

.jb-count-wrap:hover {
  border-color: rgba(14, 90, 58, 0.25);
  box-shadow: 0 6px 16px rgba(14, 90, 58, 0.08);
}

/* دکمه‌های + و - */
.jb-count-btn {
  width: 48px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  border: none;
  color: #0e5a3a;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-count-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
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
  min-width: 60px;
  padding: 0 16px;
  text-align: center;
  font-size: 1.1rem;
  font-weight: 900;
  color: #0e5a3a;
  line-height: 48px;
  border-left: 2px dashed rgba(14, 90, 58, 0.12);
  border-right: 2px dashed rgba(14, 90, 58, 0.12);
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
}

/* ریسپانسیو */
@media (max-width: 480px) {
  .jb-count-btn {
    width: 42px;
    height: 42px;
  }
  .jb-count-value {
    min-width: 50px;
    padding: 0 12px;
    line-height: 42px;
    font-size: 1rem;
  }
}
</style>