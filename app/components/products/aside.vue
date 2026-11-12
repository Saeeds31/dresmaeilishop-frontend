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
    <button class="jb-apply-filter-btn" @click="applyFilters">
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
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
  border: 2px solid rgba(14, 90, 58, 0.1);
  border-radius: 18px;
  box-shadow: 0 6px 18px rgba(14, 90, 58, 0.06);
}

.jb-filter-header-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 0.95rem;
  font-weight: 900;
  color: #1a202c;
}

.jb-filter-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 10px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.3);
  border: 2px solid #fff;
}

/* دکمه اعمال فیلتر - سبز لوکس */
.jb-apply-filter-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
  padding: 13px 24px;
  background: linear-gradient(90deg, #0e5a3a 0%, #1a7a50 50%, #0e5a3a 100%);
  background-size: 200% auto;
  color: #fff;
  font-size: 0.9rem;
  font-weight: 800;
  font-family: inherit;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 14px;
  cursor: pointer;
  transition: all 0.4s ease;
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.3);
  position: relative;
  overflow: hidden;
}

.jb-apply-filter-btn::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, transparent 0%, rgba(255, 255, 255, 0.35) 50%, transparent 100%);
  transform: translateX(-100%);
  transition: transform 0.6s ease;
}

.jb-apply-filter-btn:hover {
  background-position: 100% center;
  transform: translateY(-2px);
  box-shadow: 0 14px 32px rgba(14, 90, 58, 0.45);
}

.jb-apply-filter-btn:hover::before {
  transform: translateX(100%);
}
</style>