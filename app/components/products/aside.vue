<template>
  <aside class="jb-filter-aside lg:w-[25%] w-full flex flex-col gap-5">
    <!-- هدر سایدبار -->
    <div class="jb-filter-header">
      <h3 class="jb-filter-header-title">
        <span class="jb-filter-header-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="M3 4a1 1 0 011-1h16a1 1 0 011 1v2.586a1 1 0 01-.293.707l-6.414 6.414a1 1 0 00-.293.707V17l-4 4v-6.586a1 1 0 00-.293-.707L3.293 7.293A1 1 0 013 6.586V4z" />
          </svg>
        </span>
        <span>فیلتر کردن</span>
      </h3>
    </div>

    <ProductsFilters v-model:selectedCategories="localSelectedCategories" :categories="categories" />
    <ProductsSearch v-model="localQuery" @search="onSearch" />
    <!-- <ProductsWeight v-model:selectedWeight="localSelectedWeight" /> -->
    <ProductsAvailable v-model:onlyAvailable="localOnlyAvailable" />
    <ProductsRangprice :mainPriceRange="mainPriceRange" v-model:priceRange="localPriceRange" />

    <!-- دکمه اعمال فیلتر -->
    <button class="jb-btn-linear w-full justify-center" @click="applyFilters">
      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
      </svg>
      <span>اعمال فیلتر</span>
    </button>
  </aside>
</template>

<script setup>
import { ref, watch, computed } from "vue";
import { useCategoryStore } from "~/store/menu.js";

const categoryStore = useCategoryStore();
await categoryStore.fetchMenuData();

const filters = computed(() => categoryStore.filters);
const categories = computed(() => filters.value?.data?.categories || []);
const mainPriceRange = computed(() => filters.value?.data?.price || []);

const props = defineProps({
  selectedCategories: { type: Array, default: () => [] },
  selectedWeight: { type: Array, default: () => [] },
  onlyAvailable: { type: Boolean, default: false },
  priceRange: { type: Array, default: () => [0, 2000000] },
  query: { type: String, default: "" },
});

const emit = defineEmits([
  "update:selectedCategories",
  "update:selectedWeight",
  "update:onlyAvailable",
  "update:priceRange",
  "update:query",
  "apply-filter",
]);

const localSelectedCategories = ref([...props.selectedCategories]);
const localSelectedWeight = ref([...props.selectedWeight]);
const localOnlyAvailable = ref(props.onlyAvailable);
const localPriceRange = ref([...props.priceRange]);
const localQuery = ref(props.query);

watch(localSelectedCategories, (v) => emit("update:selectedCategories", v));
watch(localSelectedWeight, (v) => emit("update:selectedWeight", v));
watch(localOnlyAvailable, (v) => emit("update:onlyAvailable", v));
watch(localQuery, (v) => emit("update:query", v));

function applyFilters() {
  emit("update:priceRange", localPriceRange.value);
  emit("apply-filter", { categories: localSelectedCategories.value });
}
</script>

<style scoped>
.jb-filter-aside {
  position: sticky;
  top: 20px;
  align-self: flex-start;
}

.jb-filter-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 18px;
  background: linear-gradient(135deg, #fff5f8, #ffe0eb);
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  box-shadow: 0 6px 18px rgba(214, 51, 132, 0.08);
}

.jb-filter-header-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 0.95rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-filter-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 10px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
}
</style>