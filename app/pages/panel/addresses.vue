<template>
  <div class="jb-addresses-card">
    <!-- هدر -->
    <div class="jb-addresses-header">
      <h3 class="jb-addresses-header-title">
        <span class="jb-addresses-header-icon">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M17.657 16.657L13.414 20.9a2 2 0 01-2.828 0l-4.243-4.243a8 8 0 1111.314 0z" />
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
          </svg>
        </span>
        آدرس‌ها
      </h3>

      <button @click="openNewAddressModal" class="jb-add-address-btn">
        <Plus :size="16" />
        <span>افزودن آدرس جدید</span>
      </button>
    </div>

    <!-- خط جداکننده -->
    <div class="jb-divider"></div>

    <!-- حالت لودینگ -->
    <div v-if="loading" class="jb-state-block">
      <SharedLoadingSpinner size="lg" />
    </div>

    <!-- خطا -->
    <div v-else-if="error" class="jb-state-block jb-state-error">
      <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8"
          d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
      </svg>
      <span>{{ error }}</span>
    </div>

    <!-- خالی -->
    <div v-else-if="addresses.length === 0" class="jb-state-block jb-state-empty">
      <div class="jb-state-icon">
        <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M17.657 16.657L13.414 20.9a2 2 0 01-2.828 0l-4.243-4.243a8 8 0 1111.314 0z" />
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
        </svg>
      </div>
      <span>هنوز آدرسی ثبت نشده است</span>
      <button @click="openNewAddressModal" class="jb-empty-add-btn">
        <Plus :size="14" />
        افزودن اولین آدرس
      </button>
    </div>

    <!-- لیست آدرس‌ها -->
    <div v-else class="jb-addresses-grid">
      <label v-for="address in addresses" :key="address.id" class="jb-address-card"
        :class="{ 'is-selected': selectedAddress === address.id }">
        <input type="radio" name="address" :value="address.id" v-model="selectedAddress"
          class="jb-address-radio-input" />

        <div class="jb-address-card-inner">
          <!-- رادیو -->
          <div class="jb-address-radio-wrap">
            <div class="jb-address-radio" :class="{ 'is-checked': selectedAddress === address.id }"></div>
          </div>

          <!-- اطلاعات -->
          <div class="jb-address-info">
            <p class="jb-address-line">
              <span class="jb-address-icon">
                <MapPin :size="14" />
              </span>
              <span>{{ address.address_line }}</span>
            </p>

            <p class="jb-address-meta">
              <span class="jb-address-icon">
                <Hash :size="14" />
              </span>
              <span>کدپستی: {{ address.postal_code }}</span>
            </p>

            <div class="jb-address-receiver">
              <span class="jb-address-icon">
                <User :size="14" />
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
            <button @click.prevent="handleDeleteAddress(address.id)" type="button"
              class="jb-address-action-btn jb-delete-btn" aria-label="حذف">
              <Trash2 :size="14" />
            </button>

            <button @click.prevent="openEditModal(address)" type="button" class="jb-address-action-btn jb-edit-btn"
              aria-label="ویرایش">
              <MoreVertical :size="14" />
            </button>
          </div>
        </div>
      </label>
    </div>

    <!-- مودال -->
    <CartModalAdderss :show="showAddressModal" :edit-address="editingAddress" @close="showAddressModal = false"
      @save="addNewAddress" />
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
  meta: [
    { name: 'robots', content: 'noindex, nofollow, noarchive, nosnippet' },
  ],
})
</script>

<style scoped>
/* =========================================================
   صفحه لیست آدرس‌ها - مینیمال سبز
   ========================================================= */

.jb-addresses-card {
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

/* ===== هدر ===== */
.jb-addresses-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
}

.jb-addresses-header-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-addresses-header-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 6px;
  background: #0e5a3a;
  color: #fff;
}

/* دکمه افزودن */
.jb-add-address-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 9px 18px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.8rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-add-address-btn:hover {
  background: #0b472e;
}

/* خط جداکننده */
.jb-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 16px 0 20px;
}

/* ===== حالت‌ها ===== */
.jb-state-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 32px 20px;
  text-align: center;
  gap: 10px;
  color: #718096;
  font-size: 0.85rem;
  font-weight: 500;
}

.jb-state-error {
  color: #c53030;
  font-weight: 600;
}

.jb-state-icon {
  width: 64px;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.08);
  color: #a0aec0;
  margin-bottom: 4px;
}

.jb-empty-add-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  margin-top: 6px;
  padding: 9px 18px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.8rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-empty-add-btn:hover {
  background: #0b472e;
}

/* =========================================================
   گرید آدرس‌ها
   ========================================================= */
.jb-addresses-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 10px;
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
  padding: 14px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-address-card:hover {
  border-color: rgba(14, 90, 58, 0.25);
}

.jb-address-card.is-selected {
  border-color: #0e5a3a;
  background: #f7fafc;
}

.jb-address-card-inner {
  display: flex;
  align-items: flex-start;
  gap: 10px;
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
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 2px solid rgba(14, 90, 58, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  background: #fff;
}

.jb-address-radio.is-checked {
  border-color: #0e5a3a;
  background: #0e5a3a;
}

/* ===== اطلاعات ===== */
.jb-address-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 5px;
  min-width: 0;
}

.jb-address-line {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 0.82rem;
  font-weight: 700;
  color: #1a202c;
  line-height: 1.6;
}

.jb-address-meta {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.75rem;
  color: #718096;
  font-weight: 500;
}

.jb-address-receiver {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
  font-size: 0.75rem;
  color: #4a5568;
  font-weight: 600;
}

.jb-address-receiver-name {
  color: #1a202c;
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
  color: #0e5a3a;
  margin-top: 1px;
}

.jb-sep {
  color: #cbd5e1;
  font-weight: 400;
}

/* ===== دکمه‌های عملیات ===== */
.jb-address-actions {
  display: flex;
  flex-direction: column;
  gap: 5px;
  flex-shrink: 0;
}

.jb-address-action-btn {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  border: 1px solid transparent;
}

.jb-delete-btn {
  background: #fff5f5;
  border-color: #fed7d7;
  color: #e53e3e;
}

.jb-delete-btn:hover {
  background: #e53e3e;
  color: #fff;
  border-color: transparent;
}

.jb-edit-btn {
  background: #f7fafc;
  border-color: rgba(14, 90, 58, 0.08);
  color: #0e5a3a;
}

.jb-edit-btn:hover {
  background: #0e5a3a;
  color: #fff;
  border-color: transparent;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-addresses-card {
    padding: 14px;
    border-radius: 8px;
  }

  .jb-addresses-header {
    flex-direction: column;
    align-items: stretch;
  }

  .jb-addresses-header-title {
    font-size: 0.9rem;
  }

  .jb-add-address-btn {
    width: 100%;
    justify-content: center;
  }

  .jb-address-card {
    padding: 12px;
  }

  .jb-address-actions {
    flex-direction: row;
  }
}
</style>