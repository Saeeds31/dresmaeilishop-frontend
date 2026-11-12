
<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch } from "vue";
import { X } from "lucide-vue-next";

const props = defineProps({
  show: Boolean,
  editAddress: Object,
});

const emit = defineEmits(["close", "save"]);

watch(
  () => props.show,
  (isOpen) => {
    if (!isOpen && isSaving.value) {
      cancelProgress();
      if (abortController) {
        abortController.abort();
        abortController = null;
      }
    }
  }
);

const { createAddress, updateAddress, loading } = useAddresses();
const { provinces, cities, fetchProvinces, fetchCities } = useLocations();

const isSaving = ref(false);
const progress = ref(0);
const saveAttempt = ref(0);
let progressInterval = null;
let abortController = null;

const first_name = ref("");
const last_name = ref("");
const TIMEOUT_MS = 3000;
const MAX_ATTEMPTS = 2;

const province_id = ref("");
const city_id = ref("");
const postal_code = ref("");
const phone = ref("");
const address_line = ref("");

const citySearch = ref("");
const cityDropdownOpen = ref(false);
const cityBox = ref(null);

const provinceSearch = ref("");
const provinceDropdownOpen = ref(false);
const provinceBox = ref(null);

const filteredProvinces = computed(() => {
  const list = provinces.value || [];
  if (!provinceSearch.value) return list;
  return list.filter((p) => p.name.includes(provinceSearch.value));
});

function openProvinceDropdown() {
  provinceDropdownOpen.value = true;
}

function onProvinceSearchInput() {
  province_id.value = "";
  provinceDropdownOpen.value = true;
  errors.value.province_id = "";
}

function selectProvince(province) {
  province_id.value = province.id;
  provinceSearch.value = province.name;
  provinceDropdownOpen.value = false;
  validateProvince();
}

const filteredCities = computed(() => {
  const list = cities.value || [];
  if (!citySearch.value) return list;
  return list.filter((c) => c.name.includes(citySearch.value));
});

function openCityDropdown() {
  if (!province_id.value) return;
  cityDropdownOpen.value = true;
}

function onCitySearchInput() {
  city_id.value = "";
  cityDropdownOpen.value = true;
  errors.value.city_id = "";
}

function selectCity(city) {
  city_id.value = city.id;
  citySearch.value = city.name;
  cityDropdownOpen.value = false;
  validateCity();
}

function handleClickOutside(e) {
  if (cityBox.value && !cityBox.value.contains(e.target)) {
    cityDropdownOpen.value = false;
  }
  if (provinceBox.value && !provinceBox.value.contains(e.target)) {
    provinceDropdownOpen.value = false;
  }
}

onMounted(async () => {
  if (provinces.value.length === 0) {
    await fetchProvinces();
  }
  document.addEventListener("click", handleClickOutside);
});

onBeforeUnmount(() => {
  document.removeEventListener("click", handleClickOutside);
});

watch(
  () => props.show,
  async (isOpen) => {
    if (isOpen) {
      if (provinces.value.length === 0) {
        await fetchProvinces();
      }

      if (props.editAddress) {
        const fullName = (props.editAddress.receiver_name || "").trim();
        const parts = fullName.split(/\s+/);
        first_name.value = parts[0] || "";
        last_name.value = parts.slice(1).join(" ") || "";
        province_id.value = props.editAddress.province_id || "";
        city_id.value = props.editAddress.city_id || "";
        postal_code.value = props.editAddress.postal_code || "";
        phone.value = props.editAddress.phone || "";
        address_line.value = props.editAddress.address_line || "";

        const foundProvince = (provinces.value || []).find(
          (p) => p.id === province_id.value
        );
        provinceSearch.value = foundProvince ? foundProvince.name : "";

        if (province_id.value) {
          await fetchCities(province_id.value);
          const found = (cities.value || []).find(
            (c) => c.id === city_id.value
          );
          citySearch.value = found ? found.name : "";
        }
      } else {
        province_id.value = "";
        city_id.value = "";
        first_name.value = "";
        last_name.value = "";
        postal_code.value = "";
        phone.value = "";
        address_line.value = "";
        citySearch.value = "";
        cityDropdownOpen.value = false;
        provinceSearch.value = "";
        provinceDropdownOpen.value = false;
      }
    }
  }
);

watch(province_id, async (newProvinceId, oldProvinceId) => {
  if (newProvinceId && newProvinceId !== oldProvinceId) {
    city_id.value = "";
    citySearch.value = "";
    cityDropdownOpen.value = false;
    await fetchCities(newProvinceId);
  } else if (!newProvinceId) {
    cities.value = [];
    citySearch.value = "";
    city_id.value = "";
    cityDropdownOpen.value = false;
  }
});

const errors = ref({
  province_id: "",
  city_id: "",
  first_name: "",
  last_name: "",
  postal_code: "",
  phone: "",
  address_line: "",
});

const validateFirstName = () => {
  if (!first_name.value.trim()) {
    errors.value.first_name = "نام الزامی است";
    return false;
  }
  if (first_name.value.trim().length < 2) {
    errors.value.first_name = "نام باید حداقل 2 حرف باشد";
    return false;
  }
  errors.value.first_name = "";
  return true;
};

const validateLastName = () => {
  if (!last_name.value.trim()) {
    errors.value.last_name = "نام خانوادگی الزامی است";
    return false;
  }
  if (last_name.value.trim().length < 2) {
    errors.value.last_name = "نام خانوادگی باید حداقل 2 حرف باشد";
    return false;
  }
  errors.value.last_name = "";
  return true;
};

const validatePhone = () => {
  if (!phone.value.trim()) {
    errors.value.phone = "شماره تماس الزامی است";
    return false;
  }
  const phoneRegex = /^09\d{9}$/;
  if (!phoneRegex.test(phone.value)) {
    errors.value.phone = "شماره تماس باید 11 رقم و با 09 شروع شود";
    return false;
  }
  errors.value.phone = "";
  return true;
};

const validatePostalCode = () => {
  if (!postal_code.value.trim()) {
    errors.value.postal_code = "کد پستی الزامی است";
    return false;
  }
  const postalRegex = /^\d{10}$/;
  if (!postalRegex.test(postal_code.value)) {
    errors.value.postal_code = "کد پستی باید 10 رقم باشد";
    return false;
  }
  errors.value.postal_code = "";
  return true;
};

const validateAddressLine = () => {
  if (!address_line.value.trim()) {
    errors.value.address_line = "آدرس الزامی است";
    return false;
  }
  const wordCount = address_line.value.trim().split(/\s+/).length;
  if (wordCount < 3) {
    errors.value.address_line = "آدرس باید حداقل 3 کلمه داشته باشد";
    return false;
  }
  errors.value.address_line = "";
  return true;
};

const validateProvince = () => {
  if (!province_id.value) {
    errors.value.province_id = "انتخاب استان الزامی است";
    return false;
  }
  errors.value.province_id = "";
  return true;
};

const validateCity = () => {
  if (!city_id.value) {
    errors.value.city_id = "انتخاب شهر الزامی است";
    return false;
  }
  errors.value.city_id = "";
  return true;
};

const isFormValid = computed(() => {
  return (
    !errors.value.province_id &&
    !errors.value.city_id &&
    !errors.value.first_name &&
    !errors.value.last_name &&
    !errors.value.postal_code &&
    !errors.value.phone &&
    !errors.value.address_line &&
    province_id.value &&
    city_id.value &&
    postal_code.value &&
    phone.value &&
    address_line.value
  );
});

async function executeWithTimeoutAndRetry(
  fn,
  { timeout = TIMEOUT_MS, retries = 1 } = {}
) {
  let lastError = null;

  for (let attempt = 0; attempt <= retries; attempt++) {
    saveAttempt.value = attempt + 1;
    abortController = new AbortController();

    const timeoutId = setTimeout(() => {
      abortController.abort();
    }, timeout);

    try {
      const result = await fn(abortController.signal);
      clearTimeout(timeoutId);
      return result;
    } catch (err) {
      clearTimeout(timeoutId);
      lastError = err;

      if (attempt === retries) {
        throw err;
      }

      console.warn(`⏱️ Attempt ${attempt + 1} failed, retrying...`, err?.message);
      await new Promise((r) => setTimeout(r, 300));
    }
  }

  throw lastError;
}

function startProgress() {
  progress.value = 0;
  isSaving.value = true;

  progressInterval = setInterval(() => {
    if (progress.value < 90) {
      progress.value += Math.random() * 8;
    } else if (progress.value < 98) {
      progress.value += 0.5;
    }
  }, 200);
}

function finishProgress() {
  clearInterval(progressInterval);
  progressInterval = null;
  progress.value = 100;

  setTimeout(() => {
    isSaving.value = false;
    progress.value = 0;
    saveAttempt.value = 0;
  }, 400);
}

function cancelProgress() {
  clearInterval(progressInterval);
  progressInterval = null;
  isSaving.value = false;
  progress.value = 0;
  saveAttempt.value = 0;
}

async function saveAddress() {
  const isProvinceValid = validateProvince();
  const isCityValid = validateCity();
  const isFirstNameValid = validateFirstName();
  const isLastNameValid = validateLastName();
  const isPhoneValid = validatePhone();
  const isAddressValid = validateAddressLine();

  if (
    !isProvinceValid ||
    !isCityValid ||
    !isFirstNameValid ||
    !isLastNameValid ||
    !isPhoneValid ||
    !isAddressValid
  ) {
    return;
  }

  const receiver_name = `${first_name.value.trim()} ${last_name.value.trim()}`.trim();
  const addressData = {
    receiver_name: receiver_name,
    province_id: province_id.value,
    city_id: city_id.value,
    postal_code: postal_code.value,
    address_line: address_line.value,
    phone: phone.value,
  };

  startProgress();

  try {
    const isEdit = props.editAddress && props.editAddress.id;

    await executeWithTimeoutAndRetry(
      (signal) =>
        isEdit
          ? updateAddress(props.editAddress.id, addressData, signal)
          : createAddress(addressData, signal),
      { timeout: TIMEOUT_MS, retries: 1 }
    );

    finishProgress();

    province_id.value = "";
    city_id.value = "";
    first_name.value = "";
    last_name.value = "";
    postal_code.value = "";
    phone.value = "";
    address_line.value = "";
    citySearch.value = "";
    cityDropdownOpen.value = false;
    provinceSearch.value = "";
    provinceDropdownOpen.value = false;
    errors.value = {
      province_id: "",
      city_id: "",
      first_name: "",
      last_name: "",
      postal_code: "",
      phone: "",
      address_line: "",
    };

    emit("save");
  } catch (err) {
    cancelProgress();
    const isTimeout =
      err?.name === "AbortError" || err?.code === "ERR_CANCELED";

    if (err.response?.data?.errors) {
      const serverErrors = err.response.data.errors;
      if (serverErrors.receiver_name)
        errors.value.first_name = serverErrors.receiver_name[0];
      if (serverErrors.province_id)
        errors.value.province_id = serverErrors.province_id[0];
      if (serverErrors.city_id)
        errors.value.city_id = serverErrors.city_id[0];
      if (serverErrors.postal_code)
        errors.value.postal_code = serverErrors.postal_code[0];
      if (serverErrors.address_line)
        errors.value.address_line = serverErrors.address_line[0];
      if (serverErrors.phone) errors.value.phone = serverErrors.phone[0];
    }
  }
}
</script>
<template>
  <div
    v-if="show"
    class="jb-address-modal-overlay"
    @click.self="emit('close')"
  >
    <div class="jb-address-modal">
      <!-- نوار پیشرفت -->
      <div v-if="isSaving" class="jb-address-progress-track">
        <div class="jb-address-progress-bar" :style="{ width: progress + '%' }"></div>
      </div>

      <!-- هدر -->
      <div class="jb-address-header">
        <button class="jb-address-close-btn" @click="emit('close')" aria-label="بستن">
          <X class="w-4 h-4" />
        </button>

        <h2 class="jb-address-title">
          <span class="jb-address-title-icon">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M17.657 16.657L13.414 20.9a2 2 0 01-2.828 0l-4.243-4.243a8 8 0 1111.314 0z" />
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
            </svg>
          </span>
          {{ editAddress ? "ویرایش آدرس" : "افزودن آدرس" }}
        </h2>
      </div>

      <!-- خط جداکننده -->
      <div class="jb-address-dashed"></div>

      <!-- بدنه فرم -->
      <div class="jb-address-body jb-scroll">
        <form id="addressForm" @submit.prevent="saveAddress" class="jb-address-form"
          :class="{ 'is-loading': isSaving }">
          <!-- ردیف ۱: استان و شهر -->
          <div class="jb-form-row">
            <div class="jb-form-field" ref="provinceBox">
              <label for="province" class="jb-form-label">استان</label>
              <input type="hidden" v-model="province_id" />
              <input id="province" type="text" v-model="provinceSearch" @focus="openProvinceDropdown"
                @input="onProvinceSearchInput" placeholder="جستجو یا انتخاب استان" class="jb-form-input"
                :class="{ 'is-error': errors.province_id }" autocomplete="new-password" />

              <ul v-if="provinceDropdownOpen && filteredProvinces.length" class="jb-form-dropdown">
                <li v-for="province in filteredProvinces" :key="province.id" @click="selectProvince(province)"
                  class="jb-form-dropdown-item">
                  {{ province.name }}
                </li>
              </ul>

              <div v-if="provinceDropdownOpen && !filteredProvinces.length"
                class="jb-form-dropdown jb-form-dropdown-empty">
                استانی یافت نشد
              </div>

              <span v-if="errors.province_id" class="jb-form-error">
                {{ errors.province_id }}
              </span>
            </div>

            <div class="jb-form-field" ref="cityBox">
              <label for="city" class="jb-form-label">شهر</label>
              <input type="hidden" v-model="city_id" />
              <input id="city" type="text" v-model="citySearch" @focus="openCityDropdown"
                @input="onCitySearchInput" placeholder="جستجو یا انتخاب شهر" class="jb-form-input"
                :class="{ 'is-error': errors.city_id }" :disabled="!province_id" autocomplete="new-password" />

              <ul v-if="cityDropdownOpen && filteredCities.length" class="jb-form-dropdown">
                <li v-for="city in filteredCities" :key="city.id" @click="selectCity(city)"
                  class="jb-form-dropdown-item">
                  {{ city.name }}
                </li>
              </ul>

              <div v-if="cityDropdownOpen && !filteredCities.length" class="jb-form-dropdown jb-form-dropdown-empty">
                شهری یافت نشد
              </div>

              <span v-if="errors.city_id" class="jb-form-error">
                {{ errors.city_id }}
              </span>
            </div>
          </div>

          <!-- ردیف ۲: نام گیرنده -->
          <div class="jb-form-row">
            <div class="jb-form-field">
              <label class="jb-form-label">نام گیرنده</label>
              <input id="first_name" autocomplete="new-password" v-model="first_name" @blur="validateFirstName"
                type="text" placeholder="نام" class="jb-form-input" :class="{ 'is-error': errors.first_name }" />
              <span v-if="errors.first_name" class="jb-form-error">
                {{ errors.first_name }}
              </span>
            </div>

            <div class="jb-form-field">
              <label class="jb-form-label">نام خانوادگی گیرنده</label>
              <input id="last_name" autocomplete="new-password" v-model="last_name" @blur="validateLastName"
                type="text" placeholder="نام خانوادگی" class="jb-form-input"
                :class="{ 'is-error': errors.last_name }" />
              <span v-if="errors.last_name" class="jb-form-error">
                {{ errors.last_name }}
              </span>
            </div>
          </div>

          <!-- ردیف ۳: کدپستی و تلفن -->
          <div class="jb-form-row">
            <div class="jb-form-field">
              <label for="postal" class="jb-form-label">کد پستی</label>
              <input id="postal" v-model="postal_code" autocomplete="new-password" type="text" maxlength="10"
                placeholder="۱۰ رقم" class="jb-form-input" :class="{ 'is-error': errors.postal_code }" />
              <span v-if="errors.postal_code" class="jb-form-error">
                {{ errors.postal_code }}
              </span>
            </div>

            <div class="jb-form-field">
              <label for="phone" class="jb-form-label">شماره تماس</label>
              <input id="phone" v-model="phone" @blur="validatePhone" type="tel" autocomplete="new-password"
                maxlength="11" placeholder="۰۹۱۲۳۴۵۶۷۸۹" class="jb-form-input" :class="{ 'is-error': errors.phone }"
                dir="ltr" />
              <span v-if="errors.phone" class="jb-form-error">
                {{ errors.phone }}
              </span>
            </div>
          </div>

          <!-- ردیف ۴: آدرس کامل -->
          <div class="jb-form-field jb-form-field-full">
            <label for="address_line" class="jb-form-label">آدرس کامل</label>
            <textarea id="address_line" v-model="address_line" @blur="validateAddressLine"
              autocomplete="new-password" rows="3" placeholder="خیابان، کوچه، پلاک، واحد..."
              class="jb-form-input jb-form-textarea" :class="{ 'is-error': errors.address_line }"></textarea>
            <span v-if="errors.address_line" class="jb-form-error">
              {{ errors.address_line }}
            </span>
          </div>
        </form>
      </div>

      <!-- فوتر -->
      <div class="jb-address-footer">
        <button type="submit" form="addressForm" :disabled="isSaving" class="jb-address-save-btn">
          <span v-if="!isSaving" class="jb-address-save-inner">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
                d="M5 13l4 4L19 7" />
            </svg>
            ذخیره آدرس
          </span>

          <span v-else class="jb-address-save-inner">
            <svg class="jb-spinner" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
              <path class="opacity-75" fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
            </svg>
            {{ saveAttempt > 1 ? "تلاش مجدد..." : "در حال ذخیره..." }}
            <span class="jb-address-save-percent">({{ Math.round(progress) }}%)</span>
          </span>
        </button>
      </div>
    </div>
  </div>
</template>


<style scoped>
/* =========================================================
   مودال آدرس - مینیمال سبز
   ========================================================= */

/* ===== Overlay ===== */
.jb-address-modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 50;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
  background: rgba(14, 90, 58, 0.4);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
}

/* ===== مودال ===== */
.jb-address-modal {
  position: relative;
  width: 100%;
  max-width: 600px;
  height: 90vh;
  max-height: 750px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  box-shadow: 0 10px 40px rgba(14, 90, 58, 0.15);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  animation: jbModalIn 0.3s ease;
}

@keyframes jbModalIn {
  0% { opacity: 0; transform: translateY(20px); }
  100% { opacity: 1; transform: translateY(0); }
}

/* ===== نوار پیشرفت ===== */
.jb-address-progress-track {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: rgba(14, 90, 58, 0.08);
  z-index: 20;
}

.jb-address-progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #0e5a3a, #1a7a50);
  transition: width 0.2s ease-out;
}

/* ===== هدر ===== */
.jb-address-header {
  position: relative;
  flex-shrink: 0;
  padding: 18px 20px 14px;
  border-bottom: 1px solid rgba(14, 90, 58, 0.08);
}

.jb-address-title {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-size: 1rem;
  font-weight: 800;
  color: #1a202c;
  text-align: center;
}

.jb-address-title-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 30px;
  height: 30px;
  border-radius: 6px;
  background: #0e5a3a;
  color: #fff;
}

.jb-address-close-btn {
  position: absolute;
  top: 14px;
  left: 14px;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.1);
  color: #4a5568;
  cursor: pointer;
  transition: all 0.2s ease;
  z-index: 10;
}

.jb-address-close-btn:hover {
  background: #0e5a3a;
  color: #fff;
  border-color: transparent;
}

/* خط جداکننده */
.jb-address-dashed {
  height: 0;
  border-top: 1px dashed rgba(14, 90, 58, 0.15);
  margin: 0 16px;
  flex-shrink: 0;
}

/* ===== بدنه ===== */
.jb-address-body {
  flex: 1;
  overflow-y: auto;
  padding: 18px;
}

.jb-address-body.jb-scroll::-webkit-scrollbar {
  width: 5px;
}

.jb-address-body.jb-scroll::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 3px;
}

.jb-address-form {
  display: flex;
  flex-direction: column;
  gap: 14px;
  transition: opacity 0.3s ease;
}

.jb-address-form.is-loading {
  opacity: 0.6;
  pointer-events: none;
}

/* ===== ردیف‌های فرم ===== */
.jb-form-row {
  display: grid;
  grid-template-columns: 1fr;
  gap: 12px;
}

@media (min-width: 640px) {
  .jb-form-row {
    grid-template-columns: repeat(2, 1fr);
  }
}

.jb-form-field {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.jb-form-field-full {
  grid-column: 1 / -1;
}

/* ===== لیبل ===== */
.jb-form-label {
  font-size: 0.82rem;
  font-weight: 700;
  color: #1a202c;
}

/* ===== اینپوت‌ها ===== */
.jb-form-input {
  width: 100%;
  padding: 10px 12px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.12);
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 600;
  color: #1a202c;
  outline: none;
  transition: all 0.2s ease;
  font-family: inherit;
}

.jb-form-input::placeholder {
  color: #a0aec0;
  font-weight: 500;
}

.jb-form-input:focus {
  border-color: #0e5a3a;
  box-shadow: 0 0 0 3px rgba(14, 90, 58, 0.1);
}

.jb-form-input:disabled {
  background: #f7fafc;
  opacity: 0.7;
  cursor: not-allowed;
}

.jb-form-input.is-error {
  border-color: #e53e3e;
  box-shadow: 0 0 0 3px rgba(229, 62, 62, 0.1);
}

.jb-form-textarea {
  resize: none;
  min-height: 80px;
  line-height: 1.8;
}

/* ===== دراپ‌داون ===== */
.jb-form-dropdown {
  position: absolute;
  top: calc(100% - 18px);
  right: 0;
  left: 0;
  max-height: 200px;
  overflow-y: auto;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.15);
  border-radius: 6px;
  box-shadow: 0 8px 24px rgba(14, 90, 58, 0.12);
  padding: 4px;
  z-index: 50;
  margin-top: 4px;
  animation: jbDropdownIn 0.15s ease;
}

@keyframes jbDropdownIn {
  from { opacity: 0; transform: translateY(-4px); }
  to { opacity: 1; transform: translateY(0); }
}

.jb-form-dropdown-item {
  padding: 9px 12px;
  border-radius: 4px;
  font-size: 0.82rem;
  font-weight: 500;
  color: #4a5568;
  cursor: pointer;
  transition: all 0.15s ease;
}

.jb-form-dropdown-item:hover {
  background: #f1f7f3;
  color: #0e5a3a;
  font-weight: 600;
}

.jb-form-dropdown-empty {
  padding: 12px;
  text-align: center;
  font-size: 0.8rem;
  font-weight: 500;
  color: #a0aec0;
}

/* ===== خطا ===== */
.jb-form-error {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 0.7rem;
  font-weight: 600;
  color: #c53030;
  padding: 3px 8px;
  background: #fff5f5;
  border-radius: 4px;
  width: fit-content;
}

/* ===== فوتر ===== */
.jb-address-footer {
  flex-shrink: 0;
  padding: 14px 18px 18px;
  border-top: 1px solid rgba(14, 90, 58, 0.08);
}

.jb-address-save-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 11px 20px;
  background: #0e5a3a;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.88rem;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-address-save-btn:hover:not(:disabled) {
  background: #0b472e;
}

.jb-address-save-btn:disabled {
  opacity: 0.6;
  cursor: wait;
}

.jb-address-save-inner {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.jb-address-save-percent {
  font-size: 0.7rem;
  opacity: 0.85;
}

/* اسپینر */
.jb-spinner {
  width: 16px;
  height: 16px;
  animation: jbSpin 0.9s linear infinite;
}

@keyframes jbSpin {
  to { transform: rotate(360deg); }
}

/* ===== ریسپانسیو ===== */
@media (max-width: 640px) {
  .jb-address-modal {
    height: 95vh;
    max-height: none;
    border-radius: 8px;
  }

  .jb-address-header {
    padding: 14px 16px 12px;
  }

  .jb-address-body {
    padding: 14px;
  }

  .jb-address-footer {
    padding: 12px 14px 16px;
  }

  .jb-address-title {
    font-size: 0.9rem;
  }

  .jb-address-title-icon {
    width: 26px;
    height: 26px;
  }

  .jb-form-input {
    padding: 9px 10px;
    font-size: 0.8rem;
  }
}
</style>