<template>
  <form @submit.prevent="onSubmit" class="jb-search-box  mb-3 mt-3">
    <div class="jb-search-header">
      <span class="jb-filter-box-dot"></span>
      <h3 class="jb-search-title">جستجو کنید</h3>
    </div>

    <div class="jb-search-input-wrap">
      <input
        v-model="localQuery"
        class="jb-search-input"
        type="search"
        placeholder="نام محصول را وارد کنید..."
      />
      <button type="submit" class="jb-search-btn" aria-label="جستجو">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
            d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
        </svg>
      </button>
    </div>
  </form>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  modelValue: { type: String, default: "" },
});

const emit = defineEmits(["update:modelValue", "search"]);
const localQuery = ref(props.modelValue);

watch(localQuery, (val) => emit("update:modelValue", val));

function onSubmit() {
  emit("search", localQuery.value);
}
</script>

<style scoped>
.jb-search-box {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  padding: 14px 16px;
  box-shadow: 0 4px 14px rgba(214, 51, 132, 0.06);
  transition: all 0.3s ease;
}

.jb-search-box:hover {
  border-color: var(--jb-pink-300);
}

.jb-search-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 12px;
}

.jb-filter-box-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
}

.jb-search-title {
  font-size: 0.9rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-search-input-wrap {
  position: relative;
}

.jb-search-input {
  width: 100%;
  padding: 11px 42px 11px 14px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 12px;
  font-size: 0.82rem;
  font-weight: 600;
  color: var(--jb-text);
  transition: all 0.25s ease;
  outline: none;
}

.jb-search-input::placeholder {
  color: var(--jb-text-muted);
  font-weight: 500;
}

.jb-search-input:focus {
  border-color: var(--jb-fuchsia);
  box-shadow: 0 0 0 4px rgba(214, 51, 132, 0.15);
}

.jb-search-btn {
  position: absolute;
  left: 6px;
  top: 50%;
  transform: translateY(-50%);
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 9px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border: none;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.35);
}

.jb-search-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-600), var(--jb-pink-800));
  transform: translateY(-50%) scale(1.05);
}
</style>