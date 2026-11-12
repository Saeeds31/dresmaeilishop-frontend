<template>
  <!-- ====== وارد نشده ====== -->
  <button 
    v-if="!user" 
    @click="showLoginModal()" 
    class="w-max flex items-center gap-2 bg-white border-2 border-[#0e5a3a] text-[#0e5a3a] hover:bg-[#0e5a3a] hover:text-white transition-all duration-300 rounded-full py-2  px-4 shadow-sm hover:shadow-md group"
  >
    <span class="hidden md:block font-bold text-sm whitespace-nowrap">ورود و عضویت</span>
    <img src="/icons/user.svg" class="w-5 h-5 transition-transform group-hover:scale-110" />
  </button>

  <!-- ====== وارد شده ====== -->
  <div v-else class="relative" ref="dropdownRef">
    <button 
      @click="toggleDropdown" 
      class="flex items-center gap-2 bg-white border border-gray-200 hover:border-[#0e5a3a] rounded-full py-1.5 px-3 transition-all duration-300 shadow-sm hover:shadow"
    >
      <span class="font-bold text-xs text-gray-700 whitespace-nowrap max-w-[80px] truncate">
        {{ user.full_name || user.mobile }}
      </span>
      <div class="w-7 h-7 rounded-full bg-[#f4f7f6] flex items-center justify-center text-[#0e5a3a]">
        <img src="/icons/user.svg" class="w-4 h-4" />
      </div>
    </button>

    <transition 
      enter-active-class="transition ease-out duration-200" 
      enter-from-class="transform opacity-0 scale-95 translate-y-2"
      enter-to-class="transform opacity-100 scale-100 translate-y-0" 
      leave-active-class="transition ease-in duration-150"
      leave-from-class="transform opacity-100 scale-100 translate-y-0" 
      leave-to-class="transform opacity-0 scale-95 translate-y-2"
    >
      <div v-if="isDropdownOpen" class="absolute left-0 mt-3 w-56 bg-white rounded-2xl shadow-xl border border-gray-100 z-50 overflow-hidden py-2">
        <!-- هدر کوچک پروفایل در منو -->
        <div class="px-4 py-2 border-b border-gray-50 mb-1">
          <p class="text-[10px] text-gray-400">حساب کاربری</p>
          <p class="text-xs font-bold text-gray-700 truncate">{{ user.full_name || user.mobile }}</p>
        </div>

        <NuxtLink 
          to="/panel" 
          @click="isDropdownOpen = false" 
          class="flex items-center gap-3 px-4 py-2.5 text-sm font-medium text-gray-600 hover:text-[#0e5a3a] hover:bg-[#f4f7f6] transition-colors"
        >
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
          </svg>
          <span>پنل کاربری</span>
        </NuxtLink>

        <div class="h-px bg-gray-50 mx-2 my-1"></div>

        <button 
          @click="handleLogout"
          class="w-full flex items-center gap-3 px-4 py-2.5 text-sm font-medium text-red-500 hover:bg-red-50 transition-colors"
        >
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
          </svg>
          <span>خروج از حساب</span>
        </button>
      </div>
    </transition>
  </div>

  <AuthLoginModal v-if="loginModal" />
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import { storeToRefs } from "pinia";
import { useSettingStore } from "~/store/settings";
import { useAuth, useApiAuth } from "~/composables/useAuth";
import { toast } from "vue3-toastify";

const { logout } = useApiAuth();
const { loginModal } = useAuth();

const settingStore = useSettingStore();
const { user } = storeToRefs(settingStore);

const isDropdownOpen = ref(false);
const dropdownRef = ref(null);

const toggleDropdown = () => (isDropdownOpen.value = !isDropdownOpen.value);

const handleLogout = async () => {
  isDropdownOpen.value = false;
  await logout();
  toast.success("با موفقیت خارج شدید");
  navigateTo("/");
};

const handleClickOutside = (event) => {
  if (dropdownRef.value && !dropdownRef.value.contains(event.target)) {
    isDropdownOpen.value = false;
  }
};

onMounted(() => document.addEventListener("click", handleClickOutside));
onUnmounted(() => document.removeEventListener("click", handleClickOutside));

const route = useRoute();
function showLoginModal() {
  localStorage.setItem("lastPage", route.fullPath);
  loginModal.value = route.fullPath;
}
</script>