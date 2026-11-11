<template>
  <div class="jb-addresses-card">
    <!-- ===== هدر ===== -->
    <div class="jb-addresses-header">
      <h3 class="jb-addresses-header-title">
        <span class="jb-addresses-header-icon">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M17.657 16.657L13.414 20.9a2 2 0 01-2.828 0l-4.243-4.243a8 8 0 1111.314 0z" />
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
          </svg>
        </span>
        آدرس‌ها
      </h3>

      <button @click="openNewAddressModal" class="jb-btn-linear jb-add-address-btn">
        <Plus :size="18" />
        <span>افزودن آدرس جدید</span>
      </button>
    </div>

    <!-- خط بریده‌بریده -->
    <div class="jb-divider"></div>

    <!-- ===== حالت لودینگ ===== -->
    <div v-if="loading" class="jb-state-block">
      <SharedLoadingSpinner size="lg" />
    </div>

    <!-- ===== خطا ===== -->
    <div v-else-if="error" class="jb-state-block jb-state-error">
      <svg class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8"
          d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
      </svg>
      <span>{{ error }}</span>
    </div>

    <!-- ===== خالی ===== -->
    <div v-else-if="addresses.length === 0" class="jb-state-block jb-state-empty">
      <div class="jb-state-icon">
        <svg class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M17.657 16.657L13.414 20.9a2 2 0 01-2.828 0l-4.243-4.243a8 8 0 1111.314 0z" />
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
        </svg>
      </div>
      <span>هنوز آدرسی ثبت نشده است</span>
      <button @click="openNewAddressModal" class="jb-empty-add-btn">
        <Plus :size="16" />
        افزودن اولین آدرس
      </button>
    </div>

    <!-- ===== لیست آدرس‌ها ===== -->
    <div v-else class="jb-addresses-grid">
      <label
        v-for="address in addresses"
        :key="address.id"
        class="jb-address-card"
        :class="{ 'is-selected': selectedAddress === address.id }"
      >
        <input
          type="radio"
          name="address"
          :value="address.id"
          v-model="selectedAddress"
          class="jb-address-radio-input"
        />

        <div class="jb-address-card-inner">
          <!-- رادیو -->
          <div class="jb-address-radio-wrap">
            <div
              class="jb-address-radio"
              :class="{ 'is-checked': selectedAddress === address.id }"
            >
              <div
                v-if="selectedAddress === address.id"
                class="jb-address-radio-dot"
              ></div>
            </div>
          </div>

          <!-- اطلاعات -->
          <div class="jb-address-info">
            <p class="jb-address-line">
              <span class="jb-address-icon">
                <MapPin :size="16" />
              </span>
              <span>{{ address.address_line }}</span>
            </p>

            <p class="jb-address-meta">
              <span class="jb-address-icon">
                <Hash :size="16" />
              </span>
              <span>کدپستی: {{ address.postal_code }}</span>
            </p>

            <div class="jb-address-receiver">
              <span class="jb-address-icon">
                <User :size="16" />
              </span>
              <span class="jb-address-receiver-name">
                {{ address.receiver_name }}
              </span>
              <span class="jb-sep">|</span>
              <span class="jb-address-receiver-phone" dir="ltr">
                {{ address.phone }}
              </span>
            </div>
          </div>

          <!-- دکمه‌های عملیات -->
          <div class="jb-address-actions">
            <button
              @click.prevent="handleDeleteAddress(address.id)"
              type="button"
              class="jb-address-action-btn jb-delete-btn"
              aria-label="حذف"
            >
              <Trash2 :size="16" />
            </button>

            <button
              @click.prevent="openEditModal(address)"
              type="button"
              class="jb-address-action-btn jb-edit-btn"
              aria-label="ویرایش"
            >
              <MoreVertical :size="16" />
            </button>
          </div>
        </div>
      </label>
    </div>

    <!-- مودال -->
    <CartModalAdderss
      :show="showAddressModal"
      :edit-address="editingAddress"
      @close="showAddressModal = false"
      @save="addNewAddress"
    />
  </div>
</template>

<script setup>
import { Plus, MapPin, Hash, User, Trash2, MoreVertical } from "lucide-vue-next";

definePageMeta({
  layout: "panel",
  middleware: "auth",
});

import { ref, onMounted } from "vue";
import { toast } from "vue3-toastify";

const { addresses, loading, error, fetchAddresses, deleteAddress } =
  useAddresses();

const showAddressModal = ref(false);
const editingAddress = ref(null);
const selectedAddress = ref(null);
const selectedShipping = ref(1);

onMounted(async () => {
  await fetchAddresses();
  if (addresses.value.length > 0) {
    selectedAddress.value = addresses.value[0].id;
  }
});

async function handleDeleteAddress(id) {
  const shouldDelete = confirm(
    "آیا مطمئن هستید که می‌خواهید این آدرس را حذف کنید؟"
  );

  if (shouldDelete) {
    try {
      await deleteAddress(id);
      if (selectedAddress.value === id && addresses.value.length > 0) {
        selectedAddress.value = addresses.value[0].id;
      }

      toast.success("آدرس با موفقیت حذف شد");
    } catch (err) {
      toast.error("خطا در حذف آدرس");
    }
  }
}

function addNewAddress() {
  fetchAddresses();
  showAddressModal.value = false;
  editingAddress.value = null;
}

function openEditModal(address) {
  editingAddress.value = address;
  showAddressModal.value = true;
}

function openNewAddressModal() {
  editingAddress.value = null;
  showAddressModal.value = true;
}

useSeoMeta({
  title: () => 'آدرس ها پنل کاربری | فروشگاه دکتر اسماعیلی',
  description: 'لیست آدرس ها کاربر در پنل اختصاصی',
  robots: 'noindex, nofollow',
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/panel/addresses' }],
  // 🔹 جلوگیری از ایندکس شدن با متا تگ اضافی برای اطمینان بیشتر
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه لیست آدرس‌ها - تم پودری
   ========================================================= */

.jb-addresses-card {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 24px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(214, 51, 132, 0.1);
}

/* ===== هدر ===== */
.jb-addresses-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
}

.jb-addresses-header-title {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  font-size: 1.15rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-addresses-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  border-radius: 12px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
}

/* دکمه افزودن */
.jb-add-address-btn {
  padding: 11px 22px !important;
  font-size: 0.85rem !important;
}

/* خط بریده‌بریده */
.jb-divider {
  height: 0;
  border-top: 2px dashed var(--jb-pink-200);
  margin: 18px 0 24px;
  position: relative;
}

.jb-divider::before,
.jb-divider::after {
  content: "";
  position: absolute;
  top: -5px;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: var(--jb-pink-400);
  box-shadow: 0 0 0 3px #fff;
}

.jb-divider::before { right: -4px; }
.jb-divider::after  { left: -4px; }

/* ===== حالت‌ها ===== */
.jb-state-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px 20px;
  text-align: center;
  gap: 12px;
  color: var(--jb-text-muted);
  font-size: 0.88rem;
  font-weight: 600;
}

.jb-state-error {
  color: #c9184a;
  font-weight: 800;
}

.jb-state-icon {
  width: 84px;
  height: 84px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border: 3px dashed var(--jb-pink-300);
  color: var(--jb-pink-400);
  margin-bottom: 4px;
  animation: jbEmptyFloat 3s ease-in-out infinite;
}

@keyframes jbEmptyFloat {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-8px); }
}

.jb-empty-add-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  margin-top: 8px;
  padding: 10px 22px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border: none;
  border-radius: 12px;
  font-size: 0.82rem;
  font-weight: 800;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 6px 18px rgba(214, 51, 132, 0.35);
}

.jb-empty-add-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-600), var(--jb-pink-800));
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.5);
}

/* =========================================================
   گرید آدرس‌ها
   ========================================================= */
.jb-addresses-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
}

@media (min-width: 768px) {
  .jb-addresses-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* =========================================================
   کارت آدرس
   ========================================================= */
.jb-address-card {
  position: relative;
  display: block;
  padding: 16px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 18px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.jb-address-card:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.14);
  transform: translateY(-2px);
}

.jb-address-card.is-selected {
  border-color: var(--jb-fuchsia);
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.22);
}

.jb-address-card-inner {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

/* رادیو مخفی */
.jb-address-radio-input {
  position: absolute;
  opacity: 0;
  pointer-events: none;
}

/* رادیو سفارشی */
.jb-address-radio-wrap {
  flex-shrink: 0;
  margin-top: 2px;
}

.jb-address-radio {
  width: 22px;
  height: 22px;
  border-radius: 50%;
  border: 2.5px solid var(--jb-pink-300);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.25s ease;
  background: #fff;
}

.jb-address-radio.is-checked {
  border-color: var(--jb-fuchsia);
  background: rgba(214, 51, 132, 0.08);
}

.jb-address-radio-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 2px 6px rgba(214, 51, 132, 0.4);
}

/* ===== اطلاعات ===== */
.jb-address-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 6px;
  min-width: 0;
}

.jb-address-line {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 0.85rem;
  font-weight: 700;
  color: var(--jb-text);
  line-height: 1.6;
}

.jb-address-meta {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.78rem;
  color: var(--jb-text-muted);
  font-weight: 600;
}

.jb-address-receiver {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
  font-size: 0.78rem;
  color: var(--jb-text-soft);
  font-weight: 700;
}

.jb-address-receiver-name {
  color: var(--jb-text);
}

.jb-address-receiver-phone {
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
}

.jb-address-icon {
  flex-shrink: 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  color: var(--jb-fuchsia);
  margin-top: 1px;
}

.jb-sep {
  color: var(--jb-pink-300);
  font-weight: 400;
}

/* ===== دکمه‌های عملیات ===== */
.jb-address-actions {
  display: flex;
  flex-direction: column;
  gap: 6px;
  flex-shrink: 0;
}

.jb-address-action-btn {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.25s ease;
  border: 1.5px solid transparent;
}

.jb-delete-btn {
  background: #fff5f8;
  border-color: #ffb8c8;
  color: #c9184a;
}

.jb-delete-btn:hover {
  background: linear-gradient(135deg, #ff4d6d, #c9184a);
  color: #fff;
  border-color: transparent;
  transform: scale(1.08);
  box-shadow: 0 6px 16px rgba(201, 24, 74, 0.35);
}

.jb-edit-btn {
  background: var(--jb-pink-50);
  border-color: var(--jb-pink-200);
  color: var(--jb-fuchsia);
}

.jb-edit-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  transform: scale(1.08);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-addresses-card {
    padding: 16px;
    border-radius: 20px;
  }

  .jb-addresses-header {
    flex-direction: column;
    align-items: stretch;
  }

  .jb-addresses-header-title {
    font-size: 1rem;
  }

  .jb-add-address-btn {
    width: 100%;
    justify-content: center;
  }

  .jb-address-card {
    padding: 14px;
  }

  .jb-address-actions {
    flex-direction: row;
  }
}
</style>