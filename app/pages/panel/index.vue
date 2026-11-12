<template>
  <div class="jb-profile-card">
    <!-- هدر -->
    <div class="jb-profile-header">
      <h2 class="jb-profile-header-title">
        <span class="jb-profile-header-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
          </svg>
        </span>
        اطلاعات کاربری
      </h2>
    </div>

    <!-- خط جداکننده -->
    <div class="jb-divider"></div>

    <!-- لودینگ -->
    <div v-if="profilePending" class="jb-state-block">
      <SharedLoadingSpinner size="lg" />
    </div>

    <!-- اطلاعات -->
    <div v-else-if="profileData" class="jb-profile-grid">
      <!-- نام و نام خانوادگی -->
      <div class="jb-profile-field">
        <div class="jb-field-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
          </svg>
        </div>
        <div class="jb-field-content">
          <span class="jb-field-label">نام و نام خانوادگی</span>
          <span class="jb-field-value">
            {{ profileData?.user?.full_name || "-" }}
          </span>
        </div>
      </div>

      <!-- کد ملی -->
      <div class="jb-profile-field">
        <div class="jb-field-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M10 6H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V8a2 2 0 00-2-2h-5m-4 0V5a2 2 0 114 0v1m-4 0a2 2 0 104 0m-5 8a2 2 0 100-4 2 2 0 000 4zm0 0c1.306 0 2.418.835 2.83 2M9 14a3.001 3.001 0 00-2 2M15 11h.01V14h3v-3H15zm0 0h3m-3 4h.01v.01H15V15zm0 0h3v.01H15V15z" />
          </svg>
        </div>
        <div class="jb-field-content">
          <span class="jb-field-label">کد ملی</span>
          <span class="jb-field-value jb-field-value-mono" dir="ltr">
            {{ profileData?.user?.national_code || "-" }}
          </span>
        </div>
      </div>

      <!-- ایمیل -->
      <div class="jb-profile-field">
        <div class="jb-field-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
          </svg>
        </div>
        <div class="jb-field-content">
          <span class="jb-field-label">ایمیل</span>
          <span class="jb-field-value jb-field-value-mono" dir="ltr">
            {{ profileData?.user?.email || "-" }}
          </span>
        </div>
      </div>

      <!-- شماره تماس -->
      <div class="jb-profile-field">
        <div class="jb-field-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z" />
          </svg>
        </div>
        <div class="jb-field-content">
          <span class="jb-field-label">شماره تماس</span>
          <span class="jb-field-value jb-field-value-mono" dir="ltr">
            {{ profileData?.user?.mobile || "-" }}
          </span>
        </div>
      </div>
    </div>

    <!-- دکمه ویرایش -->
    <div v-if="profileData" class="jb-profile-actions">
      <button class="jb-edit-btn" @click="showModal = true">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
        </svg>
        <span>ویرایش اطلاعات</span>
      </button>
    </div>

    <!-- مودال -->
    <panelEditUserModal :show="showModal" :userData="profileData?.user" @close="showModal = false" @save="handleSave" />
  </div>
</template>

<script setup>
import { useApi } from "@/composables/api/useApi";
import { ref } from "vue";

definePageMeta({
  layout: "panel",
  middleware: "auth",
});

const { getUserProfile } = useApi();

const {
  data: profileData,
  pending: profilePending,
  refresh: refreshProfile,
} = await useAsyncData(
  "userProfile",
  async () => {
    try {
      return await getUserProfile();
    } catch (error) {
      console.error("❌ Failed to fetch user profile:", error);
      return { data: null };
    }
  },
  {
    lazy: true,
    server: false,
  }
);

const showModal = ref(false);

async function handleSave() {
  await refreshProfile();
}
useSeoMeta({
  title: () => ' پنل کاربری | فروشگاه دکتر اسماعیلی',
  description: '  کاربر در پنل اختصاصی',
  robots: 'noindex, nofollow',
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/panel' }],
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه پروفایل - مینیمال سبز
   ========================================================= */

.jb-profile-card {
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

/* ===== هدر ===== */
.jb-profile-header {
  margin-bottom: 4px;
}

.jb-profile-header-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-profile-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 6px;
  background: #0e5a3a;
  color: #fff;
}

/* خط جداکننده */
.jb-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 14px 0 20px;
}

/* ===== حالت لودینگ ===== */
.jb-state-block {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px 20px;
}

/* ===== گرید اطلاعات ===== */
.jb-profile-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
}

@media (min-width: 768px) {
  .jb-profile-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* ===== فیلد ===== */
.jb-profile-field {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  border-radius: 8px;
  transition: all 0.2s ease;
}

.jb-profile-field:hover {
  border-color: rgba(14, 90, 58, 0.2);
}

.jb-field-icon {
  flex-shrink: 0;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  background: #0e5a3a;
  color: #fff;
}

.jb-field-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 3px;
  min-width: 0;
}

.jb-field-label {
  font-size: 0.72rem;
  color: #718096;
  font-weight: 500;
}

.jb-field-value {
  font-size: 0.85rem;
  color: #1a202c;
  font-weight: 700;
  word-break: break-word;
}

.jb-field-value-mono {
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
}

/* ===== دکمه ویرایش ===== */
.jb-profile-actions {
  display: flex;
  justify-content: flex-start;
  margin-top: 20px;
  padding-top: 16px;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
}

.jb-edit-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 10px 20px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-edit-btn:hover {
  background: #0b472e;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-profile-card {
    padding: 14px;
    border-radius: 8px;
  }

  .jb-profile-header-title {
    font-size: 0.9rem;
  }

  .jb-profile-header-icon {
    width: 28px;
    height: 28px;
  }

  .jb-profile-field {
    padding: 12px;
  }

  .jb-field-icon {
    width: 32px;
    height: 32px;
  }

  .jb-field-value {
    font-size: 0.8rem;
  }

  .jb-profile-actions {
    justify-content: stretch;
  }

  .jb-edit-btn {
    width: 100%;
  }
}
</style>