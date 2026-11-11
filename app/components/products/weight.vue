<template>
  <div class="jb-filter-box  mb-3 mt-3">
    <div class="jb-filter-box-header" @click="collapsed = !collapsed">
      <h3 class="jb-filter-box-title">
        <span class="jb-filter-box-dot"></span>
        سایز
      </h3>
      <button class="jb-filter-collapse-btn" :class="{ 'is-open': !collapsed }">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M19 9l-7 7-7-7" />
        </svg>
      </button>
    </div>

    <transition name="jb-collapse">
      <div v-show="!collapsed" class="jb-weight-grid">
        <button
          v-for="(item, index) in weight"
          :key="index"
          type="button"
          @click="selectWeight(item)"
          class="jb-weight-chip"
          :class="{ 'is-active': localSelectedWeight.includes(item) }"
        >
          {{ item }}
        </button>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  selectedWeight: { type: Array, default: () => [] },
});

const emit = defineEmits(["update:selectedWeight"]);
const collapsed = ref(false);
const weight = [36, 38, 40, 42, 44, 48];
const localSelectedWeight = ref([...props.selectedWeight]);

watch(localSelectedWeight, (val) => emit("update:selectedWeight", val));
watch(
  () => props.selectedWeight,
  (val) => (localSelectedWeight.value = [...val]),
  { deep: true }
);

function toggleCollapse() {
  collapsed.value = !collapsed.value;
}

function selectWeight(item) {
  if (localSelectedWeight.value.includes(item)) {
    localSelectedWeight.value = [];
  } else {
    localSelectedWeight.value = [item];
  }
}
</script>

<style scoped>
/* استایل‌های پایه از Filters.vue با کلاس‌های مشترک */
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

/* گرید سایز */
.jb-weight-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
  padding: 12px;
}

.jb-weight-chip {
  padding: 10px 6px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 12px;
  font-size: 0.78rem;
  font-weight: 800;
  color: var(--jb-text-soft);
  cursor: pointer;
  transition: all 0.25s ease;
  font-family: inherit;
}

.jb-weight-chip:hover {
  border-color: var(--jb-pink-400);
  background: var(--jb-pink-50);
  color: var(--jb-fuchsia);
  transform: translateY(-2px);
}

.jb-weight-chip.is-active {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.4);
  transform: translateY(-2px);
}

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