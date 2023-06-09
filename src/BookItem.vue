<script setup lang="ts">
const props = defineProps<{
  imageLink?: string;
  title?: string;
  authors?: [string];
  publishedDate: string;
  buyLink: string;
}>();
function handleBuy() {
  window.open(props.buyLink, "_blank");
}
</script>

<template>
  <n-card>
    <n-grid x-gap="12" :cols="4">
      <n-gi :span="1">
        <img :src="imageLink" alt="" />
      </n-gi>
      <n-gi :span="2">
        <div :class="$style.title">{{ title }}</div>
        <div>{{ authors?.join(", ") }}★{{ publishedDate }}</div>
      </n-gi>
      <n-gi :span="1">
        <n-button @click="handleBuy">BUY</n-button>
      </n-gi>
    </n-grid>
  </n-card>
</template>

<style module>
.title {
  font-size: 20px;
  font-weight: 700;
}
</style>
