<script setup>
import { computed, ref, watch } from "vue";

const props = defineProps({
  modelValue: { type: [String, Number], default: null },
  weight: { type: [Array, String], default: () => [] },
  options: { type: Array, default: () => [] },
  label: { type: String, default: "" },
  disabled: { type: Boolean, default: false },
  showError: { type: Boolean, default: false },
});

const emit = defineEmits(["update:modelValue"]);
const selectedOption = ref(null);

const safeDecode = (value) => {
  if (typeof value !== "string") return value;
  try {
    return decodeURIComponent(value);
  } catch (error) {
    return value;
  }
};

const normalizeToArray = (input) => {
  if (Array.isArray(input)) return input;

  if (typeof input === "string") {
    try {
      const parsed = JSON.parse(input);
      if (Array.isArray(parsed)) return parsed;
    } catch (error) {}

    if (input.includes(",")) return input.split(",").map((item) => item.trim());
    return [input];
  }

  return [];
};

const normalizedOptions = computed(() => {
  const rawOptions = props.options?.length
    ? props.options
    : normalizeToArray(props.weight);

  return rawOptions.map((item) => {
    if (item && typeof item === "object") {
      const id = item.id ?? item.value ?? item.title ?? item.label ?? item;
      const value = item.value ?? item.id ?? item.title ?? item.label ?? "";
      const label = safeDecode(item.label ?? item.title ?? item.value ?? "");
      return {
        id,
        value,
        label,
        isAvailable: item.isAvailable ?? item.is_available ?? true,
      };
    }

    const normalizedValue = safeDecode(String(item));
    return {
      id: normalizedValue,
      value: normalizedValue,
      label: normalizedValue,
      isAvailable: true,
    };
  });
});

watch(
  () => props.modelValue,
  (newValue) => {
    if (!newValue) {
      selectedOption.value = null;
      return;
    }
    const option = normalizedOptions.value.find(
      (item) => item.id === newValue || item.value === newValue
    );
    selectedOption.value = option ?? null;
  },
  { immediate: true }
);

const selectOption = (option) => {
  if (props.disabled) return;
  selectedOption.value = option;
  emit("update:modelValue", option.id);
};
</script>
<template>
  <div class="jb-weight-block">
    <!-- لیبل -->
    <div v-if="label" class="jb-weight-label-row">
      <span class="jb-weight-label-dot"></span>
      <span class="jb-weight-label">{{ label }}</span>
      <span v-if="showError && !selectedOption" class="jb-weight-error-tag">الزامی</span>
    </div>

    <!-- گرید گزینه‌ها -->
    <div class="jb-weight-options">
      <button
        v-for="option in normalizedOptions"
        :key="option.id"
        type="button"
        @click="selectOption(option)"
        class="jb-weight-option"
        :class="{
          'is-selected': selectedOption?.id === option.id && !disabled,
          'is-disabled': disabled,
          'is-unavailable': !option.isAvailable && !disabled,
        }"
      >
        <span class="jb-weight-option-text">{{ option.label }}</span>

        <!-- ضربدر روی گزینه ناموجود -->
        <svg
          v-if="!option.isAvailable && !disabled"
          class="jb-weight-option-cross"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
        >
          <line x1="6" y1="6" x2="18" y2="18" />
          <line x1="18" y1="6" x2="6" y2="18" />
        </svg>

        <!-- تیک گزینه انتخاب‌شده -->
        <svg
          v-if="selectedOption?.id === option.id && !disabled"
          class="jb-weight-option-check"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="3"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <polyline points="20 6 9 17 4 12" />
        </svg>
      </button>
    </div>

    <!-- پیام خطا -->
    <p v-if="showError && !selectedOption" class="jb-weight-error-msg">
      لطفاً ابتدا گزینه مورد نظر را انتخاب کنید
    </p>
  </div>
</template>

<style scoped>
/* =========================================================
   گزینه‌های تنوع - تم سبز لوکس
   ========================================================= */

.jb-weight-block {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

/* لیبل */
.jb-weight-label-row {
  display: flex;
  align-items: center;
  gap: 10px;
}

.jb-weight-label-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  box-shadow: 0 0 0 3px rgba(140, 198, 63, 0.18);
}

.jb-weight-label {
  font-size: 0.9rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-weight-error-tag {
  font-size: 0.68rem;
  font-weight: 800;
  color: #c53030;
  padding: 2px 8px;
  background: #ffe8e8;
  border-radius: 999px;
}

/* گرید گزینه‌ها */
.jb-weight-options {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

/* خود گزینه */
.jb-weight-option {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 12px 10px;
  background: #fff;
  border: 2px solid rgba(14, 90, 58, 0.12);
  border-radius: 14px;
  font-size: 0.85rem;
  font-weight: 800;
  color: #4a5568;
  cursor: pointer;
  transition: all 0.25s ease;
  font-family: inherit;
  overflow: hidden;
  min-height: 50px;
}

.jb-weight-option:hover:not(.is-disabled):not(.is-selected) {
  border-color: rgba(14, 90, 58, 0.3);
  background: #f1f7f3;
  color: #0e5a3a;
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(14, 90, 58, 0.1);
}

/* گزینه انتخاب‌شده */
.jb-weight-option.is-selected {
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  border-color: transparent;
  box-shadow: 0 8px 20px rgba(14, 90, 58, 0.3);
  transform: translateY(-2px);
}

/* گزینه ناموجود */
.jb-weight-option.is-unavailable {
  opacity: 0.55;
  background: #f8f9fa;
}

.jb-weight-option.is-unavailable .jb-weight-option-text {
  text-decoration: line-through;
  text-decoration-color: #fc8181;
  text-decoration-thickness: 2px;
  color: #a0aec0;
}

/* گزینه disabled */
.jb-weight-option.is-disabled {
  opacity: 0.5;
  cursor: not-allowed;
  background: #f8f9fa;
}

/* متن گزینه */
.jb-weight-option-text {
  position: relative;
  z-index: 1;
  transition: color 0.2s ease;
}

/* ضربدر روی گزینه ناموجود */
.jb-weight-option-cross {
  position: absolute;
  inset: 0;
  margin: auto;
  width: 26px;
  height: 26px;
  color: #fc8181;
  pointer-events: none;
  opacity: 0.9;
}

/* تیک گزینه انتخاب‌شده */
.jb-weight-option-check {
  position: absolute;
  top: 6px;
  left: 6px;
  width: 14px;
  height: 14px;
  color: #fff;
  opacity: 0.9;
}

/* پیام خطا */
.jb-weight-error-msg {
  font-size: 0.78rem;
  font-weight: 700;
  color: #c53030;
  padding: 6px 12px;
  background: #ffe8e8;
  border-radius: 10px;
  display: inline-block;
  width: fit-content;
}

/* ریسپانسیو */
@media (max-width: 480px) {
  .jb-weight-options {
    grid-template-columns: repeat(3, 1fr);
  }
  .jb-weight-option {
    font-size: 0.78rem;
    padding: 10px 8px;
    min-height: 46px;
  }
}
</style>