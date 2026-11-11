<template>
  <section class="jb-contact-form-section">
    <!-- ===== ستون چپ: کاراکتر سه‌بعدی ===== -->
    <div class="jb-form-side jb-form-side-character">
      <div class="jb-form-character-wrap">
        <img
          src="/contact/3d-girl.svg"
          class="jb-form-character"
          alt="پشتیبانی فروشگاه دکتر اسماعیلی"
        />
      </div>
    </div>

    <!-- ===== ستون راست: اطلاعات + فرم ===== -->
    <div class="jb-form-main">
      <!-- اطلاعات کنار فرم -->
      <div class="jb-form-info">
        <div class="jb-form-logo-wrap">
          <img
            src="~/assets/images/logo.png"
            alt="لوگو"
            class="jb-form-logo"
          />
        </div>

        <h3 class="jb-form-info-title">
          <span class="jb-title-dot"></span>
          صدای شما، ارزشمند است
        </h3>

        <ul class="jb-form-info-list">
          <li class="jb-form-info-item">
            <span class="jb-form-info-dot"></span>
            تیم فروشگاه دکتر اسماعیلی همیشه آماده شنیدن نظرات و پاسخگویی به سوالات
            شماست.
          </li>
          <li class="jb-form-info-item">
            <span class="jb-form-info-dot"></span>
            نظرات و پیشنهادهایتان را با ما به اشتراک بگذارید؛ هر بازخورد شما
            برای بهبود تجربه‌ی خرید ارزشمند است.
          </li>
          <li class="jb-form-info-item jb-info-highlight">
            <span class="jb-form-info-dot"></span>
            منتظر دیدگاه‌های گرم شما هستیم! 🌸
          </li>
        </ul>
      </div>

      <!-- فرم -->
      <div class="jb-form-wrap">
        <div class="jb-form-header">
          <h3 class="jb-form-title">
            <span class="jb-form-title-icon">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                  d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
              </svg>
            </span>
            فرم ارتباطی
          </h3>
          <p class="jb-form-subtitle">پیام خود را برای ما بنویسید</p>
        </div>

        <form @submit.prevent="onSubmit" class="jb-form">
          <div class="jb-form-row">
            <!-- نام -->
            <div class="jb-form-field">
              <label class="jb-form-label">
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
                  v-model="model.name"
                  type="text"
                  placeholder="سارا مهدوی"
                  class="jb-form-input"
                />
              </div>
            </div>

            <!-- شماره تماس -->
            <div class="jb-form-field">
              <label class="jb-form-label">
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
                  v-model="model.phone"
                  type="tel"
                  placeholder="۰۹۱۱ ۰۹۹ ۸۸۷۷"
                  class="jb-form-input"
                  dir="ltr"
                />
              </div>
            </div>
          </div>

          <!-- پیام -->
          <div class="jb-form-field">
            <label class="jb-form-label">
              <span class="jb-form-dot"></span>
              پیشنهاد و نظر شما
            </label>
            <textarea
              v-model="model.message"
              rows="5"
              placeholder="نظر یا پیشنهاد خود را با ما در میان بگذارید..."
              class="jb-form-input jb-form-textarea"
            ></textarea>
          </div>

          <!-- دکمه -->
          <div class="jb-form-actions">
            <button
              type="submit"
              :disabled="loading"
              class="jb-btn-linear jb-form-submit"
            >
              <span v-if="!loading" class="jb-form-submit-inner">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5"
                    d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" />
                </svg>
                ارسال پیام
              </span>
              <span v-else class="jb-form-submit-inner">
                <svg class="jb-spinner" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none" />
                  <path class="opacity-75" fill="currentColor"
                    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
                </svg>
                در حال ارسال...
              </span>
            </button>
          </div>
        </form>
      </div>
    </div>
  </section>
</template>

<script setup>
const props = defineProps({
  model: { type: Object, required: true },
  loading: { type: Boolean, required: true },
});

const emit = defineEmits(["submit"]);

const onSubmit = () => {
  emit("submit");
};
</script>

<style scoped>
/* =========================================================
   فرم تماس با ما - تم پودری
   ========================================================= */

.jb-contact-form-section {
  display: flex;
  flex-direction: column;
  gap: 32px;
  padding: 32px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid var(--jb-pink-200);
  border-radius: 28px;
  box-shadow: 0 12px 35px rgba(214, 51, 132, 0.12);
}

@media (min-width: 1024px) {
  .jb-contact-form-section {
    flex-direction: row;
    align-items: flex-end;
    padding: 40px;
    gap: 24px;
  }
}

/* ===== ستون کاراکتر ===== */
.jb-form-side-character {
  display: none;
}

@media (min-width: 1024px) {
  .jb-form-side-character {
    display: block;
    flex-shrink: 0;
    width: 22%;
  }
}

.jb-form-character-wrap {
  padding: 16px;
  border-radius: 24px;
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  border: 2px dashed var(--jb-pink-300);
  animation: jbCharFloat 4s ease-in-out infinite;
}

@keyframes jbCharFloat {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-8px); }
}

.jb-form-character {
  width: 100%;
  height: auto;
  object-fit: contain;
  mix-blend-mode: multiply;
}

/* ===== ستون اصلی ===== */
.jb-form-main {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
}

@media (min-width: 1024px) {
  .jb-form-main {
    flex-direction: row;
    align-items: flex-start;
    gap: 36px;
  }
}

/* ===== اطلاعات کنار فرم ===== */
.jb-form-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

@media (min-width: 1024px) {
  .jb-form-info {
    flex: 0 0 40%;
  }
}

.jb-form-logo-wrap {
  display: none;
}

@media (min-width: 1024px) {
  .jb-form-logo-wrap {
    display: block;
    padding: 12px;
    background: linear-gradient(135deg, #fff8fb, #ffeaf2);
    border: 2px solid var(--jb-pink-200);
    border-radius: 18px;
    width: fit-content;
  }
}

.jb-form-logo {
  width: 80px;
  height: auto;
  opacity: 0.9;
}

.jb-form-info-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 1rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-title-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
  flex-shrink: 0;
}

.jb-form-info-list {
  display: flex;
  flex-direction: column;
  gap: 14px;
  list-style: none;
  padding: 0;
  margin: 0;
}

.jb-form-info-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  font-size: 0.88rem;
  line-height: 2;
  color: var(--jb-text-soft);
  font-weight: 600;
}

.jb-form-info-dot {
  flex-shrink: 0;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 3px rgba(214, 51, 132, 0.18);
  margin-top: 0.7em;
}

.jb-info-highlight {
  color: var(--jb-fuchsia);
  font-weight: 800;
}

/* ===== فرم ===== */
.jb-form-wrap {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

@media (min-width: 1024px) {
  .jb-form-wrap {
    flex: 0 0 55%;
  }
}

.jb-form-header {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-bottom: 4px;
}

.jb-form-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 1.15rem;
  font-weight: 900;
  color: var(--jb-text);
}

.jb-form-title-icon {
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

.jb-form-subtitle {
  font-size: 0.82rem;
  color: var(--jb-text-muted);
  font-weight: 600;
  padding-right: 44px;
}

.jb-form {
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
  padding: 13px 54px 13px 16px;
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

.jb-input-wrap:focus-within .jb-input-icon {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.35);
}

.jb-form-textarea {
  padding: 14px 16px;
  resize: none;
  min-height: 130px;
  line-height: 1.9;
}

/* دکمه */
.jb-form-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 4px;
}

.jb-form-submit {
  padding: 13px 32px !important;
  font-size: 0.9rem !important;
}

.jb-form-submit:disabled {
  opacity: 0.7;
  cursor: wait;
}

.jb-form-submit-inner {
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

/* ===== ریسپانسیو ===== */
@media (max-width: 640px) {
  .jb-contact-form-section {
    padding: 20px;
    border-radius: 22px;
  }

  .jb-form-title {
    font-size: 1rem;
  }

  .jb-form-subtitle {
    padding-right: 40px;
  }

  .jb-form-input {
    padding: 12px 48px 12px 14px;
    font-size: 0.84rem;
  }

  .jb-input-icon {
    width: 26px;
    height: 26px;
    right: 12px;
  }

  .jb-form-submit {
    width: 100%;
    justify-content: center;
  }
}
</style>