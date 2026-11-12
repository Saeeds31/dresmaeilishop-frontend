<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { toast } from "vue3-toastify";
import { useApi } from "@/composables/api/useApi";

definePageMeta({
  middleware: 'auth'
})

const { addresses, loading, error, fetchAddresses, deleteAddress } = useAddresses()
const { shippingMethods, fetchShippingMethods, loading: shippingLoading } = useShipping()

const {
  getOrder
} = useApi();
const showAddressModal = ref(false)
const editingAddress = ref(null)
const selectedAddress = ref(null)
const selectedShipping = ref(1)

const cartData = ref(null)
const subtotal = ref(900000)
const reservationStorageKey = "selected-reservation";

const selectedReservation = ref(null);
const reservationMode = ref(false);
const loadSelectedReservation = () => {
  if (!process.client) {
    return null;
  }

  const saved = localStorage.getItem(reservationStorageKey);

  if (!saved) {
    return null;
  }

  try {
    const reservation = JSON.parse(saved);

    if (!reservation?.order_id) {
      localStorage.removeItem(reservationStorageKey);
      return null;
    }

    selectedReservation.value = reservation;
    reservationMode.value = true;

    return reservation;
  } catch (error) {
    console.error("Invalid reservation data:", error);

    localStorage.removeItem(reservationStorageKey);

    selectedReservation.value = null;
    reservationMode.value = false;

    return null;
  }
};
const loadReservationOrder = async () => {
  const reservation = loadSelectedReservation();

  if (!reservation) {
    return;
  }

  try {
    const response = await getOrder(reservation.order_id);

    const order = response?.data;

    if (!order) {
      return;
    }

    selectedAddress.value = order.address_id;

    // اگر آدرس در لیست آدرس‌های کاربر هست
    const reservationAddress = addresses.value.find(
      address => address.id === order.address_id
    );

    if (reservationAddress) {
      selectedAddress.value = reservationAddress.id;
    }
  } catch (error) {
    console.error("Failed to load reservation order:", error);
  }
};
// 🔥 Watch برای تغییر آدرس
watch(selectedAddress, async (newAddressId, oldAddressId) => {
  if (newAddressId && newAddressId !== oldAddressId) {

    // ✅ فراخوانی API با address_id جدید
    await fetchShippingMethods(newAddressId, selectedReservation.value?.order_id)

    // انتخاب خودکار اولین روش ارسال
    if (shippingMethods.value.length > 0) {
      selectedShipping.value = shippingMethods.value[0].id
    }
  }
})

onMounted(async () => {
  loadReservationOrder()
  const savedCart = localStorage.getItem('cart-summary')
  if (savedCart) {
    try {
      cartData.value = JSON.parse(savedCart)
      subtotal.value = cartData.value.totalPrice || 900000
    } catch (err) {
      console.error('❌ Failed to parse cart data:', err)
    }
  }

  await fetchAddresses()

  if (addresses.value.length > 0) {
    selectedAddress.value = addresses.value[0].id
  }

  // ✅ بارگذاری اولیه روش‌های ارسال با address_id
  if (selectedAddress.value) {
    await fetchShippingMethods(selectedAddress.value, selectedReservation.value?.order_id)

    if (shippingMethods.value.length > 0) {
      selectedShipping.value = shippingMethods.value[0].id
    }
  }
})

const shippingCost = computed(() => {
  const method = shippingMethods.value.find(m => m.id === selectedShipping.value)
  return method ? method.cost : 0
})

const total = computed(() => subtotal.value + shippingCost.value)

async function handleDeleteAddress(id) {

  const shouldDelete = confirm('آیا مطمئن هستید که می‌خواهید این آدرس را حذف کنید؟')

  if (shouldDelete) {
    try {
      await deleteAddress(id)
      if (selectedAddress.value === id && addresses.value.length > 0) {
        selectedAddress.value = addresses.value[0].id
      }

      toast.success('آدرس با موفقیت حذف شد')
    } catch (err) {
      toast.error('خطا در حذف آدرس')
    }
  }
}

async function addNewAddress() {
  await fetchAddresses()
  showAddressModal.value = false
  editingAddress.value = null
  if (addresses.value.length > 0) {
    selectedAddress.value = addresses.value[0].id
  }

  // ✅ بارگذاری اولیه روش‌های ارسال با address_id
  if (selectedAddress.value) {
    await fetchShippingMethods(selectedAddress.value, selectedReservation.value?.order_id)
    if (shippingMethods.value.length > 0) {
      selectedShipping.value = shippingMethods.value[0].id
    }
  }
}

function openEditModal(address) {
  editingAddress.value = address
  showAddressModal.value = true
}

function openNewAddressModal() {
  editingAddress.value = null
  showAddressModal.value = true
}

async function handleNextStep() {
  const { $api } = useNuxtApp()

  if (!selectedAddress.value) {
    toast.error('لطفاً یک آدرس انتخاب کنید')
    return
  }

  if (!selectedShipping.value) {
    toast.error('لطفاً یک روش ارسال انتخاب کنید')
    return
  }

  try {

    const response = await $api.post('/front/shippings', {
      address_id: selectedAddress.value,
      shipping_method: selectedShipping.value
    })

    localStorage.setItem('shipping-info', JSON.stringify({
      address_id: selectedAddress.value,
      shipping_method_id: selectedShipping.value
    }))


    navigateTo('/payment')
  } catch (err) {
    console.error('❌ Shipping error:', err)
    toast.error(err.response?.data?.message || 'خطا در ثبت اطلاعات ارسال')
  }
}
useSeoMeta({
  title: 'روش ارسال | فروشگاه دکتر اسماعیلی',
  description: 'مشاهده و مدیریت آدرس و روش ارسال در فروشگاه فروشگاه دکتر اسماعیلی.',
  robots: 'noindex, nofollow', // 🔴 مهم‌ترین خط
})

useHead({
  link: [{ rel: 'canonical', href: 'https://dresmaeilishop.ir/shipping' }],
})
</script>

<template>
  <div class="container jb-shipping-page">
    <SharedBreadCrumb :items="[
      { label: 'سبد خرید', href: '/cart' },
      { label: 'انتخاب آدرس و روش ارسال', href: '/shipping' },
    ]" />
    <CartProgressSteps :current-step="2" />

    <div class="jb-shipping-card">
      <!-- هدر -->
      <div class="jb-shipping-header">
        <div>
          <h3 class="jb-shipping-title">مکان تحویل سفارش</h3>
          <p v-if="reservationMode" class="jb-shipping-subtitle">
            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
            آدرس مربوط به سفارش رزرو شده
          </p>
        </div>

        <button v-if="!reservationMode" @click="openNewAddressModal" class="jb-btn-add-address">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 4v16m8-8H4" />
          </svg>
          <span>افزودن آدرس جدید</span>
        </button>
      </div>

      <!-- خط جداکننده -->
      <div class="jb-divider"></div>

      <!-- حالت‌ها -->
      <div v-if="loading" class="jb-state-block">
        <SharedLoadingSpinner size="lg" />
      </div>

      <div v-else-if="error" class="jb-state-block jb-state-error">
        <svg class="w-8 h-8" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8"
            d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
        </svg>
        <span>{{ error }}</span>
      </div>

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
      </div>

      <!-- رزرو مود -->
      <div v-else-if="reservationMode" class="jb-reservation-address">
        <div v-for="address in addresses.filter((a) => a.id === selectedAddress)" :key="address.id"
          class="jb-address-card jb-address-reserved">
          <div class="jb-address-card-inner">
            <div class="jb-address-radio-wrap">
              <div class="jb-address-radio is-checked"></div>
            </div>

            <div class="jb-address-info">
              <p class="jb-address-line">
                <span class="jb-address-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M17.657 16.657L13.414 20.9a2 2 0 01-2.828 0l-4.243-4.243a8 8 0 1111.314 0z" />
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
                  </svg>
                </span>
                <span>{{ address.address_line }}</span>
              </p>

              <p class="jb-address-meta">
                <span class="jb-address-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a2 2 0 01-2.828 0l-7-7A1.994 1.994 0 013 12V7a4 4 0 014-4z" />
                  </svg>
                </span>
                <span>کدپستی: {{ address.postal_code }}</span>
              </p>

              <div class="jb-address-receiver">
                <span class="jb-address-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
                  </svg>
                </span>
                <span>گیرنده: {{ address.receiver_name }}</span>
                <span class="jb-sep">|</span>
                <span>تلفن: {{ address.phone }}</span>
              </div>
            </div>

            <div class="jb-badge-reserved">آدرس رزرو</div>
          </div>
        </div>
      </div>

      <!-- حالت عادی -->
      <div v-else class="jb-address-grid">
        <label v-for="address in addresses" :key="address.id" class="jb-address-card"
          :class="{ 'is-selected': selectedAddress === address.id }">
          <input type="radio" name="address" :value="address.id" v-model="selectedAddress"
            class="jb-address-radio-input" />

          <div class="jb-address-card-inner">
            <div class="jb-address-radio-wrap">
              <div class="jb-address-radio" :class="{ 'is-checked': selectedAddress === address.id }"></div>
            </div>

            <div class="jb-address-info">
              <p class="jb-address-line">
                <span class="jb-address-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M17.657 16.657L13.414 20.9a2 2 0 01-2.828 0l-4.243-4.243a8 8 0 1111.314 0z" />
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
                  </svg>
                </span>
                <span>{{ address.address_line }}</span>
              </p>

              <p class="jb-address-meta">
                <span class="jb-address-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a2 2 0 01-2.828 0l-7-7A1.994 1.994 0 013 12V7a4 4 0 014-4z" />
                  </svg>
                </span>
                <span>کدپستی: {{ address.postal_code }}</span>
              </p>

              <div class="jb-address-receiver">
                <span class="jb-address-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
                  </svg>
                </span>
                <span>گیرنده: {{ address.receiver_name }}</span>
                <span class="jb-sep">|</span>
                <span>تلفن: {{ address.phone }}</span>
              </div>
            </div>

            <div class="jb-address-actions">
              <button @click.prevent="handleDeleteAddress(address.id)" type="button"
                class="jb-address-action-btn jb-delete-btn" aria-label="حذف">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6M1 7h22M9 7V4a1 1 0 011-1h4a1 1 0 011 1v3" />
                </svg>
              </button>

              <button @click.prevent="openEditModal(address)" type="button" class="jb-address-action-btn jb-edit-btn"
                aria-label="ویرایش">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
                </svg>
              </button>
            </div>
          </div>
        </label>
      </div>

      <!-- خط جداکننده بزرگ -->
      <div class="jb-divider jb-divider-lg"></div>

      <!-- شیوه ارسال -->
      <div class="jb-shipping-methods">
        <h3 class="jb-shipping-methods-title">شیوه ارسال</h3>

        <div v-if="shippingLoading" class="jb-state-block">
          <SharedLoadingSpinner size="sm" />
        </div>

        <div v-else-if="shippingMethods.length === 0" class="jb-state-block jb-state-empty">
          <span>روش ارسالی موجود نیست</span>
        </div>

        <div v-else class="jb-shipping-grid">
          <label v-for="method in shippingMethods" :key="method.id" class="jb-shipping-card"
            :class="{ 'is-selected': selectedShipping === method.id }">
            <input type="radio" name="shipping" :value="method.id" v-model="selectedShipping"
              class="jb-shipping-radio-input" />

            <div class="jb-shipping-card-inner">
              <div class="jb-shipping-card-header">
                <div class="jb-address-radio" :class="{ 'is-checked': selectedShipping === method.id }"></div>
                <span class="jb-shipping-card-name">{{ method.name }}</span>
              </div>

              <p class="jb-shipping-card-desc">
                {{ method.message ?? method.description }}
              </p>

              <div class="jb-shipping-card-footer">
                <span class="jb-shipping-cost-label">هزینه ارسال</span>
                <span class="jb-shipping-cost-value">
                  {{ method.cost.toLocaleString("fa-IR") }}
                  <span class="jb-shipping-cost-unit">تومان</span>
                </span>
              </div>
            </div>
          </label>
        </div>
      </div>

      <!-- دکمه‌های ناوبری -->
      <div class="jb-shipping-actions">
        <NuxtLink to="/cart" class="jb-shipping-btn-secondary">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M9 5l7 7-7 7" />
          </svg>
          مرحله قبل
        </NuxtLink>

        <button @click="handleNextStep" class="jb-shipping-btn-primary">
          <span>مرحله بعد</span>
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" />
          </svg>
        </button>
      </div>
    </div>

    <CartModalAdderss :show="showAddressModal" :edit-address="editingAddress" @close="showAddressModal = false"
      @save="addNewAddress" />
  </div>
</template>



<style scoped>
/* =========================================================
   صفحه ارسال - مینیمال سبز
   ========================================================= */

.jb-shipping-page {
  padding-bottom: 40px;
}

.jb-shipping-card {
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

/* ===== هدر ===== */
.jb-shipping-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
}

.jb-shipping-title {
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
}

.jb-shipping-subtitle {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 0.75rem;
  color: #718096;
  font-weight: 500;
  margin-top: 6px;
}

.jb-shipping-subtitle svg {
  color: #0e5a3a;
}

/* دکمه افزودن آدرس */
.jb-btn-add-address {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 9px 16px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.8rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.jb-btn-add-address:hover {
  background: #0b472e;
}

/* خط جداکننده */
.jb-divider {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 16px 0;
}

.jb-divider-lg {
  margin: 24px 0;
}

/* ===== حالت لودینگ / خطا / خالی ===== */
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

/* =========================================================
   کارت آدرس
   ========================================================= */
.jb-address-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 10px;
}

@media (min-width: 768px) {
  .jb-address-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

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
.jb-address-radio-input,
.jb-shipping-radio-input {
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

.jb-address-card.is-selected .jb-address-radio,
.jb-shipping-card.is-selected .jb-address-radio {
  border-color: #0e5a3a;
}

.jb-address-radio.is-checked {
  border-color: #0e5a3a;
  background: #0e5a3a;
}

/* اطلاعات */
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

/* دکمه‌های اقدام */
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
   آدرس رزرو شده
   ========================================================= */
.jb-reservation-address {
  margin-bottom: 6px;
}

.jb-address-reserved {
  background: #f7fafc;
  border-color: #0e5a3a;
  cursor: default;
}

.jb-address-reserved:hover {
  transform: none;
}

.jb-badge-reserved {
  flex-shrink: 0;
  display: inline-flex;
  align-items: center;
  padding: 4px 10px;
  background: #0e5a3a;
  color: #fff;
  font-size: 0.68rem;
  font-weight: 600;
  border-radius: 4px;
  align-self: flex-start;
}

/* =========================================================
   شیوه ارسال
   ========================================================= */
.jb-shipping-methods {
  margin-top: 6px;
}

.jb-shipping-methods-title {
  font-size: 0.95rem;
  font-weight: 800;
  color: #1a202c;
  margin-bottom: 14px;
}

.jb-shipping-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 10px;
}

@media (min-width: 768px) {
  .jb-shipping-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.jb-shipping-card {
  position: relative;
  display: block;
  padding: 14px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-shipping-card:hover {
  border-color: rgba(14, 90, 58, 0.25);
}

.jb-shipping-card.is-selected {
  border-color: #0e5a3a;
  background: #f7fafc;
}

.jb-shipping-card-inner {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.jb-shipping-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
}

.jb-shipping-card-name {
  font-size: 0.85rem;
  font-weight: 700;
  color: #1a202c;
}

.jb-shipping-card-desc {
  font-size: 0.72rem;
  color: #718096;
  font-weight: 500;
  line-height: 1.8;
  min-height: 2.4em;
}

.jb-shipping-card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  padding-top: 8px;
  border-top: 1px dashed rgba(14, 90, 58, 0.1);
}

.jb-shipping-cost-label {
  font-size: 0.75rem;
  color: #718096;
  font-weight: 500;
}

.jb-shipping-cost-value {
  font-size: 0.9rem;
  font-weight: 800;
  color: #0e5a3a;
  display: inline-flex;
  align-items: baseline;
  gap: 3px;
}

.jb-shipping-cost-unit {
  font-size: 0.65rem;
  color: #a0aec0;
  font-weight: 500;
}

/* =========================================================
   دکمه‌های ناوبری
   ========================================================= */
.jb-shipping-actions {
  display: flex;
  gap: 10px;
  margin-top: 24px;
  flex-wrap: wrap;
}

.jb-shipping-btn-secondary {
  flex: 1;
  min-width: 140px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 10px 20px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.15);
  border-radius: 6px;
  color: #0e5a3a;
  font-size: 0.85rem;
  font-weight: 700;
  font-family: inherit;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-shipping-btn-secondary:hover {
  background: #f7fafc;
  border-color: rgba(14, 90, 58, 0.3);
}

.jb-shipping-btn-primary {
  flex: 1;
  min-width: 160px;
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

.jb-shipping-btn-primary:hover {
  background: #0b472e;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-shipping-card {
    padding: 14px;
    border-radius: 8px;
  }

  .jb-shipping-title {
    font-size: 0.9rem;
  }

  .jb-btn-add-address {
    width: 100%;
    justify-content: center;
  }

  .jb-address-card {
    padding: 12px;
  }

  .jb-address-actions {
    flex-direction: row;
  }

  .jb-shipping-actions {
    flex-direction: column;
  }

  .jb-shipping-btn-secondary,
  .jb-shipping-btn-primary {
    width: 100%;
  }
}
</style>