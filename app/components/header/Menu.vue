<template>
  <nav class="w-full">
    <!-- ===== دسکتاپ ===== -->
    <ul class="hidden md:flex items-center gap-1 justify-end">
      <li
        v-for="item in menuItems"
        :key="item.id"
        class="group relative"
      >
        <NuxtLink
          :to="item.link || '#'"
          class="inline-flex items-center gap-1.5 px-4 py-2 text-sm font-medium rounded-full transition-all duration-300"
          :class="
            isActive(item.link)
              ? 'text-white bg-[#0e5a3a] shadow-md'
              : 'text-gray-600 hover:text-[#0e5a3a] hover:bg-[#f4f7f6]'
          "
        >
          {{ item.title }}
          <svg
            v-if="item.children?.length"
            class="w-3.5 h-3.5 opacity-70 transition-transform duration-300 group-hover:rotate-180"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
          >
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M19 9l-7 7-7-7" />
          </svg>
        </NuxtLink>

        <!-- زیرمنو دسکتاپ -->
        <div
          v-if="item.children?.length"
          class="absolute top-full right-0 pt-3 opacity-0 invisible translate-y-3 group-hover:opacity-100 group-hover:visible group-hover:translate-y-0 transition-all duration-300 z-[1500]"
        >
          <div class="bg-white rounded-2xl shadow-xl border border-gray-100 min-w-[200px] overflow-hidden py-2">
            <ul class="space-y-0.5">
              <li v-for="sub in item.children" :key="sub.id">
                <NuxtLink
                  :to="sub.link || '#'"
                  class="flex items-center gap-2 px-4 py-2.5 text-sm transition-colors duration-200"
                  :class="
                    isActive(sub.link)
                      ? 'text-[#0e5a3a] bg-[#f4f7f6] font-bold border-r-4 border-[#0e5a3a]'
                      : 'text-gray-600 hover:text-[#0e5a3a] hover:bg-[#f4f7f6]'
                  "
                >
                  <span class="w-1.5 h-1.5 rounded-full bg-gray-300 transition-colors" :class="{'bg-[#8cc63f]': isActive(sub.link)}"></span>
                  {{ sub.title }}
                </NuxtLink>
              </li>
            </ul>
          </div>
        </div>
      </li>
    </ul>

    <!-- ===== موبایل ===== -->
    <ul class="md:hidden flex flex-col gap-1">
      <li v-for="item in menuItems" :key="item.id">
        <div class="flex flex-col">
          <div class="flex items-center justify-between">
            <NuxtLink
              :to="item.link || '#'"
              class="flex-1 py-3 text-base font-bold rounded-xl px-3 transition-colors"
              :class="
                isActive(item.link)
                  ? 'text-[#0e5a3a] bg-[#f4f7f6]'
                  : 'text-gray-700 hover:bg-gray-50'
              "
              @click="!item.children?.length && props.menuToggleFunction()"
            >
              {{ item.title }}
            </NuxtLink>

            <button
              v-if="item.children?.length"
              @click="toggleMobileSubmenu(item.id)"
              class="p-2 mr-1 rounded-full hover:bg-gray-100 transition-colors"
              :aria-expanded="activeMobileSub === item.id"
            >
              <svg
                class="w-4 h-4 text-[#0e5a3a] transition-transform duration-300"
                :class="{ 'rotate-180': activeMobileSub === item.id }"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M19 9l-7 7-7-7" />
              </svg>
            </button>
          </div>

          <!-- زیرمنو موبایل -->
          <ul
            v-if="item.children?.length"
            v-show="activeMobileSub === item.id"
            class="mt-1 mr-3 pr-3 border-r-2 border-dashed border-[#8cc63f]/40 flex flex-col gap-0.5"
          >
            <li v-for="sub in item.children" :key="sub.id">
              <NuxtLink
                :to="sub.link || '#'"
                class="block py-2 px-3 text-sm font-semibold rounded-lg transition-colors"
                :class="
                  isActive(sub.link)
                    ? 'text-[#0e5a3a] bg-[#f4f7f6]'
                    : 'text-gray-500 hover:text-[#0e5a3a] hover:bg-[#f4f7f6]'
                "
                @click="props.menuToggleFunction()"
              >
                {{ sub.title }}
              </NuxtLink>
            </li>
          </ul>
        </div>
      </li>
    </ul>
  </nav>
</template>


<script setup>
import { ref, computed, watch } from "vue";
import { useRoute } from "vue-router";
import { useApi } from "@/composables/api/useApi";

const props = defineProps({
  menuToggleFunction: {
    type: Function,
    default: () => {},
  },
});

const route = useRoute();
const activeMobileSub = ref(null);

const { getBase } = useApi();

const { data: baseResponse } = await useAsyncData(
  "layout-base-data",
  () => getBase(),
  { server: true, lazy: true }
);

const resolveLink = (item) => {
  if (item?.link) return item.link;
  const title = (item?.title || "").trim();
  if (title === "درباره ما") return "/about-us";
  if (title === "تماس با ما") return "/contact-us";
  if (item?.id) return `/products?category_ids=${item.id}`;
  return "#";
};

const normalizeMenus = (rawMenus) => {
  if (!Array.isArray(rawMenus)) return [];
  return rawMenus.map((m) => ({
    id: m.id ?? m.title,
    title: m.title ?? "",
    link: resolveLink(m),
    children: Array.isArray(m.children)
      ? m.children.map((c) => ({
          id: c.id ?? c.title,
          title: c.title ?? "",
          link: resolveLink(c),
        }))
      : [],
  }));
};

const menuItems = computed(() => {
  const data = baseResponse.value?.data ?? baseResponse.value ?? {};
  const rawMenus = data.menus ?? [];
  return normalizeMenus(rawMenus);
});

const isActive = (link) => {
  if (!link || link === "#") return false;
  return route.fullPath === link;
};

watch(
  () => route.fullPath,
  () => {
    activeMobileSub.value = null;
  }
);

const toggleMobileSubmenu = (id) => {
  activeMobileSub.value = activeMobileSub.value === id ? null : id;
};
</script>