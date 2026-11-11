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
   OTP Input - تم پودری
   ========================================================= */

.jb-otp-container {
  display: flex;
  gap: 10px;
  justify-content: center;
  direction: ltr;
}

.jb-otp-input {
  position: relative;
  width: 52px;
  height: 62px;
  text-align: center;
  font-size: 26px;
  font-weight: 900;
  color: var(--jb-text);
  border: 2px solid var(--jb-pink-200);
  border-radius: 16px;
  background: #fff;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  outline: none;
  font-family: inherit;
  caret-color: var(--jb-fuchsia);
}

.jb-otp-input::placeholder {
  color: var(--jb-text-muted);
}

/* هاور */
.jb-otp-input:hover {
  border-color: var(--jb-pink-400);
}

/* فوکوس */
.jb-otp-input:focus {
  border-color: var(--jb-fuchsia);
  background: #fff;
  box-shadow:
    0 0 0 4px rgba(214, 51, 132, 0.18),
    0 8px 20px rgba(214, 51, 132, 0.15);
  transform: translateY(-3px) scale(1.06);
}

/* پر شده */
.jb-otp-input.is-filled {
  border-color: var(--jb-pink-400);
  background: linear-gradient(135deg, #fff8fb, #ffeaf2);
  color: var(--jb-fuchsia);
  box-shadow: 0 6px 16px rgba(214, 51, 132, 0.15);
}

/* پر شده + فوکوس */
.jb-otp-input.is-filled:focus {
  background: linear-gradient(135deg, #ffeaf2, #ffe0eb);
  border-color: var(--jb-fuchsia);
  box-shadow:
    0 0 0 4px rgba(214, 51, 132, 0.22),
    0 10px 24px rgba(214, 51, 132, 0.22);
}

/* =========================================================
   ریسپانسیو
   ========================================================= */
@media (max-width: 480px) {
  .jb-otp-container {
    gap: 7px;
  }

  .jb-otp-input {
    width: 42px;
    height: 52px;
    font-size: 20px;
    border-radius: 13px;
  }
}

@media (max-width: 360px) {
  .jb-otp-container {
    gap: 5px;
  }

  .jb-otp-input {
    width: 36px;
    height: 46px;
    font-size: 18px;
    border-radius: 11px;
  }
}
</style>