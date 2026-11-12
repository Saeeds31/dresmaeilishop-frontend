<!-- components/layout/Footer.vue -->
<template>
  <footer class="jb-footer">
    <!-- حباب‌های تزئینی پس‌زمینه -->
    <div class="jb-footer-bubble jb-footer-bubble-1"></div>
    <div class="jb-footer-bubble jb-footer-bubble-2"></div>

    <div class="container mx-auto px-4 md:px-8 lg:px-12 xl:px-28 relative z-10">
      <!-- ============================================================
           ردیف بالا: لوگو + مزیت‌ها (کارت‌های کوچک)
      ============================================================ -->
      <div class="jb-advantages-row">
        <!-- لوگو -->
        <NuxtLink to="/" class="jb-footer-logo">
          <img src="/images/logo.png" :alt="siteName" />
        </NuxtLink>

        <!-- مزیت‌ها -->
        <div class="jb-advantages-grid">
          <div v-for="item in advantages" :key="item.title" class="jb-advantage-card">
            <div class="jb-advantage-icon">
              <img v-if="!item.enamad" :src="item.icon" :alt="item.title" />
              <a referrerpolicy='origin' v-else target='_blank'
                href='https://trustseal.enamad.ir/?id=702342&Code=u8S0xaRkdoZnDjpDmRlQLwXDczeitXnV'><img
                  referrerpolicy='origin'
                  src='https://trustseal.enamad.ir/logo.aspx?id=702342&Code=u8S0xaRkdoZnDjpDmRlQLwXDczeitXnV' alt=''
                  style='cursor:pointer' code='u8S0xaRkdoZnDjpDmRlQLwXDczeitXnV'></a>

            </div>
            <h4 class="jb-advantage-title">{{ item.title }}</h4>
            <p class="jb-advantage-subtitle">{{ item.subtitle }}</p>
          </div>
        </div>
      </div>

      <!-- خط جداکننده بریده‌بریده -->
      <div class="jb-footer-divider"></div>

      <!-- ============================================================
           ردیف میانی: ستون‌های لینک
      ============================================================ -->
      <div class="jb-footer-columns">
        <!-- ستون ۱: دسترسی سریع -->
        <div class="jb-footer-col">
          <h5 class="jb-footer-col-title">
            <span class="jb-footer-col-dot"></span>
            دسترسی سریع
          </h5>
          <ul class="jb-footer-links">
            <li v-for="(item, i) in flatQuickAccess" :key="`q-${i}`">
              <NuxtLink :to="item.href" class="jb-footer-link">
                <span class="jb-footer-arrow">‹</span>
                {{ item.label }}
              </NuxtLink>
            </li>
          </ul>
        </div>

        <!-- ستون ۲: دسته‌بندی‌ها -->
        <div class="jb-footer-col">
          <h5 class="jb-footer-col-title">
            <span class="jb-footer-col-dot"></span>
            دسته‌بندی‌ها
          </h5>
          <ul class="jb-footer-links">
            <li v-for="(item, i) in flatCategories" :key="`c-${i}`">
              <NuxtLink :to="item.href" class="jb-footer-link">
                <span class="jb-footer-arrow">‹</span>
                {{ item.label }}
              </NuxtLink>
            </li>
          </ul>
        </div>

        <!-- ستون ۳: شبکه‌های اجتماعی -->
        <div class="jb-footer-col">
          <h5 class="jb-footer-col-title">
            <span class="jb-footer-col-dot"></span>
            ما را دنبال کنید
          </h5>
          <div class="jb-footer-socials">
            <a v-for="item in socialLinks" :key="item.label" :href="item.href" target="_blank" rel="noopener"
              class="jb-social-card" :title="item.label">
              <img :src="item.icon" :alt="item.label" />
              <span>{{ item.label }}</span>
            </a>
            <a referrerpolicy='origin' target='_blank'
              href='https://trustseal.enamad.ir/?id=702342&Code=u8S0xaRkdoZnDjpDmRlQLwXDczeitXnV'><img
                referrerpolicy='origin' class="jb-social-card"
                src='https://trustseal.enamad.ir/logo.aspx?id=702342&Code=u8S0xaRkdoZnDjpDmRlQLwXDczeitXnV' alt=''
                style='cursor:pointer' code='u8S0xaRkdoZnDjpDmRlQLwXDczeitXnV'></a>
          </div>
        </div>

        <!-- ستون ۴: اطلاعات تماس -->
        <div class="jb-footer-col">
          <h5 class="jb-footer-col-title">
            <span class="jb-footer-col-dot"></span>
            اطلاعات تماس
          </h5>
          <ul class="jb-footer-contacts">
            <li v-for="item in contactItems" :key="item.label" class="jb-contact-item">
              <span v-if="item.icon" class="jb-contact-icon">
                <img :src="item.icon" :alt="item.label" />
              </span>
              <a v-if="item.href" :href="item.href" class="jb-contact-text" :dir="item.dir || 'rtl'">
                {{ item.label }}
              </a>
              <span v-else class="jb-contact-text">{{ item.label }}</span>
            </li>
          </ul>
        </div>
      </div>

      <!-- خط جداکننده -->
      <div class="jb-footer-divider"></div>

      <!-- ============================================================
           ردیف پایین: کپی‌رایت + برگشت به بالا
      ============================================================ -->
      <div class="jb-footer-bottom">
        <p class="jb-footer-copy">{{ footerNote }}</p>

        <a href="https://padhosha.ir" target="_blank" class="jb-footer-copy jb-footer-copy-link">
          پیاده‌سازی توسط پدهوشا
        </a>

        <!-- دکمه برگشت به بالا -->
        <button type="button" class="jb-scroll-top" @click="scrollTop" aria-label="برگشت به بالا">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 15l7-7 7 7" />
          </svg>
          <span>برگشت به بالا</span>
        </button>
      </div>
    </div>
  </footer>
</template>

<script setup>
import { computed } from "vue";
import { useSettingStore } from "~/store/settings.js";
import { storeToRefs } from "pinia";

const settingStore = useSettingStore();
const { settings, menus } = storeToRefs(settingStore);

const siteName = computed(
  () => settings.value?.site?.site_name || "فروشگاه دکتر اسماعیلی"
);

const footerNote = computed(() => {
  return settings.value?.site?.site_footer_note || "© تمامی حقوق برای فروشگاه دکتر اسماعیلی محفوظ است";
});

/* ============================================================
   مزیت‌ها
============================================================ */
const advantages = [
  { title: "ارسال سریع", subtitle: "۳ تا ۷ روز کاری", icon: "/icons/delivery.svg" },
  { title: "پشتیبانی", subtitle: "در روزهای کاری", icon: "/icons/support.svg" },
  { title: "قیمت مناسب", subtitle: "خرید بدون واسطه", icon: "/icons/quality.svg" },
  { title: "پرداخت امن", subtitle: "درگاه بانکی", icon: "/icons/payment.svg" },
  { title: "ضمانت کیفیت", subtitle: "خرید با اطمینان", icon: "/icons/quality.svg", enamad: true },
];
/* ============================================================
   دسترسی سریع - از منوها یا fallback
============================================================ */
const fallbackQuickAccess = [
  { label: "درباره ما", href: "/about-us" },
  { label: "تماس با ما", href: "/contact-us" },
  { label: "مقالات", href: "/blogs" },
  { label: "پیگیری سفارش", href: "/panel/orders" },
  { label: "فروشگاه", href: "/products" },
];

const flatQuickAccess = computed(() => {
  if (menus.value?.length) {
    return menus.value.slice(0, 6).map((menu) => ({
      label: menu.title,
      href: menu.link || "#",
    }));
  }
  return fallbackQuickAccess;
});

/* ============================================================
   دسته‌بندی‌ها
============================================================ */
const fallbackCategories = [
  { label: "مراقبت پوست", href: "/products/مراقبت-پوست" },
  { label: "مراقبت مو", href: "/products/مراقبت-مو" },
  { label: "مراقبت چشم", href: "/products/مراقبت-چشم" },
  { label: "مکمل ها", href: "/products/مکمل-ها" },
  { label: "محصولات مراقبتی کودک", href: "/products/محصولات-مراقبتی-کودک" },
];

const flatCategories = computed(() => fallbackCategories);

/* ============================================================
   شبکه‌های اجتماعی
============================================================ */
const fallbackSocials = [
  { label: "اینستاگرام", href: "https://instagram.com/dresmaeilishop", icon: "/icons/instagram.svg" },
  { label: "تلگرام", href: "https://t.me/dresmaeilishop", icon: "/icons/telegram.svg" },
];

const socialLinks = computed(() => {
  const socials = settings.value?.social || {};
  const map = [
    { key: "instagram", label: "اینستاگرام", icon: "/icons/instagram.svg" },
    { key: "telegram", label: "تلگرام", icon: "/icons/telegram.svg" },
    { key: "whatsapp", label: "واتساپ", icon: "/icons/whatsapp.svg" },
    { key: "youtube", label: "یوتیوب", icon: "/icons/youtube.svg" },
    { key: "rubika", label: "روبیکا", icon: "/icons/rubika.png" },
  ];

  const dynamicItems = map
    .filter((item) => socials[item.key])
    .map((item) => ({
      label: item.label,
      href: socials[item.key],
      icon: item.icon,
    }));

  return dynamicItems.length ? dynamicItems : fallbackSocials;
});

/* ============================================================
   اطلاعات تماس
============================================================ */
const fallbackContacts = [
  { label: "09210133186-09022072235", href: "tel:09375015769", icon: "/icons/phone.svg", dir: "ltr" },
  { label: "گرگان، خیابان سرخواجه، مجتمع آفتاب، طبقه همکف", icon: "/icons/location.svg" },
];

const contactItems = computed(() => {
  const site = settings.value?.site || {};
  const items = [];

  if (site.site_phone) {
    items.push({
      label: site.site_phone,
      href: `tel:${site.site_phone.replace(/\s+/g, "")}`,
      icon: "/icons/phone.svg",
      dir: "ltr",
    });
  }

  if (site.site_email) {
    items.push({
      label: site.site_email,
      href: `mailto:${site.site_email}`,
      icon: "/icons/mail.svg",
      dir: "ltr",
    });
  }

  if (site.site_address) {
    items.push({
      label: site.site_address,
      icon: "/icons/location.svg",
    });
  }

  return items.length ? items : fallbackContacts;
});

/* ============================================================
   برگشت به بالا
============================================================ */
const scrollTop = () => {
  window.scrollTo({ top: 0, behavior: "smooth" });
};
</script>

<style scoped>
/* =========================================================
   فوتر - تم سفید و روشن با لهجه‌های سبز لوگو
   ========================================================= */

.jb-footer {
  position: relative;
  background: linear-gradient(180deg, #ffffff 0%, #f8fcf9 100%);
  color: #1a202c;
  padding: 56px 0 20px;
  overflow: hidden;
  border-top: 3px solid transparent;
  border-image: linear-gradient(90deg, transparent, #8cc63f, transparent) 1;
}

/* حباب‌های تزئینی - بسیار ملایم با لهجه سبز */
.jb-footer-bubble {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  z-index: 0;
}

.jb-footer-bubble-1 {
  width: 380px;
  height: 380px;
  background: radial-gradient(circle, rgba(140, 198, 63, 0.08), transparent 70%);
  top: -140px;
  right: -100px;
}

.jb-footer-bubble-2 {
  width: 280px;
  height: 280px;
  background: radial-gradient(circle, rgba(14, 90, 58, 0.05), transparent 70%);
  bottom: -100px;
  left: -80px;
}

/* =========================================================
   ردیف مزیت‌ها
   ========================================================= */
.jb-advantages-row {
  display: flex;
  flex-direction: column;
  gap: 28px;
  margin-bottom: 40px;
}

.jb-footer-logo {
  display: block;
  width: 180px;
  margin: 0 auto;
  padding: 18px;
  border-radius: 24px;
  background: rgba(255, 255, 255, 0.9);
  border: 2px dashed rgba(14, 90, 58, 0.15);
  transition: all 0.3s ease;
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
}

.jb-footer-logo:hover {
  background: #fff;
  border-color: rgba(14, 90, 58, 0.3);
  transform: translateY(-3px);
  box-shadow: 0 12px 30px rgba(14, 90, 58, 0.1);
}

.jb-footer-logo img {
  width: 100%;
  height: auto;
  opacity: 1;
}

/* گرید مزیت‌ها */
.jb-advantages-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.jb-advantage-card {
  background: rgba(255, 255, 255, 0.9);
  border: 1.5px solid rgba(14, 90, 58, 0.08);
  border-radius: 16px;
  padding: 16px 12px;
  text-align: center;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}

.jb-advantage-card:hover {
  background: #fff;
  border-color: rgba(14, 90, 58, 0.2);
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(14, 90, 58, 0.1);
}

.jb-advantage-icon {
  width: 46px;
  height: 46px;
  margin: 0 auto 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 13px;
  background: #f1f7f3;
  border: 1.5px solid rgba(14, 90, 58, 0.15);
  transition: all 0.3s ease;
}

.jb-advantage-card:hover .jb-advantage-icon {
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  border-color: transparent;
  box-shadow: 0 8px 20px rgba(14, 90, 58, 0.25);
  transform: scale(1.08) rotate(-4deg);
}

.jb-advantage-icon img {
  width: 22px;
  height: 22px;
  filter: brightness(0) saturate(100%) invert(24%) sepia(15%) saturate(1435%) hue-rotate(101deg) brightness(95%) contrast(91%);
  opacity: 0.9;
  transition: all 0.3s ease;
}

.jb-advantage-card:hover .jb-advantage-icon img {
  filter: brightness(0) invert(1);
}

.jb-advantage-title {
  font-size: 0.82rem;
  font-weight: 800;
  color: #1a202c;
  margin-bottom: 4px;
}

.jb-advantage-subtitle {
  font-size: 0.68rem;
  color: #718096;
  line-height: 1.5;
  font-weight: 500;
}

/* =========================================================
   خط بریده‌بریده
   ========================================================= */
.jb-footer-divider {
  height: 0;
  border-top: 1.5px dashed rgba(14, 90, 58, 0.15);
  margin: 40px 0;
  position: relative;
}

.jb-footer-divider::before,
.jb-footer-divider::after {
  content: "";
  position: absolute;
  top: -5px;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: #8cc63f;
  box-shadow: 0 0 0 3px #fff;
}

.jb-footer-divider::before {
  right: -4px;
}

.jb-footer-divider::after {
  left: -4px;
}

/* =========================================================
   ستون‌ها
   ========================================================= */
.jb-footer-columns {
  display: grid;
  grid-template-columns: 1fr;
  gap: 32px;
}

.jb-footer-col-title {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.9rem;
  font-weight: 800;
  color: #1a202c;
  margin-bottom: 18px;
  position: relative;
  padding-bottom: 10px;
}

.jb-footer-col-title::after {
  content: "";
  position: absolute;
  bottom: 0;
  right: 0;
  width: 40px;
  height: 2.5px;
  border-radius: 999px;
  background: linear-gradient(90deg, #0e5a3a, #8cc63f);
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.3);
}

.jb-footer-col-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  box-shadow: 0 0 0 3px rgba(140, 198, 63, 0.2);
}

/* ===== لینک‌ها ===== */
.jb-footer-links {
  display: flex;
  flex-direction: column;
  gap: 11px;
}

.jb-footer-link {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.82rem;
  color: #4a5568;
  font-weight: 500;
  transition: all 0.25s ease;
  position: relative;
}

.jb-footer-link:hover {
  color: #0e5a3a;
  transform: translateX(-4px);
}

.jb-footer-arrow {
  color: #8cc63f;
  font-size: 1.1rem;
  line-height: 1;
  transition: all 0.25s ease;
  opacity: 0.7;
}

.jb-footer-link:hover .jb-footer-arrow {
  color: #0e5a3a;
  opacity: 1;
  transform: translateX(-2px);
}

/* =========================================================
   شبکه‌های اجتماعی
   ========================================================= */
.jb-footer-socials {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.jb-social-card {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 9px 14px;
  background: rgba(255, 255, 255, 0.9);
  border: 1.5px solid rgba(14, 90, 58, 0.08);
  border-radius: 12px;
  font-size: 0.8rem;
  font-weight: 600;
  color: #4a5568;
  transition: all 0.25s ease;
  width: fit-content;
}

.jb-social-card:hover {
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  border-color: transparent;
  color: #fff;
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(14, 90, 58, 0.3);
}

.jb-social-card img {
  width: 18px;
  height: 18px;
  opacity: 0.85;
  transition: all 0.25s ease;
}

.jb-social-card:hover img {
  opacity: 1;
  filter: brightness(0) invert(1);
}

/* =========================================================
   اطلاعات تماس
   ========================================================= */
.jb-footer-contacts {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.jb-contact-item {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  font-size: 0.82rem;
}

.jb-contact-icon {
  flex-shrink: 0;
  width: 34px;
  height: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 11px;
  background: #f1f7f3;
  border: 1.5px solid rgba(14, 90, 58, 0.15);
  margin-top: 1px;
  transition: all 0.25s ease;
}

.jb-contact-item:hover .jb-contact-icon {
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  border-color: transparent;
  box-shadow: 0 6px 16px rgba(14, 90, 58, 0.3);
  transform: scale(1.05);
}

.jb-contact-icon img {
  width: 15px;
  height: 15px;
  filter: brightness(0) saturate(100%) invert(24%) sepia(15%) saturate(1435%) hue-rotate(101deg) brightness(95%) contrast(91%);
  opacity: 0.9;
  transition: all 0.25s ease;
}

.jb-contact-item:hover .jb-contact-icon img {
  filter: brightness(0) invert(1);
}

.jb-contact-text {
  color: #4a5568;
  font-weight: 500;
  line-height: 1.8;
  transition: color 0.25s ease;
}

a.jb-contact-text:hover {
  color: #0e5a3a;
}

/* =========================================================
   ردیف پایین
   ========================================================= */
.jb-footer-bottom {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  text-align: center;
}

.jb-footer-copy {
  font-size: 0.8rem;
  color: #718096;
  font-weight: 500;
  line-height: 1.8;
}

.jb-footer-copy-link {
  color: #0e5a3a;
  font-weight: 700;
  transition: color 0.25s ease;
}

.jb-footer-copy-link:hover {
  color: #8cc63f;
}

/* دکمه برگشت به بالا */
.jb-scroll-top {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 11px 24px;
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
  border: 2px solid rgba(255, 255, 255, 0.15);
  border-radius: 999px;
  font-size: 0.78rem;
  font-weight: 800;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.3);
  margin-top: 6px;
  position: relative;
  overflow: hidden;
}

.jb-scroll-top::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.35), transparent);
  transform: translateX(-100%);
  transition: transform 0.6s ease;
}

.jb-scroll-top:hover::before {
  transform: translateX(100%);
}

.jb-scroll-top:hover {
  background: linear-gradient(135deg, #0b472e, #0e5a3a);
  transform: translateY(-3px);
  box-shadow: 0 14px 32px rgba(14, 90, 58, 0.4);
}

/* =========================================================
   ریسپانسیو
   ========================================================= */

/* تبلت */
@media (min-width: 640px) {
  .jb-advantages-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* دسکتاپ */
@media (min-width: 768px) {
  .jb-footer {
    padding: 64px 0 24px;
  }

  .jb-footer-columns {
    grid-template-columns: repeat(2, 1fr);
    gap: 36px;
  }

  .jb-footer-bottom {
    flex-direction: row;
    justify-content: space-between;
    text-align: right;
  }

  .jb-scroll-top {
    margin-top: 0;
  }
}

@media (min-width: 1024px) {
  .jb-advantages-row {
    flex-direction: row;
    align-items: center;
    gap: 40px;
  }

  .jb-footer-logo {
    margin: 0;
    flex-shrink: 0;
  }

  .jb-advantages-grid {
    flex: 1;
    grid-template-columns: repeat(5, 1fr);
    gap: 14px;
  }

  .jb-footer-columns {
    grid-template-columns: 1fr 1fr 1fr 1.4fr;
    gap: 32px;
  }
}

@media (min-width: 1280px) {
  .jb-advantages-grid {
    gap: 18px;
  }
}
</style>