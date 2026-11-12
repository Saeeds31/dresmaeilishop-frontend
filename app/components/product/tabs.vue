

<script setup>
import { onMounted, ref, computed } from "vue";
import { useApi } from "@/composables/api/useApi";

const props = defineProps({
  specifications: Array,
  description: { type: String, default: "" },
  products: { type: Array, default: () => [] },
  title: { type: String, default: "محصولات مشابه " },
  productId: { type: [String, Number], required: true },
});

console.log("🔵 tabs.vue setup, productId:", props.productId, typeof props.productId);

const { getSimilarProducts } = useApi();
const { data: similarData } = await useAsyncData(
  () => `similar-products-${props.productId}`,
  () => {
    if (!props.productId) return null;
    return getSimilarProducts(props.productId);
  },
  { watch: [() => props.productId] }
);

const similarProducts = computed(() => {
  return similarData.value?.data?.similar_products || [];
});
</script>
<template>
  <Section class="jb-tabs-section flex flex-col lg:flex-row gap-4 mt-8">
    <div class="w-full">
      <ProductTabsSectionTab
        :productId="props.productId"
        :specifications="props.specifications"
        :description="props.description"
      />
    </div>
  </Section>

  <section class="relative pt-10 mt-8">

    <HomeSlidersHomeOfferSlider :mainTitle="title" :products="similarProducts" />
  </section>
</template>

<style scoped>
.jb-related-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
  font-size: 1.25rem;
  font-weight: 900;
  color: #1a202c;
}

.jb-related-title-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  box-shadow: 0 0 0 4px rgba(140, 198, 63, 0.18);
}
</style>