<!-- app/components/about/FAQSection.vue -->
<template>
  <section class="jb-faq-section">
    <div class="container mx-auto px-4 md:px-8 lg:px-12 xl:px-28">
      <!-- تیتر -->
      <div class="jb-faq-header">
        <h2 class="jb-faq-title">
          <span class="jb-faq-icon">❓</span>
          سوالات <span class="jb-faq-title-strong">متداول</span>
        </h2>
        <div class="jb-faq-line"></div>
        <p class="jb-faq-subtitle">
          پاسخ پرتکرارترین سوالات شما درباره‌ی خرید از فروشگاه دکتر اسماعیلی
        </p>
      </div>

      <!-- لیست سوالات -->
      <ul class="jb-faq-list">
        <li
          v-for="(item, idx) in faqs"
          :key="item.id + '-' + idx"
          class="jb-faq-item"
          :class="{ 'is-active': activeId === item.id }"
        >
          <button
            class="jb-faq-btn"
            :aria-expanded="activeId === item.id"
            @click="toggle(item.id)"
          >
            <!-- بج شماره -->
            <span class="jb-faq-num">{{ String(idx + 1).padStart(2, "0") }}</span>

            <!-- سوال -->
            <span class="jb-faq-question">{{ item.question }}</span>

            <!-- فلش -->
            <span class="jb-faq-arrow" :class="{ 'is-open': activeId === item.id }">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M19 9l-7 7-7-7" />
              </svg>
            </span>
          </button>

          <!-- جواب -->
          <transition name="faq-slide">
            <div v-if="activeId === item.id && item.answer" class="jb-faq-answer-wrap">
              <div class="jb-faq-dashed"></div>
              <p class="jb-faq-answer">{{ item.answer }}</p>
            </div>
          </transition>
        </li>
      </ul>
    </div>
  </section>
</template>

<script setup>
import { ref } from "vue";

const faqs = ref([
  {
    id: 1,
    question: "چگونه می‌توانم سفارش خود را ثبت کنم؟",
    answer: "برای ثبت سفارش کافیه وارد سایت فروشگاه دکتر اسماعیلی بشید و اجناس مورد نظر به سبدخرید اضافه کنید و‌پرداخت انجام بدید🌹",
  },
  {
    id: 2,
    question: "آیا می‌توان بدون ثبت‌نام در سایت یا داشتن حساب کاربری، سفارش خود را ثبت نمایم؟",
    answer: "شما به عنوان کاربر مهمان میتونید از سایت دیدن کنید کارایی که خوشتون اومده را به سبدخرید اضافه کنید اما در نهایت در مرحله پرداخت حتما باید ثبت نام انجام بدید✅",
  },
  {
    id: 3,
    question: "چگونه می‌توانم سفارشات خود را پیگیری کنم و از وضعیت آن‌ها مطلع شوم؟",
    answer: "برای پیگیری سفارشات به پنل کاربری خودتون مراجعه کنید و در کل بعد از حداقل ۴۸ و در مواقع شلوغ ۷۲ ساعت طول میکشه کدرهگیری براتون پیامک بشه و شما با داشتن کدرهگیری میتوانید در سایت اداره پست از وضعیت بسته خود مطلع شوید",
  },
  {
    id: 4,
    question: "پس از اینکه سفارشم را ثبت کردم، چقدر طول می‌کشد به دستم برسد؟",
    answer: "حداقل بین ۴۸-۷۲ ساعت کاری طول میکشه بسته شما ارسال بشه و کدرهگیری براتون پیامک بشه و بعد از این مرحله به پست مربوط میشه که ممکنه ۲ روز تا ۱ هفته طول بکشه✅",
  },
  {
    id: 5,
    question: "امکان ارسال سفارش به یک شهر دیگر وجود دارد؟",
    answer: "به همه نقاط کشور ما ارسال خواهیم داشت فرقی نداره کدوم شهر یا روستا باشید🙏",
  },
  {
    id: 6,
    question: "تحویل حضوری",
    answer: "با توجه به حجم سفارشات روزانه، تحویل سفارشات به صورت حضوری لطفاً بعد از گذشت ۴۸ تا ۷۲ ساعت از ثبت سفارش برای دریافت آن مراجعه نمایید.",
  },
  {
    id: 7,
    question: "اگر مشکلی در خرید داشتیم چطور میتونیم با شما در ارتباط باشیم؟",
    answer: "برای ارتباط با پشتیبانی کافیه در تایم کاری ۱۰ صبح تا ۵ عصر تماس برقرار کنید: 09375015769 یا از طریق تلگرام و روبیکا برامون پیام بزارید🙏",
  },
]);

const activeId = ref(null);

function toggle(id) {
  activeId.value = activeId.value === id ? null : id;
}
</script>

<style scoped>
/* =========================================================
   سکشن سوالات متداول - تم پودری
   ========================================================= */

.jb-faq-section {
  padding: 48px 0 56px;
  position: relative;
}

/* ===== هدر ===== */
.jb-faq-header {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 12px;
  margin-bottom: 32px;
}

.jb-faq-title {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  font-size: 1.2rem;
  font-weight: 900;
  color: var(--jb-text);
  letter-spacing: -0.3px;
}

.jb-faq-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  border-radius: 12px;
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  font-size: 1rem;
  box-shadow: 0 6px 18px rgba(214, 51, 132, 0.35);
  border: 2px solid #fff;
}

.jb-faq-title-strong {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 900;
}

.jb-faq-line {
  width: 72px;
  height: 3px;
  border-radius: 999px;
  background: linear-gradient(
    90deg,
    var(--jb-pink-400),
    var(--jb-fuchsia),
    var(--jb-pink-400)
  );
  box-shadow: 0 2px 8px rgba(214, 51, 132, 0.25);
}

.jb-faq-subtitle {
  font-size: 0.88rem;
  color: var(--jb-text-muted);
  font-weight: 500;
  line-height: 1.8;
}

/* ===== لیست ===== */
.jb-faq-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

/* ===== آیتم ===== */
.jb-faq-item {
  background: #ffffff;
  border: 2px solid var(--jb-pink-200);
  border-radius: 20px;
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 4px 14px rgba(214, 51, 132, 0.06);
}

.jb-faq-item:hover {
  border-color: var(--jb-pink-400);
  box-shadow: 0 8px 22px rgba(214, 51, 132, 0.14);
  transform: translateY(-2px);
}

.jb-faq-item.is-active {
  border-color: var(--jb-fuchsia);
  box-shadow: 0 12px 28px rgba(214, 51, 132, 0.22);
  background: linear-gradient(180deg, #fff 0%, #fff8fb 100%);
}

/* ===== دکمه سوال ===== */
.jb-faq-btn {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 16px 18px;
  background: transparent;
  border: none;
  cursor: pointer;
  text-align: right;
  font-family: inherit;
  transition: all 0.25s ease;
}

/* بج شماره */
.jb-faq-num {
  flex-shrink: 0;
  width: 38px;
  height: 38px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  background: var(--jb-pink-100);
  color: var(--jb-pink-700);
  font-size: 0.8rem;
  font-weight: 900;
  letter-spacing: 0.5px;
  transition: all 0.3s ease;
  border: 1.5px solid var(--jb-pink-200);
}

.jb-faq-item.is-active .jb-faq-num {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  border-color: transparent;
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.4);
}

/* سوال */
.jb-faq-question {
  flex: 1;
  font-size: 0.92rem;
  font-weight: 700;
  color: var(--jb-text);
  line-height: 1.7;
  transition: color 0.25s ease;
  text-align: right;
}

.jb-faq-item.is-active .jb-faq-question {
  color: var(--jb-fuchsia);
  font-weight: 800;
}

/* فلش */
.jb-faq-arrow {
  flex-shrink: 0;
  width: 34px;
  height: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  background: var(--jb-pink-100);
  color: var(--jb-pink-600);
  transition: all 0.3s ease;
}

.jb-faq-arrow.is-open {
  background: linear-gradient(135deg, var(--jb-pink-500), var(--jb-fuchsia));
  color: #fff;
  transform: rotate(180deg);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.4);
}

/* ===== جواب ===== */
.jb-faq-answer-wrap {
  padding: 0 18px 18px;
  overflow: hidden;
}

.jb-faq-dashed {
  height: 0;
  border-top: 1.5px dashed var(--jb-pink-300);
  margin-bottom: 14px;
  position: relative;
}

.jb-faq-dashed::before,
.jb-faq-dashed::after {
  content: "";
  position: absolute;
  top: -4px;
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--jb-pink-400);
  box-shadow: 0 0 0 2px #fff;
}

.jb-faq-dashed::before { right: -3px; }
.jb-faq-dashed::after  { left: -3px; }

.jb-faq-answer {
  font-size: 0.86rem;
  line-height: 2;
  color: var(--jb-text-soft);
  font-weight: 500;
  padding-right: 52px;
  text-align: right;
}

/* ===== انیمیشن باز شدن ===== */
.faq-slide-enter-active,
.faq-slide-leave-active {
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  max-height: 500px;
}

.faq-slide-enter-from,
.faq-slide-leave-to {
  opacity: 0;
  max-height: 0;
  padding-top: 0;
  padding-bottom: 0;
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 768px) {
  .jb-faq-section {
    padding: 32px 0 40px;
  }

  .jb-faq-title {
    font-size: 1.05rem;
    gap: 8px;
  }

  .jb-faq-icon {
    width: 32px;
    height: 32px;
    font-size: 0.85rem;
  }

  .jb-faq-line {
    width: 60px;
  }

  .jb-faq-subtitle {
    font-size: 0.8rem;
  }

  .jb-faq-btn {
    padding: 14px 14px;
    gap: 10px;
  }

  .jb-faq-num {
    width: 32px;
    height: 32px;
    font-size: 0.72rem;
  }

  .jb-faq-question {
    font-size: 0.84rem;
    line-height: 1.6;
  }

  .jb-faq-arrow {
    width: 30px;
    height: 30px;
  }

  .jb-faq-answer-wrap {
    padding: 0 14px 14px;
  }

  .jb-faq-answer {
    font-size: 0.8rem;
    line-height: 1.95;
    padding-right: 42px;
  }
}

@media (max-width: 480px) {
  .jb-faq-question {
    font-size: 0.8rem;
  }

  .jb-faq-num {
    width: 28px;
    height: 28px;
    font-size: 0.68rem;
    border-radius: 8px;
  }

  .jb-faq-arrow {
    width: 26px;
    height: 26px;
  }

  .jb-faq-answer {
    padding-right: 38px;
  }
}
</style>