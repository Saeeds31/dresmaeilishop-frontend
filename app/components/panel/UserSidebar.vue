<template>
  <aside class="jb-panel-sidebar">
    <!-- ===== بخش پروفایل ===== -->
    <div class="jb-profile-card">
      <div class="jb-profile-avatar">
        <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
        </svg>
        <span class="jb-profile-avatar-ring"></span>
      </div>

      <p class="jb-profile-name">
        {{ userProfile?.full_name || "کاربر" }}
      </p>
      <p class="jb-profile-mobile" dir="ltr">
        {{ formatMobile(userProfile?.mobile) }}
      </p>
    </div>

    <!-- خط بریده‌بریده -->
    <div class="jb-sidebar-divider"></div>

    <!-- ===== منوها ===== -->
    <ul class="jb-panel-menu">
      <li
        v-for="(item, index) in menu"
        :key="index"
      >
        <NuxtLink
          :to="item.to"
          class="jb-panel-menu-item"
          :class="{ 'is-active': route.path === item.to }"
          @click="handleMenuClick(item.to)"
        >
          <span class="jb-panel-menu-icon">
            <component :is="item.icon" class="w-4 h-4" />
          </span>
          <span class="jb-panel-menu-label">{{ item.label }}</span>

          <svg
            v-if="route.path === item.to"
            class="jb-panel-menu-arrow"
            fill="none" stroke="currentColor" viewBox="0 0 24 24"
          >
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
   سایدبار پنل کاربری - تم پودری
   ========================================================= */

.jb-panel-sidebar {
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 24px;
  box-shadow: 0 10px 30px rgba(214, 51, 132, 0.1);
}

/* =========================================================
   کارت پروفایل
   ========================================================= */
.jb-profile-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 12px 0;
}

.jb-profile-avatar {
  position: relative;
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow:
    0 12px 30px rgba(214, 51, 132, 0.35),
    inset 0 -3px 6px rgba(110, 15, 46, 0.2),
    inset 0 3px 6px rgba(255, 255, 255, 0.3);
  border: 4px solid #fff;
  margin-bottom: 6px;
  animation: jbAvatarFloat 3.5s ease-in-out infinite;
}

@keyframes jbAvatarFloat {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-4px); }
}

.jb-profile-avatar-ring {
  position: absolute;
  inset: -10px;
  border-radius: 50%;
  border: 2px dashed var(--jb-pink-400);
  animation: jbRingRotate 12s linear infinite;
  pointer-events: none;
}

@keyframes jbRingRotate {
  to { transform: rotate(360deg); }
}

.jb-profile-name {
  font-size: 1rem;
  font-weight: 900;
  color: var(--jb-text);
  margin-top: 4px;
}

.jb-profile-mobile {
  font-size: 0.78rem;
  color: var(--jb-text-muted);
  font-weight: 700;
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
  padding: 4px 12px;
  background: var(--jb-pink-50);
  border-radius: 999px;
  border: 1.5px solid var(--jb-pink-200);
}

/* خط بریده‌بریده */
.jb-sidebar-divider {
  height: 0;
  border-top: 1.5px dashed var(--jb-pink-300);
  position: relative;
  margin: 4px 0;
}

.jb-sidebar-divider::before,
.jb-sidebar-divider::after {
  content: "";
  position: absolute;
  top: -4px;
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--jb-pink-400);
  box-shadow: 0 0 0 2px #fff;
}

.jb-sidebar-divider::before { right: -3px; }
.jb-sidebar-divider::after  { left: -3px; }

/* =========================================================
   منو
   ========================================================= */
.jb-panel-menu {
  display: flex;
  flex-direction: column;
  gap: 6px;
  list-style: none;
  padding: 0;
  margin: 0;
}

.jb-panel-menu-item {
  position: relative;
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 11px 14px;
  border-radius: 14px;
  color: var(--jb-text-soft);
  font-size: 0.88rem;
  font-weight: 700;
  text-decoration: none;
  transition: all 0.25s ease;
  border: 1.5px solid transparent;
}

.jb-panel-menu-item:hover {
  background: var(--jb-pink-50);
  color: var(--jb-fuchsia);
  border-color: var(--jb-pink-200);
  transform: translateX(-3px);
}

.jb-panel-menu-item.is-active {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.4);
  transform: translateX(0);
}

.jb-panel-menu-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 10px;
  background: var(--jb-pink-100);
  color: var(--jb-fuchsia);
  flex-shrink: 0;
  transition: all 0.25s ease;
}

.jb-panel-menu-item.is-active .jb-panel-menu-icon {
  background: rgba(255, 255, 255, 0.2);
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
  animation: jbArrowSlide 1.5s ease-in-out infinite;
}

@keyframes jbArrowSlide {
  0%, 100% { transform: translateX(0); }
  50%      { transform: translateX(-4px); }
}

/* =========================================================
   دکمه خروج
   ========================================================= */
.jb-panel-logout-li {
  margin-top: 8px;
  padding-top: 12px;
  border-top: 1.5px dashed var(--jb-pink-200);
}

.jb-panel-logout-btn {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 11px 14px;
  background: #fff5f8;
  border: 1.5px solid #ffb8c8;
  border-radius: 14px;
  color: #c9184a;
  font-size: 0.88rem;
  font-weight: 800;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-panel-logout-btn:hover {
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  color: #fff;
  border-color: transparent;
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(201, 24, 74, 0.35);
}

.jb-panel-logout-icon {
  background: #ffe0e8;
  color: #c9184a;
  transition: all 0.25s ease;
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
    padding: 16px;
    border-radius: 20px;
  }

  .jb-profile-avatar {
    width: 68px;
    height: 68px;
  }

  .jb-profile-avatar svg {
    width: 26px;
    height: 26px;
  }

  .jb-panel-menu-item {
    padding: 10px 12px;
    font-size: 0.84rem;
  }

  .jb-panel-menu-icon {
    width: 28px;
    height: 28px;
  }
}
</style>