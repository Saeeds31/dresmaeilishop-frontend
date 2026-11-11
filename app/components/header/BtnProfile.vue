<template>
  <!-- ====== وارد نشده ====== -->
  <button v-if="!user" @click="showLoginModal()" class="jb-btn-secondary flex items-center gap-2 !py-2 !px-4 text-sm">
    <span class="font-bold whitespace-nowrap">ورود و عضویت</span>
    <img src="/icons/user.svg" class="w-5 h-5" />
  </button>

  <!-- ====== وارد شده ====== -->
  <div v-else class="relative" ref="dropdownRef">
    <button @click="toggleDropdown" class="jb-icon-btn flex items-center gap-2 !py-2 !px-3">
      <span class="font-bold text-xs jb-text whitespace-nowrap max-w-[80px] truncate">
        {{ user.full_name || user.mobile }}
      </span>
      <img src="/icons/user.svg" class="w-5 h-5" />
    </button>

    <transition enter-active-class="transition ease-out duration-100" enter-from-class="transform opacity-0 scale-95"
      enter-to-class="transform opacity-100 scale-100" leave-active-class="transition ease-in duration-75"
      leave-from-class="transform opacity-100 scale-100" leave-to-class="transform opacity-0 scale-95">
      <div v-if="isDropdownOpen" class="absolute left-0 mt-2 w-52 jb-submenu z-50">
        <NuxtLink to="/panel" @click="isDropdownOpen = false" class="jb-submenu-item !flex items-center gap-2">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
          </svg>
          <span>پنل کاربری</span>
        </NuxtLink>

        <button @click="handleLogout"
          class="w-full flex items-center gap-2 px-3 py-2 text-sm font-semibold text-red-600 hover:bg-red-50 rounded-lg transition mt-1">
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