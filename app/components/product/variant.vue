<script setup>
import { computed, ref, watch } from "vue";
import ProductVerityWeight from "~/components/product/verity/weight.vue";
import ProductVerityCount from "~/components/product/verity/count.vue";

import { useCartStore } from "~/store/cart";
const cartStore = useCartStore();
let addToCartMessage = computed(() => cartStore.addToCartMessage);

const props = defineProps({
  primaryAttribute: { type: Object, default: null },
  secondaryAttribute: { type: Object, default: null },
  primaryLabel: { type: String, default: "گزینه ۱" },
  secondaryLabel: { type: String, default: "گزینه ۲" },
  nestedMap: { type: Object, default: () => ({}) },
  variants: { type: Array, default: () => [] },
  showError: { type: Boolean, default: false },
});

const emit = defineEmits([
  "update:selectedVariant",
  "update:quantity",
  "update:selectedPrimaryId",
  "update:selectedSecondaryId",
]);

const selectedPrimaryId = ref(null);
const selectedSecondaryId = ref(null);
const quantity = ref(1);

const isDualAttribute = computed(
  () => !!props.primaryAttribute && !!props.secondaryAttribute
);

const hasAnyAvailableVariant = (attributeId, valueId) => {
  if (attributeId === props.primaryAttribute?.id) {
    const map = props.nestedMap?.[valueId];
    if (!map) {
      return props.variants.some(
        (v) =>
          v.is_available === true &&
          v.stock > 0 &&
          v.values?.some((val) => val.id === valueId)
      );
    }
    return Object.values(map).some(
      (variant) => variant.is_available === true && variant.stock > 0
    );
  }

  if (attributeId === props.secondaryAttribute?.id) {
    for (const primaryKey of Object.keys(props.nestedMap || {})) {
      const variant = props.nestedMap[primaryKey]?.[valueId];
      if (variant?.is_available === true && variant.stock > 0) {
        return true;
      }
    }
    return false;
  }

  return true;
};

const primaryOptions = computed(() => {
  const attribute = props.primaryAttribute;
  if (!attribute?.values?.length) return [];

  return attribute.values.map((value) => {
    if (isDualAttribute.value) {
      const available = hasAnyAvailableVariant(attribute.id, value.id);
      return {
        id: value.id,
        value: value.value,
        label: value.value,
        isAvailable: available,
      };
    }

    const variant = props.variants.find(
      (v) => v.values?.length === 1 && v.values[0].id === value.id
    );
    const available = variant
      ? variant.is_available === true && variant.stock > 0
      : value.is_available !== false;
    return {
      id: value.id,
      value: value.value,
      label: value.value,
      isAvailable: available,
    };
  });
});

const secondaryOptions = computed(() => {
  const attribute = props.secondaryAttribute;
  if (!attribute?.values?.length) return [];

  if (isDualAttribute.value && !selectedPrimaryId.value) {
    return attribute.values.map((value) => ({
      id: value.id,
      value: value.value,
      label: value.value,
      isAvailable: true,
    }));
  }

  if (isDualAttribute.value && selectedPrimaryId.value) {
    const map = props.nestedMap?.[selectedPrimaryId.value] ?? {};
    return attribute.values.map((value) => {
      const variant = map[value.id];
      const available = variant
        ? variant.is_available === true && variant.stock > 0
        : false;
      return {
        id: value.id,
        value: value.value,
        label: value.value,
        isAvailable: available,
      };
    });
  }

  return attribute.values.map((value) => {
    const variant = props.variants.find(
      (v) => v.values?.length === 1 && v.values[0].id === value.id
    );
    const available = variant
      ? variant.is_available === true && variant.stock > 0
      : value.is_available !== false;
    return {
      id: value.id,
      value: value.value,
      label: value.value,
      isAvailable: available,
    };
  });
});

const secondaryRequired = computed(() => !!props.secondaryAttribute);

const resolveVariant = (primaryId, secondaryId) => {
  if (!primaryId) return null;

  if (secondaryId) {
    const forward = props.nestedMap?.[primaryId]?.[secondaryId];
    if (forward) return forward;

    const backward = props.nestedMap?.[secondaryId]?.[primaryId];
    if (backward) return backward;
  }

  return props.variants.find((variant) =>
    variant.values?.some((value) => value.id === primaryId)
  );
};

watch(selectedPrimaryId, () => {
  emit("update:selectedPrimaryId", selectedPrimaryId.value);

  if (!secondaryRequired.value) return;
  if (!selectedPrimaryId.value) {
    selectedSecondaryId.value = null;
    return;
  }

  const allowedMap = props.nestedMap?.[selectedPrimaryId.value];
  if (!allowedMap) {
    selectedSecondaryId.value = null;
    return;
  }

  const allowedIds = Object.keys(allowedMap).map(Number);
  if (!allowedIds.includes(selectedSecondaryId.value)) {
    const firstAvailable = secondaryOptions.value.find(
      (option) => option.isAvailable
    );
    selectedSecondaryId.value = firstAvailable?.id ?? null;
  }
});

watch(selectedSecondaryId, () => {
  emit("update:selectedSecondaryId", selectedSecondaryId.value);
});

watch([selectedPrimaryId, selectedSecondaryId], ([primaryId, secondaryId]) => {
  if (!primaryId) {
    emit("update:selectedVariant", null);
    return;
  }

  if (secondaryRequired.value && !secondaryId) {
    emit("update:selectedVariant", null);
    return;
  }

  const variant = resolveVariant(primaryId, secondaryId);
  emit("update:selectedVariant", variant?.id ?? null);
});

watch(quantity, (newQuantity) => {
  emit("update:quantity", newQuantity);
});
</script>

<template>
  <div class="jb-variant-container">
    <ProductVerityWeight
      v-if="primaryOptions.length"
      v-model="selectedPrimaryId"
      :options="primaryOptions"
      :label="primaryLabel"
      :show-error="showError && !selectedPrimaryId"
    />

    <ProductVerityWeight
      v-if="secondaryOptions.length"
      v-model="selectedSecondaryId"
      :options="secondaryOptions"
      :label="secondaryLabel"
      :show-error="showError && secondaryRequired && !selectedSecondaryId"
    />

    <ProductVerityCount v-model="quantity" />

    <!-- پیام خطای افزودن به سبد -->
    <transition name="jb-error-slide">
      <div v-if="addToCartMessage" class="jb-variant-error-msg">
        <span class="jb-variant-error-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
              d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
          </svg>
        </span>
        <span>{{ addToCartMessage }}</span>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.jb-variant-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px 0;
}

/* پیام خطا */
.jb-variant-error-msg {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  background: linear-gradient(135deg, #fff5f8, #ffe0e8);
  border: 2px solid #ffb8c8;
  border-radius: 14px;
  color: #c9184a;
  font-size: 0.85rem;
  font-weight: 700;
  box-shadow: 0 4px 14px rgba(201, 24, 74, 0.12);
}

.jb-variant-error-icon {
  flex-shrink: 0;
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  color: #fff;
  box-shadow: 0 4px 12px rgba(201, 24, 74, 0.35);
}

.jb-error-slide-enter-active,
.jb-error-slide-leave-active {
  transition: all 0.35s ease;
}

.jb-error-slide-enter-from,
.jb-error-slide-leave-to {
  opacity: 0;
  transform: translateY(-10px);
  max-height: 0;
  padding-top: 0;
  padding-bottom: 0;
  margin: 0;
}
</style>