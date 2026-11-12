
<script setup>
import { computed } from "vue";
import { toast } from "vue3-toastify";

const props = defineProps({
  title: String,
  inStock: Boolean,
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
<template>
  <div class="jb-product-title-block">
    <!-- ردیف بالا: عنوان + دکمه اشتراک‌گذاری -->
    <div class="jb-title-row">
      <div class="flex-1">
        <h1 class="jb-title">{{ title }}</h1>
        <!-- وضعیت موجودی -->
        <div class="mt-2 flex items-center gap-2">
          <span 
            class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-xs font-bold"
            :class="inStock ? 'bg-[#f1f7f3] text-[#0e5a3a]' : 'bg-red-50 text-red-500'"
          >
            <span class="w-1.5 h-1.5 rounded-full" :class="inStock ? 'bg-[#8cc63f]' : 'bg-red-400'"></span>
            {{ inStock ? 'موجود در انبار' : 'ناموجود' }}
          </span>
        </div>
      </div>

      <button class="jb-share-btn" @click="shareContent" aria-label="اشتراک‌گذاری">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
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
  font-size: 1.4rem;
  font-weight: 900;
  color: #1a202c;
  line-height: 1.6;
  letter-spacing: -0.3px;
}

.jb-share-btn {
  flex-shrink: 0;
  width: 44px;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 14px;
  background: #fff;
  border: 2px solid rgba(14, 90, 58, 0.1);
  color: #0e5a3a;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-share-btn:hover {
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  border-color: transparent;
  transform: translateY(-2px) rotate(-8deg);
  box-shadow: 0 8px 20px rgba(14, 90, 58, 0.3);
}

/* قیمت */
.jb-price-row {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
  padding: 16px 20px;
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
  border-radius: 16px;
  border: 1.5px solid rgba(14, 90, 58, 0.08);
}

.jb-price-current {
  font-size: 1.8rem;
  font-weight: 900;
  color: #0e5a3a;
  line-height: 1;
  display: inline-flex;
  align-items: baseline;
  gap: 6px;
}

.jb-price-unit {
  font-size: 0.82rem;
  font-weight: 600;
  color: #4a5568;
}

.jb-price-old {
  font-size: 1rem;
  color: #a0aec0;
  text-decoration: line-through;
  text-decoration-color: #8cc63f;
  text-decoration-thickness: 2px;
  font-weight: 600;
}

.jb-price-discount {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 6px 14px;
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
  border-radius: 999px;
  font-weight: 900;
  font-size: 0.85rem;
  box-shadow: 0 6px 16px rgba(14, 90, 58, 0.3);
  border: 2px solid rgba(255, 255, 255, 0.4);
}

.jb-discount-label {
  font-size: 0.68rem;
  opacity: 0.95;
  font-weight: 700;
}

@media (min-width: 768px) {
  .jb-title { font-size: 1.6rem; }
  .jb-price-current { font-size: 2rem; }
}
</style>