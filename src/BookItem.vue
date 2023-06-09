<script setup lang="ts">
defineProps<{
  imageLink?: string;
  title?: string;
  authors?: [string];
  publishedDate: string;
}>();
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
        <n-button>PREVIEW</n-button>
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
