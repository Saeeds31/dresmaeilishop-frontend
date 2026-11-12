<template>
  <main class="jb-blogs-page">
    <div class="container mx-auto px-4 md:px-8 lg:px-12 xl:px-28">
      <!-- ===== Skeleton لودینگ ===== -->
      <div v-if="pending" class="jb-blogs-skeleton">
        <!-- Featured skeleton -->
        <div class="jb-skeleton-featured"></div>

        <!-- Grid skeleton -->
        <div class="jb-blogs-grid">
          <div v-for="n in 6" :key="n" class="jb-skeleton-card">
            <div class="jb-skeleton-img"></div>
            <div class="jb-skeleton-body">
              <div class="jb-skeleton-line jb-skeleton-line-lg"></div>
              <div class="jb-skeleton-line jb-skeleton-line-md"></div>
              <div class="jb-skeleton-line jb-skeleton-line-sm"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- ===== خطا ===== -->
      <div v-else-if="error" class="jb-blogs-error">
        <div class="jb-error-icon">
          <svg class="w-12 h-12" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
              d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
          </svg>
        </div>
        <h3 class="jb-error-title">خطایی در بارگذاری مقالات رخ داد</h3>
        <p class="jb-error-desc">
          لطفاً اتصال اینترنت خود را بررسی کنید و دوباره تلاش کنید
        </p>
        <button @click="refresh" class="jb-btn-linear">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
              d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
          </svg>
          دوباره تلاش کن
        </button>
      </div>

      <!-- ===== لیست مقالات ===== -->
      <div v-else-if="blogs && blogs.length > 0">
        <SharedBreadCrumb :items="[{ label: 'مقالات', href: '/blogs' }]" />

        <!-- هدر سکشن -->
        <div class="jb-blogs-header">
          <div class="jb-blogs-header-title">
            <span class="jb-blogs-header-icon">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253" />
              </svg>
            </span>
            <h1 class="jb-blogs-title">
              مقالات
              <span class="jb-blogs-title-strong">فروشگاه دکتر اسماعیلی</span>
            </h1>
          </div>
          <p class="jb-blogs-desc">
            راهنماها، نکات و تجربه‌های ارزشمند برای سلامت و زیبایی شما
          </p>
        </div>

        <!-- خط بریده‌بریده -->
        <div class="jb-divider"></div>

        <!-- ===== گرید مقالات ===== -->
        <div v-if="gridBlogs.length > 0" class="jb-blogs-grid" id="blogsList">
          <HomeArticlesBigCard v-for="blog in gridBlogs" :key="blog.id" :card="blog" />
        </div>

        <!-- ===== Pagination ===== -->
        <ProductsPagination v-if="pagination" :model-value="currentPage" :total-pages="totalPages" class="mt-10"
          @update:model-value="goToPage" />

      </div>

      <!-- ===== خالی ===== -->
      <div v-else class="jb-blogs-empty">
        <div class="jb-empty-icon">
          <svg class="w-14 h-14" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
              d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
          </svg>
        </div>
        <h3 class="jb-empty-title">مقاله‌ای یافت نشد</h3>
        <p class="jb-empty-desc">به‌زودی مقالات جدید در این بخش منتشر می‌شود</p>
      </div>
    </div>
  </main>
</template>

<script setup>
import { computed, watch, ref } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useBlogStore } from "~/store/blog";
import { useApi } from "@/composables/api/useApi";

const blogStore = useBlogStore();
const { getBlogs } = useApi();
const route = useRoute();
const router = useRouter();

// 🔹 خواندن شماره صفحه از URL (پیش‌فرض ۱)
const currentPage = computed(() => {
  const page = Number(route.query.page);
  return page > 0 ? page : 1;
});

// 🔹 دریافت داده اولیه بر اساس صفحه فعلی URL
const { pending, error, refresh } = await useAsyncData(
  () => `blogs-page-${currentPage.value}`,
  async () => {
    const response = await getBlogs({ page: currentPage.value });
    const fetchedData = response?.data || response;
    blogStore.setBlogs(fetchedData);
    return fetchedData;
  },
  {
    watch: [currentPage],
  }
);

const blogs = computed(() => blogStore.blogs || []);
const pagination = computed(() => blogStore.pagination);
const gridBlogs = computed(() => blogs.value);
const totalPages = computed(() => pagination.value?.last_page || 1);

// 🔹 تغییر صفحه با به‌روزرسانی URL (نه فقط تغییر ref)
const goToPage = (page) => {
  if (page < 1 || page > totalPages.value) return;
  router.push({ query: { ...route.query, page: page === 1 ? undefined : page } });
};

// 🔹 اسکرول به لیست بعد از تغییر صفحه
watch(currentPage, () => {
  if (process.client) {
    document
      .getElementById("blogsList")
      ?.scrollIntoView({ behavior: "smooth" });
  }
});

// ============ SEO ============
const siteUrl = "https://dresmaeilishop.ir";
const currentPageUrl = computed(() => {
  const base = `${siteUrl}/blogs`;
  return currentPage.value > 1 ? `${base}?page=${currentPage.value}` : base;
});

useSeoMeta({
  title: () =>
    currentPage.value > 1
      ? `مقالات فروشگاه دکتر اسماعیلی - صفحه ${currentPage.value} | راهنمای سلامت و زیبایی`
      : "مقالات فروشگاه دکتر اسماعیلی | راهنمای سلامت و زیبایی",
  description:
    "مجموعه مقالات آموزشی و راهنمای خرید محصولات مراقبت از پوست، مو، مکمل‌های غذایی و لوازم بهداشتی. نکات و تجربه‌های ارزشمند برای سلامت و زیبایی شما از فروشگاه دکتر اسماعیلی.",
  ogTitle: () =>
    currentPage.value > 1
      ? `مقالات فروشگاه دکتر اسماعیلی - صفحه ${currentPage.value}`
      : "مقالات فروشگاه دکتر اسماعیلی",
  ogDescription: "راهنماها و نکات کاربردی برای سلامت و زیبایی شما.",
  ogImage: "https://dresmaeilishop.ir/_nuxt/logo.BGpC-RES.png",
  ogUrl: () => currentPageUrl.value,
  ogType: "website",
  twitterCard: "summary_large_image",
  robots: "index, follow",
});

useHead({
  link: [
    {
      rel: "canonical",
      href: () => currentPageUrl.value,
    },
  ],
  script: [
    {
      type: "application/ld+json",
      key: "blogs-list-jsonld",
      innerHTML: computed(() => {
        if (!blogs.value?.length) return null;

        return JSON.stringify({
          "@context": "https://schema.org",
          "@type": "ItemList",
          itemListElement: blogs.value.map((blog, index) => ({
            "@type": "ListItem",
            position: (currentPage.value - 1) * (pagination.value?.per_page || 12) + index + 1,
            url: `${siteUrl}/blog/${blog.slug || blog.id}`,
          })),
        });
      }),
    },
  ],
});
</script>

<style scoped>
/* =========================================================
   صفحه مقالات - تم سبز لوکس
   ========================================================= */

.jb-blogs-page {
  padding-bottom: 80px;
  background: linear-gradient(135deg, #f8fcf9 0%, #e8f3ec 50%, #d1e8db 100%);
  background-attachment: fixed;
  min-height: 100vh;
  padding-top: 20px;
}

/* =========================================================
   هدر سکشن
   ========================================================= */
.jb-blogs-header {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-top: 8px;
  margin-bottom: 20px;
}

.jb-blogs-header-title {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.jb-blogs-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 42px;
  height: 42px;
  border-radius: 14px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.3);
  border: 3px solid #fff;
  flex-shrink: 0;
}

.jb-blogs-title {
  font-size: 1.5rem;
  font-weight: 900;
  color: #1a202c;
  letter-spacing: -0.5px;
}

.jb-blogs-title-strong {
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.jb-blogs-desc {
  font-size: 0.88rem;
  color: #4a5568;
  font-weight: 600;
  line-height: 1.9;
  padding-right: 54px;
}

/* خط بریده‌بریده */
.jb-divider {
  height: 0;
  border-top: 2px dashed rgba(14, 90, 58, 0.15);
  margin: 24px 0;
  position: relative;
}

.jb-divider::before,
.jb-divider::after {
  content: "";
  position: absolute;
  top: -5px;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: #8cc63f;
  box-shadow: 0 0 0 3px #fff;
}

.jb-divider::before {
  right: -4px;
}

.jb-divider::after {
  left: -4px;
}

/* =========================================================
   Featured Blog
   ========================================================= */
.jb-blog-featured-wrap {
  margin-bottom: 32px;
  position: relative;
}

.jb-blog-featured-wrap::before {
  content: "مقاله ویژه";
  position: absolute;
  top: -10px;
  right: 20px;
  padding: 6px 16px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  font-size: 0.72rem;
  font-weight: 900;
  border-radius: 999px;
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.35);
  border: 2px solid #fff;
  z-index: 10;
}

/* =========================================================
   گرید مقالات
   ========================================================= */
.jb-blogs-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 18px;
  margin-top: 8px;
}

@media (min-width: 640px) {
  .jb-blogs-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
  }
}

@media (min-width: 1024px) {
  .jb-blogs-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 22px;
  }
}

/* =========================================================
   Skeleton
   ========================================================= */
.jb-blogs-skeleton {
  padding-top: 40px;
}

.jb-skeleton-featured {
  height: 340px;
  border-radius: 28px;
  background: linear-gradient(90deg, #e8f3ec 0%, #f8fcf9 50%, #e8f3ec 100%);
  background-size: 200% 100%;
  animation: jbShimmer 1.6s ease-in-out infinite;
  margin-bottom: 32px;
  border: 2px solid rgba(14, 90, 58, 0.1);
}

.jb-skeleton-card {
  background: #fff;
  border: 2px solid rgba(14, 90, 58, 0.1);
  border-radius: 24px;
  overflow: hidden;
}

.jb-skeleton-img {
  height: 200px;
  background: linear-gradient(90deg, #e8f3ec 0%, #f8fcf9 50%, #e8f3ec 100%);
  background-size: 200% 100%;
  animation: jbShimmer 1.6s ease-in-out infinite;
}

.jb-skeleton-body {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.jb-skeleton-line {
  height: 14px;
  border-radius: 8px;
  background: linear-gradient(90deg, #e8f3ec 0%, #f8fcf9 50%, #e8f3ec 100%);
  background-size: 200% 100%;
  animation: jbShimmer 1.6s ease-in-out infinite;
}

.jb-skeleton-line-lg {
  width: 85%;
  height: 18px;
}

.jb-skeleton-line-md {
  width: 100%;
}

.jb-skeleton-line-sm {
  width: 60%;
}

@keyframes jbShimmer {
  0% {
    background-position: -200% 0;
  }

  100% {
    background-position: 200% 0;
  }
}

/* =========================================================
   خطا / خالی
   ========================================================= */
.jb-blogs-error,
.jb-blogs-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 14px;
  padding: 80px 24px;
  text-align: center;
  min-height: 60vh;
}

.jb-error-icon,
.jb-empty-icon {
  width: 110px;
  height: 110px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, #e8f3ec, #d1e8db);
  border: 3px dashed rgba(14, 90, 58, 0.25);
  color: #0e5a3a;
  margin-bottom: 8px;
  animation: jbEmptyFloat 3s ease-in-out infinite;
}

.jb-empty-icon {
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
  border-color: rgba(140, 198, 63, 0.5);
  color: #8cc63f;
}

@keyframes jbEmptyFloat {

  0%,
  100% {
    transform: translateY(0);
  }

  50% {
    transform: translateY(-8px);
  }
}

.jb-error-title,
.jb-empty-title {
  font-size: 1.15rem;
  font-weight: 900;
  color: #1a202c;
}

.jb-error-title {
  color: #0e5a3a;
}

.jb-error-desc,
.jb-empty-desc {
  font-size: 0.88rem;
  color: #4a5568;
  font-weight: 600;
  line-height: 1.9;
  max-width: 400px;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-blogs-title {
    font-size: 1.2rem;
  }

  .jb-blogs-header-icon {
    width: 36px;
    height: 36px;
  }

  .jb-blogs-desc {
    font-size: 0.82rem;
    padding-right: 0;
  }

  .jb-skeleton-featured {
    height: 240px;
    border-radius: 22px;
  }

  .jb-skeleton-img {
    height: 180px;
  }
}
</style>