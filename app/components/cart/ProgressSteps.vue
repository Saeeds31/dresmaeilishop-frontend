<!-- components/cart/ProgressSteps.vue -->
<template>
  <div class="jb-steps-wrapper">
    <div class="jb-steps-row">
      <!-- ============ Step 1: سبد خرید ============ -->
      <div class="jb-step" :class="{ 'is-active': currentStep >= 1 }">
        <div class="jb-step-icon-wrap">
          <div class="jb-step-icon">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
                d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
            </svg>
          </div>
        </div>
        <span class="jb-step-label">سبد خرید شما</span>
      </div>

      <!-- خط اتصال ۱ -->
      <div class="jb-step-line" :class="{ 'is-filled': currentStep >= 2 }">
        <span class="jb-step-line-dot"></span>
      </div>

      <!-- ============ Step 2: اطلاعات ارسال ============ -->
      <div class="jb-step" :class="{ 'is-active': currentStep >= 2 }">
        <div class="jb-step-icon-wrap">
          <div class="jb-step-icon">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
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
      <div class="jb-step-line" :class="{ 'is-filled': currentStep > 2 }">
        <span class="jb-step-line-dot"></span>
      </div>

      <!-- ============ Step 3: اطلاعات پرداخت ============ -->
      <div class="jb-step" :class="{ 'is-active': currentStep >= 3 }">
        <div class="jb-step-icon-wrap">
          <div class="jb-step-icon">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
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
   مراحل خرید - تم پودری
   ========================================================= */

.jb-steps-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 32px;
  padding: 20px 16px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 2px solid var(--jb-pink-200);
  border-radius: 22px;
  box-shadow: 0 6px 18px rgba(214, 51, 132, 0.08);
}

.jb-steps-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  flex-wrap: wrap;
  width: 100%;
}

/* ===== استپ ===== */
.jb-step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  transition: all 0.35s ease;
  min-width: 78px;
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
  width: 46px;
  height: 46px;
  border-radius: 16px;
  background: #fff;
  color: var(--jb-text-muted);
  border: 2px solid var(--jb-pink-200);
  transition: all 0.35s ease;
  box-shadow: 0 4px 12px rgba(214, 51, 132, 0.08);
}

.jb-step:hover .jb-step-icon {
  transform: translateY(-2px);
  border-color: var(--jb-pink-400);
}

/* استپ فعال */
.jb-step.is-active .jb-step-icon {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.45);
  animation: jbStepPulse 2.5s ease-in-out infinite;
}

@keyframes jbStepPulse {
  0%, 100% {
    box-shadow: 0 8px 22px rgba(214, 51, 132, 0.45);
  }
  50% {
    box-shadow: 0 10px 28px rgba(214, 51, 132, 0.65);
  }
}

/* ===== لیبل استپ ===== */
.jb-step-label {
  font-size: 0.72rem;
  font-weight: 700;
  color: var(--jb-text-muted);
  text-align: center;
  transition: all 0.35s ease;
  white-space: nowrap;
}

.jb-step.is-active .jb-step-label {
  color: var(--jb-fuchsia);
  font-weight: 900;
  font-size: 0.78rem;
}

/* ===== خط اتصال ===== */
.jb-step-line {
  position: relative;
  flex: 1;
  min-width: 30px;
  max-width: 100px;
  height: 3px;
  background: linear-gradient(90deg, var(--jb-pink-100), var(--jb-pink-200));
  border-radius: 999px;
  margin: 0 4px;
  transition: all 0.4s ease;
  align-self: flex-start;
  margin-top: 22px;
}

/* نقطه وسط خط */
.jb-step-line-dot {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--jb-pink-200);
  border: 2px solid #fff;
  box-shadow: 0 2px 6px rgba(214, 51, 132, 0.2);
  transition: all 0.35s ease;
}

/* خط پرشده */
.jb-step-line.is-filled {
  background: linear-gradient(90deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 2px 8px rgba(214, 51, 132, 0.35);
}

.jb-step-line.is-filled .jb-step-line-dot {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  box-shadow: 0 0 0 4px rgba(214, 51, 132, 0.15);
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 640px) {
  .jb-steps-wrapper {
    padding: 16px 8px;
    margin-bottom: 22px;
  }

  .jb-steps-row {
    gap: 4px;
  }

  .jb-step {
    min-width: 60px;
    gap: 6px;
  }

  .jb-step-icon {
    width: 40px;
    height: 40px;
    border-radius: 13px;
  }

  .jb-step-icon svg {
    width: 18px;
    height: 18px;
  }

  .jb-step-label {
    font-size: 0.62rem;
    white-space: normal;
    line-height: 1.3;
  }

  .jb-step.is-active .jb-step-label {
    font-size: 0.66rem;
  }

  .jb-step-line {
    min-width: 16px;
    max-width: 40px;
    margin-top: 19px;
    height: 2.5px;
  }

  .jb-step-line-dot {
    width: 6px;
    height: 6px;
  }
}
</style>