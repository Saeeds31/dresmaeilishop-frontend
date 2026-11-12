<template>
  <SharedBreadCrumb :items="[
    { label: 'مقالات', href: '/blogs' },
    {
      label: article?.title || 'جزئیات مقاله',
      href: `/blog/${route.params.id}`,
    },
  ]" />

  <main class="jb-article-page">
    <div class="container mx-auto px-4 md:px-8 lg:px-12 xl:px-28">

      <!-- ===== لودینگ ===== -->
      <div v-if="pending" class="jb-state-block">
        <SharedLoadingSpinner size="lg" />
      </div>

      <!-- ===== خطا ===== -->
      <div v-else-if="error" class="jb-state-block jb-state-error">
        <div class="jb-error-icon">
          <svg class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8"
              d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
          </svg>
        </div>
        <h3 class="jb-error-title">خطا در بارگذاری مقاله</h3>
        <button @click="refresh" class="jb-btn-linear">دوباره تلاش کن</button>
      </div>

      <!-- ===== مقاله ===== -->
      <div v-else-if="article" class="jb-article-wrapper">
        <!-- ============================================================
             هدر مقاله: عنوان + تصویر
        ============================================================ -->
        <section class="jb-article-hero">
          <!-- تصویر -->
          <div class="jb-article-hero-media">
            <img v-if="article.image" :src="$getImageUrl(article.image)" :alt="article.title"
              class="jb-article-hero-img" />
            <div v-else class="jb-article-hero-placeholder">
              <svg class="w-16 h-16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
                  d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
              </svg>
            </div>
          </div>

          <!-- محتوای هدر -->
          <div class="jb-article-hero-content">
            <!-- دسته‌بندی -->
            <div v-if="article.categories?.length" class="jb-article-cats">
              <span v-for="cat in article.categories" :key="cat.id" class="jb-article-cat">
                {{ cat.title }}
              </span>
            </div>

            <!-- عنوان -->
            <h1 class="jb-article-title">{{ article.title }}</h1>

            <!-- خط تزئینی -->
            <div class="jb-article-divider"></div>

            <!-- متادیتا -->
            <div class="jb-article-meta">
              <span v-if="article.author?.full_name" class="jb-meta-item">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
                </svg>
                {{ article.author.full_name }}
              </span>

              <span v-if="article.created_at" class="jb-meta-item">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                </svg>
                {{ formatDate(article.created_at) }}
              </span>

              <span v-if="article.read_time" class="jb-meta-item">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
                {{ article.read_time }} دقیقه مطالعه
              </span>
            </div>

            <!-- خلاصه -->
            <p v-if="article.short_description" class="jb-article-excerpt">
              {{ article.short_description }}
            </p>

            <!-- اشتراک‌گذاری -->
            <div class="jb-article-share">
              <span class="jb-share-label">اشتراک‌گذاری:</span>

              <a v-if="settings?.social?.instagram" :href="settings.social.instagram" target="_blank" rel="noopener"
                class="jb-share-btn jb-share-instagram" aria-label="اینستاگرام">
                <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
                  <path
                    d="M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.069 1.645.069 4.849 0 3.205-.012 3.584-.069 4.849-.149 3.225-1.664 4.771-4.919 4.919-1.266.058-1.644.07-4.85.07-3.204 0-3.584-.012-4.849-.07-3.26-.149-4.771-1.699-4.919-4.92-.058-1.265-.07-1.644-.07-4.849 0-3.204.013-3.583.07-4.849.149-3.227 1.664-4.771 4.919-4.919 1.266-.057 1.645-.069 4.849-.069zm0-2.163C8.741 0 8.333.014 7.053.072 2.695.272.273 2.69.073 7.052.014 8.333 0 8.741 0 12c0 3.259.014 3.668.072 4.948.2 4.358 2.618 6.78 6.98 6.98C8.333 23.986 8.741 24 12 24c3.259 0 3.668-.014 4.948-.072 4.354-.2 6.782-2.618 6.979-6.98.059-1.28.073-1.689.073-4.948 0-3.259-.014-3.667-.072-4.947-.196-4.354-2.617-6.78-6.979-6.98C15.668.014 15.259 0 12 0zm0 5.838a6.162 6.162 0 100 12.324 6.162 6.162 0 000-12.324zM12 16a4 4 0 110-8 4 4 0 010 8zm6.406-11.845a1.44 1.44 0 100 2.881 1.44 1.44 0 000-2.881z" />
                </svg>
              </a>

              <a v-if="settings?.social?.telegram" :href="settings.social.telegram" target="_blank" rel="noopener"
                class="jb-share-btn jb-share-telegram" aria-label="تلگرام">
                <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
                  <path
                    d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm5.894 8.221l-1.97 9.28c-.145.658-.537.818-1.084.508l-3-2.21-1.447 1.394c-.16.16-.295.295-.605.295l.213-3.053 5.56-5.023c.24-.213-.054-.333-.373-.12L7.24 13.51l-2.96-.924c-.643-.204-.657-.643.136-.953l11.57-4.461c.537-.194 1.006.131.908.95z" />
                </svg>
              </a>

              <a v-if="settings?.social?.whatsapp" :href="settings.social.whatsapp" target="_blank" rel="noopener"
                class="jb-share-btn jb-share-whatsapp" aria-label="واتساپ">
                <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
                  <path
                    d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893a11.821 11.821 0 00-3.48-8.413z" />
                </svg>
              </a>

              <button @click="copyLink" class="jb-share-copy">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
                </svg>
                کپی لینک
              </button>
            </div>
          </div>
        </section>

        <!-- ============================================================
             محتوای مقاله (با استایل توضیحات محصول)
        ============================================================ -->
        <section v-if="article.description" class="jb-article-content-section">
          <div class="jb-article-content-top-line">
            <span class="jb-article-content-line"></span>
            <span class="jb-article-content-icon">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
              </svg>
            </span>
            <span class="jb-article-content-line"></span>
          </div>

          <div class="jb-article-content" v-html="article.description"></div>
        </section>

        <!-- ============================================================
             نظرات
        ============================================================ -->
        <!-- <BlogTabsComment /> -->
      </div>

      <!-- ===== یافت نشد ===== -->
      <div v-else class="jb-state-block jb-state-empty">
        <div class="jb-empty-icon">
          <svg class="w-12 h-12" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
              d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
          </svg>
        </div>
        <h3 class="jb-empty-title">مقاله‌ای یافت نشد</h3>
      </div>
    </div>
  </main>
</template>

<script setup>
import { useApi } from "@/composables/api/useApi";
import { useSettingStore } from "~/store/settings";
import { storeToRefs } from "pinia";
import { toast } from "vue3-toastify";

const route = useRoute();
const { $getImageUrl } = useNuxtApp();
const { getBlogDetail } = useApi();

const settingStore = useSettingStore();
const { settings } = storeToRefs(settingStore);

/* ============================================================
   اصلاح: ساختار پاسخ API -> data.article
============================================================ */
const {
  pending,
  error,
  data: articleData,
  refresh,
} = await useAsyncData(`blog-${route.params.id}`, async () => {
  try {
    const response = await getBlogDetail(route.params.id);
    // ساختار پاسخ: { success, message, data: { article: {...}, related: [] } }
    return response?.data?.article ?? response?.data ?? null;
  } catch (err) {
    console.error("❌ Failed to fetch blog:", err);
    throw err;
  }
});

const article = computed(() => articleData.value || null);

const formatDate = (dateString) => {
  if (!dateString) return "";
  try {
    return new Date(dateString).toLocaleDateString("fa-IR", {
      year: "numeric",
      month: "long",
      day: "numeric",
    });
  } catch {
    return dateString;
  }
};

const copyLink = async () => {
  try {
    await navigator.clipboard.writeText(window.location.href);
    toast.success("لینک کپی شد");
  } catch (err) {
    console.error("Failed to copy:", err);
    toast.error("خطا در کپی کردن لینک");
  }
};
const siteUrl = 'https://dresmaeilishop.ir'
const articleUrl = computed(() => `${siteUrl}/blog/${article.value?.slug || route.params.id}`)
const articleImage = computed(() => article.value?.image ? $getImageUrl(article.value.image) : '')

useSeoMeta({
  title: () => article.value?.meta_title || article.value?.title,
  description: () => article.value?.meta_description || article.value?.short_description?.slice(0, 160),
  ogTitle: () => article.value?.title,
  ogDescription: () => article.value?.short_description,
  ogImage: () => articleImage.value,
  ogUrl: () => articleUrl.value,
  ogType: 'article',
  articlePublishedTime: () => article.value?.created_at,
  articleModifiedTime: () => article.value?.updated_at,
  twitterCard: 'summary_large_image',
  robots: 'index, follow, max-image-preview:large',
})

useHead({
  link: [{ rel: 'canonical', href: () => articleUrl.value }],
  script: [
    {
      type: 'application/ld+json',
      key: 'article-jsonld',
      innerHTML: computed(() => {
        if (!article.value) return null

        const a = article.value
        const authorName = a.author?.full_name || 'فروشگاه دکتر اسماعیلی'

        const jsonLd = {
          '@context': 'https://schema.org',
          '@type': 'BlogPosting', // BlogPosting برای مقالات وبلاگ مناسب‌تر از Article است
          'headline': a.title,
          'description': a.meta_description || a.short_description,
          'image': articleImage.value ? [articleImage.value] : [],
          'datePublished': a.created_at,
          'dateModified': a.updated_at,
          'author': {
            '@type': 'Person',
            'name': authorName,
            // چون صفحه اختصاصی نویسنده نداریم، از URL سایت استفاده می‌کنیم
            'url': siteUrl,
          },
          'publisher': {
            '@type': 'Organization',
            'name': 'فروشگاه دکتر اسماعیلی',
            'logo': {
              '@type': 'ImageObject',
              'url': `${siteUrl}/logo.png`, // مسیر لوگوی سایت
            },
          },
          'mainEntityOfPage': {
            '@type': 'WebPage',
            '@id': articleUrl.value,
          },
          // افزودن دسته‌بندی به عنوان بخش مقاله
          'articleSection': a.categories?.map(c => c.title).join(', ') || '',
          // افزودن زمان مطالعه (اختیاری اما مفید)
          'timeRequired': `PT${a.read_time || 5}M`,
        }

        // اضافه کردن BreadcrumbList جداگانه
        const breadcrumbLd = {
          '@context': 'https://schema.org',
          '@type': 'BreadcrumbList',
          'itemListElement': [
            {
              '@type': 'ListItem',
              'position': 1,
              'name': 'خانه',
              'item': siteUrl,
            },
            {
              '@type': 'ListItem',
              'position': 2,
              'name': 'مقالات',
              'item': `${siteUrl}/blogs`,
            },
            {
              '@type': 'ListItem',
              'position': 3,
              'name': a.title,
              'item': articleUrl.value,
            },
          ],
        }

        return JSON.stringify([jsonLd, breadcrumbLd])
      }),
    },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه جزئیات مقاله - تم سبز لوکس
   ========================================================= */

.jb-article-page {
  padding-top: 20px;
  padding-bottom: 80px;
  background: linear-gradient(135deg, #f8fcf9 0%, #e8f3ec 50%, #d1e8db 100%);
  background-attachment: fixed;
  min-height: 100vh;
}

.jb-article-wrapper {
  display: flex;
  flex-direction: column;
  gap: 40px;
}

/* =========================================================
   هدر مقاله
   ========================================================= */
.jb-article-hero {
  display: grid;
  grid-template-columns: 1fr;
  gap: 28px;
  padding: 32px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid rgba(14, 90, 58, 0.1);
  border-radius: 28px;
  box-shadow: 0 15px 40px rgba(14, 90, 58, 0.08);
  position: relative;
  overflow: hidden;
}

.jb-article-hero::before {
  content: "";
  position: absolute;
  width: 320px;
  height: 320px;
  background: radial-gradient(circle, rgba(140, 198, 63, 0.12), transparent 70%);
  top: -140px;
  left: -100px;
  border-radius: 50%;
  pointer-events: none;
}

@media (min-width: 1024px) {
  .jb-article-hero {
    grid-template-columns: 1.15fr 1fr;
    gap: 40px;
    padding: 40px;
    align-items: center;
  }
}

/* ===== تصویر ===== */
.jb-article-hero-media {
  position: relative;
  width: 100%;
  aspect-ratio: 4 / 3;
  border-radius: 24px;
  overflow: hidden;
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
  border: 3px solid #fff;
  box-shadow: 0 12px 30px rgba(14, 90, 58, 0.15);
  order: 1;
}

@media (min-width: 1024px) {
  .jb-article-hero-media {
    order: 0;
    aspect-ratio: 1 / 1;
    max-height: 460px;
  }
}

.jb-article-hero-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.6s ease;
}

.jb-article-hero:hover .jb-article-hero-img {
  transform: scale(1.04);
}

.jb-article-hero-placeholder {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #8cc63f;
}

/* ===== محتوای هدر ===== */
.jb-article-hero-content {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* دسته‌بندی */
.jb-article-cats {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.jb-article-cat {
  display: inline-flex;
  align-items: center;
  padding: 6px 14px;
  background: linear-gradient(135deg, #f4f9f6, #e8f3ec);
  border: 1.5px solid rgba(140, 198, 63, 0.4);
  border-radius: 999px;
  color: #0e5a3a;
  font-size: 0.72rem;
  font-weight: 900;
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.08);
}

/* تیتر */
.jb-article-title {
  font-size: 1.5rem;
  font-weight: 900;
  line-height: 1.5;
  color: #1a202c;
  letter-spacing: -0.4px;
}

@media (min-width: 1024px) {
  .jb-article-title {
    font-size: 1.9rem;
  }
}

/* خط تزئینی */
.jb-article-divider {
  width: 90px;
  height: 3px;
  border-radius: 999px;
  background: linear-gradient(90deg, #0e5a3a, #8cc63f, transparent);
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.2);
}

/* متادیتا */
.jb-article-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: center;
}

.jb-meta-item {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 5px 12px;
  background: #f1f7f3;
  border: 1.5px solid rgba(14, 90, 58, 0.1);
  border-radius: 999px;
  font-size: 0.72rem;
  font-weight: 800;
  color: #4a5568;
}

.jb-meta-item svg {
  color: #0e5a3a;
  flex-shrink: 0;
}

/* خلاصه */
.jb-article-excerpt {
  font-size: 0.92rem;
  line-height: 2.1;
  color: #4a5568;
  font-weight: 600;
  padding: 16px 20px;
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
  border-right: 4px solid #8cc63f;
  border-radius: 14px;
}

/* اشتراک‌گذاری */
.jb-article-share {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
  padding-top: 8px;
}

.jb-share-label {
  font-size: 0.78rem;
  font-weight: 800;
  color: #718096;
}

.jb-share-btn {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  color: #fff;
  text-decoration: none;
  transition: all 0.25s ease;
  border: 2px solid #fff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.jb-share-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
}

.jb-share-instagram {
  background: linear-gradient(135deg, #f58529, #dd2a7b, #8134af);
}

.jb-share-telegram {
  background: linear-gradient(135deg, #29a9eb, #0088cc);
}

.jb-share-whatsapp {
  background: linear-gradient(135deg, #25d366, #128c7e);
}

.jb-share-copy {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: #fff;
  border: 2px solid rgba(14, 90, 58, 0.2);
  border-radius: 12px;
  color: #0e5a3a;
  font-size: 0.78rem;
  font-weight: 800;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-share-copy:hover {
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
  border-color: transparent;
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(14, 90, 58, 0.3);
}

/* =========================================================
   محتوای مقاله (استایل توضیحات محصول)
   ========================================================= */
.jb-article-content-section {
  padding: 32px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid rgba(14, 90, 58, 0.1);
  border-radius: 28px;
  box-shadow: 0 12px 30px rgba(14, 90, 58, 0.06);
}

/* خط تزئینی بالای متن */
.jb-article-content-top-line {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 28px;
}

.jb-article-content-line {
  flex: 1;
  height: 2px;
  background: linear-gradient(90deg, transparent, rgba(140, 198, 63, 0.5), transparent);
  border-radius: 999px;
}

.jb-article-content-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 14px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.3);
  border: 2px solid #fff;
  flex-shrink: 0;
}

/* تایپوگرافی محتوا */
.jb-article-content {
  font-size: 0.95rem;
  line-height: 2.15;
  color: #4a5568;
  font-weight: 500;
  word-break: break-word;
}

/* پاراگراف‌ها */
.jb-article-content :deep(p) {
  margin-bottom: 18px;
  text-align: justify;
}

.jb-article-content :deep(p:last-child) {
  margin-bottom: 0;
}

.jb-article-content :deep(p:first-of-type) {
  font-size: 1rem;
  font-weight: 600;
  color: #1a202c;
  line-height: 2.25;
}

/* تیترها */
.jb-article-content :deep(h1),
.jb-article-content :deep(h2),
.jb-article-content :deep(h3),
.jb-article-content :deep(h4),
.jb-article-content :deep(h5),
.jb-article-content :deep(h6) {
  position: relative;
  color: #1a202c;
  font-weight: 900;
  line-height: 1.6;
  margin-top: 32px;
  margin-bottom: 16px;
  padding-right: 18px;
  padding-bottom: 8px;
  letter-spacing: -0.2px;
}

.jb-article-content :deep(h1:first-child),
.jb-article-content :deep(h2:first-child),
.jb-article-content :deep(h3:first-child) {
  margin-top: 0;
}

.jb-article-content :deep(h1::before),
.jb-article-content :deep(h2::before),
.jb-article-content :deep(h3::before),
.jb-article-content :deep(h4::before),
.jb-article-content :deep(h5::before),
.jb-article-content :deep(h6::before) {
  content: "";
  position: absolute;
  right: 0;
  top: 0.15em;
  bottom: 0.4em;
  width: 5px;
  border-radius: 999px;
  background: linear-gradient(180deg, #0e5a3a, #8cc63f);
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.3);
}

.jb-article-content :deep(h1::after),
.jb-article-content :deep(h2::after),
.jb-article-content :deep(h3::after) {
  content: "";
  position: absolute;
  bottom: 0;
  right: 18px;
  left: 0;
  height: 2px;
  background: linear-gradient(90deg, rgba(140, 198, 63, 0.5), transparent);
  border-radius: 999px;
}

.jb-article-content :deep(h1) {
  font-size: 1.5rem;
}

.jb-article-content :deep(h2) {
  font-size: 1.3rem;
}

.jb-article-content :deep(h3) {
  font-size: 1.15rem;
}

.jb-article-content :deep(h4),
.jb-article-content :deep(h5),
.jb-article-content :deep(h6) {
  font-size: 1rem;
}

/* لیست‌ها */
.jb-article-content :deep(ul),
.jb-article-content :deep(ol) {
  margin: 16px 0 20px;
  padding-right: 22px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.jb-article-content :deep(ul li),
.jb-article-content :deep(ol li) {
  position: relative;
  line-height: 2;
  padding-right: 8px;
}

.jb-article-content :deep(ul) {
  list-style: none;
  padding-right: 8px;
}

.jb-article-content :deep(ul li) {
  padding-right: 26px;
}

.jb-article-content :deep(ul li::before) {
  content: "";
  position: absolute;
  right: 4px;
  top: 0.85em;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  box-shadow: 0 0 0 3px rgba(140, 198, 63, 0.15);
}

.jb-article-content :deep(ol) {
  counter-reset: jb-counter;
  list-style: none;
  padding-right: 8px;
}

.jb-article-content :deep(ol li) {
  counter-increment: jb-counter;
  padding-right: 42px;
}

.jb-article-content :deep(ol li::before) {
  content: counter(jb-counter);
  position: absolute;
  right: 0;
  top: 0.4em;
  width: 26px;
  height: 26px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  font-size: 0.72rem;
  font-weight: 900;
  box-shadow: 0 4px 10px rgba(14, 90, 58, 0.3);
  line-height: 1;
}

/* بولد و ایتالیک */
.jb-article-content :deep(strong),
.jb-article-content :deep(b) {
  color: #0e5a3a;
  font-weight: 900;
}

.jb-article-content :deep(em),
.jb-article-content :deep(i) {
  color: #4a5568;
  font-style: italic;
}

/* لینک */
.jb-article-content :deep(a) {
  color: #0e5a3a;
  font-weight: 800;
  text-decoration: none;
  border-bottom: 2px dashed rgba(140, 198, 63, 0.6);
  transition: all 0.25s ease;
}

.jb-article-content :deep(a:hover) {
  color: #8cc63f;
  border-bottom-color: #0e5a3a;
}

/* نقل‌قول */
.jb-article-content :deep(blockquote) {
  position: relative;
  margin: 24px 0;
  padding: 18px 22px;
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
  border-right: 4px solid transparent;
  border-image: linear-gradient(180deg, #0e5a3a, #8cc63f) 1;
  border-radius: 16px;
  color: #1a202c;
  font-weight: 700;
  font-style: italic;
  line-height: 2;
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.08);
}

.jb-article-content :deep(blockquote::before) {
  content: "❞";
  position: absolute;
  top: -12px;
  right: 16px;
  font-size: 2.2rem;
  color: #8cc63f;
  line-height: 1;
  background: #fff;
  padding: 0 8px;
  border-radius: 8px;
}

/* کد */
.jb-article-content :deep(code) {
  background: linear-gradient(135deg, #f4f9f6, #e8f3ec);
  color: #0e5a3a;
  padding: 2px 10px;
  border-radius: 8px;
  font-family: "Vazirmatn", monospace;
  font-size: 0.85em;
  font-weight: 700;
  border: 1.5px solid rgba(14, 90, 58, 0.1);
}

/* تصاویر */
.jb-article-content :deep(img) {
  display: block;
  max-width: 100%;
  height: auto;
  margin: 22px auto;
  border-radius: 20px;
  border: 3px solid #fff;
  box-shadow: 0 12px 30px rgba(14, 90, 58, 0.15);
  transition: transform 0.35s ease, box-shadow 0.35s ease;
}

.jb-article-content :deep(img:hover) {
  transform: scale(1.02);
  box-shadow: 0 18px 40px rgba(14, 90, 58, 0.25);
}

/* جدول */
.jb-article-content :deep(table) {
  width: 100%;
  margin: 22px 0;
  border-collapse: separate;
  border-spacing: 0;
  border-radius: 16px;
  overflow: hidden;
  border: 2px solid rgba(14, 90, 58, 0.1);
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.08);
}

.jb-article-content :deep(thead) {
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
}

.jb-article-content :deep(th) {
  padding: 14px 16px;
  font-size: 0.85rem;
  font-weight: 900;
  text-align: right;
}

.jb-article-content :deep(td) {
  padding: 12px 16px;
  font-size: 0.85rem;
  border-top: 1.5px dashed rgba(14, 90, 58, 0.1);
  background: #fff;
  color: #4a5568;
  font-weight: 600;
}

.jb-article-content :deep(hr) {
  margin: 28px 0;
  border: none;
  height: 2px;
  background: linear-gradient(90deg, transparent, rgba(140, 198, 63, 0.5), transparent);
  border-radius: 999px;
}

/* =========================================================
   حالت‌ها
   ========================================================= */
.jb-state-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 14px;
  padding: 80px 24px;
  text-align: center;
  min-height: 60vh;
}

.jb-state-error {
  color: #0e5a3a;
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

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {

  .jb-article-hero,
  .jb-article-content-section {
    padding: 20px;
    border-radius: 22px;
  }

  .jb-article-title {
    font-size: 1.2rem;
  }

  .jb-article-content {
    font-size: 0.88rem;
    line-height: 2.05;
  }

  .jb-article-content :deep(h1) {
    font-size: 1.25rem;
  }

  .jb-article-content :deep(h2) {
    font-size: 1.1rem;
  }

  .jb-article-content :deep(h3) {
    font-size: 1rem;
  }

  .jb-article-content-icon {
    width: 34px;
    height: 34px;
    border-radius: 11px;
  }

  .jb-share-copy {
    width: 100%;
    justify-content: center;
  }
}
</style>