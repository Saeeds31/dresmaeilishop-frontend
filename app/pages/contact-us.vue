<template>
  <main class="jb-contact-page">
    <div class="container mx-auto px-4 md:px-8 lg:px-12 xl:px-28">
      <SharedBreadCrumb class="mb-8" :items="[{ label: 'تماس با ما', href: '/contact-us' }]" />

      <ContactHeroSection :contact-info="contactInfo" />

      <ContactFormSection class="mt-16" :model="form" :loading="loading" @submit="submitForm" />
    </div>
  </main>
</template>

<script setup>
import { reactive, ref, computed } from "vue";

const config = useRuntimeConfig();
const apiBase = (config.public.apiBase || config.public.baseURL || "").replace(
  /\/$/,
  ""
);

const defaultContactInfo = {
  address: "گرگان - خیابان سرخواجه , مجتمع آفتاب سه طبقه همکف",
  mobile: "09112808522",
  phone: "01732330749",
  supportHours: "۰۹:۰۰ تا ۱۷:۰۰",
  storeHours: "۰۹:۰۰ تا ۲۱:۰۰",
  telegram: "https://t.me/dresmaeilishop",
  telegramId: "dresmaeilishop",
  instagram: "https://instagram.com/dresmaeilishop",
  instagramId: "dresmaeilishop",
};

const { data } = await useAsyncData("contact-page", async () => {
  return null;
});

const contactInfo = computed(
  () => data.value?.contactInfo ?? defaultContactInfo
);

const form = reactive({
  name: "",
  phone: "",
  message: "",
});

const loading = ref(false);

const toEnglishDigits = (value) => {
  const persianNums = ["۰", "۱", "۲", "۳", "۴", "۵", "۶", "۷", "۸", "۹"];
  return value
    .toString()
    .split("")
    .map((char) => {
      const index = persianNums.indexOf(char);
      if (index > -1) return index.toString();
      return char;
    })
    .join("");
};

const resetForm = () => {
  form.name = "";
  form.phone = "";
  form.message = "";
};

const submitForm = async () => {
  const name = form.name.trim();
  const message = form.message.trim();
  const normalizedPhone = toEnglishDigits(form.phone).replace(/\D/g, "");

  if (!name) {
    alert("لطفاً نام و نام خانوادگی را وارد کنید.");
    return;
  }

  if (normalizedPhone.length !== 11) {
    alert("شماره موبایل باید دقیقاً ۱۱ رقم باشد.");
    return;
  }

  if (!message) {
    alert("متن پیام نمی‌تواند خالی باشد.");
    return;
  }

  loading.value = true;

  try {
    const fd = new FormData();
    fd.append("full_name", name);
    fd.append("subject", "پیام از فرم تماس وب‌سایت");
    fd.append("mobile", normalizedPhone);
    fd.append("email", "");
    fd.append("body", message);

    await $fetch("/front/contact", {
      baseURL: apiBase,
      method: "POST",
      body: fd,
      headers: {
        Accept: "application/json",
      },
    });

    alert("پیام شما با موفقیت ارسال شد.");
    resetForm();
  } catch (error) {
    console.error(error);

    const status = error?.response?.status;
    const apiErrors = error?.response?._data;

    if (status === 422 && apiErrors?.errors) {
      const messages = Object.values(apiErrors.errors).flat().join("\n");
      alert(messages || apiErrors.message || "اعتبارسنجی با خطا مواجه شد.");
    } else {
      alert("ارسال پیام با خطا مواجه شد. لطفاً دوباره تلاش کنید.");
    }
  } finally {
    loading.value = false;
  }
};
useSeoMeta({
  title: 'تماس با فروشگاه دکتر اسماعیلی | آدرس و شماره تماس فروشگاه در گرگان',
  description: 'راه‌های ارتباط با فروشگاه فروشگاه دکتر اسماعیلی. آدرس: گرگان، خیابان سرخواجه، مجتمع آفتاب. شماره تماس: ۰۹۳۷۵۰۱۵۷۶۹. ساعات کاری فروشگاه و پشتیبانی.',
  ogTitle: 'تماس با فروشگاه دکتر اسماعیلی',
  ogDescription: 'آدرس فروشگاه، شماره تماس و فرم ارتباط با فروشگاه دکتر اسماعیلی در گرگان.',
  ogUrl: 'https://dresmaeilishop.ir/contact-us',
  ogType: 'website',
  robots: 'index, follow',
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/contact-us' }],
  script: [
    {
      type: 'application/ld+json',
      key: 'contact-jsonld',
      innerHTML: JSON.stringify({
        '@context': 'https://schema.org',
        '@type': 'ContactPage',
        'mainEntity': {
          '@type': 'LocalBusiness',
          'name': 'فروشگاه دکتر اسماعیلی',
          'url': 'https://dresmaeilishop.ir',
          'telephone': '+989375015769',
          'address': {
            '@type': 'PostalAddress',
            'streetAddress': 'خیابان سرخواجه، مجتمع آفتاب، طبقه همکف',
            'addressLocality': 'گرگان',
            'addressRegion': 'گلستان',
            'addressCountry': 'IR',
          },
          'openingHoursSpecification': [
            {
              '@type': 'OpeningHoursSpecification',
              'dayOfWeek': ['Saturday', 'Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday'],
              'opens': '09:00',
              'closes': '21:00',
            },
          ],
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
.jb-contact-page {
  padding-bottom: 80px;
  background: linear-gradient(135deg, #fff8fb 0%, #fff0f5 50%, #ffeaf2 100%);
  background-attachment: fixed;
  min-height: 100vh;
}
</style>