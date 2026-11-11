<template>
  <div class="jb-product-title-block">
    <!-- ردیف بالا: عنوان + دکمه شیر -->
    <div class="jb-title-row">
      <h1 class="jb-title">{{ title }}</h1>

      <button class="jb-share-btn" @click="shareContent" aria-label="اشتراک‌گذاری">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.368 2.684 3 3 0 00-5.368-2.684z" />
        </svg>
      </button>
    </div>

    <!-- قیمت‌ها -->
    <div class="jb-price-row">
      <div class="jb-price-current">
        {{ Number(price).toLocaleString("fa") }}
        <span class="jb-price-unit">تومان</span>
      </div>

      <div v-if="oldPrice && oldPrice != price" class="jb-price-old">
        {{ Number(oldPrice).toLocaleString("fa") }} تومان
      </div>

      <!-- بج تخفیف -->
      <div v-if="oldPrice && oldPrice != price" class="jb-price-discount">
        <span>{{ discountPercent }}٪</span>
        <span class="jb-discount-label">تخفیف</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from "vue";
import { toast } from "vue3-toastify";

const props = defineProps({
  title: String,
  price: Number,
  oldPrice: Number,
});

const discountPercent = computed(() => {
  if (!props.oldPrice || props.oldPrice === props.price) return 0;
  return Math.round(((props.oldPrice - props.price) / props.oldPrice) * 100);
});

const shareContent = async () => {
  const shareData = {
    title: props.title,
    text: `مشاهده محصول: ${props.title}`,
    url: window.location.href,
  };

  if (navigator.share) {
    try {
      await navigator.share(shareData);
    } catch (err) {}
  } else {
    toast.error("مرورگر شما از اشتراک‌گذاری پشتیبانی نمی‌کند.");
  }
};
</script>

<style scoped>
.jb-product-title-block {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.jb-title-row {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
}

.jb-title {
  flex: 1;
  font-size: 1.25rem;
  font-weight: 900;
  color: var(--jb-text);
  line-height: 1.7;
  letter-spacing: -0.3px;
}

.jb-share-btn {
  flex-shrink: 0;
  width: 42px;
  height: 42px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  color: var(--jb-fuchsia);
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-share-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  transform: translateY(-2px) rotate(-8deg);
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.35);
}

/* قیمت */
.jb-price-row {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
  padding: 14px 0;
  border-top: 2px dashed var(--jb-pink-200);
  border-bottom: 2px dashed var(--jb-pink-200);
}

.jb-price-current {
  font-size: 1.5rem;
  font-weight: 900;
  color: var(--jb-pink-700);
  line-height: 1;
  display: inline-flex;
  align-items: baseline;
  gap: 6px;
}

.jb-price-unit {
  font-size: 0.82rem;
  font-weight: 600;
  color: var(--jb-text-muted);
}

.jb-price-old {
  font-size: 1rem;
  color: var(--jb-text-muted);
  text-decoration: line-through;
  text-decoration-color: #ff4d6d;
  text-decoration-thickness: 2px;
  font-weight: 600;
}

.jb-price-discount {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 5px 12px;
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  color: #fff;
  border-radius: 999px;
  font-weight: 900;
  font-size: 0.8rem;
  box-shadow: 0 6px 16px rgba(201, 24, 74, 0.4);
  border: 2px solid rgba(255, 255, 255, 0.6);
}

.jb-discount-label {
  font-size: 0.68rem;
  opacity: 0.95;
  font-weight: 700;
}

@media (min-width: 768px) {
  .jb-title { font-size: 1.5rem; }
  .jb-price-current { font-size: 1.75rem; }
}
</style>