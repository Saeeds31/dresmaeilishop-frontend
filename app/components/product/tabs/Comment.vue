<template>
   <div>
      <productTabsForm :productId="props.productId" />
      <productTabsCardcomment :productId="props.productId"   />
   </div>
</template>

<script setup>

const props = defineProps({
  productId: { type: [String, Number], required: true },

});

</script>