<!-- pages/products/index.js -->
<template>
  <main class="container mx-auto md:p-6" dir="rtl">
    <sharedBreadCrumb :items="[{ label: 'محصولات', href: '/products' }]" />

    <section class="flex flex-col lg:flex-row gap-6 mt-0 md:mt-10">
      <!-- سایدبار فیلتر -->
      <ProductsAside class="hidden lg:block" v-model:selectedCategories="filters.categories"
        v-model:selectedWeight="filters.weights" v-model:onlyAvailable="filters.inStock"
        v-model:priceRange="filters.priceRange" v-model:query="filters.search" @apply-filter="applyFilters" />

      <!-- نمایش محصولات -->
      <div class="flex-1">
        <ProductsSort class="hidden lg:block" v-model="sortBy" />
        <ProductsFilterMobile class="lg:hidden" v-model:selectedCategories="filters.categories"
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
    <section v-if="!pending && !error && products?.length"
      class="jb-products-seo-content mt-12 pt-8 border-t-2 border-dashed border-pink-200">
      <h2 class="text-xl font-black text-gray-800 mb-6">راهنمای خرید از مجموعه محصولات فروشگاه دکتر اسماعیلی</h2>

      <div class="space-y-4 text-sm text-gray-600 leading-8 font-semibold">
        <p>
          مجموعه‌ای که در حال مشاهده آن هستید، شامل طیف گسترده‌ای از محصولات مورد نیاز برای نوزاد و کودک شماست.
          از لباس‌های نوزادی و سیسمونی اولیه گرفته تا اسباب‌بازی، لوازم بهداشتی و اقلام ایمنی، همه در تلاش بوده‌ایم
          تا انتخاب‌های متنوعی را متناسب با سلیقه و نیازهای مختلف در اختیار شما قرار دهیم.
        </p>
        <p>
          برای انتخاب آسان‌تر، می‌توانید از فیلترهای کنار صفحه (مانند دسته‌بندی، محدوده قیمت و موجودی) استفاده کنید
          تا سریع‌تر به محصول مورد نظر خود برسید. اگر در انتخاب تردید دارید، پیشنهاد می‌کنیم ابتدا دسته‌بندی مرتبط
          (مثلاً «لباس نوزاد» یا «لوازم حمام») را انتخاب کنید تا گزینه‌های مرتبط‌تر را ببینید.
        </p>
        <p>
          همه محصولات این فروشگاه با تمرکز بر کیفیت و ایمنی برای پوست حساس نوزاد انتخاب شده‌اند. در صورت نیاز به
          مشاوره بیشتر برای تکمیل سبد سیسمونی، می‌توانید از طریق صفحه
          <NuxtLink to="/contact-us" class="text-pink-600 underline decoration-dotted hover:text-pink-700">تماس با ما
          </NuxtLink>
          با کارشناسان فروشگاه دکتر اسماعیلی در ارتباط باشید.
        </p>
      </div>

      <!-- یک بخش سوالات متداول کوتاه (اختیاری اما عالی برای سئو) -->
      <div class="mt-8 pt-6 border-t border-gray-200">
        <h3 class="text-base font-bold text-gray-700 mb-4">سوالات پرتکرار</h3>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6 text-sm">
          <div>
            <p class="font-bold text-gray-800 mb-1">آیا امکان ارسال به سراسر کشور وجود دارد؟</p>
            <p class="text-gray-600 leading-7">بله، سفارشات فروشگاه دکتر اسماعیلی به تمام نقاط ایران ارسال می‌شود.</p>
          </div>
          <div>
            <p class="font-bold text-gray-800 mb-1">اگر سایز لباس مناسب نبود چه کنم؟</p>
            <p class="text-gray-600 leading-7">با توجه به شرایط تعویض، می‌توانید از طریق پشتیبانی پیگیری کنید.</p>
          </div>
        </div>
      </div>
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

// state فیلترها
const filters = ref({
  search: route.query.search || "",
  categories: route.query.category_ids ? route.query.category_ids.split(',') : [],
  weights: [],
  inStock: route.query.in_stock === "1",
  priceRange: [
    route.query.min_price ? Number(route.query.min_price) : '',
    route.query.max_price ? Number(route.query.max_price) : '',
  ],
});

// state مرتب‌سازی
const sortBy = ref(route.query.sort_by || "newest");
const currentPage = ref(Number(route.query.page) || 1);

const apiParams = computed(() => ({
  search: filters.value.search || undefined,
  in_stock: filters.value.inStock ? 1 : undefined,
  min_price: filters.value.priceRange[0] || undefined,
  max_price: filters.value.priceRange[1] || undefined,
  category_ids: filters.value.categories ? filters.value.categories.join(",") : undefined,
  weight: filters.value.weights.length
    ? filters.value.weights.join(",")
    : undefined,
  sort_by: sortBy.value || undefined,
  page: currentPage.value || undefined,
}));

// بارگذاری همزمان base + products + wishlist
const pending = ref(true);
const error = ref(null);
const data = ref(null);

try {
  await loadPageWithBase(
    [
      async () => {
        const response = await getProducts(apiParams.value);
        data.value = response;
        return response;
      },
      async () => await fetchWishlist(),
    ],
    "در حال بارگذاری محصولات...",
    getBase,
  );
} catch (err) {
  error.value = err;
  console.error("خطا در بارگذاری محصولات:", err);
} finally {
  pending.value = false;
}

const products = computed(() => data.value?.data?.data || []);
const paginationData = computed(() => data.value?.data || {});
const totalPages = computed(() => paginationData.value.last_page || 1);

const refresh = async () => {
  pending.value = true;
  error.value = null;
  try {
    await loadPageWithBase(
      [
        async () => {
          const response = await getProducts(apiParams.value);
          data.value = response;
          return response;
        },
      ],
      "در حال بارگذاری مجدد...",
    );
  } catch (err) {
    error.value = err;
  } finally {
    pending.value = false;
  }
};

// تابع تغییر URL وقتی کاربر روی دکمه اعمال فیلتر کلیک می‌کند
async function applyFilters() {
  const query = {};

  if (apiParams.value.search) query.search = apiParams.value.search;
  if (apiParams.value.in_stock) query.in_stock = apiParams.value.in_stock;
  if (apiParams.value.min_price) query.min_price = apiParams.value.min_price;
  if (apiParams.value.max_price) query.max_price = apiParams.value.max_price;
  if (apiParams.value.category_ids)
    query.category_ids = apiParams.value.category_ids;
  if (filters.value.weights.length)
    query.weight = filters.value.weights.join(",");
  if (sortBy.value && sortBy.value !== "newest")
    query.sort_by = sortBy.value;

  // وقتی فیلتر اعمال میشه، صفحه به ۱ برگرده
  currentPage.value = 1;

  await router.push({ query });
}

// watch برای تغییرات sortBy
watch(sortBy, () => {
  applyFilters();
});

// watch برای تغییرات currentPage
watch(currentPage, async (newPage) => {
  const query = { ...route.query };
  if (newPage > 1) {
    query.page = newPage;
  } else {
    delete query.page;
  }
  await router.push({ query });
});

watch(
  () => route.query,
  (newQuery) => {
    // update filters from URL
    filters.value.search = newQuery.search || "";
    filters.value.inStock = newQuery.in_stock === "1";
    filters.value.categories = newQuery.category_id
      ? [Number(newQuery.category_id)]
      : [];
    filters.value.weights = newQuery.weight
      ? newQuery.weight.split(",").map(Number)
      : [];
    filters.value.priceRange = [
      newQuery.min_price ? Number(newQuery.min_price) : 0,
      newQuery.max_price ? Number(newQuery.max_price) : 2000000,
    ];
    sortBy.value = newQuery.sort_by || "newest";
    currentPage.value = Number(newQuery.page) || 1;
    refresh();
  },
);
const siteUrl = "https://dresmaeilishop.ir";
const pageTitle = computed(() => {
  const base = "فروشگاه محصولات فروشگاه دکتر اسماعیلی | خرید لوازم نوزاد و کودک";
  return currentPage.value > 1 ? `${base} - صفحه ${currentPage.value}` : base;
});

useSeoMeta({
  title: () => pageTitle.value,
  description: "خرید آنلاین انواع سیسمونی، لباس نوزاد، اسباب‌بازی و لوازم ایمنی از فروشگاه فروشگاه دکتر اسماعیلی. مشاهده و فیلتر محصولات بر اساس دسته‌بندی، قیمت و موجودی.",
  ogTitle: () => pageTitle.value,
  ogDescription: "مجموعه کامل محصولات فروشگاه دکتر اسماعیلی با امکان فیلتر و جستجوی سریع.",
  ogUrl: () => {
    const base = `${siteUrl}/products`;
    return currentPage.value > 1 ? `${base}?page=${currentPage.value}` : base;
  },
  ogType: "website",
  twitterCard: "summary_large_image",
  robots: "index, follow",
});
useHead({
  link: [
    // کنونیکال خودارجاع برای هر صفحه
    {
      rel: "canonical",
      href: () => {
        const base = `${siteUrl}/products`;
        return currentPage.value > 1 ? `${base}?page=${currentPage.value}` : base;
      },
    },
    // لینک به صفحه بعد (اگر وجود دارد)
    ...(currentPage.value < totalPages.value
      ? [{
        rel: "next",
        href: `${siteUrl}/products?page=${currentPage.value + 1}`,
      }]
      : []),
    // لینک به صفحه قبل (اگر صفحه فعلی بزرگ‌تر از ۱ است)
    ...(currentPage.value > 1
      ? [{
        rel: "prev",
        href: currentPage.value === 2
          ? `${siteUrl}/products`
          : `${siteUrl}/products?page=${currentPage.value - 1}`,
      }]
      : []),
  ],
});
</script>
