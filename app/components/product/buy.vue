<template>
  <div class="jb-buy-block">
    <div class="jb-buy-row">
      <!-- دکمه افزودن به سبد -->
      <button
        v-if="props.inStock"
        @click="handleAddToCart"
        :disabled="isAddingToCart"
        class="jb-buy-btn"
        :class="{ 'is-loading': isAddingToCart }"
      >
        <ShoppingCart v-if="!isAddingToCart" :size="20" />
        <Loader v-else :size="20" class="animate-spin" />
        <span>
          {{ isAddingToCart ? "در حال افزودن..." : "افزودن به سبد خرید" }}
        </span>
      </button>

      <!-- حالت ناموجود -->
      <template v-else>
        <button disabled class="jb-buy-btn is-disabled">
          <span>ناموجود</span>
        </button>
        <StockAlertButton
          :product-id="props.productId"
          :is-in-stock="props.inStock"
          @request-updated="handleStockAlertUpdated"
        />
      </template>

      <!-- دکمه علاقه‌مندی -->
      <button
        @click="toggleWishlist"
        class="jb-wish-btn"
        :class="{ 'is-active': isInWishlist }"
        aria-label="افزودن به علاقه‌مندی‌ها"
      >
        <svg v-if="isInWishlist" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
          <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z" />
        </svg>
        <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="w-5 h-5">
          <path stroke-linecap="round" stroke-linejoin="round"
            d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { ShoppingCart, Loader } from "lucide-vue-next";
import { useApi } from "@/composables/api/useApi";
import { toast } from "vue3-toastify";
import StockAlertButton from "@/components/product/StockAlertButton.vue";
import { useCartStore } from "~/store/cart";

const cartStore = useCartStore();

const props = defineProps({
  price: Number,
  oldPrice: Number,
  inStock: Boolean,
  variantId: Number,
  variants: Array,
  quantity: { type: Number, default: 1 },
  productId: Number,
  productTitle: String,
  productImage: String,
  weight: { type: String, default: "" },
  isInWishlist: { type: Boolean, default: false },
});

const emit = defineEmits([
  "wishlist-updated",
  "show-weight-error",
  "cart-updated",
]);

const { addToCart, addToWishlist, removeFromWishlist } = useApi();
const isAddingToCart = ref(false);

const handleAddToCart = async () => {
  if (!props.variantId) {
    toast.error("لطفا یکی از ویژگی ها را انتخاب کنید");
    emit("show-weight-error");
    return;
  }

  isAddingToCart.value = true;
  let selectedVariant = props.variants.find((vari) => vari.id == props.variantId);
  if (!selectedVariant) return;
  selectedVariant.attributes = [];
  selectedVariant.values.forEach((vals) => {
    selectedVariant.attributes.push({ name: vals.attribute, value: vals.value });
  });
  try {
    const productData = {
      id: props.productId,
      product_id: props.productId,
      name: props.productTitle,
      title: props.productTitle,
      variant: selectedVariant,
      price_original: selectedVariant.price,
      price_final: selectedVariant.final_price,
      line_original_total: selectedVariant.price * props.quantity,
      line_final_total: selectedVariant.final_price * props.quantity,
      line_discount:
        (selectedVariant.price - selectedVariant.final_price) * props.quantity,
      image: props.productImage,
      weight: props.weight,
    };

    const result = await addToCart(props.variantId, props.quantity, productData);
    emit("cart-updated");
  } catch (error) {
    console.log(error);
    cartStore.addToCartMessage =
      error.response.data.message || "خطا در افزودن به سبد خرید";
    setTimeout(() => {
      cartStore.addToCartMessage = "";
    }, 5000);
    toast.error(error.response.data.message || "خطا در افزودن به سبد خرید");
  } finally {
    isAddingToCart.value = false;
  }
};

const toggleWishlist = async () => {
  if (!props.productId) return;
  try {
    if (props.isInWishlist) {
      await removeFromWishlist(props.productId);
    } else {
      await addToWishlist(props.productId);
    }
    emit("wishlist-updated");
  } catch (error) {
    console.error("❌ Wishlist toggle error:", error);
    toast.error("خطا در تغییر وضعیت علاقه‌مندی");
  }
};
</script>

<style scoped>
.jb-buy-block {
  margin-top: 20px;
}

.jb-buy-row {
  display: flex;
  align-items: center;
  gap: 12px;
}

/* دکمه خرید */
.jb-buy-btn {
  flex: 1;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 14px 28px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border: none;
  border-radius: 16px;
  font-size: 0.95rem;
  font-weight: 800;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.4);
  position: relative;
  overflow: hidden;
}

.jb-buy-btn::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.35), transparent);
  transform: translateX(-100%);
  transition: transform 0.6s ease;
}

.jb-buy-btn:hover::before {
  transform: translateX(100%);
}

.jb-buy-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-600), var(--jb-pink-800));
  transform: translateY(-3px);
  box-shadow: 0 14px 32px rgba(214, 51, 132, 0.55);
}

.jb-buy-btn.is-loading {
  cursor: wait;
  opacity: 0.85;
}

.jb-buy-btn.is-disabled {
  background: linear-gradient(135deg, #d1d5db, #9ca3af);
  box-shadow: none;
  cursor: not-allowed;
}

.jb-buy-btn.is-disabled:hover {
  transform: none;
}

/* دکمه علاقه‌مندی */
.jb-wish-btn {
  flex-shrink: 0;
  width: 52px;
  height: 52px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  border: 2px solid var(--jb-pink-300);
  border-radius: 16px;
  color: var(--jb-pink-400);
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-wish-btn:hover {
  background: var(--jb-pink-100);
  color: var(--jb-pink-600);
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.2);
}

.jb-wish-btn.is-active {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.5);
  animation: jbHeartBeat 0.5s ease;
}

.jb-wish-btn:active {
  transform: scale(0.92);
}

@keyframes jbHeartBeat {
  0% { transform: scale(1); }
  30% { transform: scale(1.25); }
  60% { transform: scale(0.95); }
  100% { transform: scale(1); }
}

@media (max-width: 480px) {
  .jb-buy-btn {
    padding: 12px 18px;
    font-size: 0.85rem;
  }
  .jb-wish-btn {
    width: 46px;
    height: 46px;
  }
}
</style>