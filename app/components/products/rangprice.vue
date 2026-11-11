<template>
  <div class="jb-filter-box  mb-3 mt-3">
    <div class="jb-price-header">
      <span class="jb-filter-box-dot"></span>
      <h3 class="jb-price-title">محدوده قیمت</h3>
    </div>

    <div class="jb-price-inputs">
      <div class="jb-price-input-wrap">
        <span class="jb-price-label">از</span>
        <input
          type="number"
          v-model.number="localPriceRange[0]"
          class="jb-price-input"
          :placeholder="Number(mainPriceRange.min_price).toLocaleString('fa-IR')"
          min="0"
        />
      </div>
      <span class="jb-price-sep">تا</span>
      <div class="jb-price-input-wrap">
        <span class="jb-price-label">تا</span>
        <input
          type="number"
          v-model.number="localPriceRange[1]"
          class="jb-price-input"
          :placeholder="Number(mainPriceRange.max_price).toLocaleString('fa-IR')"
          min="0"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  priceRange: { type: Array },
  mainPriceRange: { type: Object },
});

const emit = defineEmits(["update:priceRange"]);
const localPriceRange = ref([...props.priceRange]);

watch(localPriceRange, (newVal) => emit("update:priceRange", newVal), { deep: true });
watch(() => props.priceRange, (newVal) => {
  localPriceRange.value = [...newVal];
}, { deep: true });
</script>

<style scoped>
.jb-filter-box {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  padding: 14px 16px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 14px rgba(214, 51, 132, 0.06);
}

.jb-filter-box:hover {
  border-color: var(--jb-pink-300);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.12);
}

.jb-price-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 14px;
}

.jb-filter-box-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
}

.jb-price-title {
  font-size: 0.9rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-price-inputs {
  display: flex;
  align-items: flex-end;
  gap: 8px;
}

.jb-price-input-wrap {
  flex: 1;
  position: relative;
}

.jb-price-label {
  position: absolute;
  top: -8px;
  right: 10px;
  padding: 0 6px;
  background: #fff;
  font-size: 0.65rem;
  font-weight: 800;
  color: var(--jb-fuchsia);
  border-radius: 4px;
}

.jb-price-input {
  width: 100%;
  padding: 10px 12px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 12px;
  font-size: 0.82rem;
  font-weight: 700;
  color: var(--jb-text);
  outline: none;
  transition: all 0.25s ease;
  text-align: center;
  font-family: inherit;
}

.jb-price-input:focus {
  border-color: var(--jb-fuchsia);
  box-shadow: 0 0 0 4px rgba(214, 51, 132, 0.15);
}

.jb-price-input::placeholder {
  color: var(--jb-text-muted);
  font-weight: 500;
}

.jb-price-sep {
  color: var(--jb-text-muted);
  font-weight: 800;
  font-size: 0.82rem;
  padding-bottom: 12px;
}
</style>