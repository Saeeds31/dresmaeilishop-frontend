<script setup>
import { useApi } from "@/composables/api/useApi";
import { useRoute } from "vue-router";
import { useProductStore } from "~/store/product.js"
const route = useRoute();
const { getProduct, getWishlist } = useApi();
const productId = computed(() => route.params.id);
const productStore = useProductStore();
const wishlistData = ref(null);
const commentsData = ref(null);
console.log('PARAM:', route.params.id)

const { data: productData, error: productError, refresh } = useAsyncData(
  () => `product-detail-${productId.value}`,
  async () => {
    const id = productId.value;
    if (!id) return null;
    if (productStore.product[id]) return productStore.product[id];
    const res = await getProduct(id);
    productStore.product[id] = res.data;
    return res;
  },
  {
    watch: [() => productId.value],
    immediate: true,
    dedupe: 'defer',
  }
);

const productPayload = computed(() => productStore.product[productId.value] ?? {});
const productCore = computed(() => productPayload.value.product ?? {});
const specifications = computed(() => productPayload.value.specifications ?? []);
const attributeOrder = computed(
  () => productPayload.value.attributes_order ?? [],
);
const attributes = computed(() => productPayload.value.attributes ?? []);
const attributeMap = computed(() => {
  return attributes.value.reduce((acc, attr) => {
    acc[attr.id] = attr;
    return acc;
  }, {});
});

const primaryAttribute = computed(
  () =>
    attributeMap.value[attributeOrder.value?.[0]] ??
    attributes.value?.[0] ??
    null,
);
const secondaryAttribute = computed(
  () =>
    attributeMap.value[attributeOrder.value?.[1]] ??
    attributes.value?.[1] ??
    null,
);

// 🔹 اصلاح شد: id 1 = رنگ، id 2 = سایز
const attributeLabels = {
  1: "رنگ",
  2: "سایز",
};

const variantAttributes = computed(() => ({
  primary: primaryAttribute.value,
  secondary: secondaryAttribute.value,
  primaryLabel:
    attributeLabels[primaryAttribute.value?.id] ??
    (primaryAttribute.value ? "گزینه ۱" : null),
  secondaryLabel:
    attributeLabels[secondaryAttribute.value?.id] ??
    (secondaryAttribute.value ? "گزینه ۲" : null),
}));

const nestedMap = computed(() => productPayload.value.nested_map ?? {});
const variants = computed(() => productPayload.value.variants ?? []);

const refreshWishlist = async () => {
  try {
    const response = await getWishlist();
    wishlistData.value = response;
  } catch (error) {
    console.error("خطا در رفرش wishlist:", error);
  }
};

const selectedVariantId = ref(null);
const selectedQuantity = ref(1);
const selectedPrimaryId = ref(null);
const selectedSecondaryId = ref(null);

const showWeightError = ref(false);
const cartItems = ref([]);

const isInWishlist = computed(() => {
  const wishlistItems = wishlistData.value?.data || wishlistData.value?.items;

  if (!wishlistItems || !Array.isArray(wishlistItems)) {
    if (productData.value?.data?.wishlist !== undefined) {
      return productData.value.data.wishlist;
    }
    return false;
  }

  return wishlistItems.some((item) => item.product_id === parseInt(productId));
});

// 🔹 تشخیص اینکه هیچ تنوعی موجود نیست
const isCompletelyOutOfStock = computed(() => {
  const list = variants.value;
  if (!list?.length) return true;
  return !list.some(
    (variant) => variant.is_available === true && variant.stock > 0,
  );
});

const product = computed(() => {
  const core = productCore.value;
  if (!core?.id) return null;

  const mainImage = core.main_image ?? null;
  if (variants.value && variants.value.length == 1) {
    selectedVariantId.value = variants.value[0].id;
  }
  let images = [core.main_image];
  if (core?.images?.length) {
    images.push(...core.images.map((image) => image.path));
  }

  const thumbnails = [mainImage].filter(Boolean);

  return {
    id: core.id,
    title: core.title,
    video: core.video,
    description: core.description,
    price: core.price,
    meta_title: core.meta_title,
    meta_description: core.meta_description,
    final_price: core.final_price,
    // 🔹 اصلاح: بر اساس موجودی واقعی تنوع‌ها
    inStock: !isCompletelyOutOfStock.value,
    images,
    thumbnails,
    attributes: attributes.value,
    variants: variants.value,
    nested_map: nestedMap.value,
  };
});

const currentPrice = computed(() => {
  if (!selectedVariantId.value || !product.value?.variants) {
    return product.value?.final_price || 0;
  }

  const selectedVariant = product.value.variants.find(
    (v) => v.id === selectedVariantId.value,
  );
  return selectedVariant?.final_price || product.value?.final_price || 0;
});

const oldPrice = computed(() => {
  if (!selectedVariantId.value || !product.value?.variants) {
    return product.value?.price || 0;
  }

  const selectedVariant = product.value.variants.find(
    (v) => v.id === selectedVariantId.value,
  );
  return selectedVariant?.price || product.value?.price || 0;
});

const handleQuantityChange = (quantity) => {
  selectedQuantity.value = quantity;
};

const handleWishlistUpdated = async () => {
  await refreshWishlist();
};

const handleShowWeightError = () => {
  showWeightError.value = true;
};

const loadCartData = async () => {
  if (!process.client) return;

  const token = localStorage.getItem("auth-token");

  if (token) {
    try {
      const { getCart } = useApi();
      const response = await getCart();
      cartItems.value = response.items || [];
    } catch (err) {
      console.error("❌ Error loading cart from API:", err);
      cartItems.value = [];
    }
  } else {
    const guestCart = localStorage.getItem("guest-cart");
    if (guestCart) {
      try {
        const cartData = JSON.parse(guestCart);
        cartItems.value = cartData.items || [];
      } catch (err) {
        console.error("❌ Error loading cart from localStorage:", err);
        cartItems.value = [];
      }
    } else {
      cartItems.value = [];
    }
  }
};

const handleCartUpdated = async () => {
  // await loadCartData();

  if (typeof window !== "undefined") {
    window.dispatchEvent(new Event("cart-updated"));
    window.dispatchEvent(new Event("open-cart-sidebar"));
  }
};


watch(
  isInWishlist,
  (newValue) => { },
  { immediate: true },
);
// ============ SEO ============
const config = useRuntimeConfig()
const siteUrl = config.public.siteUrl   // از nuxt.config تعریف کن
const siteImageUrl = config.public.siteImageUrl   // از nuxt.config تعریف کن

const siteName = 'فروشگاه دکتر اسماعیلی' // بهتره از config بیاد

// آدرس امن برای تصاویر
const absoluteUrl = (path) => {
  if (!path) return `${siteImageUrl}/og-default.jpg`
  if (path.startsWith('http')) return path
  return `${siteImageUrl}/${path.replace(/^\/+/, '')}`
}

// متن خالص از HTML برای meta description
const stripHtml = (html) => {
  if (!html) return ''
  return html
    .replace(/<[^>]*>/g, ' ')
    .replace(/&nbsp;/g, ' ')
    .replace(/\s+/g, ' ')
    .trim()
}

// قیمت‌ها به عدد
const numericPrice = computed(() => Number(currentPrice.value) || 0)
const numericOldPrice = computed(() => Number(oldPrice.value) || 0)

// درصد تخفیف
const discountPercent = computed(() => {
  if (numericOldPrice.value <= numericPrice.value) return 0
  return Math.round(
    ((numericOldPrice.value - numericPrice.value) / numericOldPrice.value) * 100
  )
})

// canonical با کوئری تمیز (بدون tracking)
const canonicalUrl = computed(() =>
  productId.value ? `${siteUrl}/product/${productId.value}` : siteUrl
)

// متا تگ‌ها
useSeoMeta({
  title: () => product.value?.meta_title,
  description: () => {
    const desc = product.value?.meta_description || ''
    return desc.length > 160 ? desc.slice(0, 157) + '...' : desc
  },
  ogTitle: () => product.value?.meta_title || 'محصول',
  ogDescription: () => stripHtml(product.value?.meta_description || '').slice(0, 200),
  ogImage: () => absoluteUrl(product.value?.images?.[0]),
  ogUrl: () => canonicalUrl.value,
  ogType: 'product',
  ogSiteName: siteName,
  ogLocale: 'fa_IR',
  twitterCard: 'summary_large_image',
  twitterTitle: () => product.value?.meta_title || '',
  twitterDescription: () => stripHtml(product.value?.meta_description || '').slice(0, 200),
  twitterImage: () => absoluteUrl(product.value?.images?.[0]),
  robots: 'index, follow, max-image-preview:large',
})



// ============ Structured Data (JSON-LD) ============
useHead({
  link: [
    { rel: 'canonical', href: canonicalUrl },
  ],
  script: [
    {
      type: 'application/ld+json',
      key: 'product-jsonld',
      innerHTML: computed(() => {
        if (!product.value) return null
        const p = product.value

        const availability = p.inStock
          ? 'https://schema.org/InStock'
          : 'https://schema.org/OutOfStock'

        const jsonLd = {
          '@context': 'https://schema.org',
          '@type': 'Product',
          name: p.title,
          image: p.images.map(absoluteUrl),
          description: stripHtml(p.description).slice(0, 500),
          sku: String(p.id),
          brand: {
            '@type': 'Brand',
            name: siteName,
          },
          offers: {
            '@type': 'Offer',
            url: canonicalUrl.value,
            priceCurrency: 'IRR', // تومان نیست! ریال. اگه تومانه از واحد دیگه استفاده کن
            price: numericPrice.value,
            availability,
            itemCondition: 'https://schema.org/NewCondition',
            seller: {
              '@type': 'Organization',
              name: siteName,
            },
            ...(numericOldPrice.value > numericPrice.value && {
              priceSpecification: {
                '@type': 'UnitPriceSpecification',
                price: numericPrice.value,
                priceCurrency: 'IRR',
              },
            }),
          },
          ...(p.variants?.length > 1 && {
            hasVariant: p.variants.map((v) => ({
              '@type': 'Product',
              name: `${p.title} - ${v.values?.map(x => x.value).join(' / ') || ''}`,
              sku: String(v.id),
              offers: {
                '@type': 'Offer',
                price: Number(v.final_price),
                priceCurrency: 'IRR',
                availability: v.is_available && v.stock > 0
                  ? 'https://schema.org/InStock'
                  : 'https://schema.org/OutOfStock',
                url: canonicalUrl.value,
              },
            })),
          }),
        }

        // Breadcrumb
        const breadcrumbLd = {
          '@context': 'https://schema.org',
          '@type': 'BreadcrumbList',
          itemListElement: [
            {
              '@type': 'ListItem',
              position: 1,
              name: 'خانه',
              item: siteUrl,
            },
            {
              '@type': 'ListItem',
              position: 2,
              name: 'محصولات',
              item: `${siteUrl}/products`,
            },
            {
              '@type': 'ListItem',
              position: 3,
              name: p.title,
              item: canonicalUrl.value,
            },
          ],
        }

        return JSON.stringify([jsonLd, breadcrumbLd])
      }),
    },
  ],
})
</script>
<template>
  <main class="container mx-auto px-4 md:px-6 lg:px-8 py-6">
    <SharedBreadCrumb :items="[
      { label: 'محصولات', href: '/products' },
      { label: product?.title || 'محصول', href: `/product/${productId}` },
    ]" />

    <div v-if="product" class="mt-6">
      <!-- ===== بخش اصلی: گالری + اطلاعات ===== -->
      <section class="grid grid-cols-1 lg:grid-cols-12 gap-8 lg:gap-12 items-start">

        <!-- ستون راست: گالری تصاویر (۷ از ۱۲) -->
        <div class="lg:col-span-7 order-2 lg:order-1">
          <ProductImage :images="product.images" :video="product.video" :thumbnails="product.images" />
        </div>

        <!-- ستون چپ: اطلاعات محصول (۵ از ۱۲) -->
        <div class="lg:col-span-5 order-1 lg:order-2">
          <div class="lg:sticky lg:top-24 space-y-6">

            <!-- کارت اصلی اطلاعات -->
            <div class="bg-white rounded-3xl border border-[rgba(14,90,58,0.1)] shadow-sm p-6 md:p-8">

              <!-- عنوان و قیمت -->
              <ProductTitle  :title="product.title" :price="currentPrice" :oldPrice="oldPrice"
                :in-stock="product.inStock" />

              <!-- خط جداکننده -->
              <div class="my-6 h-px bg-gradient-to-l from-transparent via-[rgba(14,90,58,0.15)] to-transparent"></div>

              <!-- تنوع‌ها -->
              <ProductVariant v-model:selectedVariant="selectedVariantId" v-model:quantity="selectedQuantity"
                v-model:selectedPrimaryId="selectedPrimaryId" v-model:selectedSecondaryId="selectedSecondaryId"
                :primary-attribute="variantAttributes.primary" :secondary-attribute="variantAttributes.secondary"
                :primary-label="variantAttributes.primaryLabel" :secondary-label="variantAttributes.secondaryLabel"
                :nested-map="nestedMap" :variants="variants" :show-error="showWeightError" />
            </div>

            <!-- کارت خرید (چسبان) -->
            <div class="bg-white rounded-3xl border border-[rgba(14,90,58,0.1)] shadow-lg p-5 md:p-6">
              <ProductBuy :price="currentPrice" :old-price="oldPrice" :in-stock="product.inStock"
                :variant-id="selectedVariantId" :variants="variants" :quantity="selectedQuantity"
                :product-id="product.id" :product-title="product.title" :product-image="product.images[0]"
                :is-in-wishlist="isInWishlist" @wishlist-updated="handleWishlistUpdated"
                @show-weight-error="handleShowWeightError" @cart-updated="handleCartUpdated" />
            </div>

          </div>
        </div>
      </section>

      <!-- ===== بخش‌های بعدی (تب‌ها و محصولات مرتبط) ===== -->
      <ProductTabs class="mt-16 lg:mt-24" :specifications="specifications" :description="product.description"
        :comments="commentsData" :productId="product.id" />
      <ProductRelatedProducts class="mt-16 lg:mt-24" />
    </div>

    <div v-else class="text-center py-16">
      <SharedLoadingSpinner size="xl" />
    </div>
  </main>
</template>