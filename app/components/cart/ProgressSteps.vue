<!-- components/cart/ProgressSteps.vue -->
<template>
  <div class="jb-steps-wrapper">
    <div class="jb-steps-row">
      <!-- ============ Step 1: سبد خرید ============ -->
      <div class="jb-step" :class="{ 'is-active': currentStep >= 1 }">
        <div class="jb-step-icon-wrap">
          <div class="jb-step-icon">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
            </svg>
          </div>
        </div>
        <span class="jb-step-label">سبد خرید شما</span>
      </div>

      <!-- خط اتصال ۱ -->
      <div class="jb-step-line" :class="{ 'is-filled': currentStep >= 2 }"></div>

      <!-- ============ Step 2: اطلاعات ارسال ============ -->
      <div class="jb-step" :class="{ 'is-active': currentStep >= 2 }">
        <div class="jb-step-icon-wrap">
          <div class="jb-step-icon">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M17.657 16.657L13.414 20.9a2 2 0 01-2.828 0l-4.243-4.243a8 8 0 1111.314 0z" />
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
            </svg>
          </div>
        </div>
        <span class="jb-step-label">اطلاعات ارسال</span>
      </div>

      <!-- خط اتصال ۲ -->
      <div class="jb-step-line" :class="{ 'is-filled': currentStep > 2 }"></div>

      <!-- ============ Step 3: اطلاعات پرداخت ============ -->
      <div class="jb-step" :class="{ 'is-active': currentStep >= 3 }">
        <div class="jb-step-icon-wrap">
          <div class="jb-step-icon">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M3 10h18M7 15h1m4 0h1m-7 4h12a3 3 0 003-3V8a3 3 0 00-3-3H6a3 3 0 00-3 3v8a3 3 0 003 3z" />
            </svg>
          </div>
        </div>
        <span class="jb-step-label">اطلاعات پرداخت</span>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  currentStep: {
    type: Number,
    required: true,
    validator: (value) => value >= 1 && value <= 3,
  },
});
</script>

<style scoped>
/* =========================================================
   مراحل خرید - مینیمال سبز
   ========================================================= */

.jb-steps-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 24px;
  padding: 16px 20px;
  background: #fff;
  border: 1px solid rgba(14, 90, 58, 0.1);
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.04);
}

.jb-steps-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  flex-wrap: wrap;
  width: 100%;
}

/* ===== استپ ===== */
.jb-step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  transition: all 0.3s ease;
  min-width: 70px;
}

/* ===== آیکون استپ ===== */
.jb-step-icon-wrap {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.jb-step-icon {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 6px;
  background: #f7fafc;
  color: #a0aec0;
  border: 1px solid rgba(14, 90, 58, 0.08);
  transition: all 0.3s ease;
}

.jb-step:hover .jb-step-icon {
  border-color: rgba(14, 90, 58, 0.2);
}

/* استپ فعال */
.jb-step.is-active .jb-step-icon {
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
  border-color: transparent;
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.2);
}

/* ===== لیبل استپ ===== */
.jb-step-label {
  font-size: 0.7rem;
  font-weight: 600;
  color: #a0aec0;
  text-align: center;
  transition: all 0.3s ease;
  white-space: nowrap;
}

.jb-step.is-active .jb-step-label {
  color: #0e5a3a;
  font-weight: 800;
  font-size: 0.74rem;
}

/* ===== خط اتصال ===== */
.jb-step-line {
  position: relative;
  flex: 1;
  min-width: 30px;
  max-width: 100px;
  height: 2px;
  background: #e2e8f0;
  border-radius: 1px;
  margin: 0 4px;
  transition: all 0.4s ease;
  align-self: flex-start;
  margin-top: 19px;
}

/* خط پرشده */
.jb-step-line.is-filled {
  background: linear-gradient(90deg, #0e5a3a, #1a7a50);
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-steps-wrapper {
    padding: 14px 8px;
    margin-bottom: 18px;
  }

  .jb-steps-row {
    gap: 2px;
  }

  .jb-step {
    min-width: 58px;
    gap: 4px;
  }

  .jb-step-icon {
    width: 36px;
    height: 36px;
    border-radius: 6px;
  }

  .jb-step-icon svg {
    width: 16px;
    height: 16px;
  }

  .jb-step-label {
    font-size: 0.6rem;
    white-space: normal;
    line-height: 1.3;
  }

  .jb-step.is-active .jb-step-label {
    font-size: 0.64rem;
  }

  .jb-step-line {
    min-width: 16px;
    max-width: 40px;
    margin-top: 17px;
    height: 2px;
  }
}
</style>