<template>
  <div class="jb-sort-bar mb-2">
    <div class="jb-sort-label">
      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
          d="M3 4h13M3 8h9m-9 4h6m4 0l4-4m0 0l4 4m-4-4v12" />
      </svg>
      <span>مرتب‌سازی:</span>
    </div>

    <ul class="jb-sort-list">
      <li v-for="option in sortOptions" :key="option.label" :class="{ active: selectedLabel === option.label }"
        @click="selectSort(option)" class="jb-sort-item">
        {{ option.label }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  modelValue: { type: String, default: "newest" },
});

const emit = defineEmits(["update:modelValue"]);

const sortOptions = [
  { label: "جدیدترین", value: "newest" },
  { label: "ارزان‌ترین", value: "cheapest" },
  { label: "گران‌ترین", value: "expensive" },
  { label: "پرفروش‌ترین", value: "best_selling" },
];

const selectedLabel = ref(
  sortOptions.find((opt) => opt.value === props.modelValue)?.label || "جدیدترین"
);

watch(() => props.modelValue, (newVal) => {
  const found = sortOptions.find((opt) => opt.value === newVal);
  selectedLabel.value = found ? found.label : "جدیدترین";
});

const selectSort = (option) => {
  selectedLabel.value = option.label;
  emit("update:modelValue", option.value);
};
</script>

<style scoped>
.jb-sort-bar {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 14px 18px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  box-shadow: 0 4px 14px rgba(214, 51, 132, 0.06);
  flex-wrap: wrap;
}

.jb-sort-label {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
  flex-shrink: 0;
}

.jb-sort-label svg {
  color: var(--jb-fuchsia);
}

.jb-sort-list {
  display: flex;
  gap: 8px;
  list-style: none;
  margin: 0;
  padding: 0;
  flex-wrap: nowrap;
  overflow-x: auto;
  scrollbar-width: none;
}

.jb-sort-item {
  padding: 7px 16px;
  border-radius: 999px;
  font-size: 0.8rem;
  font-weight: 700;
  color: var(--jb-text-soft);
  background: #fff;
  border: 1.5px solid var(--jb-pink-200);
  cursor: pointer;
  transition: all 0.25s ease;
  user-select: none;
}

.jb-sort-item:hover {
  background: var(--jb-pink-50);
  border-color: var(--jb-pink-400);
  color: var(--jb-fuchsia);
  transform: translateY(-2px);
}

.jb-sort-item.active {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.4);
  transform: translateY(-2px);
}

@media (max-width: 768px) {
  .jb-sort-bar {
    padding: 12px;
    gap: 10px;
  }

  .jb-sort-item {
    padding: 6px 12px;
    font-size: 0.75rem;
  }
}
</style>