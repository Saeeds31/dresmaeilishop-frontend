<template>
  <div class="lg:hidden jb-mobile-filter">
    <!-- ===== تب‌های بالا ===== -->
    <div class="jb-mf-tabs">
      <button
        v-for="(tab, index) in tabs"
        :key="index"
        @click="openDrawer(tab.key)"
        class="jb-mf-tab"
      >
        <span class="jb-mf-tab-icon">
          <component :is="tab.icon" class="w-4 h-4" />
        </span>
        <span>{{ tab.label }}</span>
      </button>
    </div>

    <!-- ===== Overlay ===== -->
    <transition name="jb-overlay">
      <div
        v-if="drawerOpen"
        class="jb-mf-overlay"
        @click="drawerOpen = false"
      />
    </transition>

    <!-- ===== Drawer ===== -->
    <transition name="jb-slide">
      <div v-if="drawerOpen" class="jb-mf-drawer">
        <!-- هدر -->
        <div class="jb-mf-drawer-header">
          <div class="jb-mf-drawer-header-title">
            <span class="jb-mf-header-icon">
              <component :is="tabs.find((t) => t.key === currentTab)?.icon" class="w-4 h-4" />
            </span>
            <h3>{{ currentTabLabel }}</h3>
          </div>

          <button class="jb-mf-close-btn" @click="drawerOpen = false" aria-label="بستن">
            <X class="w-4 h-4" />
          </button>
        </div>

        <!-- خط جداکننده بریده‌بریده -->
        <div class="jb-mf-dashed"></div>

        <!-- محتوا -->
        <div class="jb-mf-drawer-body">
          <!-- دسته بندی -->
          <div v-if="currentTab === 'categories'" class="mb-2">
            <ProductsFilters
              v-model:selectedCategories="localSelectedCategories"
              :categories="categories"
            />
          </div>

          <!-- قیمت -->
          <div v-else-if="currentTab === 'price'">
            <ProductsRangprice
              :mainPriceRange="mainPriceRange"
              v-model:priceRange="localPriceRange"
            />
          </div>

          <!-- فیلتر -->
          <div v-else-if="currentTab === 'filter'" class="flex flex-col gap-5">
            <!-- <ProductsWeight v-model:selectedWeight="localSelectedWeight" /> -->
            <ProductsAvailable v-model:onlyAvailable="localOnlyAvailable" />
            <ProductsSearch v-model="localQuery" @search="onSearch" />
          </div>

          <!-- مرتب سازی -->
          <div v-else-if="currentTab === 'sort'" class="jb-mf-sort-list">
            <button
              v-for="option in sortOptions"
              :key="option.value"
              @click="selectSort(option.value)"
              class="jb-mf-sort-item"
              :class="{ 'is-active': localSortBy === option.value }"
            >
              <div class="jb-mf-sort-item-inner">
                <span class="jb-mf-sort-item-icon">
                  <component :is="option.icon" class="w-4 h-4" />
                </span>
                <span class="jb-mf-sort-item-label">{{ option.label }}</span>
              </div>

              <CheckCircle2
                v-if="localSortBy === option.value"
                class="w-5 h-5 jb-mf-sort-check"
              />
            </button>
          </div>
        </div>

        <!-- فوتر -->
        <div class="jb-mf-drawer-footer">
          <button class="jb-btn-linear w-full justify-center" @click="applyFilters">
            <Filter class="w-4 h-4" />
            <span>اعمال فیلتر</span>
          </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import {
  X,
  CheckCircle2,
  Filter,
  Layers,
  SlidersHorizontal,
  ArrowUpDown,
  Tag,
  Flame,
  Eye,
  BadgePercent,
} from "lucide-vue-next";

import { ref, computed, watch } from "vue";
import { useCategoryStore } from "~/store/menu.js";

const tabs = [
  { key: "categories", label: "دسته‌بندی", icon: Layers },
  { key: "price", label: "محدوده قیمت", icon: SlidersHorizontal },
  { key: "filter", label: "فیلتر", icon: Filter },
  { key: "sort", label: "مرتب‌سازی", icon: ArrowUpDown },
];

const sortOptions = [
  { label: "جدیدترین", value: "newest", icon: Tag },
  { label: "ارزان‌ترین", value: "cheapest", icon: BadgePercent },
  { label: "گران‌ترین", value: "expensive", icon: Eye },
  { label: "پرفروش‌ترین", value: "best_selling", icon: Flame },
];

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
  sortBy: { type: String, default: "newest" },
});

const emit = defineEmits([
  "update:selectedCategories",
  "update:selectedWeight",
  "update:onlyAvailable",
  "update:priceRange",
  "update:query",
  "update:sortBy",
  "apply-filter",
]);

const drawerOpen = ref(false);
const currentTab = ref("");
const currentTabLabel = computed(
  () => tabs.find((t) => t.key === currentTab.value)?.label || ""
);

const localSelectedCategories = ref([...props.selectedCategories]);
const localSelectedWeight = ref([...props.selectedWeight]);
const localOnlyAvailable = ref(props.onlyAvailable);
const localPriceRange = ref([...props.priceRange]);
const localQuery = ref(props.query);
const localSortBy = ref(props.sortBy);

watch(localSelectedCategories, (v) => emit("update:selectedCategories", v));
watch(localSelectedWeight, (v) => emit("update:selectedWeight", v));
watch(localOnlyAvailable, (v) => emit("update:onlyAvailable", v));
watch(localQuery, (v) => emit("update:query", v));

function openDrawer(tab) {
  currentTab.value = tab;
  drawerOpen.value = true;
}

function onSearch() {
  applyFilters();
}

function selectSort(value) {
  localSortBy.value = value;
  emit("update:sortBy", value);
  applyFilters();
}

function applyFilters() {
  emit("update:priceRange", localPriceRange.value);
  emit("apply-filter");
  drawerOpen.value = false;
}
</script>

<style scoped>
/* =========================================================
   فیلتر موبایل - تم پودری
   ========================================================= */

.jb-mobile-filter {
  position: relative;
}

/* ===== تب‌های بالا ===== */
.jb-mf-tabs {
  display: flex;
  overflow-x: auto;
  gap: 8px;
  padding: 12px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 2px solid var(--jb-pink-200);
  border-radius: 16px;
  box-shadow: 0 4px 14px rgba(214, 51, 132, 0.08);
  scrollbar-width: none;
}

.jb-mf-tabs::-webkit-scrollbar {
  display: none;
}

.jb-mf-tab {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 9px 16px;
  background: #fff;
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 999px;
  font-size: 0.78rem;
  font-weight: 800;
  color: var(--jb-text-soft);
  cursor: pointer;
  transition: all 0.25s ease;
  white-space: nowrap;
  flex-shrink: 0;
  font-family: inherit;
}

.jb-mf-tab:hover {
  background: var(--jb-pink-50);
  border-color: var(--jb-pink-400);
  color: var(--jb-fuchsia);
  transform: translateY(-2px);
}

.jb-mf-tab-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  border-radius: 7px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 4px 10px rgba(214, 51, 132, 0.3);
}

/* ===== Overlay ===== */
.jb-mf-overlay {
  position: fixed;
  inset: 0;
  background: rgba(45, 20, 32, 0.55);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  z-index: 40;
}

/* ===== Drawer ===== */
.jb-mf-drawer {
  position: fixed;
  top: 0;
  right: 0;
  height: 100vh;
  width: 88vw;
  max-width: 380px;
  background: linear-gradient(180deg, #fff8fb 0%, #ffeaf2 100%);
  border-left: 2px solid var(--jb-pink-300);
  box-shadow: -10px 0 40px rgba(214, 51, 132, 0.35);
  z-index: 50;
  display: flex;
  flex-direction: column;
}

/* هدر */
.jb-mf-drawer-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  background: rgba(255, 255, 255, 0.7);
  border-bottom: 1.5px solid rgba(255, 184, 207, 0.6);
  flex-shrink: 0;
}

.jb-mf-drawer-header-title {
  display: flex;
  align-items: center;
  gap: 10px;
}

.jb-mf-drawer-header-title h3 {
  font-size: 0.95rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-mf-header-icon {
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

.jb-mf-close-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 10px;
  background: #fff;
  border: 1.5px solid var(--jb-pink-200);
  color: var(--jb-text-soft);
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-mf-close-btn:hover {
  background: var(--jb-pink-100);
  color: var(--jb-fuchsia);
  border-color: var(--jb-pink-400);
  transform: rotate(90deg);
}

/* خط بریده‌بریده */
.jb-mf-dashed {
  height: 0;
  border-top: 1.5px dashed var(--jb-pink-300);
  margin: 0 16px;
  position: relative;
}

.jb-mf-dashed::before,
.jb-mf-dashed::after {
  content: "";
  position: absolute;
  top: -4px;
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--jb-pink-400);
  box-shadow: 0 0 0 2px #fff8fb;
}

.jb-mf-dashed::before { right: -3px; }
.jb-mf-dashed::after  { left: -3px; }

/* بدنه */
.jb-mf-drawer-body {
  flex: 1;
  overflow-y: auto;
  padding: 20px 16px;
  scrollbar-width: thin;
}

.jb-mf-drawer-body::-webkit-scrollbar {
  width: 6px;
}

.jb-mf-drawer-body::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, var(--jb-pink-400), var(--jb-fuchsia));
  border-radius: 6px;
}

/* فوتر */
.jb-mf-drawer-footer {
  flex-shrink: 0;
  padding: 14px 16px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  border-top: 1.5px dashed var(--jb-pink-300);
}

/* ===== لیست مرتب‌سازی ===== */
.jb-mf-sort-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.jb-mf-sort-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 16px;
  cursor: pointer;
  transition: all 0.25s ease;
  font-family: inherit;
}

.jb-mf-sort-item:hover {
  border-color: var(--jb-pink-400);
  transform: translateY(-2px);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.15);
}

.jb-mf-sort-item.is-active {
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border-color: var(--jb-fuchsia);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.22);
}

.jb-mf-sort-item-inner {
  display: flex;
  align-items: center;
  gap: 12px;
}

.jb-mf-sort-item-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 10px;
  background: var(--jb-pink-100);
  color: var(--jb-pink-700);
  transition: all 0.25s ease;
}

.jb-mf-sort-item.is-active .jb-mf-sort-item-icon {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.4);
}

.jb-mf-sort-item-label {
  font-size: 0.88rem;
  font-weight: 800;
  color: var(--jb-text-soft);
}

.jb-mf-sort-item.is-active .jb-mf-sort-item-label {
  color: var(--jb-fuchsia);
}

.jb-mf-sort-check {
  color: var(--jb-fuchsia);
}

/* ===== انیمیشن‌ها ===== */
.jb-slide-enter-active,
.jb-slide-leave-active {
  transition: transform 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}

.jb-slide-enter-from,
.jb-slide-leave-to {
  transform: translateX(100%);
}

.jb-overlay-enter-active,
.jb-overlay-leave-active {
  transition: opacity 0.3s ease;
}

.jb-overlay-enter-from,
.jb-overlay-leave-to {
  opacity: 0;
}
</style>