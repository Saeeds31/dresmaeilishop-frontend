<template>
  <header class="mb-4">
    <!-- ==================== دسکتاپ ==================== -->
    <div class="hidden md:block">
      <!-- ردیف بالا: لوگو + منوها -->
      <div class="jb-header relative z-[50]">
        <div class="container w-full flex items-center justify-between py-1">
          <!-- لوگو -->
          <SharedLogo :width="'110px'" height="'74px'" />

          <!-- منوها -->
          <HeaderMenu class="flex-1 mr-8" />
        </div>
      </div>

      <!-- ردیف پایین: شبکه‌های اجتماعی | سرچ | پروفایل + سبد -->
      <div class="container py-4 flex items-center justify-between gap-6">
        <!-- راست: شبکه‌های اجتماعی -->
        <div class="flex items-center gap-1 shrink-0">
          <a
            v-for="item in socialLinks"
            :key="item.label"
            :href="item.href"
            target="_blank"
            rel="noopener"
            :title="item.label"
            class="jb-icon-btn w-9 h-9"
          >
            <img :src="item.icon" :alt="item.label" class="w-5 h-5" />
          </a>
        </div>

        <!-- وسط: سرچ (بدون دسته‌بندی) -->
        <div class="flex-1 max-w-2xl">
          <HeaderSearch />
        </div>

        <!-- چپ: پروفایل + سبد خرید -->
        <div class="flex items-center gap-2 shrink-0">
          <HeaderBtnProfile />
          <button
            type="button"
            class="relative jb-btn-secondary flex items-center gap-2 !py-2 !px-4"
            @click="openCartSidebar"
          >
            <span class="font-bold text-sm">سبد خرید</span>
            <img src="/icons/bag.svg" class="w-5 h-5" />
            <span v-if="cartItemsCount > 0" class="jb-cart-badge">
              {{ cartItemsCount }}
            </span>
          </button>
        </div>
      </div>
    </div>

    <!-- ==================== موبایل ==================== -->
    <div class="md:hidden">
      <div class="jb-header">
        <div class="container w-full flex items-center justify-between py-3">
          <!-- راست: دکمه منو -->
          <button
            @click="toggleMenu"
            class="jb-icon-btn p-2"
            aria-label="باز کردن منو"
          >
            <IconMenu class="w-6 h-6 text-[color:var(--jb-fuchsia)]" />
          </button>

          <!-- وسط: لوگو -->
          <SharedLogo :width="'80px'" :height="'54px'" />

          <!-- چپ: سبد خرید + پروفایل -->
          <div class="flex items-center gap-2">
            <button
              type="button"
              class="relative jb-icon-btn p-2"
              @click="openCartSidebar"
              aria-label="سبد خرید"
            >
              <img src="/icons/bag.svg" class="w-5 h-5" />
              <span v-if="cartItemsCount > 0" class="jb-cart-badge">
                {{ cartItemsCount }}
              </span>
            </button>
            <HeaderBtnProfile />
          </div>
        </div>
      </div>

      <!-- منوی موبایل -->
      <transition name="slide">
        <div
          v-if="isMenuOpen"
          class="fixed inset-0 bg-black/50 z-[10000]"
          @click="toggleMenu"
        >
          <div
            class="jb-mobile-drawer fixed right-0 top-0 h-full w-[88vw] max-w-[380px] flex flex-col"
            @click.stop
          >
            <!-- هدر: لوگو + بستن -->
            <div class="flex items-center justify-between px-4 py-3 border-b-2 border-dashed border-[color:var(--jb-pink-300)] shrink-0">
              <SharedLogo :width="'70px'" :height="'47px'" />
              <button
                @click="toggleMenu"
                class="jb-icon-btn p-2"
                aria-label="بستن منو"
              >
                <IconClose class="w-5 h-5 text-[color:var(--jb-fuchsia)]" />
              </button>
            </div>

            <!-- سرچ بالا -->
            <div class="px-4 py-3 shrink-0 border-b-2 border-dashed border-[color:var(--jb-pink-200)]">
              <HeaderSearch />
            </div>

            <!-- محتوای تب (قابل اسکرول) -->
            <div class="flex-1 overflow-y-auto overscroll-contain jb-scroll">
              <!-- تب منو -->
              <div v-show="activeTab === 'menu'" class="p-4">
                <HeaderMenu :menuToggleFunction="toggleMenu" />
              </div>

            </div>

            <!-- پایین: شبکه‌های اجتماعی -->
            <div class="shrink-0 border-t-2 border-dashed border-[color:var(--jb-pink-300)] px-4 py-3 bg-white/60">
              <p class="text-xs font-bold jb-text-pink mb-2">ما را دنبال کنید</p>
              <div class="flex items-center gap-2 flex-wrap">
                <a
                  v-for="item in socialLinks"
                  :key="item.label"
                  :href="item.href"
                  target="_blank"
                  rel="noopener"
                  :title="item.label"
                  class="jb-icon-btn w-9 h-9"
                >
                  <img :src="item.icon" :alt="item.label" class="w-4 h-4" />
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
  { key: "telegram",  label: "تلگرام",    icon: "/icons/telegram.svg" },
  { key: "whatsapp",  label: "واتساپ",    icon: "/icons/whatsapp.svg" },
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
    { key: "telegram",  label: "تلگرام",    icon: "/icons/telegram.svg" },
    { key: "whatsapp",  label: "واتساپ",    icon: "/icons/whatsapp.svg" },
    { key: "youtube",   label: "یوتیوب",    icon: "/icons/youtube.svg" },
    { key: "rubika",    label: "روبیکا",    icon: "/icons/rubika.png" },
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
    categories.value = [{ id: 0, title: "همه دسته‌ها" }, ...list];
  } catch {
    categories.value = [{ id: 0, title: "همه دسته‌ها" }];
  } finally {
    loadingCategories.value = false;
  }
};

watch(activeTab, (tab) => {
  if (tab === "categories" && categories.value.length === 0) {
    fetchCategories();
  }
});

watch(isMenuOpen, (open) => {
  if (!open) activeTab.value = "menu";
});
</script>