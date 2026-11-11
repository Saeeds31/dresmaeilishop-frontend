<template>
  <div
    v-if="show"
    class="jb-address-modal-overlay"
    @click.self="emit('close')"
  >
    <div class="jb-address-modal">
      <!-- نوار پیشرفت -->
      <div v-if="isSaving" class="jb-address-progress-track">
        <div
          class="jb-address-progress-bar"
          :style="{ width: progress + '%' }"
        ></div>
      </div>

      <!-- ============================================================
           هدر
      ============================================================ -->
      <div class="jb-address-header">
        <button
          class="jb-address-close-btn"
          @click="emit('close')"
          aria-label="بستن"
        >
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

      <!-- خط بریده‌بریده -->
      <div class="jb-address-dashed"></div>

      <!-- ============================================================
           بدنه فرم
      ============================================================ -->
      <div class="jb-address-body jb-scroll">
        <form
          id="addressForm"
          @submit.prevent="saveAddress"
          class="jb-address-form"
          :class="{ 'is-loading': isSaving }"
        >
          <!-- ===== ردیف ۱: استان و شهر ===== -->
          <div class="jb-form-row">
            <!-- استان -->
            <div class="jb-form-field" ref="provinceBox">
              <label for="province" class="jb-form-label">
                <span class="jb-form-label-dot"></span>
                استان
              </label>

              <input type="hidden" v-model="province_id" />
              <input
                id="province"
                type="text"
                v-model="provinceSearch"
                @focus="openProvinceDropdown"
                @input="onProvinceSearchInput"
                placeholder="جستجو یا انتخاب استان"
                class="jb-form-input"
                :class="{ 'is-error': errors.province_id }"
                autocomplete="new-password"
              />

              <ul
                v-if="provinceDropdownOpen && filteredProvinces.length"
                class="jb-form-dropdown"
              >
                <li
                  v-for="province in filteredProvinces"
                  :key="province.id"
                  @click="selectProvince(province)"
                  class="jb-form-dropdown-item"
                >
                  {{ province.name }}
                </li>
              </ul>

              <div
                v-if="provinceDropdownOpen && !filteredProvinces.length"
                class="jb-form-dropdown jb-form-dropdown-empty"
              >
                استانی یافت نشد
              </div>

              <span v-if="errors.province_id" class="jb-form-error">
                {{ errors.province_id }}
              </span>
            </div>

            <!-- شهر -->
            <div class="jb-form-field" ref="cityBox">
              <label for="city" class="jb-form-label">
                <span class="jb-form-label-dot"></span>
                شهر
              </label>

              <input type="hidden" v-model="city_id" />
              <input
                id="city"
                type="text"
                v-model="citySearch"
                @focus="openCityDropdown"
                @input="onCitySearchInput"
                placeholder="جستجو یا انتخاب شهر"
                class="jb-form-input"
                :class="{ 'is-error': errors.city_id }"
                :disabled="!province_id"
                autocomplete="new-password"
              />

              <ul
                v-if="cityDropdownOpen && filteredCities.length"
                class="jb-form-dropdown"
              >
                <li
                  v-for="city in filteredCities"
                  :key="city.id"
                  @click="selectCity(city)"
                  class="jb-form-dropdown-item"
                >
                  {{ city.name }}
                </li>
              </ul>

              <div
                v-if="cityDropdownOpen && !filteredCities.length"
                class="jb-form-dropdown jb-form-dropdown-empty"
              >
                شهری یافت نشد
              </div>

              <span v-if="errors.city_id" class="jb-form-error">
                {{ errors.city_id }}
              </span>
            </div>
          </div>

          <!-- ===== ردیف ۲: نام گیرنده ===== -->
          <div class="jb-form-row">
            <div class="jb-form-field">
              <label class="jb-form-label">
                <span class="jb-form-label-dot"></span>
                نام گیرنده
              </label>
              <input
                id="first_name"
                autocomplete="new-password"
                v-model="first_name"
                @blur="validateFirstName"
                type="text"
                placeholder="نام"
                class="jb-form-input"
                :class="{ 'is-error': errors.first_name }"
              />
              <span v-if="errors.first_name" class="jb-form-error">
                {{ errors.first_name }}
              </span>
            </div>

            <div class="jb-form-field">
              <label class="jb-form-label">
                <span class="jb-form-label-dot"></span>
                نام خانوادگی گیرنده
              </label>
              <input
                id="last_name"
                autocomplete="new-password"
                v-model="last_name"
                @blur="validateLastName"
                type="text"
                placeholder="نام خانوادگی"
                class="jb-form-input"
                :class="{ 'is-error': errors.last_name }"
              />
              <span v-if="errors.last_name" class="jb-form-error">
                {{ errors.last_name }}
              </span>
            </div>
          </div>

          <!-- ===== ردیف ۳: کدپستی و تلفن ===== -->
          <div class="jb-form-row">
            <div class="jb-form-field">
              <label for="postal" class="jb-form-label">
                <span class="jb-form-label-dot"></span>
                کد پستی
              </label>
              <input
                id="postal"
                v-model="postal_code"
                autocomplete="new-password"
                type="text"
                maxlength="10"
                placeholder="۱۰ رقم"
                class="jb-form-input"
                :class="{ 'is-error': errors.postal_code }"
              />
              <span v-if="errors.postal_code" class="jb-form-error">
                {{ errors.postal_code }}
              </span>
            </div>

            <div class="jb-form-field">
              <label for="phone" class="jb-form-label">
                <span class="jb-form-label-dot"></span>
                شماره تماس
              </label>
              <input
                id="phone"
                v-model="phone"
                @blur="validatePhone"
                type="tel"
                autocomplete="new-password"
                maxlength="11"
                placeholder="۰۹۱۲۳۴۵۶۷۸۹"
                class="jb-form-input"
                :class="{ 'is-error': errors.phone }"
                dir="ltr"
              />
              <span v-if="errors.phone" class="jb-form-error">
                {{ errors.phone }}
              </span>
            </div>
          </div>

          <!-- ===== ردیف ۴: آدرس کامل ===== -->
          <div class="jb-form-field jb-form-field-full">
            <label for="address_line" class="jb-form-label">
              <span class="jb-form-label-dot"></span>
              آدرس کامل
            </label>
            <textarea
              id="address_line"
              v-model="address_line"
              @blur="validateAddressLine"
              autocomplete="new-password"
              rows="3"
              placeholder="خیابان، کوچه، پلاک، واحد..."
              class="jb-form-input jb-form-textarea"
              :class="{ 'is-error': errors.address_line }"
            ></textarea>
            <span v-if="errors.address_line" class="jb-form-error">
              {{ errors.address_line }}
            </span>
          </div>
        </form>
      </div>

      <!-- ============================================================
           فوتر
      ============================================================ -->
      <div class="jb-address-footer">
        <button
          type="submit"
          form="addressForm"
          :disabled="isSaving"
          class="jb-btn-linear jb-address-save-btn"
        >
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

<style scoped>
/* =========================================================
   مودال آدرس - تم پودری
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
  background: rgba(45, 20, 32, 0.55);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
}

/* ===== مودال ===== */
.jb-address-modal {
  position: relative;
  width: 100%;
  max-width: 640px;
  height: 90vh;
  max-height: 780px;
  background: #fff;
  border: 2px solid var(--jb-pink-300);
  border-radius: 26px;
  box-shadow:
    0 25px 60px rgba(214, 51, 132, 0.3),
    0 10px 25px rgba(255, 122, 160, 0.2);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  animation: jbModalIn 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes jbModalIn {
  0% {
    opacity: 0;
    transform: translateY(30px) scale(0.96);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* ===== نوار پیشرفت ===== */
.jb-address-progress-track {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: var(--jb-pink-100);
  z-index: 20;
}

.jb-address-progress-bar {
  height: 100%;
  background: linear-gradient(90deg, var(--jb-pink-500), var(--jb-fuchsia));
  transition: width 0.2s ease-out;
  border-radius: 0 4px 4px 0;
  box-shadow: 0 0 12px rgba(214, 51, 132, 0.6);
}

/* ===== هدر ===== */
.jb-address-header {
  position: relative;
  flex-shrink: 0;
  padding: 20px 20px 16px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
}

.jb-address-title {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  font-size: 1.05rem;
  font-weight: 900;
  color: var(--jb-text);
  text-align: center;
}

.jb-address-title-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 11px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
}

.jb-address-close-btn {
  position: absolute;
  top: 16px;
  left: 16px;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 11px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  color: var(--jb-text-soft);
  cursor: pointer;
  transition: all 0.25s ease;
  z-index: 10;
}

.jb-address-close-btn:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  transform: rotate(90deg);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
}

/* خط بریده‌بریده */
.jb-address-dashed {
  height: 0;
  border-top: 1.5px dashed var(--jb-pink-300);
  margin: 0 16px;
  position: relative;
  flex-shrink: 0;
}

.jb-address-dashed::before,
.jb-address-dashed::after {
  content: "";
  position: absolute;
  top: -4px;
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--jb-pink-400);
  box-shadow: 0 0 0 2px #fff;
}

.jb-address-dashed::before { right: -3px; }
.jb-address-dashed::after  { left: -3px; }

/* ===== بدنه ===== */
.jb-address-body {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.jb-address-body.jb-scroll::-webkit-scrollbar {
  width: 6px;
}

.jb-address-body.jb-scroll::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, var(--jb-pink-400), var(--jb-fuchsia));
  border-radius: 6px;
}

.jb-address-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
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
  gap: 14px;
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
  gap: 8px;
}

.jb-form-field-full {
  grid-column: 1 / -1;
}

/* ===== لیبل ===== */
.jb-form-label {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
}

.jb-form-label-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 2px rgba(214, 51, 132, 0.18);
}

/* ===== اینپوت‌ها ===== */
.jb-form-input {
  width: 100%;
  padding: 12px 14px;
  background: #fff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 14px;
  font-size: 0.88rem;
  font-weight: 700;
  color: var(--jb-text);
  outline: none;
  transition: all 0.25s ease;
  font-family: inherit;
}

.jb-form-input::placeholder {
  color: var(--jb-text-muted);
  font-weight: 500;
}

.jb-form-input:focus {
  border-color: var(--jb-fuchsia);
  box-shadow: 0 0 0 4px rgba(214, 51, 132, 0.15);
}

.jb-form-input:disabled {
  background: var(--jb-pink-50);
  opacity: 0.7;
  cursor: not-allowed;
}

.jb-form-input.is-error {
  border-color: #ef4444;
  box-shadow: 0 0 0 4px rgba(239, 68, 68, 0.12);
}

.jb-form-textarea {
  resize: none;
  min-height: 88px;
  line-height: 1.9;
}

/* ===== دراپ‌داون ===== */
.jb-form-dropdown {
  position: absolute;
  top: calc(100% - 22px);
  right: 0;
  left: 0;
  max-height: 220px;
  overflow-y: auto;
  background: #fff;
  border: 2px solid var(--jb-pink-300);
  border-radius: 14px;
  box-shadow: 0 12px 30px rgba(214, 51, 132, 0.2);
  padding: 6px;
  z-index: 50;
  margin-top: 6px;
  animation: jbDropdownIn 0.2s ease;
}

@keyframes jbDropdownIn {
  from { opacity: 0; transform: translateY(-6px); }
  to   { opacity: 1; transform: translateY(0); }
}

.jb-form-dropdown-item {
  padding: 10px 14px;
  border-radius: 10px;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--jb-text-soft);
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-form-dropdown-item:hover {
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  color: var(--jb-fuchsia);
  font-weight: 800;
}

.jb-form-dropdown-empty {
  padding: 14px;
  text-align: center;
  font-size: 0.82rem;
  font-weight: 600;
  color: var(--jb-text-muted);
}

/* ===== خطا ===== */
.jb-form-error {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 0.72rem;
  font-weight: 700;
  color: #dc2626;
  padding: 4px 10px;
  background: #fef2f2;
  border-radius: 8px;
  width: fit-content;
}

/* ===== فوتر ===== */
.jb-address-footer {
  flex-shrink: 0;
  padding: 16px 20px 20px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  border-top: 1.5px dashed var(--jb-pink-300);
}

.jb-address-save-btn {
  width: 100%;
  justify-content: center;
  padding: 14px 24px !important;
  font-size: 0.92rem !important;
}

.jb-address-save-btn:disabled {
  opacity: 0.7;
  cursor: wait;
}

.jb-address-save-inner {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.jb-address-save-percent {
  font-size: 0.72rem;
  opacity: 0.85;
}

/* اسپینر */
.jb-spinner {
  width: 18px;
  height: 18px;
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
    border-radius: 22px;
  }

  .jb-address-header {
    padding: 16px;
  }

  .jb-address-body {
    padding: 16px;
  }

  .jb-address-footer {
    padding: 14px 16px 18px;
  }

  .jb-address-title {
    font-size: 0.95rem;
  }

  .jb-address-title-icon {
    width: 30px;
    height: 30px;
  }

  .jb-form-input {
    padding: 11px 12px;
    font-size: 0.82rem;
  }
}
</style>