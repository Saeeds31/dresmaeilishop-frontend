<template>
  <div class="jb-otp-container" dir="ltr">
    <input
      v-for="(digit, index) in length"
      :key="index"
      :ref="el => inputRefs[index] = el"
      v-model="digits[index]"
      type="text"
      inputmode="numeric"
      pattern="[0-9]*"
      maxlength="1"
      class="jb-otp-input"
      :class="{ 'is-filled': digits[index] }"
      @input="handleInput(index, $event)"
      @keydown="handleKeyDown(index, $event)"
      @paste="handlePaste"
      autocomplete="off"
    />
  </div>
</template>

<script setup>
import { ref, watch, onMounted, nextTick } from "vue";

const props = defineProps({
  code: { type: String, default: "" },
  length: { type: Number, default: 6 },
});

const emit = defineEmits(["update:code"]);

const digits = ref(Array(props.length).fill(""));
const inputRefs = ref([]);

watch(
  () => props.code,
  (newCode) => {
    if (newCode !== digits.value.join("")) {
      digits.value = newCode.split("").slice(0, props.length);
      while (digits.value.length < props.length) {
        digits.value.push("");
      }
    }
  },
  { immediate: true }
);

const handleInput = (index, event) => {
  const value = event.target.value;

  if (!/^\d*$/.test(value)) {
    event.target.value = "";
    return;
  }

  if (value.length > 1) {
    const numbers = value.split("").filter((char) => /\d/.test(char));
    numbers.forEach((num, i) => {
      if (index + i < props.length) {
        digits.value[index + i] = num;
      }
    });

    const nextIndex = Math.min(index + numbers.length, props.length - 1);
    focusInput(nextIndex);
  } else {
    digits.value[index] = value;

    if (value && index < props.length - 1) {
      focusInput(index + 1);
    }
  }

  emitCode();
};

const handleKeyDown = (index, event) => {
  if (event.key === "Backspace") {
    if (!digits.value[index] && index > 0) {
      digits.value[index - 1] = "";
      focusInput(index - 1);
      emitCode();
    } else if (digits.value[index]) {
      digits.value[index] = "";
      emitCode();
    }
    event.preventDefault();
  }

  if (event.key === "ArrowLeft" && index > 0) {
    focusInput(index - 1);
    event.preventDefault();
  }

  if (event.key === "ArrowRight" && index < props.length - 1) {
    focusInput(index + 1);
    event.preventDefault();
  }

  if (/^\d$/.test(event.key) && digits.value[index]) {
    digits.value[index] = "";
  }
};

const handlePaste = (event) => {
  event.preventDefault();
  const pastedData = event.clipboardData.getData("text");
  const numbers = pastedData.replace(/\D/g, "").split("").slice(0, props.length);

  numbers.forEach((num, i) => {
    digits.value[i] = num;
  });

  const nextEmptyIndex = digits.value.findIndex((d) => !d);
  const focusIndex = nextEmptyIndex === -1 ? props.length - 1 : nextEmptyIndex;
  focusInput(focusIndex);

  emitCode();
};

const focusInput = (index) => {
  nextTick(() => {
    inputRefs.value[index]?.focus();
    inputRefs.value[index]?.select();
  });
};

const emitCode = () => {
  const code = digits.value.join("");
  emit("update:code", code);
};

onMounted(() => {
  focusInput(0);
});
</script>

<style scoped>
/* =========================================================
   OTP Input - مینیمال سبز
   ========================================================= */

.jb-otp-container {
  display: flex;
  gap: 8px;
  justify-content: center;
  direction: ltr;
}

.jb-otp-input {
  position: relative;
  width: 46px;
  height: 52px;
  text-align: center;
  font-size: 22px;
  font-weight: 800;
  color: #1a202c;
  border: 1px solid rgba(14, 90, 58, 0.12);
  border-radius: 6px;
  background: #fff;
  transition: all 0.2s ease;
  outline: none;
  font-family: inherit;
  caret-color: #0e5a3a;
}

.jb-otp-input::placeholder {
  color: #a0aec0;
}

/* هاور */
.jb-otp-input:hover {
  border-color: rgba(14, 90, 58, 0.3);
}

/* فوکوس */
.jb-otp-input:focus {
  border-color: #0e5a3a;
  background: #fff;
  box-shadow: 0 0 0 3px rgba(14, 90, 58, 0.1);
}

/* پر شده */
.jb-otp-input.is-filled {
  border-color: rgba(14, 90, 58, 0.25);
  background: #f7fafc;
  color: #0e5a3a;
}

/* پر شده + فوکوس */
.jb-otp-input.is-filled:focus {
  background: #f1f7f3;
  border-color: #0e5a3a;
  box-shadow: 0 0 0 3px rgba(14, 90, 58, 0.15);
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 480px) {
  .jb-otp-container {
    gap: 6px;
  }

  .jb-otp-input {
    width: 40px;
    height: 46px;
    font-size: 18px;
    border-radius: 5px;
  }
}

@media (max-width: 360px) {
  .jb-otp-container {
    gap: 4px;
  }

  .jb-otp-input {
    width: 34px;
    height: 42px;
    font-size: 16px;
    border-radius: 5px;
  }
}
</style>