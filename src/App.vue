<script setup>
import axios from "axios";
import { ref, reactive, h } from "vue";
import { IosMenu } from "@vicons/ionicons4";
import { NIcon } from "naive-ui";
import BookItem from "./BookItem.vue";
const options = reactive([
  {
    label: "a",
    key: "a",
  },
]);
function renderIcon() {
  return h(NIcon, null, {
    default: () => h(IosMenu),
  });
}

const result = {
  url: "",
  items: ref([]),
  totalItems: 0,
  cntIndex: 0,
};
const activeFlag = ref(false);

async function search(url) {
  activeFlag.value = true;
  result.items.value = [];
  result.cntIndex = 0;
  result.url =
    "https://www.googleapis.com/books/v1/volumes?q=flowers+inauthor:keyes";
  const { data } = await axios.get(result.url, {
    params: {
      startIndex: result.cntIndex,
    },
  });
  console.log(`共${data.totalItems}条`);
  result.totalItems = data.totalItems;
  for (let i = 0; i < data.items.length; ++i) {
    result.items.value.push((await axios.get(data.items[i].selfLink)).data);
  }
  result.cntIndex += data.items.length;
}

async function load() {
  const { data } = await axios.get(result.url, {
    params: {
      startIndex: result.cntIndex,
    },
  });
  result.totalItems = data.totalItems;
  for (let i = 0; i < data.items.length; ++i) {
    result.items.value.push((await axios.get(data.items[i].selfLink)).data);
  }
  result.cntIndex += data.items.length;
  console.log(result.cntIndex);
}

const style = reactive({});
window.addEventListener("scroll", () => {
  if (window.scrollY >= 100 && window.scrollY <= 150) {
    style.top = window.scrollY - 150 + "px";
    style.position = "fixed";
  } else if (window.scrollY < 100) {
    style.top = "";
    style.position = "";
  } else {
    style.top = "0";
    style.position = "fixed";
  }
  if (
    window.scrollY + window.innerHeight >=
    document.documentElement.offsetHeight
  ) {
    load();
  }
});
</script>

<template>
  <header :class="{ active: activeFlag }" :style="style">
    <n-input-group style="width: 500px">
      <n-dropdown trigger="click" :options="options">
        <n-button color="#757575" :render-icon="renderIcon" />
      </n-dropdown>
      <n-input />
      <n-button color="#757575" @click="search()"> 搜索 </n-button>
    </n-input-group>
  </header>
  <main v-if="activeFlag">
    <BookItem
      v-for="i in result.items.value"
      :image-link="i.volumeInfo?.imageLinks?.thumbnail"
      :title="i.volumeInfo.title"
      :authors="i.volumeInfo.authors"
      :published-date="i.volumeInfo.publishedDate"
    />
  </main>
</template>

<style module></style>

<style>
body {
  background-image: url(/oldbooks.webp);
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
  background-attachment: fixed;
}
header {
  position: absolute;
  top: 300px;
  width: 100%;
  height: 50px;
  align-items: center;
  display: flex;
  justify-content: center;
  z-index: 1;
}
main {
  box-sizing: border-box;
  margin-top: 100px;
  width: 100%;
  padding: 0 20px;
}
.active {
  top: 25px;
}
</style>
