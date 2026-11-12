<template>
  <main class="container mx-auto md:p-6" dir="rtl">
    <sharedBreadCrumb :items="[{ label: 'محصولات', href: '/products' }]" />

    <section class="flex flex-col lg:flex-row gap-6 mt-0 md:mt-10">
      <!-- سایدبار فیلتر -->
      <ProductsAside class="hidden lg:block" :selectedCategories="hasCategory ? [category.id] : []"
        v-model:selectedWeight="filters.weights" v-model:onlyAvailable="filters.inStock"
        v-model:priceRange="filters.priceRange" v-model:query="filters.search" @apply-filter="applyFilters" />

      <!-- نمایش محصولات -->
      <div class="flex-1">
        <ProductsSort class="hidden lg:block" v-model="sortBy" />
        <ProductsFilterMobile class="lg:hidden" :selectedCategories="hasCategory ? [category.id] : []"
          v-model:selectedWeight="filters.weights" v-model:onlyAvailable="filters.inStock"
          v-model:priceRange="filters.priceRange" v-model:query="filters.search" v-model:sortBy="sortBy"
          @apply-filter="applyFilters" />

        <div v-if="pending" class="text-center text-gray-500 mt-6">
          <SharedLoadingSpinner size="lg" />
        </div>

        <div v-else-if="error" class="text-center text-red-500 mt-6">
          خطا در دریافت محصولات
        </div>

        <div v-else-if="!products?.length" class="text-center text-gray-500 mt-6">
          محصولی یافت نشد.
        </div>

        <div v-else id="productList" class="grid grid-cols-2 sm:grid-cols-2 lg:grid-cols-3 gap-6 max-md:gap-3 mt-6">
          <SharedProductCard v-for="item in products" :key="item.id" :product="item" />
        </div>

        <ProductsPagination v-if="!pending && !error && products?.length" v-model="currentPage"
          :total-pages="totalPages" class="mt-6" />
      </div>
    </section>
    <section v-if="!pending && hasCategory" class="mt-12 border-t border-gray-200 pt-8">
      <h1 class="text-xl font-bold mb-4 text-gray-800">
        {{ categoryHeading }}
      </h1>
      <!-- description از API به صورت HTML برمی‌گردد -->
      <div class="category-description" v-html="categoryDescription" />
    </section>
  </main>
</template>

<script setup>
import { ref, computed, watch } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useApi } from "@/composables/api/useApi";
import { usePageLoading } from "@/composables/usePageLoading";

const { getProducts, getBase } = useApi();
const { fetchWishlist } = useWishlist();
const { loadPageWithBase } = usePageLoading();
const route = useRoute();
const router = useRouter();
const config = useRuntimeConfig();
const siteUrl = config.public.siteUrl || "https://dresmaeilishop.ir";

// ✅ مقدار امن برای آدرس تصاویر (قبل از هر await)
const siteImageUrl = config.public.siteImageUrl || "https://dresmaeilishop.ir";

const slugArray = computed(() => {
  const s = route.params.slug;
  if (!s) return [];
  return Array.isArray(s) ? s : [s];
});

const lastSlug = computed(() => {
  const arr = slugArray.value;
  return arr.length ? arr[arr.length - 1] : "";
});

const filters = ref({
  search: route.query.search || "",
  category: lastSlug.value,
  weights: [],
  inStock: route.query.in_stock === "1",
  priceRange: [
    route.query.min_price ? Number(route.query.min_price) : "",
    route.query.max_price ? Number(route.query.max_price) : "",
  ],
});

const sortBy = ref(route.query.sort_by || "newest");
const currentPage = ref(Number(route.query.page) || 1);

const apiParams = computed(() => ({
  search: filters.value.search || undefined,
  in_stock: filters.value.inStock ? 1 : undefined,
  min_price: filters.value.priceRange[0] || undefined,
  max_price: filters.value.priceRange[1] || undefined,
  category_slug: filters.value.category || undefined,
  weight: filters.value.weights.length ? filters.value.weights.join(",") : undefined,
  sort_by: sortBy.value || undefined,
  page: currentPage.value || undefined,
}));

// ✅ داده‌ها را با useAsyncData مدیریت می‌کنیم تا context حفظ شود
const { data, pending, error, refresh } = await useAsyncData(
  "products-page",
  async () => {
    const response = await getProducts(apiParams.value);
    return response;
  },
  { watch: [apiParams] } // ✅ با تغییر فیلترها خودکار refetch می‌شود
);

// ✅ wishlist را جداگانه و بدون await در setup اصلی لود می‌کنیم
onMounted(() => {
  fetchWishlist().catch(() => { });
});

const products = computed(() => data.value?.data?.data || []);
const category = computed(() => data.value?.category || null);
const paginationData = computed(() => data.value?.data || {});
const totalPages = computed(() => paginationData.value.last_page || 1);

// ✅ بررسی وجود دسته بر اساس id (نه فقط truthy بودن شیء)
const hasCategory = computed(() => !!category.value?.id);

// ✅ حالا useSeoMeta را قبل از هر await (که دیگر وجود ندارد) صدا می‌زنیم
const pageTitle = computed(() =>
  hasCategory.value
    ? category.value.meta_title || `${category.value.title} | فروشگاه دکتر اسماعیلی`
    : "همه محصولات | فروشگاه دکتر اسماعیلی"
);

const pageDescription = computed(() =>
  hasCategory.value
    ? category.value.meta_description ||
    (category.value.description
      ? category.value.description.replace(/<[^>]*>/g, "").slice(0, 160).trim()
      : `خرید ${category.value.title} از فروشگاه دکتر اسماعیلی با ارسال سریع.`)
    : "مشاهده‌ی همه‌ی محصولات فروشگاه دکتر اسماعیلی با امکان فیلتر بر اساس دسته‌بندی، قیمت و موجودی."
);

const canonicalUrl = computed(() => {
  const path = route.path.replace(/\/$/, "");
  return `${siteUrl}${path}`;
});

const breadcrumbItems = computed(() => {
  const items = [{ label: "محصولات", href: "/products" }];
  if (!hasCategory.value) return items;
  const slugParts = slugArray.value;
  let accumulated = "/products";
  slugParts.forEach((part, i) => {
    accumulated += `/${part}`;
    const isLast = i === slugParts.length - 1;
    const label =
      isLast && category.value?.title
        ? category.value.title
        : decodeURIComponent(part).replace(/-/g, " ");
    items.push({ label, href: accumulated });
  });
  return items;
});

const categoryHeading = computed(() =>
  hasCategory.value ? `درباره‌ی ${category.value.title}` : "درباره‌ی محصولات"
);

const categoryDescription = computed(() => {
  if (!hasCategory.value) return "";
  const desc = category.value.description;
  if (!desc) return "";
  if (desc === "null") return "";
  return desc;
});

// ✅ سئو را اینجا صدا می‌زنیم (بدون await قبل از آن)
useSeoMeta({
  title: () => pageTitle.value,
  description: () => pageDescription.value,
  ogTitle: () => pageTitle.value,
  ogDescription: () => pageDescription.value,
  ogType: "website",
  ogUrl: () => canonicalUrl.value,
  ogImage: () =>
    hasCategory.value && category.value.main_image
      ? `${siteImageUrl}/${category.value.main_image.replace(/^\/+/, "")}`
      : `${siteImageUrl}/og-default.jpg`,
  twitterCard: "summary_large_image",
  robots: "index, follow",
});

useHead({
  link: [
    {
      rel: "canonical",
      href: () =>
        currentPage.value > 1
          ? `${useRequestURL().origin}${route.path.replace(/\/$/, "")}`
          : canonicalUrl.value,
    },
  ],
  script: [
    {
      type: "application/ld+json",
      innerHTML: JSON.stringify({
        "@context": "https://schema.org",
        "@type": "BreadcrumbList",
        itemListElement: breadcrumbItems.value.map((item, index) => ({
          "@type": "ListItem",
          position: index + 1,
          name: item.label,
          item: item.href
            ? `${useRequestURL().origin}${item.href}`
            : undefined,
        })),
      }),
    },
  ],
});

// توابع و watch ها
async function applyFilters(dataSended) {
  const query = {};

  if (apiParams.value.search) query.search = apiParams.value.search;
  if (apiParams.value.in_stock) query.in_stock = apiParams.value.in_stock;
  if (apiParams.value.min_price) query.min_price = apiParams.value.min_price;
  if (apiParams.value.max_price) query.max_price = apiParams.value.max_price;
  if (filters.value.weights.length) query.weight = filters.value.weights.join(",");
  if (sortBy.value && sortBy.value !== "newest") query.sort_by = sortBy.value;
  currentPage.value = 1;
  if (dataSended.categories && dataSended.categories.length == 1 && dataSended.categories[0] == category.value.id) {

    await router.push({ query });
  } else {
    query.category_ids = dataSended.categories.join(',')
    await router.push({ path: '/products', query: query })
  }
}

watch(sortBy, () => { applyFilters(); });

watch(currentPage, async (newPage) => {
  const query = { ...route.query };
  if (newPage > 1) query.page = newPage;
  else delete query.page;
  await router.push({ query });
});

watch(lastSlug, (newSlug) => {
  filters.value.category = newSlug;
  filters.value.search = "";
  filters.value.weights = [];
  filters.value.inStock = false;
  filters.value.priceRange = ["", ""];
  sortBy.value = "newest";
  currentPage.value = 1;
});

watch(
  () => route.query,
  (newQuery) => {
    filters.value.search = newQuery.search || "";
    filters.value.inStock = newQuery.in_stock === "1";
    filters.value.category = lastSlug.value;
    filters.value.weights = newQuery.weight ? newQuery.weight.split(",").map(Number) : [];
    filters.value.priceRange = [
      newQuery.min_price ? Number(newQuery.min_price) : 0,
      newQuery.max_price ? Number(newQuery.max_price) : 2000000,
    ];
    sortBy.value = newQuery.sort_by || "newest";
    currentPage.value = Number(newQuery.page) || 1;
  }
);
</script>

<style scoped>
/* استایل پایه */
.category-description {
  color: #4b5563;
  line-height: 2;
  font-size: 0.95rem;
}

.category-description :deep(h1),
.category-description :deep(h2),
.category-description :deep(h3) {
  font-weight: 800;
  color: var(--jb-text, #1f2937);
  margin: 1.5rem 0 0.75rem;
  padding-right: 12px;
  border-right: 4px solid #8cc63f;
}

.category-description :deep(p) {
  margin-bottom: 1rem;
}

.category-description :deep(ul),
.category-description :deep(ol) {
  padding-right: 1.5rem;
  margin-bottom: 1rem;
}

.category-description :deep(li) {
  margin-bottom: 0.5rem;
}

/* جدول‌ها شبیه jb-spec-list */
.category-description :deep(table) {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  border: 1.5px solid rgba(14, 90, 58, 0.15);
  border-radius: 14px;
  overflow: hidden;
  margin: 1.25rem 0;
}

.category-description :deep(thead) {
  background: linear-gradient(135deg, #f4f9f6, #e8f3ec);
}

.category-description :deep(th) {
  padding: 14px 18px;
  font-weight: 800;
  font-size: 0.85rem;
  color: #1a202c;
  text-align: right;
  border-bottom: 2px dashed rgba(14, 90, 58, 0.2);
}

.category-description :deep(td) {
  padding: 14px 18px;
  font-size: 0.85rem;
  font-weight: 600;
  color: #4a5568;
  border-bottom: 1px solid rgba(14, 90, 58, 0.08);
}

.category-description :deep(tr:last-child td) {
  border-bottom: none;
}

.category-description :deep(tbody tr:hover) {
  background: #f1f7f3;
}
</style>