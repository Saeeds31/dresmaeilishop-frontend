<template>
  <div class="jb-pagination">
    <div class="jb-pagination-row">
      <!-- دکمه صفحه قبل -->
      <button @click="prevPage" :disabled="currentPageInternal === 1" class="jb-page-btn jb-page-arrow"
        aria-label="صفحه قبل">
        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none"
          stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="9 18 15 12 9 6"></polyline>
        </svg>
      </button>

      <!-- شماره صفحات -->
      <template v-for="(page, index) in visiblePages" :key="index">
        <div v-if="page === '...'" class="jb-page-btn jb-page-ellipsis">...</div>
        <button v-else @click="goToPage(page)" :class="['jb-page-btn', { active: currentPageInternal === page }]"
          :aria-label="`صفحه ${toPersianNum(page)}`">
          {{ toPersianNum(page) }}
        </button>
      </template>

      <!-- دکمه صفحه بعد -->
      <button @click="nextPage" :disabled="currentPageInternal === totalPages" class="jb-page-btn jb-page-arrow"
        aria-label="صفحه بعد">
        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none"
          stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="15 18 9 12 15 6"></polyline>
        </svg>
      </button>
    </div>

    <!-- اطلاعات صفحه -->
    <div class="jb-page-info">
      صفحه {{ toPersianNum(currentPageInternal) }} از {{ toPersianNum(totalPages) }}
    </div>
  </div>
</template>

<script setup>
import { computed, watch, ref } from "vue";

const props = defineProps({
  modelValue: { type: Number, default: 1 },
  totalPages: { type: Number, default: 25 },
});

const emit = defineEmits(["update:modelValue", "page-change"]);
const currentPageInternal = ref(props.modelValue);

watch(() => props.modelValue, (val) => {
  currentPageInternal.value = val;
});

watch(currentPageInternal, (val) => {
  emit("update:modelValue", val);
  emit("page-change", val);
});

const toPersianNum = (num) => {
  const persianDigits = ["۰", "۱", "۲", "۳", "۴", "۵", "۶", "۷", "۸", "۹"];
  return num.toString().split("").map((d) => persianDigits[d] || d).join("");
};

const visiblePages = computed(() => {
  const pages = [];
  const total = props.totalPages;
  const current = currentPageInternal.value;

  if (total <= 7) {
    for (let i = 1; i <= total; i++) pages.push(i);
  } else {
    if (current <= 4) {
      pages.push(1, 2, 3, 4, 5, "...", total);
    } else if (current >= total - 3) {
      pages.push(1, "...", total - 4, total - 3, total - 2, total - 1, total);
    } else {
      pages.push(1, "...", current - 1, current, current + 1, "...", total);
    }
  }
  return pages;
});

const goToPage = (page) => {
  if (page >= 1 && page <= props.totalPages) currentPageInternal.value = page;
};

const nextPage = () => {
  if (currentPageInternal.value < props.totalPages) currentPageInternal.value++;
};

const prevPage = () => {
  if (currentPageInternal.value > 1) currentPageInternal.value--;
};
</script>

<style scoped>
.jb-pagination {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  margin-top: 40px;
  direction: rtl;
}

.jb-pagination-row {
  display: flex;
  align-items: center;
  gap: 6px;
  flex-wrap: wrap;
  justify-content: center;
}

.jb-page-btn {
  min-width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  border: 2px solid var(--jb-pink-200);
  background: #fff;
  color: var(--jb-text-soft);
  font-size: 0.85rem;
  font-weight: 800;
  cursor: pointer;
  transition: all 0.25s ease;
  user-select: none;
  font-family: inherit;
}

.jb-page-btn:hover:not(.active):not(.jb-page-ellipsis):not(:disabled) {
  background: var(--jb-pink-100);
  border-color: var(--jb-pink-400);
  color: var(--jb-fuchsia);
  transform: translateY(-2px);
}

.jb-page-btn.active {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.45);
  transform: translateY(-2px);
}

.jb-page-btn.jb-page-ellipsis {
  cursor: default;
  border: none;
  background: transparent;
  color: var(--jb-text-muted);
}

.jb-page-btn:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

.jb-page-arrow {
  background: #fff;
  color: var(--jb-fuchsia);
}

.jb-page-arrow:hover:not(:disabled) {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.4);
}

.jb-page-info {
  color: var(--jb-text-muted);
  font-size: 0.82rem;
  font-weight: 600;
  text-align: center;
}

@media (max-width: 768px) {
  .jb-page-btn {
    min-width: 36px;
    height: 36px;
    font-size: 0.78rem;
  }
}
</style>