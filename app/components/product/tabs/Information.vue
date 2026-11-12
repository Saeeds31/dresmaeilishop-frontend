
<script setup>
const props = defineProps({
  description: { type: String, default: "" },
});
</script>
<template>
  <div class="jb-description-wrapper">
    <!-- خط تزئینی بالای متن -->
    <div class="jb-desc-top-line">
      <span class="jb-desc-line"></span>
      <span class="jb-desc-icon">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2"
            d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
        </svg>
      </span>
      <span class="jb-desc-line"></span>
    </div>

    <!-- محتوای اصلی (از ادیتور) -->
    <div class="jb-description-content" v-html="props.description"></div>
  </div>
</template>

<style scoped>
/* =========================================================
   توضیحات محصول - تم سبز لوکس
   ========================================================= */

.jb-description-wrapper {
  position: relative;
  padding: 8px 0;
}

/* خط تزئینی بالای متن */
.jb-desc-top-line {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 28px;
}

.jb-desc-line {
  flex: 1;
  height: 2px;
  background: linear-gradient(90deg, transparent, rgba(140, 198, 63, 0.5), transparent);
  border-radius: 999px;
}

.jb-desc-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 14px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.3);
  border: 2px solid #fff;
  flex-shrink: 0;
}

/* =========================================================
   تایپوگرافی محتوای ادیتور
   ========================================================= */

.jb-description-content {
  font-size: 0.95rem;
  line-height: 2.15;
  color: #4a5568;
  font-weight: 500;
  word-break: break-word;
}

/* ===== پاراگراف‌ها ===== */
.jb-description-content :deep(p) {
  margin-bottom: 18px;
  text-align: justify;
}

.jb-description-content :deep(p:last-child) {
  margin-bottom: 0;
}

.jb-description-content :deep(p:first-of-type) {
  font-size: 1rem;
  font-weight: 600;
  color: #1a202c;
  line-height: 2.25;
}

/* ===== تیترها ===== */
.jb-description-content :deep(h1),
.jb-description-content :deep(h2),
.jb-description-content :deep(h3),
.jb-description-content :deep(h4),
.jb-description-content :deep(h5),
.jb-description-content :deep(h6) {
  position: relative;
  color: #1a202c;
  font-weight: 900;
  line-height: 1.6;
  margin-top: 32px;
  margin-bottom: 16px;
  padding-right: 18px;
  padding-bottom: 8px;
  letter-spacing: -0.2px;
}

.jb-description-content :deep(h1:first-child),
.jb-description-content :deep(h2:first-child),
.jb-description-content :deep(h3:first-child) {
  margin-top: 0;
}

.jb-description-content :deep(h1::before),
.jb-description-content :deep(h2::before),
.jb-description-content :deep(h3::before),
.jb-description-content :deep(h4::before),
.jb-description-content :deep(h5::before),
.jb-description-content :deep(h6::before) {
  content: "";
  position: absolute;
  right: 0;
  top: 0.15em;
  bottom: 0.4em;
  width: 5px;
  border-radius: 999px;
  background: linear-gradient(180deg, #0e5a3a, #8cc63f);
  box-shadow: 0 4px 12px rgba(14, 90, 58, 0.3);
}

.jb-description-content :deep(h1::after),
.jb-description-content :deep(h2::after),
.jb-description-content :deep(h3::after) {
  content: "";
  position: absolute;
  bottom: 0;
  right: 18px;
  left: 0;
  height: 2px;
  background: linear-gradient(90deg, rgba(140, 198, 63, 0.5), transparent);
  border-radius: 999px;
}

.jb-description-content :deep(h1) { font-size: 1.5rem; }
.jb-description-content :deep(h2) { font-size: 1.3rem; }
.jb-description-content :deep(h3) { font-size: 1.15rem; }
.jb-description-content :deep(h4),
.jb-description-content :deep(h5),
.jb-description-content :deep(h6) { font-size: 1rem; }

/* ===== لیست‌ها ===== */
.jb-description-content :deep(ul),
.jb-description-content :deep(ol) {
  margin: 16px 0 20px;
  padding-right: 22px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.jb-description-content :deep(ul li),
.jb-description-content :deep(ol li) {
  position: relative;
  line-height: 2;
  padding-right: 8px;
}

.jb-description-content :deep(ul) {
  list-style: none;
  padding-right: 8px;
}

.jb-description-content :deep(ul li) {
  padding-right: 26px;
}

.jb-description-content :deep(ul li::before) {
  content: "";
  position: absolute;
  right: 4px;
  top: 0.85em;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  box-shadow: 0 0 0 3px rgba(140, 198, 63, 0.15);
}

.jb-description-content :deep(ol) {
  counter-reset: jb-counter;
  list-style: none;
  padding-right: 8px;
}

.jb-description-content :deep(ol li) {
  counter-increment: jb-counter;
  padding-right: 42px;
}

.jb-description-content :deep(ol li::before) {
  content: counter(jb-counter);
  position: absolute;
  right: 0;
  top: 0.4em;
  width: 26px;
  height: 26px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: linear-gradient(135deg, #0e5a3a, #8cc63f);
  color: #fff;
  font-size: 0.72rem;
  font-weight: 900;
  box-shadow: 0 4px 10px rgba(14, 90, 58, 0.3);
  line-height: 1;
}

.jb-description-content :deep(li > ul),
.jb-description-content :deep(li > ol) {
  margin-top: 10px;
  margin-bottom: 4px;
}

/* ===== بولد و ایتالیک ===== */
.jb-description-content :deep(strong),
.jb-description-content :deep(b) {
  color: #0e5a3a;
  font-weight: 900;
}

.jb-description-content :deep(em),
.jb-description-content :deep(i) {
  color: #4a5568;
  font-style: italic;
}

/* ===== لینک‌ها ===== */
.jb-description-content :deep(a) {
  color: #0e5a3a;
  font-weight: 800;
  text-decoration: none;
  border-bottom: 2px dashed rgba(140, 198, 63, 0.6);
  transition: all 0.25s ease;
}

.jb-description-content :deep(a:hover) {
  color: #8cc63f;
  border-bottom-color: #0e5a3a;
}

/* ===== نقل‌قول ===== */
.jb-description-content :deep(blockquote) {
  position: relative;
  margin: 24px 0;
  padding: 18px 22px;
  background: linear-gradient(135deg, #f8fcf9, #e8f3ec);
  border-right: 4px solid transparent;
  border-image: linear-gradient(180deg, #0e5a3a, #8cc63f) 1;
  border-radius: 16px;
  color: #1a202c;
  font-weight: 700;
  font-style: italic;
  line-height: 2;
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.08);
}

.jb-description-content :deep(blockquote::before) {
  content: "❞";
  position: absolute;
  top: -12px;
  right: 16px;
  font-size: 2.2rem;
  color: #8cc63f;
  line-height: 1;
  background: #fff;
  padding: 0 8px;
  border-radius: 8px;
}

.jb-description-content :deep(blockquote p:last-child) {
  margin-bottom: 0;
}

/* ===== کد ===== */
.jb-description-content :deep(code) {
  background: linear-gradient(135deg, #f4f9f6, #e8f3ec);
  color: #0e5a3a;
  padding: 2px 10px;
  border-radius: 8px;
  font-family: "Vazirmatn", monospace;
  font-size: 0.85em;
  font-weight: 700;
  border: 1.5px solid rgba(14, 90, 58, 0.1);
}

.jb-description-content :deep(pre) {
  margin: 20px 0;
  padding: 16px 18px;
  background: linear-gradient(135deg, #1a202c, #2d3748);
  color: #e2e8f0;
  border-radius: 16px;
  overflow-x: auto;
  font-size: 0.85rem;
  line-height: 1.9;
  border: 2px solid rgba(14, 90, 58, 0.3);
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.15);
}

.jb-description-content :deep(pre code) {
  background: transparent;
  color: inherit;
  padding: 0;
  border: none;
}

/* ===== تصاویر ===== */
.jb-description-content :deep(img) {
  display: block;
  max-width: 100%;
  height: auto;
  margin: 22px auto;
  border-radius: 20px;
  border: 3px solid #fff;
  box-shadow: 0 12px 30px rgba(14, 90, 58, 0.15);
  transition: transform 0.35s ease, box-shadow 0.35s ease;
}

.jb-description-content :deep(img:hover) {
  transform: scale(1.02);
  box-shadow: 0 18px 40px rgba(14, 90, 58, 0.25);
}

/* ===== جدول ===== */
.jb-description-content :deep(table) {
  width: 100%;
  margin: 22px 0;
  border-collapse: separate;
  border-spacing: 0;
  border-radius: 16px;
  overflow: hidden;
  border: 2px solid rgba(14, 90, 58, 0.1);
  box-shadow: 0 8px 22px rgba(14, 90, 58, 0.08);
}

.jb-description-content :deep(thead) {
  background: linear-gradient(135deg, #0e5a3a, #1a7a50);
  color: #fff;
}

.jb-description-content :deep(th) {
  padding: 14px 16px;
  font-size: 0.85rem;
  font-weight: 900;
  text-align: right;
}

.jb-description-content :deep(td) {
  padding: 12px 16px;
  font-size: 0.85rem;
  border-top: 1.5px dashed rgba(14, 90, 58, 0.1);
  background: #fff;
  color: #4a5568;
  font-weight: 600;
}

.jb-description-content :deep(tbody tr:hover td) {
  background: #f1f7f3;
}

.jb-description-content :deep(tbody tr:nth-child(even) td) {
  background: #f8fcf9;
}

/* ===== خط افقی ===== */
.jb-description-content :deep(hr) {
  margin: 28px 0;
  border: none;
  height: 2px;
  background: linear-gradient(90deg, transparent, rgba(140, 198, 63, 0.5), transparent);
  border-radius: 999px;
}

/* ===== ویدیو/iframe ===== */
.jb-description-content :deep(iframe),
.jb-description-content :deep(video) {
  width: 100%;
  max-width: 100%;
  margin: 22px 0;
  border-radius: 20px;
  border: 3px solid #fff;
  box-shadow: 0 12px 30px rgba(14, 90, 58, 0.15);
}

/* ===== واکنش‌گرا ===== */
@media (max-width: 768px) {
  .jb-description-content { font-size: 0.88rem; line-height: 2.05; }
  .jb-description-content :deep(h1) { font-size: 1.25rem; }
  .jb-description-content :deep(h2) { font-size: 1.1rem; }
  .jb-description-content :deep(h3) { font-size: 1rem; }
  .jb-description-content :deep(p:first-of-type) { font-size: 0.92rem; }
  .jb-desc-top-line { margin-bottom: 20px; }
  .jb-desc-icon { width: 34px; height: 34px; border-radius: 11px; }
  .jb-description-content :deep(img) { border-radius: 16px; margin: 18px auto; }
}
</style>