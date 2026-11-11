<template>
  <div
    v-if="show"
    class="jb-edit-modal-overlay"
    @click.self="emit('close')"
  >
    <div class="jb-edit-modal">
      <!-- دکمه بستن -->
      <button class="jb-edit-modal-close" @click="emit('close')" aria-label="بستن">
        <X class="w-4 h-4" />
      </button>

      <!-- هدر -->
      <div class="jb-edit-modal-header">
        <div class="jb-edit-modal-icon">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
          </svg>
        </div>
        <h2 class="jb-edit-modal-title">ویرایش اطلاعات کاربری</h2>
      </div>

      <!-- خط بریده‌بریده -->
      <div class="jb-edit-dashed"></div>

      <!-- بدنه فرم (اسکرول‌پذیر) -->
      <div class="jb-edit-modal-body jb-scroll">
        <form id="editUserForm" @submit.prevent="saveUser" class="jb-edit-form">
          <!-- ===== ردیف ۱ ===== -->
          <div class="jb-form-row">
            <!-- نام و نام خانوادگی -->
            <div class="jb-form-field">
              <label for="fullName" class="jb-form-label">
                <span class="jb-form-dot"></span>
                نام و نام خانوادگی
              </label>
              <div class="jb-input-wrap">
                <span class="jb-input-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
                  </svg>
                </span>
                <input
                  id="fullName"
                  v-model="name"
                  type="text"
                  placeholder="مثال: حمیدرضا حاج‌دادی"
                  class="jb-form-input"
                />
              </div>
            </div>

            <!-- تاریخ تولد -->
            <div class="jb-form-field">
              <label for="birthDate" class="jb-form-label">
                <span class="jb-form-dot"></span>
                تاریخ تولد
              </label>
              <div class="jb-input-wrap">
                <span class="jb-input-icon">
                  <Calendar class="w-4 h-4" />
                </span>
                <input
                  id="birthDate"
                  v-model="birthDate"
                  type="date"
                  class="jb-form-input jb-date-input"
                />
              </div>
            </div>
          </div>

          <!-- ===== ردیف ۲ ===== -->
          <div class="jb-form-row">
            <!-- شماره تماس -->
            <div class="jb-form-field">
              <label for="phone" class="jb-form-label">
                <span class="jb-form-dot"></span>
                شماره تماس
              </label>
              <div class="jb-input-wrap">
                <span class="jb-input-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z" />
                  </svg>
                </span>
                <input
                  id="phone"
                  v-model="phone"
                  type="text"
                  placeholder="۰۹*********"
                  class="jb-form-input"
                  dir="ltr"
                />
              </div>
            </div>

            <!-- کد ملی -->
            <div class="jb-form-field">
              <label for="nationalCode" class="jb-form-label">
                <span class="jb-form-dot"></span>
                کد ملی
              </label>
              <div class="jb-input-wrap">
                <span class="jb-input-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M10 6H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V8a2 2 0 00-2-2h-5m-4 0V5a2 2 0 114 0v1m-4 0a2 2 0 104 0" />
                  </svg>
                </span>
                <input
                  id="nationalCode"
                  v-model="nationalCode"
                  type="text"
                  placeholder="۱۲۳۴۵۶۷۸۹۰"
                  class="jb-form-input"
                  dir="ltr"
                />
              </div>
            </div>
          </div>

          <!-- ===== ردیف ۳: ایمیل (تمام عرض) ===== -->
          <div class="jb-form-field jb-form-field-full">
            <label for="email" class="jb-form-label">
              <span class="jb-form-dot"></span>
              ایمیل
            </label>
            <div class="jb-input-wrap">
              <span class="jb-input-icon">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
                </svg>
              </span>
              <input
                id="email"
                v-model="email"
                type="email"
                placeholder="example@mail.com"
                class="jb-form-input"
                dir="ltr"
              />
            </div>
          </div>

          <!-- ===== خط جداکننده بخش رمز عبور ===== -->
          <div class="jb-form-section-divider">
            <span class="jb-form-section-text">تغییر رمز عبور (اختیاری)</span>
          </div>

          <!-- ===== ردیف ۴: رمز عبور ===== -->
          <div class="jb-form-row">
            <!-- رمز عبور -->
            <div class="jb-form-field">
              <label for="password" class="jb-form-label">
                <span class="jb-form-dot"></span>
                رمز عبور جدید
              </label>
              <div class="jb-input-wrap">
                <span class="jb-input-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                  </svg>
                </span>
                <input
                  id="password"
                  v-model="password"
                  type="password"
                  placeholder="در صورت تغییر وارد کنید"
                  class="jb-form-input"
                />
              </div>
            </div>

            <!-- تأیید رمز عبور -->
            <div class="jb-form-field">
              <label for="confirmPassword" class="jb-form-label">
                <span class="jb-form-dot"></span>
                تأیید رمز عبور
              </label>
              <div class="jb-input-wrap">
                <span class="jb-input-icon">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                      d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
                  </svg>
                </span>
                <input
                  id="confirmPassword"
                  v-model="confirmPassword"
                  type="password"
                  placeholder="تکرار رمز عبور"
                  class="jb-form-input"
                />
              </div>
            </div>
          </div>
        </form>
      </div>

      <!-- ===== فوتر ===== -->
      <div class="jb-edit-modal-footer">
        <button
          type="submit"
          form="editUserForm"
          :disabled="loading"
          class="jb-btn-linear jb-edit-save-btn"
        >
          <span v-if="!loading" class="jb-edit-save-inner">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
            </svg>
            ذخیره اطلاعات
          </span>
          <span v-else class="jb-edit-save-inner">
            <svg class="jb-spinner" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
              <path class="opacity-75" fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
            </svg>
            در حال ذخیره...
          </span>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useApi } from "@/composables/api/useApi";
import { toast } from "vue3-toastify";
import { ref, watch } from "vue";
import { X, Calendar } from "lucide-vue-next";

const props = defineProps({
  show: Boolean,
  userData: Object,
});
const emit = defineEmits(["close", "save"]);

const name = ref("");
const birthDate = ref("");
const phone = ref("");
const email = ref("");
const nationalCode = ref("");
const password = ref("");
const confirmPassword = ref("");

watch(
  () => [props.show, props.userData],
  ([show, userData]) => {
    if (show && userData) {
      name.value = userData.full_name || "";
      birthDate.value = userData.birth_date || "";
      phone.value = userData.mobile || "";
      email.value = userData.email || "";
      nationalCode.value = userData.national_code || "";
      password.value = "";
      confirmPassword.value = "";
    }
  },
  { immediate: true }
);

const { updateUserProfile } = useApi();
const loading = ref(false);

async function saveUser() {
  if (!name.value || !phone.value) {
    toast.error("لطفاً نام و شماره تماس را وارد کنید");
    return;
  }

  if (password.value && password.value !== confirmPassword.value) {
    toast.error("رمز عبور و تأیید آن یکسان نیست");
    return;
  }

  loading.value = true;

  try {
    const updateData = {
      full_name: name.value,
      mobile: phone.value,
      birth_date: birthDate.value || undefined,
      email: email.value || undefined,
      national_code: nationalCode.value || undefined,
    };

    if (password.value) {
      updateData.password = password.value;
      updateData.password_confirmation = confirmPassword.value;
    }

    await updateUserProfile(updateData);

    toast.success("اطلاعات با موفقیت به‌روزرسانی شد");
    emit("save");
    emit("close");
  } catch (error) {
    console.error("Failed to update profile:", error);
    toast.error(
      error.response?.data?.message || "خطا در به‌روزرسانی اطلاعات"
    );
  } finally {
    loading.value = false;
  }
}
</script>

<style scoped>
/* =========================================================
   مودال ویرایش اطلاعات کاربری - تم پودری
   ========================================================= */

.jb-edit-modal-overlay {
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

.jb-edit-modal {
  position: relative;
  width: 100%;
  max-width: 680px;
  max-height: 92vh;
  display: flex;
  flex-direction: column;
  background: #fff;
  border: 2px solid var(--jb-pink-300);
  border-radius: 26px;
  overflow: hidden;
  box-shadow:
    0 25px 60px rgba(214, 51, 132, 0.3),
    0 10px 25px rgba(255, 122, 160, 0.2);
  animation: jbEditModalIn 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes jbEditModalIn {
  0% {
    opacity: 0;
    transform: translateY(30px) scale(0.96);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* ===== دکمه بستن ===== */
.jb-edit-modal-close {
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

.jb-edit-modal-close:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  transform: rotate(90deg);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
}

/* ===== هدر ===== */
.jb-edit-modal-header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 20px 20px 16px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  flex-shrink: 0;
}

.jb-edit-modal-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 13px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
}

.jb-edit-modal-title {
  font-size: 1.05rem;
  font-weight: 900;
  color: var(--jb-text);
}

/* خط بریده‌بریده */
.jb-edit-dashed {
  height: 0;
  border-top: 1.5px dashed var(--jb-pink-300);
  margin: 0 16px;
  position: relative;
  flex-shrink: 0;
}

.jb-edit-dashed::before,
.jb-edit-dashed::after {
  content: "";
  position: absolute;
  top: -4px;
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--jb-pink-400);
  box-shadow: 0 0 0 2px #fff;
}

.jb-edit-dashed::before { right: -3px; }
.jb-edit-dashed::after  { left: -3px; }

/* ===== بدنه ===== */
.jb-edit-modal-body {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.jb-edit-modal-body.jb-scroll::-webkit-scrollbar {
  width: 6px;
}

.jb-edit-modal-body.jb-scroll::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, var(--jb-pink-400), var(--jb-fuchsia));
  border-radius: 6px;
}

/* ===== فرم ===== */
.jb-edit-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

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
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.jb-form-field-full {
  grid-column: 1 / -1;
}

/* لیبل */
.jb-form-label {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  font-weight: 800;
  color: var(--jb-text);
}

.jb-form-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 2px rgba(214, 51, 132, 0.18);
}

/* اینپوت با آیکون */
.jb-input-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.jb-input-icon {
  position: absolute;
  right: 14px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  align-items: center;
  justify-content: center;
  width: 30px;
  height: 30px;
  border-radius: 9px;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  color: var(--jb-fuchsia);
  pointer-events: none;
  z-index: 1;
  transition: all 0.25s ease;
}

.jb-form-input {
  width: 100%;
  padding: 12px 54px 12px 16px;
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

.jb-form-input:focus + .jb-input-icon,
.jb-input-wrap:focus-within .jb-input-icon {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.35);
}

/* ورودی تاریخ */
.jb-date-input {
  padding-left: 16px;
}

.jb-date-input::-webkit-calendar-picker-indicator {
  opacity: 0;
  display: none;
  position: absolute;
  right: 0;
  left: 0;
  top: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.jb-date-input {
  appearance: none;
  -webkit-appearance: none;
}

/* جداکننده بخش */
.jb-form-section-divider {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 8px 0;
}

.jb-form-section-divider::before,
.jb-form-section-divider::after {
  content: "";
  flex: 1;
  height: 0;
  border-top: 1.5px dashed var(--jb-pink-200);
}

.jb-form-section-text {
  font-size: 0.75rem;
  font-weight: 800;
  color: var(--jb-fuchsia);
  padding: 4px 14px;
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border-radius: 999px;
  border: 1.5px solid var(--jb-pink-200);
  white-space: nowrap;
}

/* ===== فوتر ===== */
.jb-edit-modal-footer {
  flex-shrink: 0;
  padding: 16px 20px 20px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  border-top: 1.5px dashed var(--jb-pink-300);
}

.jb-edit-save-btn {
  width: 100%;
  justify-content: center;
  padding: 14px 24px !important;
  font-size: 0.92rem !important;
}

.jb-edit-save-btn:disabled {
  opacity: 0.7;
  cursor: wait;
}

.jb-edit-save-inner {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.jb-spinner {
  width: 18px;
  height: 18px;
  animation: jbSpin 0.9s linear infinite;
}

@keyframes jbSpin {
  to { transform: rotate(360deg); }
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-edit-modal {
    max-height: 95vh;
    border-radius: 22px;
  }

  .jb-edit-modal-header {
    padding: 16px;
  }

  .jb-edit-modal-body {
    padding: 16px;
  }

  .jb-edit-modal-footer {
    padding: 14px 16px 18px;
  }

  .jb-edit-modal-title {
    font-size: 0.95rem;
  }

  .jb-form-input {
    padding: 11px 48px 11px 14px;
    font-size: 0.82rem;
  }

  .jb-input-icon {
    width: 26px;
    height: 26px;
    right: 12px;
  }
}
</style>