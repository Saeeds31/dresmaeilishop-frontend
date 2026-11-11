<template>
  <div class="jb-tabs-wrapper" dir="rtl">
    <!-- هدر تب‌ها -->
    <div class="jb-tabs-header">
      <button @click="activeTab = 'text'" :class="['jb-tab-btn', { 'is-active': activeTab === 'text' }]">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
        </svg>
        <span>توضیحات</span>
      </button>

      <button v-if="props.specifications && props.specifications.length" @click="activeTab = 'specifications'"
        :class="['jb-tab-btn', { 'is-active': activeTab === 'specifications' }]">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2" d="M4 6h16M4 10h16M4 14h10M4 18h10" />
        </svg>
        <span>ویژگی‌ها</span>
      </button>

      <button @click="activeTab = 'comment'" :class="['jb-tab-btn', { 'is-active': activeTab === 'comment' }]">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
        </svg>
        <span>نظرات</span>
      </button>
    </div>

    <!-- خط جداکننده بریده‌بریده -->
    <div class="jb-tabs-dashed"></div>

    <!-- محتوا -->
    <div class="jb-tabs-content">
      <transition name="jb-fade" mode="out-in">
        <div v-if="activeTab === 'text'" key="text" class="jb-tab-panel">
          <productTabsInformation :description="props.description" />
        </div>

        <div v-else-if="activeTab === 'comment'" key="comment" class="jb-tab-panel">
          <productTabsComment :productId="props.productId" :comments="comments" />
        </div>

        <div v-else-if="activeTab === 'specifications'" key="specifications" class="jb-tab-panel">
          <ul class="jb-spec-list">
            <li v-for="(item, index) in props.specifications" :key="index" class="jb-spec-item">
              <span class="jb-spec-title">{{ item.title }}</span>
              <span class="jb-spec-value">
                {{item.values ? item.values.map((it) => it.value).join(" , ") : ""}}
              </span>
            </li>
          </ul>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { toast } from "vue3-toastify";

const props = defineProps({
  specifications: Array,
  description: { type: String, default: "" },
  comments: { type: Object, default: null },
  productId: { type: [String, Number], required: true },

});

const activeTab = ref("text");
const comments = ref([]);
</script>

<style scoped>
.jb-tabs-wrapper {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 24px;
  overflow: hidden;
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.08);
}

/* ===== تب‌ها ===== */
.jb-tabs-header {
  display: flex;
  gap: 8px;
  padding: 16px;
  flex-wrap: wrap;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
}

.jb-tab-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 14px;
  font-size: 0.88rem;
  font-weight: 800;
  color: var(--jb-text-soft);
  cursor: pointer;
  transition: all 0.25s ease;
  font-family: inherit;
}

.jb-tab-btn:hover {
  border-color: var(--jb-pink-400);
  background: var(--jb-pink-50);
  color: var(--jb-fuchsia);
  transform: translateY(-2px);
}

.jb-tab-btn.is-active {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.4);
  transform: translateY(-2px);
}

/* خط بریده‌بریده */
.jb-tabs-dashed {
  height: 0;
  border-top: 2px dashed var(--jb-pink-200);
  margin: 0 16px;
  position: relative;
}

.jb-tabs-dashed::before,
.jb-tabs-dashed::after {
  content: "";
  position: absolute;
  top: -5px;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: var(--jb-pink-400);
  box-shadow: 0 0 0 3px #fff;
}

.jb-tabs-dashed::before {
  right: -4px;
}

.jb-tabs-dashed::after {
  left: -4px;
}

/* ===== محتوا ===== */
.jb-tabs-content {
  padding: 24px;
}

.jb-tab-panel {
  animation: jbFadeIn 0.35s ease;
}

@keyframes jbFadeIn {
  from {
    opacity: 0;
    transform: translateY(8px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* ===== لیست مشخصات ===== */
.jb-spec-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.jb-spec-item {
  display: grid;
  grid-template-columns: minmax(140px, 200px) 1fr;
  gap: 12px;
  border-radius: 14px;
  overflow: hidden;
  border: 1.5px solid var(--jb-pink-200);
  transition: all 0.25s ease;
}

.jb-spec-item:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.12);
}

.jb-spec-title {
  padding: 14px 18px;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
  display: flex;
  align-items: center;
  border-left: 2px dashed var(--jb-pink-300);
}

.jb-spec-value {
  padding: 14px 18px;
  background: #fff;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--jb-text-soft);
  display: flex;
  align-items: center;
}

/* ===== انیمیشن‌ها ===== */
.jb-fade-enter-active,
.jb-fade-leave-active {
  transition: all 0.25s ease;
}

.jb-fade-enter-from,
.jb-fade-leave-to {
  opacity: 0;
  transform: translateY(8px);
}

@media (max-width: 640px) {
  .jb-tabs-content {
    padding: 16px;
  }

  .jb-tab-btn {
    padding: 8px 14px;
    font-size: 0.8rem;
  }

  .jb-spec-item {
    grid-template-columns: 1fr;
  }

  .jb-spec-title {
    border-left: none;
    border-bottom: 2px dashed var(--jb-pink-300);
  }
}
</style>