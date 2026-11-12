<template>
  <SharedBreadCrumb :items="[{ label: 'درباره ما', href: '/about-us' }]" />
  <main class="jb-about-page">
    <div class="container mx-auto px-4 md:px-8 lg:px-12 ">

      <AboutHeroCollage :image-src="aboutData.hero.imageSrc" :alt="aboutData.hero.alt"
        :fallback-src="aboutData.hero.fallbackSrc" />

      <AboutStorySection :title="aboutData.story.title" :paragraphs="aboutData.story.paragraphs"
        :highlight="aboutData.story.highlight" :image-src="aboutData.story.imageSrc"
        :image-alt="aboutData.story.imageAlt" :fallback-src="aboutData.story.fallbackSrc" />
    </div>
  </main>
</template>

<script setup>
import { computed } from "vue";

const defaultAbout = {
  hero: {
    imageSrc: "/images/room1.jpg",
    alt: "فروشگاه دکتر اسماعیلی در گرگان",
    fallbackSrc: "/contact/Group.svg",
  },
  story: {
    title: "فروشگاه دکتر اسماعیلی، انتخابی شایسته برای سلامت و زیبایی",
    paragraphs: [
      "مجموعه فروشگاه دکتر اسماعیلی از سال ۱۳۹۵ فعالیت خود را با عشق و علاقه به دنیای سلامت و زیبایی آغاز کرد و با تلاش آقای رحیمی نژاد از یک فروشگاه محلی به یکی از فروشگاه‌های محبوب در سطح کشور تبدیل شد.",
      "ما همواره در تلاشیم با ارائه‌ی محصولات باکیفیت و قیمت‌های مناسب، بهترین تجربه‌ی خرید را برای شما عزیزان فراهم کنیم.",
      "فروشگاه دکتر اسماعیلی مجموعه‌ای متنوع از محصولات مراقبت از پوست، مو، مکمل‌های غذایی و لوازم بهداشتی را در اختیار شما قرار می‌دهد تا متناسب با سلیقه و نیاز خود، بهترین انتخاب را داشته باشید.",
    ],
    highlight:
      "ما به دنبال ایجاد تعادل میان کیفیت، ایمنی و زیبایی برای شما هستیم. محصولاتی که با مواد باکیفیت و دغدغه‌ی سلامت پوست و بدن طراحی شده‌اند؛ چون برای ما، آرامش و لبخند شما، مهم‌ترین است.",
    imageSrc: "/images/room2.jpg",

    imageAlt: "محصولات مراقبت از پوست و زیبایی در فروشگاه دکتر اسماعیلی",
    fallbackSrc: "/contact/Group.svg",
  },
};

const { data } = await useAsyncData("about-page", async () => {
  return null;
});

const aboutData = computed(() => data.value ?? defaultAbout);
useSeoMeta({
  title: 'درباره ما | فروشگاه دکتر اسماعیلی - فروشگاه سلامت و زیبایی در گرگان',
  description: 'آشنایی با فروشگاه دکتر اسماعیلی؛ از سال ۱۳۹۵ با تلاش سه خواهر، ارائه‌دهنده محصولات مراقبت از پوست، مو، مکمل‌های غذایی و لوازم بهداشتی باکیفیت در گرگان و سراسر کشور.',
  ogTitle: 'درباره فروشگاه دکتر اسماعیلی',
  ogDescription: 'داستان فروشگاه دکتر اسماعیلی؛ از یک فروشگاه محلی در گرگان تا یکی از فروشگاه‌های محبوب سلامت و زیبایی در کشور.',
  ogImage: 'https://dresmaeilishop.ir/about/Group.jpg', // آدرس کامل تصویر هیرو
  ogUrl: 'https://dresmaeilishop.ir/about-us',
  ogType: 'website',
  twitterCard: 'summary_large_image',
  robots: 'index, follow',
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/about-us' }],
  script: [
    {
      type: 'application/ld+json',
      key: 'about-jsonld',
      innerHTML: JSON.stringify({
        '@context': 'https://schema.org',
        '@type': 'AboutPage',
        'mainEntity': {
          '@type': 'OnlineStore',
          'name': 'فروشگاه دکتر اسماعیلی',
          'url': 'https://dresmaeilishop.ir',
          'logo': 'https://dresmaeilishop.ir/logo.png',
          'foundingDate': '2016', // سال ۱۳۹۵ شمسی ≈ ۲۰۱۶ میلادی
          'description': 'فروشگاه دکتر اسماعیلی از سال ۱۳۹۵ با تلاش سه خواهر فعالیت خود را آغاز کرد و اکنون یکی از فروشگاه‌های محبوب سلامت و زیبایی در سطح کشور است.',
          'telephone': '+989375015769',
          'address': {
            '@type': 'PostalAddress',
            'streetAddress': 'خیابان سرخواجه، مجتمع آفتاب، طبقه همکف',
            'addressLocality': 'گرگان',
            'addressRegion': 'گلستان',
            'addressCountry': 'IR',
          },
          'sameAs': [
            'https://t.me/dresmaeilishop',
            'https://instagram.com/dresmaeilishop',
          ],
        },
      }),
    },
  ],
})
</script>

<style scoped>
.jb-about-page {
  padding-bottom: 80px;
  background: linear-gradient(135deg, #f8fcf9 0%, #e8f3ec 50%, #d1e8db 100%);
  background-attachment: fixed;
  min-height: 100vh;
}
</style>