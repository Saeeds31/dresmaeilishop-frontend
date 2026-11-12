<template>
  <div class="jb-title-section">
    <!-- تیتر -->
    <h2 class="jb-title">
      <span class="jb-title-black">{{ titleBlack }}</span>
      <span class="jb-title-strong">{{ titleStrong }}</span>
    </h2>

    <!-- خط تزئینی -->
    <div class="jb-title-line"></div>

    <!-- توضیحات -->
    <p v-if="description" class="jb-title-desc">{{ description }}</p>
  </div>
</template>

<script>
export default {
  name: "HomeTitleSection",
  props: {
    titleBlack: { type: String, required: true },
    titleStrong: { type: String, required: true },
    description: { type: String, default: "" },
  },
};
</script>

<style scoped>
/* ===== کانتینر ===== */
.jb-title-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  text-align: center;
  padding: 8px 0;
}

/* ===== تیتر ===== */
.jb-title {
  font-size: 1.25rem;
  font-weight: 900;
  line-height: 1.5;
  display: inline-flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 6px;
  letter-spacing: -0.3px;
}

/* کلمه معمولی */
.jb-title-black {
  color: #1a202c;
}

/* کلمه تأکیدی با گرادیان سبز */
.jb-title-strong {
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 900;
  text-shadow: 0 2px 8px rgba(14, 90, 58, 0.15);
}

/* ===== خط تزئینی ===== */
.jb-title-line {
  width: 72px;
  height: 3px;
  border-radius: 999px;
  background: linear-gradient(
    90deg,
    transparent,
    #8cc63f,
    #0e5a3a,
    #8cc63f,
    transparent
  );
  box-shadow: 0 2px 8px rgba(14, 90, 58, 0.2);
}

/* ===== توضیحات ===== */
.jb-title-desc {
  color: #4a5568;
  font-size: 0.9rem;
  font-weight: 500;
  line-height: 1.8;
  max-width: 480px;
  padding: 0 8px;
}

/* ===== ریسپانسیو ===== */
@media (min-width: 768px) {
  .jb-title {
    font-size: 1.6rem;
    gap: 8px;
  }
  .jb-title-line {
    width: 90px;
    height: 4px;
  }
  .jb-title-desc {
    font-size: 0.95rem;
  }
}

@media (min-width: 1024px) {
  .jb-title {
    font-size: 1.9rem;
  }
  .jb-title-line {
    width: 110px;
  }
  .jb-title-desc {
    font-size: 1rem;
  }
}
</style>