<template>
  <Teleport to="body">
    <div
      v-if="loginModal"
      class="jb-modal-overlay"
      @click.self="closeModal"
    >
      <div class="jb-modal-box" @click.stop>
        <!-- دکمه بستن -->
        <button class="jb-modal-close" @click="closeModal" aria-label="بستن">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>

        <!-- ============================================================
             مرحله ۱: موبایل
        ============================================================ -->
        <template v-if="step === 'mobile'">
          <div class="jb-modal-header">
            <h3 class="jb-modal-title">ورود یا عضویت</h3>
            <p class="jb-modal-desc">لطفا شماره موبایل خود را وارد کنید</p>
          </div>

          <form @submit.prevent="handleMobileSubmit" class="jb-modal-form">
            <div class="jb-input-wrap">
              <span class="jb-input-icon">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z" />
                </svg>
              </span>
              <input
                v-model="mobile"
                type="tel"
                placeholder="۰۹۱۲۳۴۵۶۷۸۹"
                maxlength="11"
                class="jb-modal-input"
                :disabled="btnLoader"
                dir="ltr"
              />
            </div>

            <button
              type="submit"
              :disabled="btnLoader || !isValidMobile"
              class="jb-modal-submit"
            >
              <span v-if="!btnLoader">ادامه</span>
              <span v-else class="jb-modal-spinner-wrap">
                <svg class="jb-modal-spinner" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
                  <path class="opacity-75" fill="currentColor"
                    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
                </svg>
                در حال بررسی...
              </span>
            </button>
          </form>
        </template>

        <!-- ============================================================
             مرحله ۲: OTP
        ============================================================ -->
        <template v-else-if="step === 'otp'">
          <div class="jb-modal-header">
            <div class="jb-modal-header-row">
              <h3 class="jb-modal-title">کد تأیید</h3>
              <button @click="step = 'mobile'" type="button" class="jb-modal-edit-btn">
                <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
                </svg>
                ویرایش شماره
              </button>
            </div>
            <p class="jb-modal-desc">
              کد ارسال‌شده به
              <strong class="jb-modal-mobile">{{ mobile }}</strong>
              را وارد کنید
            </p>
          </div>

          <div class="jb-modal-otp-wrap">
            <SharedOtpInput v-model:code="otp" :length="6" class="my-2 justify-center" />
          </div>

          <div class="jb-modal-otp-actions">
            <button
              @click="verifyOtpCode"
              :disabled="btnLoader || otp.length !== 6"
              class="jb-modal-submit jb-modal-submit-half"
            >
              <span v-if="!btnLoader">تأیید</span>
              <svg v-else class="jb-modal-spinner" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
                <path class="opacity-75" fill="currentColor"
                  d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
              </svg>
            </button>

            <button
              type="button"
              @click="resendOtp"
              :disabled="timer > 0"
              class="jb-modal-secondary-btn"
            >
              <svg v-if="timer === 0" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
              </svg>
              {{ timer > 0 ? `${timer} ثانیه` : "ارسال مجدد" }}
            </button>
          </div>
        </template>

        <!-- ============================================================
             مرحله ۳: ثبت‌نام
        ============================================================ -->
        <template v-else-if="step === 'register'">
          <div class="jb-modal-header">
            <h3 class="jb-modal-title">تکمیل ثبت‌نام</h3>
            <p class="jb-modal-desc">لطفا اطلاعات خود را وارد کنید</p>
          </div>

          <form @submit.prevent="completeRegistration" class="jb-modal-form">
            <div class="jb-input-wrap">
              <span class="jb-input-icon">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
                </svg>
              </span>
              <input
                v-model="fullName"
                type="text"
                placeholder="نام و نام خانوادگی"
                class="jb-modal-input"
              />
            </div>

            <div class="jb-input-wrap">
              <span class="jb-input-icon">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                </svg>
              </span>
              <input
                v-model="password"
                type="password"
                placeholder="رمز عبور (حداقل ۶ کاراکتر)"
                class="jb-modal-input"
              />
            </div>

            <button
              type="submit"
              :disabled="btnLoader || !isRegisterValid"
              class="jb-modal-submit"
            >
              <span v-if="!btnLoader">ثبت‌نام</span>
              <span v-else class="jb-modal-spinner-wrap">
                <svg class="jb-modal-spinner" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
                  <path class="opacity-75" fill="currentColor"
                    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
                </svg>
                در حال ثبت‌نام...
              </span>
            </button>
          </form>
        </template>

        <!-- ============================================================
             مرحله ۴: رمز عبور
        ============================================================ -->
        <template v-else-if="step === 'password'">
          <div class="jb-modal-header">
            <h3 class="jb-modal-title">ورود با رمز عبور</h3>
            <p class="jb-modal-desc">
              شماره:
              <strong class="jb-modal-mobile">{{ formatMobile(mobile) }}</strong>
            </p>
          </div>

          <form @submit.prevent="loginWithPass" class="jb-modal-form">
            <div class="jb-input-wrap">
              <span class="jb-input-icon">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                    d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                </svg>
              </span>
              <input
                v-model="password"
                type="password"
                placeholder="رمز عبور"
                class="jb-modal-input"
              />
            </div>

            <button
              type="submit"
              :disabled="btnLoader || password.length < 6"
              class="jb-modal-submit"
            >
              <span v-if="!btnLoader">ورود</span>
              <span v-else class="jb-modal-spinner-wrap">
                <svg class="jb-modal-spinner" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
                  <path class="opacity-75" fill="currentColor"
                    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
                </svg>
                در حال ورود...
              </span>
            </button>

            <button type="button" @click="switchToOtp" class="jb-modal-secondary-btn">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
              </svg>
              ورود با کد یک‌بار مصرف
            </button>
          </form>
        </template>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { ref, computed, onUnmounted } from "vue";
import { useAuth } from "~/composables/useAuth";
import { useApiAuth } from "~/composables/useAuth";
import { useCartStore } from "~/store/cart";
import { toast } from "vue3-toastify";

const { checkMobile, sendOtp, verifyOtp, loginWithPassword, register } =
  useApiAuth();
const { loginModal } = useAuth();

const step = ref("mobile");
const mobile = ref("");
const otp = ref("");
const password = ref("");
const fullName = ref("");
const btnLoader = ref(false);
const timer = ref(0);
let timerInterval = null;

const isValidMobile = computed(() => /^09\d{9}$/.test(mobile.value));
const isRegisterValid = computed(
  () => fullName.value.length >= 3 && password.value.length >= 6
);

const formatMobile = (num) => num.replace(/(\d{4})(\d{3})(\d{4})/, "$1$2$3");

const closeModal = () => {
  loginModal.value = false;
  resetForm();
};

const resetForm = () => {
  step.value = "mobile";
  mobile.value = "";
  otp.value = "";
  password.value = "";
  fullName.value = "";
  btnLoader.value = false;
  clearTimer();
};

const clearTimer = () => {
  if (timerInterval) clearInterval(timerInterval);
  timer.value = 0;
};

const startTimer = () => {
  clearTimer();
  timer.value = 60;
  timerInterval = setInterval(() => {
    timer.value--;
    if (timer.value <= 0) clearInterval(timerInterval);
  }, 1000);
};

async function handleMobileSubmit() {
  if (!isValidMobile.value) return toast.error("شماره موبایل معتبر نیست");

  btnLoader.value = true;
  try {
    const res = await checkMobile(mobile.value);

    if (res.data.status === "login") {
      step.value = "password";
    } else {
      await sendOtp(mobile.value);
      step.value = "otp";
      startTimer();
    }
  } catch (err) {
    toast.error(err?.response?.data?.message || "خطایی رخ داد");
  } finally {
    btnLoader.value = false;
  }
}

async function verifyOtpCode() {
  btnLoader.value = true;
  try {
    const res = await verifyOtp(mobile.value, otp.value);
    if (res.data.status === "need_register") {
      step.value = "register";
    } else {
      await handleSuccessfulLogin(res.data);
    }
  } catch (err) {
    toast.error(err?.response?.data?.message || "کد نامعتبر است");
  } finally {
    btnLoader.value = false;
  }
}

async function completeRegistration() {
  btnLoader.value = true;
  try {
    const res = await register(mobile.value, password.value, fullName.value);
    await handleSuccessfulLogin(res.data);
  } catch (err) {
    toast.error(err?.response?.data?.message || "خطا در ثبت‌نام");
  } finally {
    btnLoader.value = false;
  }
}

async function loginWithPass() {
  btnLoader.value = true;
  try {
    const res = await loginWithPassword(mobile.value, password.value);
    await handleSuccessfulLogin(res.data);
  } catch (err) {
    toast.error(err?.response?.data?.message || "رمز عبور اشتباه است");
  } finally {
    btnLoader.value = false;
  }
}

async function resendOtp() {
  if (timer.value > 0) return;
  try {
    await sendOtp(mobile.value);
    startTimer();
    toast.success("کد یکبار مصرف ارسال شد");
  } catch (err) {
    toast.error("خطا در ارسال مجدد کد");
  }
}

function switchToOtp() {
  step.value = "otp";
  resendOtp();
}

const router = useRouter();

async function handleSuccessfulLogin(data) {
  const { $api } = useNuxtApp();

  localStorage.setItem("auth-token", data.token);
  localStorage.setItem("user", JSON.stringify(data.user));

  if ($api.defaults?.headers?.common) {
    $api.defaults.headers.common.Authorization = `Bearer ${data.token}`;
  }

  toast.success(`خوش آمدید ${data.user.full_name || data.user.mobile}`);
  closeModal();
  window.location.href = localStorage.getItem("lastPage") ?? "/";
}

onUnmounted(() => clearTimer());
</script>

<style scoped>
/* =========================================================
   مودال ورود/ثبت‌نام - مینیمال سبز
   ========================================================= */

.jb-modal-overlay {
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

/* ===== باکس مودال ===== */
.jb-modal-box {
  position: relative;
  width: 100%;
  max-width: 400px;
  padding: 32px 24px 24px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 10px;
  box-shadow: 0 10px 40px rgba(14, 90, 58, 0.15);
  animation: jbModalIn 0.3s ease;
}

@keyframes jbModalIn {
  0% {
    opacity: 0;
    transform: translateY(20px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

/* ===== دکمه بستن ===== */
.jb-modal-close {
  position: absolute;
  top: 12px;
  left: 12px;
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
}

.jb-modal-close:hover {
  background: #0e5a3a;
  color: #fff;
  border-color: transparent;
}

/* ===== هدر ===== */
.jb-modal-header {
  margin-bottom: 20px;
  text-align: center;
}

.jb-modal-header-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 6px;
  flex-wrap: wrap;
}

.jb-modal-title {
  font-size: 1.15rem;
  font-weight: 800;
  color: #1a202c;
  letter-spacing: -0.3px;
  line-height: 1.4;
}

.jb-modal-desc {
  font-size: 0.82rem;
  color: #718096;
  font-weight: 500;
  line-height: 1.8;
  margin-top: 4px;
}

.jb-modal-mobile {
  color: #0e5a3a;
  font-weight: 800;
  direction: ltr;
  display: inline-block;
}

/* دکمه ویرایش شماره */
.jb-modal-edit-btn {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 10px;
  background: #f7fafc;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 4px;
  color: #0e5a3a;
  font-size: 0.7rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  font-family: inherit;
  white-space: nowrap;
}

.jb-modal-edit-btn:hover {
  background: #f1f7f3;
  border-color: rgba(14, 90, 58, 0.25);
}

/* ===== فرم ===== */
.jb-modal-form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

/* اینپوت با آیکون */
.jb-input-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.jb-input-icon {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 6px;
  background: #f7fafc;
  color: #0e5a3a;
  pointer-events: none;
  transition: all 0.2s ease;
  z-index: 1;
}

.jb-modal-input {
  width: 100%;
  padding: 12px 48px 12px 14px;
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

.jb-modal-input::placeholder {
  color: #a0aec0;
  font-weight: 500;
}

.jb-modal-input:focus {
  border-color: #0e5a3a;
  box-shadow: 0 0 0 3px rgba(14, 90, 58, 0.1);
}

.jb-input-wrap:focus-within .jb-input-icon {
  background: #0e5a3a;
  color: #fff;
}

.jb-modal-input:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* ===== دکمه اصلی ===== */
.jb-modal-submit {
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

.jb-modal-submit:hover:not(:disabled) {
  background: #0b472e;
}

.jb-modal-submit:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.jb-modal-submit-half {
  flex: 1;
}

/* اسپینر داخل دکمه */
.jb-modal-spinner-wrap {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.jb-modal-spinner {
  width: 18px;
  height: 18px;
  animation: jbSpin 0.9s linear infinite;
}

@keyframes jbSpin {
  to { transform: rotate(360deg); }
}

/* ===== OTP ===== */
.jb-modal-otp-wrap {
  display: flex;
  justify-content: center;
  margin: 8px 0 16px;
}

.jb-modal-otp-actions {
  display: flex;
  gap: 8px;
}

/* دکمه ثانویه */
.jb-modal-secondary-btn {
  flex: 1;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 10px 16px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.15);
  border-radius: 6px;
  color: #0e5a3a;
  font-size: 0.82rem;
  font-weight: 600;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.2s ease;
}

.jb-modal-secondary-btn:hover:not(:disabled) {
  background: #f7fafc;
  border-color: rgba(14, 90, 58, 0.3);
}

.jb-modal-secondary-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 480px) {
  .jb-modal-box {
    padding: 28px 18px 20px;
    border-radius: 8px;
  }

  .jb-modal-title {
    font-size: 1.05rem;
  }

  .jb-modal-desc {
    font-size: 0.78rem;
  }

  .jb-modal-input {
    padding: 11px 44px 11px 12px;
    font-size: 0.82rem;
  }

  .jb-input-icon {
    width: 26px;
    height: 26px;
    right: 10px;
  }

  .jb-modal-otp-actions {
    flex-direction: column;
  }
}
</style>