<template>
  <div class="max-w-4xl mx-auto px-4 sm:px-6 py-5">
    <!-- Title -->
    <div class="flex flex-col sm:flex-row sm:items-center gap-2 sm:gap-3 my-6 pb-4 border-b-2 border-dashed border-[rgba(14,90,58,0.15)]">
      <h2 class="text-lg sm:text-xl font-bold text-[#0e5a3a] flex items-center gap-2">
        <span class="w-2.5 h-2.5 rounded-full bg-gradient-to-br from-[#0e5a3a] to-[#8cc63f]"></span>
        نظرات کاربران
      </h2>
      <span class="text-sm sm:text-base text-gray-500">({{ comments.length }} نظر)</span>
    </div>

    <!-- Comments -->
    <div class="space-y-6">
      <CommentItem v-for="comment in comments" :key="comment.id" :comment="comment" @like="handleLike"
        @dislike="handleDislike" @reply="handleReply" />
    </div>

    <!-- No comments -->
    <div v-if="comments.length === 0" class="text-center py-16">
      <div class="inline-flex items-center justify-center w-20 h-20 rounded-full bg-[#f1f7f3] mb-4">
        <svg class="w-10 h-10 text-[#8cc63f]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
        </svg>
      </div>
      <p class="text-base text-gray-500 m-0">
        هنوز نظری ثبت نشده است. اولین نفری باشید که نظر می‌دهد!
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import CommentItem from './CommentItem.vue'
import { useApi } from "@/composables/api/useApi";
let comments = ref(null);
const props = defineProps({
   productId: { type: [String, Number], required: true },

});
const route = useRoute();
const { getProductComments } = useApi();
try {
   const response = await getProductComments(props.productId);
   comments.value = response;
} catch (error) {
   console.error("خطا در دریافت کامنت");
}


const findComment = (commentId) => {
   for (const comment of comments.value) {
      if (comment.id === commentId) return comment

      if (comment.replies) {
         const reply = comment.replies.find(r => r.id === commentId)
         if (reply) return reply
      }
   }
   return null
}

const handleLike = (commentId) => {
   const comment = findComment(commentId)
   if (comment) {
      comment.likes++
   }
}

const handleDislike = (commentId) => {
   const comment = findComment(commentId)
   if (comment) {
      comment.dislikes++
   }
}

const handleReply = ({ commentId, text }) => {
   const comment = comments.value.find(c => c.id === commentId)
   if (comment) {
      if (!comment.replies) {
         comment.replies = []
      }
      comment.replies.push({
         id: Date.now(),
         name: 'شما',
         avatar: 'https://i.pravatar.cc/150?img=20',
         date: 'الان',
         text,
         likes: 0,
         dislikes: 0
      })
   }
}
</script>