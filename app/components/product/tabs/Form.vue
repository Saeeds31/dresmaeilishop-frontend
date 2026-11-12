
<script setup>
import { ref, computed } from "vue";
import { useAuth } from "~/composables/useAuth";
import { toast } from "vue3-toastify";
import { useApi } from "@/composables/api/useApi";
const route = useRoute();
const { loginModal } = useAuth();
const { postProductComment } = useApi();

const emit = defineEmits(["submit"]);
const props = defineProps({
  productId: { type: [String, Number], required: true },

});
const formData = ref({
  name: "",
  rating: 0,
  text: "",
});
let loading = ref(false);
const isFormValid = computed(() => {
  return (
    formData.value.rating > 0 &&
    formData.value.text.trim()
  );
});
const submitComment = async () => {
  const token = process.client ? localStorage.getItem("auth-token") : null;

  if (!token) {
    toast.warning("لطفا ابتدا وارد حساب کاربری خود شوید", {
      autoClose: 3000,
    });
    localStorage.setItem('lastPage', route.fullPath)

    // ریدایرکت به صفحه لاگین
    setTimeout(() => {
      loginModal.value = true;
    }, 1000);
    return;
  }

  if (isFormValid.value) {
    loading.value = true;
    const fd = new FormData();
    fd.append('rating', formData.value.rating);
    fd.append('content', formData.value.text);
    fd.append('parent_id', '');
    try {
      await postProductComment(props.productId, fd);
      toast.success("کامنت شما با موفقیت ثبت شد و بعد از تایید نمایش داده خواهد شد", {
        autoClose: 3000,
      });
      resetForm();

    } catch (error) {
      console.error("Error registering stock alert:", error);
      toast.error(error.response?.data?.message || "خطا در ثبت درخواست");
    } finally {
      loading.value = false;
    }
  }
};

const resetForm = () => {
  formData.value = {
    name: "",
    rating: 0,
    text: "",
  };
};
</script>
<template>
  <div class="p-5 mt-6 rounded-2xl border-2 border-[rgba(14,90,58,0.1)] bg-white shadow-sm">
    <h3 class="text-lg font-bold mb-5 text-[#0e5a3a] flex items-center gap-2">
      <span class="w-2 h-2 rounded-full bg-gradient-to-br from-[#0e5a3a] to-[#8cc63f]"></span>
      نظر خود را بنویسید
    </h3>

    <form @submit.prevent="submitComment" class="space-y-5">
      <!-- امتیاز -->
      <div class="w-full md:w-[50%]">
        <label class="block text-sm font-bold text-gray-700 mb-2">امتیاز شما</label>
        <div class="flex gap-2 justify-start">
          <button v-for="star in 5" :key="star" type="button" @click="formData.rating = star"
            class="transition-transform hover:scale-110">
            <svg class="w-9 h-9 transition-colors"
              :class="star <= formData.rating ? 'text-[#8cc63f]' : 'text-gray-200'"
              xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
              <path class="fill-current"
                d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z" />
            </svg>
          </button>
        </div>
      </div>

      <!-- متن نظر -->
      <div>
        <label for="comment" class="block text-sm font-bold text-gray-700 mb-2">
          نظر شما
        </label>
        <textarea v-model="formData.text" id="comment" rows="5" placeholder="نظر خود را در مورد محصول بنویسید..."
          required
          class="w-full px-4 py-3 border-2 border-[rgba(14,90,58,0.1)] rounded-xl text-sm resize-y focus:outline-none focus:border-[#8cc63f] focus:shadow-[0_0_0_4px_rgba(140,198,63,0.15)] transition-all"></textarea>
      </div>

      <!-- دکمه‌ها -->
      <div class="flex flex-col sm:flex-row gap-3 pt-2">
        <button type="submit" :disabled="!isFormValid || loading"
          class="inline-flex items-center justify-center gap-2 px-8 py-3 bg-gradient-to-l from-[#0e5a3a] to-[#1a7a50] text-white text-sm font-bold rounded-xl shadow-lg shadow-[#0e5a3a]/20 hover:shadow-xl hover:shadow-[#0e5a3a]/30 hover:-translate-y-0.5 transition-all disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:translate-y-0">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
          </svg>
          ثبت نظر
        </button>

        <button type="button" @click="resetForm"
          class="px-6 py-3 bg-[#f1f7f3] text-[#0e5a3a] text-sm font-bold rounded-xl hover:bg-[#e8f3ec] transition-colors">
          پاک کردن
        </button>
      </div>
    </form>
  </div>
</template>