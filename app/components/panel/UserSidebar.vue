<template>
  <aside class="jb-panel-sidebar">
    <!-- ===== بخش پروفایل ===== -->
    <div class="jb-profile-card">
      <div class="jb-profile-avatar">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
        </svg>
      </div>

      <p class="jb-profile-name">
        {{ userProfile?.full_name || "کاربر" }}
      </p>
      <p class="jb-profile-mobile" dir="ltr">
        {{ formatMobile(userProfile?.mobile) }}
      </p>
    </div>

    <!-- خط جداکننده -->
    <div class="jb-sidebar-divider"></div>

    <!-- ===== منوها ===== -->
    <ul class="jb-panel-menu">
      <li v-for="(item, index) in menu" :key="index">
        <NuxtLink :to="item.to" class="jb-panel-menu-item" :class="{ 'is-active': route.path === item.to }"
          @click="handleMenuClick(item.to)">
          <span class="jb-panel-menu-icon">
            <component :is="item.icon" class="w-4 h-4" />
          </span>
          <span class="jb-panel-menu-label">{{ item.label }}</span>

          <svg v-if="route.path === item.to" class="jb-panel-menu-arrow" fill="none" stroke="currentColor"
            viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
          </svg>
        </NuxtLink>
      </li>

      <!-- دکمه خروج -->
      <li class="jb-panel-logout-li">
        <button @click="logout" class="jb-panel-logout-btn" type="button">
          <span class="jb-panel-menu-icon jb-panel-logout-icon">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
            </svg>
          </span>
          <span>خروج از حساب</span>
        </button>
      </li>
    </ul>
  </aside>
</template>

<script setup>
import { useRoute, useRouter } from "vue-router";
import { nextTick, ref, watch, computed } from "vue";
import {
  User,
  MapPin,
  ShoppingBag,
  Heart,
  Wallet,
} from "lucide-vue-next";

const route = useRoute();
const router = useRouter();

const pendingScroll = ref(false);

const userProfile = computed(() => {
  if (process.client) {
    const userData = localStorage.getItem("user");
    if (userData) {
      try {
        return JSON.parse(userData);
      } catch (error) {
        console.error("❌ Failed to parse user data from localStorage:", error);
        return null;
      }
    }
  }
  return null;
});

const menu = [
  { label: "حساب کاربری", to: "/panel", icon: User },
  { label: "کیف پول", to: "/panel/wallet", icon: Wallet },
  { label: "لیست آدرس‌ها", to: "/panel/addresses", icon: MapPin },
  { label: "پیگیری سفارشات", to: "/panel/orders", icon: ShoppingBag },
  { label: "لیست علاقه‌مندی‌ها", to: "/panel/favorites", icon: Heart },
];

const formatMobile = (mobile) => {
  if (!mobile) return "---";
  const mobileStr = mobile.toString();
  if (mobileStr.length >= 10) {
    return `${mobileStr.slice(0, 4)}***${mobileStr.slice(-3)}`;
  }
  return mobile;
};

const handleMenuClick = (to) => {
  if (process.client && window.innerWidth < 768) {
    if (route.path === to) {
      scrollDown();
      return;
    }
    pendingScroll.value = true;
  }
};

const scrollDown = () => {
  window.scrollBy({
    top: 150,
    behavior: "smooth",
  });
};

watch(
  () => route.path,
  async () => {
    if (!pendingScroll.value) return;
    if (!process.client || window.innerWidth >= 768) {
      pendingScroll.value = false;
      return;
    }

    await nextTick();

    setTimeout(() => {
      scrollDown();
      pendingScroll.value = false;
    }, 300);
  }
);

const logout = () => {
  router.push("/");
};
</script>

<style scoped>
/* =========================================================
   سایدبار پنل کاربری - مینیمال سبز
   ========================================================= */

.jb-panel-sidebar {
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 16px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

/* =========================================================
   کارت پروفایل
   ========================================================= */
.jb-profile-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 10px 0;
}

.jb-profile-avatar {
  width: 64px;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: #0e5a3a;
  color: #fff;
  margin-bottom: 4px;
}

.jb-profile-name {
  font-size: 0.95rem;
  font-weight: 800;
  color: #1a202c;
  margin-top: 2px;
}

.jb-profile-mobile {
  font-size: 0.75rem;
  color: #718096;
  font-weight: 600;
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
  padding: 3px 10px;
  background: #f7fafc;
  border-radius: 4px;
  border: 1px solid rgba(14, 90, 58, 0.08);
}

/* خط جداکننده */
.jb-sidebar-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 2px 0;
}

/* =========================================================
   منو
   ========================================================= */
.jb-panel-menu {
  display: flex;
  flex-direction: column;
  gap: 4px;
  list-style: none;
  padding: 0;
  margin: 0;
}

.jb-panel-menu-item {
  position: relative;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 9px 12px;
  border-radius: 6px;
  color: #4a5568;
  font-size: 0.85rem;
  font-weight: 600;
  text-decoration: none;
  transition: all 0.2s ease;
  border: 1px solid transparent;
}

.jb-panel-menu-item:hover {
  background: #f7fafc;
  color: #0e5a3a;
  border-color: rgba(14, 90, 58, 0.08);
}

.jb-panel-menu-item.is-active {
  background: #0e5a3a;
  color: #fff;
  border-color: transparent;
}

.jb-panel-menu-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 6px;
  background: #f7fafc;
  color: #0e5a3a;
  flex-shrink: 0;
  transition: all 0.2s ease;
}

.jb-panel-menu-item.is-active .jb-panel-menu-icon {
  background: rgba(255, 255, 255, 0.15);
  color: #fff;
}

.jb-panel-menu-label {
  flex: 1;
  min-width: 0;
}

.jb-panel-menu-arrow {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

/* =========================================================
   دکمه خروج
   ========================================================= */
.jb-panel-logout-li {
  margin-top: 6px;
  padding-top: 10px;
  border-top: 1px dashed rgba(14, 90, 58, 0.1);
}

.jb-panel-logout-btn {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 9px 12px;
  background: #fff5f5;
  border: 1px solid #fed7d7;
  border-radius: 6px;
  color: #c53030;
  font-size: 0.85rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-panel-logout-btn:hover {
  background: #c53030;
  color: #fff;
  border-color: transparent;
}

.jb-panel-logout-icon {
  background: #ffe8e8;
  color: #c53030;
  transition: all 0.2s ease;
}

.jb-panel-logout-btn:hover .jb-panel-logout-icon {
  background: rgba(255, 255, 255, 0.2);
  color: #fff;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 768px) {
  .jb-panel-sidebar {
    padding: 14px;
    border-radius: 8px;
  }

  .jb-profile-avatar {
    width: 56px;
    height: 56px;
  }

  .jb-profile-avatar svg {
    width: 22px;
    height: 22px;
  }

  .jb-panel-menu-item {
    padding: 8px 10px;
    font-size: 0.8rem;
  }

  .jb-panel-menu-icon {
    width: 26px;
    height: 26px;
  }
}
</style>