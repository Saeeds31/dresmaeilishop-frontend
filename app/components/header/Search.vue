<template>
  <div class="relative w-full">
    <div
      class="flex items-center bg-white rounded-2xl border-2 border-gray-100 overflow-hidden h-12 focus-within:border-[#0e5a3a] focus-within:shadow-[0_0_0_4px_rgba(14,90,58,0.1)] transition-all duration-300 shadow-sm hover:shadow-md">
      <!-- اینپوت -->
      <input v-model="query" @keyup.enter="handleSearch" type="text" placeholder="جستجوی محصولات …"
        class="flex-1 text-sm font-medium px-4 outline-none text-right bg-transparent text-gray-700 placeholder:text-gray-400" />

      <!-- دکمه جستجو -->
      <button type="button"
        class="w-16 h-full flex items-center justify-center transition-all duration-300 hover:opacity-90 active:scale-95"
        style="background: linear-gradient(135deg, #0e5a3a, #8cc63f);" @click="handleSearch" aria-label="جستجو">
        <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
            d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
        </svg>
      </button>
    </div>
  </div>
</template>
<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";

const query = ref("");
const router = useRouter();

const handleSearch = () => {
  const q = query.value.trim();
  if (!q) {
    router.push("/products");
    return;
  }
  router.push(`/products?search=${encodeURIComponent(q)}`);
};
</script>