<template>
  <div class="jb-filter-box mb-3 mt-3">
    <!-- هدر -->
    <div class="jb-filter-box-header" @click="collapsed = !collapsed">
      <h3 class="jb-filter-box-title">
        <span class="jb-filter-box-dot"></span>
        دسته‌بندی
      </h3>
      <button class="jb-filter-collapse-btn" :class="{ 'is-open': !collapsed }">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M19 9l-7 7-7-7" />
        </svg>
      </button>
    </div>

    <!-- لیست -->
    <transition name="jb-collapse">
      <div v-show="!collapsed" class="jb-filter-box-body">
        <label
          v-for="item in categories"
          :key="item.id"
          class="jb-cat-item"
          :class="{ 'is-checked': localSelectedCategories.includes(item.id) }"
        >
          <span class="jb-cat-item-title">{{ item.title }}</span>
          <span class="jb-cat-checkbox">
            <input
              type="checkbox"
              :value="item.id"
              v-model="localSelectedCategories"
            />
            <span class="jb-cat-checkmark">
              <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3.5" d="M5 13l4 4L19 7" />
              </svg>
            </span>
          </span>
        </label>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  selectedCategories: { type: Array, default: () => [] },
  categories: { type: Array, default: () => [] },
});

const emit = defineEmits(["update:selectedCategories"]);
const collapsed = ref(false);
const localSelectedCategories = ref([...props.selectedCategories]);

watch(localSelectedCategories, (val) => emit("update:selectedCategories", val));
watch(() => props.selectedCategories, (val) => {
  localSelectedCategories.value = [...val];
});
</script>

<style scoped>
.jb-filter-box {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  overflow: hidden;
  transition: all 0.3s ease;
  box-shadow: 0 4px 14px rgba(214, 51, 132, 0.06);
}

.jb-filter-box:hover {
  border-color: var(--jb-pink-300);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.12);
}

.jb-filter-box-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
  cursor: pointer;
  background: linear-gradient(135deg, #fff5f8, #ffeaf2);
}

.jb-filter-box-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.9rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-filter-box-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
}

.jb-filter-collapse-btn {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: var(--jb-pink-100);
  color: var(--jb-pink-700);
  border: 1.5px solid var(--jb-pink-200);
  transition: all 0.3s ease;
  cursor: pointer;
}

.jb-filter-collapse-btn.is-open {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  transform: rotate(180deg);
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.35);
}

.jb-filter-box-body {
  padding: 8px 6px 12px;
  display: flex;
  flex-direction: column;
}

/* آیتم دسته */
.jb-cat-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding: 10px 12px;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  user-select: none;
}

.jb-cat-item:hover {
  background: var(--jb-pink-50);
}

.jb-cat-item.is-checked {
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
}

.jb-cat-item-title {
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--jb-text-soft);
  transition: color 0.2s ease;
}

.jb-cat-item.is-checked .jb-cat-item-title {
  color: var(--jb-fuchsia);
  font-weight: 800;
}

/* چک‌باکس سفارشی */
.jb-cat-checkbox {
  position: relative;
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.jb-cat-checkbox input {
  position: absolute;
  opacity: 0;
  inset: 0;
  cursor: pointer;
  z-index: 2;
}

.jb-cat-checkmark {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  border: 2px solid var(--jb-pink-300);
  background: #fff;
  color: transparent;
  transition: all 0.2s ease;
}

.jb-cat-checkbox input:checked + .jb-cat-checkmark {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  border-color: transparent;
  color: #fff;
  box-shadow: 0 4px 10px rgba(214, 51, 132, 0.35);
}

/* انیمیشن باز/بسته */
.jb-collapse-enter-active,
.jb-collapse-leave-active {
  transition: all 0.3s ease;
  max-height: 500px;
  overflow: hidden;
}

.jb-collapse-enter-from,
.jb-collapse-leave-to {
  opacity: 0;
  max-height: 0;
  padding-top: 0;
  padding-bottom: 0;
}
</style>