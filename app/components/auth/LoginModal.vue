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
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>

        <!-- بج تزئینی بالا -->
        <div class="jb-modal-badge">
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
              d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
          </svg>
        </div>

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
              class="jb-btn-linear jb-modal-submit"
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
              class="jb-btn-linear jb-modal-submit jb-modal-submit-half"
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
              class="jb-btn-linear jb-modal-submit"
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
              class="jb-btn-linear jb-modal-submit"
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
   مودال ورود/ثبت‌نام - تم پودری
   ========================================================= */

.jb-modal-overlay {
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

/* ===== باکس مودال ===== */
.jb-modal-box {
  position: relative;
  width: 100%;
  max-width: 420px;
  padding: 44px 28px 32px;
  background: #fff;
  border: 2px solid var(--jb-pink-300);
  border-radius: 28px;
  box-shadow:
    0 25px 60px rgba(214, 51, 132, 0.3),
    0 10px 25px rgba(255, 122, 160, 0.2);
  animation: jbModalIn 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes jbModalIn {
  0% {
    opacity: 0;
    transform: translateY(30px) scale(0.95);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* ===== دکمه بستن ===== */
.jb-modal-close {
  position: absolute;
  top: 16px;
  left: 16px;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 11px;
  background: var(--jb-pink-50);
  border: 1.5px solid var(--jb-pink-200);
  color: var(--jb-text-soft);
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-modal-close:hover {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  transform: rotate(90deg);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.35);
}

/* ===== بج تزئینی بالا ===== */
.jb-modal-badge {
  position: absolute;
  top: -28px;
  left: 50%;
  transform: translateX(-50%);
  width: 58px;
  height: 58px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 20px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border: 3px solid #fff;
  box-shadow: 0 10px 25px rgba(214, 51, 132, 0.45);
  animation: jbBadgeFloat 3s ease-in-out infinite;
}

@keyframes jbBadgeFloat {
  0%, 100% { transform: translateX(-50%) translateY(0); }
  50%      { transform: translateX(-50%) translateY(-6px); }
}

/* ===== هدر ===== */
.jb-modal-header {
  margin-bottom: 24px;
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
  font-size: 1.3rem;
  font-weight: 900;
  color: var(--jb-text);
  letter-spacing: -0.3px;
  line-height: 1.4;
}

.jb-modal-desc {
  font-size: 0.85rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  line-height: 1.8;
  margin-top: 4px;
}

.jb-modal-mobile {
  color: var(--jb-fuchsia);
  font-weight: 900;
  direction: ltr;
  display: inline-block;
}

/* دکمه ویرایش شماره */
.jb-modal-edit-btn {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 5px 12px;
  background: var(--jb-pink-50);
  border: 1.5px solid var(--jb-pink-200);
  border-radius: 999px;
  color: var(--jb-fuchsia);
  font-size: 0.72rem;
  font-weight: 800;
  cursor: pointer;
  transition: all 0.25s ease;
  font-family: inherit;
  white-space: nowrap;
}

.jb-modal-edit-btn:hover {
  background: var(--jb-pink-100);
  border-color: var(--jb-fuchsia);
  transform: translateY(-1px);
}

/* ===== فرم ===== */
.jb-modal-form {
  display: flex;
  flex-direction: column;
  gap: 14px;
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
  transition: all 0.25s ease;
  z-index: 1;
}

.jb-modal-input {
  width: 100%;
  padding: 14px 54px 14px 16px;
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

.jb-modal-input::placeholder {
  color: var(--jb-text-muted);
  font-weight: 500;
}

.jb-modal-input:focus {
  border-color: var(--jb-fuchsia);
  box-shadow: 0 0 0 4px rgba(214, 51, 132, 0.15);
}

.jb-modal-input:focus + .jb-input-icon,
.jb-input-wrap:focus-within .jb-input-icon {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.35);
}

.jb-modal-input:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* ===== دکمه اصلی ===== */
.jb-modal-submit {
  width: 100%;
  justify-content: center;
  padding: 14px 24px !important;
  font-size: 0.92rem !important;
}

.jb-modal-submit:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none !important;
  box-shadow: 0 4px 14px rgba(214, 51, 132, 0.25) !important;
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
  width: 20px;
  height: 20px;
  animation: jbSpin 0.9s linear infinite;
}

@keyframes jbSpin {
  to { transform: rotate(360deg); }
}

/* ===== OTP ===== */
.jb-modal-otp-wrap {
  display: flex;
  justify-content: center;
  margin: 8px 0 20px;
}

.jb-modal-otp-actions {
  display: flex;
  gap: 10px;
}

/* دکمه ثانویه */
.jb-modal-secondary-btn {
  flex: 1;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 13px 20px;
  background: #fff;
  border: 2px solid var(--jb-pink-300);
  border-radius: 14px;
  color: var(--jb-fuchsia);
  font-size: 0.85rem;
  font-weight: 800;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.25s ease;
}

.jb-modal-secondary-btn:hover:not(:disabled) {
  background: var(--jb-pink-100);
  border-color: var(--jb-fuchsia);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(214, 51, 132, 0.2);
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
    padding: 40px 20px 24px;
    border-radius: 24px;
  }

  .jb-modal-badge {
    width: 52px;
    height: 52px;
    top: -26px;
    border-radius: 17px;
  }

  .jb-modal-badge svg {
    width: 20px;
    height: 20px;
  }

  .jb-modal-title {
    font-size: 1.15rem;
  }

  .jb-modal-desc {
    font-size: 0.8rem;
  }

  .jb-modal-input {
    padding: 12px 48px 12px 14px;
    font-size: 0.82rem;
  }

  .jb-input-icon {
    width: 26px;
    height: 26px;
    right: 12px;
  }

  .jb-modal-otp-actions {
    flex-direction: column;
  }
}
</style>