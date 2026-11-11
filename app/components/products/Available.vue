<template>
  <div class="jb-filter-box">
    <div class="jb-available-header">
      <div class="jb-available-title-wrap">
        <span class="jb-filter-box-dot"></span>
        <h3 class="jb-available-title">حالت نمایش</h3>
      </div>
    </div>

    <div class="jb-available-row">
      <p class="jb-available-label">فقط کالاهای موجود</p>
      <label class="jb-switch">
        <input type="checkbox" v-model="localOnlyAvailable" />
        <span class="jb-slider">
          <span class="jb-slider-dot"></span>
        </span>
      </label>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  onlyAvailable: { type: Boolean, default: false },
});

const emit = defineEmits(["update:onlyAvailable"]);
const localOnlyAvailable = ref(props.onlyAvailable);

watch(localOnlyAvailable, (val) => emit("update:onlyAvailable", val));
watch(() => props.onlyAvailable, (val) => (localOnlyAvailable.value = val));
</script>

<style scoped>
.jb-filter-box {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  overflow: hidden;
  padding: 14px 16px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 14px rgba(214, 51, 132, 0.06);
}

.jb-filter-box:hover {
  border-color: var(--jb-pink-300);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.12);
}

.jb-available-header {
  margin-bottom: 14px;
}

.jb-available-title-wrap {
  display: flex;
  align-items: center;
  gap: 8px;
}

.jb-filter-box-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
}

.jb-available-title {
  font-size: 0.9rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-available-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding: 8px 4px;
}

.jb-available-label {
  font-size: 0.82rem;
  font-weight: 600;
  color: var(--jb-text-soft);
}

/* سوییچ */
.jb-switch {
  position: relative;
  display: inline-block;
  width: 52px;
  height: 28px;
  flex-shrink: 0;
}

.jb-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.jb-slider {
  position: absolute;
  cursor: pointer;
  inset: 0;
  background: #e9dbe1;
  border-radius: 999px;
  transition: all 0.3s ease;
  border: 2px solid var(--jb-pink-200);
}

.jb-slider-dot {
  position: absolute;
  top: 50%;
  right: 4px;
  transform: translateY(-50%);
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.jb-switch input:checked + .jb-slider {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  border-color: transparent;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.35);
}

.jb-switch input:checked + .jb-slider .jb-slider-dot {
  right: calc(100% - 22px);
  box-shadow: 0 2px 8px rgba(214, 51, 132, 0.4);
}
</style>