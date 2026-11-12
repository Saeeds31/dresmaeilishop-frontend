<template>
  <header class="mb-4 font-sans">
    <!-- ==================== دسکتاپ ==================== -->
    <div class="hidden md:block">
      <!-- ردیف اول: لوگو + پروفایل/سبد خرید -->
      <div class="bg-white border-b border-gray-100 relative z-[50]">
        <div class="container mx-auto px-4 w-full flex items-center justify-between py-4">
          <!-- راست: لوگو (بزرگ و برجسته) -->
          <nuxt-link  to="/" class="block shrink-0 transition-transform hover:scale-105">
            <img src="/images/logo.png" alt="دکتر اسماعیلی" class="w-[160px] h-auto object-contain" />
          </nuxt-link>

          <!-- چپ: پروفایل + سبد خرید -->
          <div class="flex items-center gap-3 shrink-0">
            <HeaderBtnProfile />
            <button type="button"
              class="relative flex items-center gap-2  border-2 border-[#0e5a3a]  bg-[#0e5a3a] text-white transition-all duration-300 rounded-full py-2 px-5 shadow-sm hover:shadow-md"
              @click="openCartSidebar">
              <span class="font-bold text-sm hidden lg:block">سبد خرید</span>
              <img src="/icons/bag.svg" class="w-5 h-5" />
              <span v-if="cartItemsCount > 0"
                class="absolute -top-1.5 -right-1.5 bg-[#8cc63f] text-white text-[10px] font-bold w-5 h-5 flex items-center justify-center rounded-full shadow-md border-2 border-white">
                {{ cartItemsCount }}
              </span>
            </button>
          </div>
        </div>
      </div>

      <!-- ردیف دوم: دکمه دسته‌بندی + منو + جستجو -->
      <div class="bg-[#f9fbfa] border-b border-gray-100 py-3 shadow-sm sticky top-0 z-40">
        <div class="container mx-auto px-4 flex items-center justify-between gap-6">

          <div class="flex items-center gap-6 flex-1">
            <!-- دکمه دسته‌بندی‌ها با منوی کشویی داینامیک -->
            <div class="relative shrink-0" @mouseenter="isMegaMenuOpen = true" @mouseleave="isMegaMenuOpen = false">
              <button
                class="flex items-center gap-2 bg-[#0e5a3a] text-white px-5 py-2.5 rounded-full hover:bg-[#0b472e] transition-all shadow-md hover:shadow-lg">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24"
                  stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
                </svg>
                <span class="font-medium text-sm whitespace-nowrap">دسته‌بندی‌ها</span>
                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 transition-transform duration-300"
                  :class="{ 'rotate-180': isMegaMenuOpen }" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                </svg>
              </button>

              <!-- مگا منوی داینامیک (سطح ۱ و ۲) -->
              <transition name="fade">
                <div v-if="isMegaMenuOpen && categories.length > 0"
                  class="absolute top-full right-0 mt-2 w-[700px] bg-white rounded-2xl shadow-2xl border border-gray-100 overflow-hidden z-[60]">
                  <div class="flex min-h-[300px]">
                    <!-- ستون سطح ۱ (دسته‌های اصلی) -->
                    <div
                      class="w-1/3 bg-gray-50 p-4 border-l border-gray-100 overflow-y-auto max-h-[400px] custom-scrollbar">
                      <ul class="space-y-1">
                        <li v-for="cat in categories" :key="cat.id" @mouseenter="activeCategory = cat"
                          class="px-3 py-2.5 rounded-lg cursor-pointer transition-all duration-200 text-sm font-medium flex items-center justify-between"
                          :class="activeCategory?.id === cat.id ? 'bg-white text-[#0e5a3a] shadow-sm border-r-4 border-[#0e5a3a]' : 'text-gray-600 hover:text-[#0e5a3a] hover:bg-white'">
                          <nuxt-link :to="`/products/${cat.slug ?? cat.id}`"
                            class="flex items-center justify-between w-full">
                            {{ cat.title }}
                            <svg v-if="cat.children && cat.children.length" xmlns="http://www.w3.org/2000/svg"
                              class="w-4 h-4 opacity-50" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M15 19l-7-7 7-7" />
                            </svg>

                          </nuxt-link>

                        </li>
                      </ul>
                    </div>

                    <!-- ستون سطح ۲ (زیردسته‌ها) -->
                    <div class="w-2/3 p-6 bg-white">
                      <div v-if="activeCategory">
                        <h4 class="font-bold text-gray-800 mb-4 text-base border-b pb-2 flex items-center gap-2">
                          <span class="w-2 h-2 rounded-full bg-[#8cc63f]"></span>
                          {{ activeCategory.title }}
                        </h4>
                        <div v-if="activeCategory.children && activeCategory.children.length"
                          class="grid grid-cols-2 gap-x-6 gap-y-3">
                          <nuxt-link v-for="sub in activeCategory.children" :key="sub.id"
                            :to="`/products/${activeCategory.slug ?? activeCategory.id}/${sub.slug ?? sub.id}`"
                            class="text-sm text-gray-600 hover:text-[#0e5a3a] flex items-center gap-2 transition-colors py-1 group/link">
                            <span
                              class="w-1.5 h-1.5 rounded-full bg-gray-300 group-hover/link:bg-[#8cc63f] transition-colors"></span>
                            {{ sub.title }}
                          </nuxt-link>
                        </div>
                        <p v-else class="text-sm text-gray-400 italic">زیردسته‌ای برای این گروه تعریف نشده است.</p>
                      </div>
                    </div>
                  </div>
                </div>
              </transition>
            </div>

            <!-- منوی اصلی -->
            <HeaderMenu class="flex-1" />
          </div>

          <!-- سرچ در انتهای ردیف -->
          <div class="w-72 shrink-0">
            <HeaderSearch />
          </div>
        </div>
      </div>
    </div>

    <!-- ==================== موبایل (بدون تغییر) ==================== -->
    <div class="md:hidden">
      <div class="bg-white shadow-sm sticky top-0 z-[50]">
        <div class="container mx-auto  w-full flex items-center justify-between py-3">
          <!-- راست: دکمه منو -->
          <button @click="toggleMenu" class="p-2 rounded-full hover:bg-gray-100 transition-colors"
            aria-label="باز کردن منو">
            <IconMenu class="w-6 h-6 text-[#0e5a3a]" />
          </button>

          <!-- وسط: لوگو -->
          <nuxt-link  to="/" class="block w-[110px]">
            <img src="/images/logo.png" alt="دکتر اسماعیلی" class="w-[110px] h-auto" />
          </nuxt-link>

          <!-- چپ: سبد خرید + پروفایل -->
          <div class="flex items-center gap-1">
            <button type="button" class="relative p-2 rounded-full bg-[#0e5a3a] border-2 border-[#0e5a3a]text-white transition-all duration-300"
              @click="openCartSidebar" aria-label="سبد خرید">
              <img src="/icons/bag.svg" class="w-5 h-5" />

              <span v-if="cartItemsCount > 0"
                class="absolute top-0 right-0 bg-[#8cc63f] text-white text-[10px] font-bold w-4 h-4 flex items-center justify-center rounded-full border border-white">
                {{ cartItemsCount }}
              </span>
            </button>
            <HeaderBtnProfile />
          </div>
        </div>
      </div>

      <!-- منوی موبایل -->
      <transition name="slide">
        <div v-if="isMenuOpen" class="fixed inset-0 bg-black/50 z-[10000] backdrop-blur-sm" @click="toggleMenu">
          <div class="bg-white fixed right-0 top-0 h-full w-[88vw] max-w-[380px] flex flex-col shadow-2xl" @click.stop>
            <!-- هدر: لوگو + بستن -->
            <div class="flex items-center justify-between px-4 py-4 border-b border-gray-100 shrink-0 bg-gray-50">
              <img src="/images/logo.png" alt="دکتر اسماعیلی" class="w-[90px] h-auto" />
              <button @click="toggleMenu"
                class="p-2 rounded-full bg-white shadow-sm text-gray-500 hover:text-red-500 transition-colors"
                aria-label="بستن منو">
                <IconClose class="w-5 h-5" />
              </button>
            </div>

            <!-- سرچ بالا -->
            <div class="px-4 py-3 shrink-0 border-b border-gray-100">
              <HeaderSearch />
            </div>

            <!-- محتوای تب (قابل اسکرول) -->
            <div class="flex-1 overflow-y-auto overscroll-contain jb-scroll p-4">
              <!-- دکمه دسته‌بندی‌ها در موبایل -->
              <div class="mb-6">
                <button @click="activeTab = 'categories'"
                  class="w-full flex items-center justify-between bg-[#0e5a3a] text-white px-4 py-3 rounded-xl font-bold">
                  <span class="flex items-center gap-2">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24"
                      stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                        d="M4 6h16M4 12h16M4 18h16" />
                    </svg>
                    دسته‌بندی محصولات
                  </span>
                  <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="none" viewBox="0 0 24 24"
                    stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                  </svg>
                </button>
              </div>

              <!-- تب منو -->
              <div v-show="activeTab === 'menu'">
                <HeaderMenu :menuToggleFunction="toggleMenu" />
              </div>

              <!-- تب دسته بندی ها (موبایل) -->
              <div v-show="activeTab === 'categories'" class="space-y-3">
                <div v-for="cat in categories" :key="cat.id" class="border border-gray-100 rounded-xl overflow-hidden">
                  <button @click="toggleMobileCat(cat.id)"
                    class="w-full flex items-center justify-between bg-gray-50 px-4 py-3 text-sm font-bold text-gray-800">
                    {{ cat.title }}
                    <svg v-if="cat.children && cat.children.length" xmlns="http://www.w3.org/2000/svg"
                      class="w-4 h-4 transition-transform" :class="{ 'rotate-180': openMobileCat === cat.id }"
                      fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                    </svg>
                  </button>
                  <div v-if="openMobileCat === cat.id && cat.children" class="bg-white p-3 space-y-2">
                    <a v-for="sub in cat.children" :key="sub.id" href="#"
                      class="block text-sm text-gray-600 hover:text-[#0e5a3a] py-1 border-b border-gray-50 last:border-0">
                      {{ sub.title }}
                    </a>
                  </div>
                </div>
              </div>
            </div>

            <!-- پایین: شبکه‌های اجتماعی -->
            <div class="shrink-0 border-t border-gray-100 px-4 py-4 bg-gray-50">
              <p class="text-xs font-bold text-[#0e5a3a] mb-3">ما را دنبال کنید</p>
              <div class="flex items-center gap-3 flex-wrap">
                <a v-for="item in socialLinks" :key="item.label" :href="item.href" target="_blank" rel="noopener"
                  :title="item.label"
                  class="w-10 h-10 flex items-center justify-center rounded-full bg-white shadow-sm border border-gray-100 hover:border-[#8cc63f] transition-colors">
                  <img :src="item.icon" :alt="item.label" class="w-5 h-5" />
                </a>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>
  </header>
</template>
<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch } from "vue";
import { useApi } from "@/composables/api/useApi";
import { storeToRefs } from "pinia";
import { useSettingStore } from "~/store/settings";
import { useCategoryStore } from "~/store/menu.js";

const categoryStore = useCategoryStore();
const settingStore = useSettingStore();
const { getBase } = useApi();

const isMenuOpen = ref(false);
const activeTab = ref("menu");

// --- اضافه شده برای مگا منو ---
const isMegaMenuOpen = ref(false);
const activeCategory = ref(null);
const openMobileCat = ref(null);

const toggleMobileCat = (id) => {
  openMobileCat.value = openMobileCat.value === id ? null : id;
};

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};

const openCartSidebar = () => {
  if (typeof window === "undefined") return;
  window.dispatchEvent(new Event("open-cart-sidebar"));
};

/* ---------------- سبد خرید ---------------- */
const cartItemsCount = ref(0);

const readCartCount = () => {
  if (typeof window === "undefined") return 0;
  try {
    const summary = JSON.parse(localStorage.getItem("cart-summary") || "{}");
    if (typeof summary.totalQuantity === "number") return summary.totalQuantity;
    const guestCart = JSON.parse(localStorage.getItem("guest-cart") || "{}");
    const items = guestCart.items || [];
    return items.reduce((sum, item) => sum + (Number(item.quantity) || 0), 0);
  } catch {
    return 0;
  }
};

const syncCartCount = () => {
  cartItemsCount.value = readCartCount();
};

onMounted(() => {
  syncCartCount();
  window.addEventListener("storage", syncCartCount);
  window.addEventListener("cart-updated", syncCartCount);
});

onBeforeUnmount(() => {
  window.removeEventListener("storage", syncCartCount);
  window.removeEventListener("cart-updated", syncCartCount);
});

/* ---------------- دریافت تنظیمات ---------------- */
const { data: baseData } = await useAsyncData(
  "headerBaseData",
  async () => {
    try {
      return await getBase();
    } catch (error) {
      console.error("❌ Failed to fetch base data:", error);
      return { data: { settings: [] } };
    }
  },
  { lazy: true, server: false }
);

/* ---------------- شبکه‌های اجتماعی ---------------- */
const fallbackSocials = [
  { key: "instagram", label: "اینستاگرام", icon: "/icons/instagram.svg" },
  { key: "telegram", label: "تلگرام", icon: "/icons/telegram.svg" },
  { key: "whatsapp", label: "واتساپ", icon: "/icons/whatsapp.svg" },
];

function extractHandle(url) {
  if (!url) return "";
  try {
    const u = new URL(url);
    return u.pathname.replace(/\/+$/, "").replace(/^\//, "") || u.hostname;
  } catch {
    return url;
  }
}

const socialLinks = computed(() => {
  const socials = baseData.value?.data?.settings?.site?.social || {};
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

/* ---------------- دسته‌بندی‌ها ---------------- */
const categories = ref([]);
const loadingCategories = ref(false);

const fetchCategories = async () => {
  loadingCategories.value = true;
  try {
    const res = await categoryStore.fetchFilters();
    const list = res?.data?.categories || [];
    categories.value = list;
    // انتخاب پیش‌فرض اولین دسته برای نمایش در مگا منو
    if (categories.value.length > 0) {
      activeCategory.value = categories.value[0];
    }
  } catch {
    categories.value = [];
  } finally {
    loadingCategories.value = false;
  }
};

// فراخوانی در زمان لود شدن کامپوننت
onMounted(() => {
  fetchCategories();
});

watch(activeTab, (tab) => {
  if (tab === "categories" && categories.value.length === 0) {
    fetchCategories();
  }
});

watch(isMenuOpen, (open) => {
  if (!open) activeTab.value = "menu";
});
</script>

<style scoped>
/* اسکرول بار سفارشی برای مگا منو */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #cbd5e1;
  border-radius: 10px;
}

/* انیمیشن‌ها */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(100%);
}
</style>