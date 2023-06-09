<script setup>
import axios from "axios";
import { ref, reactive, h, computed } from "vue";
import { IosMenu } from "@vicons/ionicons4";
import { NIcon } from "naive-ui";
import BookItem from "./BookItem.vue";

const placeholder = ref("please input");
const input = ref("");
const mode = ref("all");
const searchKey = computed(() => {
  if (mode.value === "title") {
    return "intitle:" + input.value;
  } else if (mode.value === "author") {
    return "inauthor:" + input.value;
  } else if (mode.value === "ibsn") {
    return "ibsn" + input.value;
  } else if (mode.value === "all") {
    return input.value;
  }
});
const options = reactive([
  {
    label: "ALL",
    key: "all",
  },
  {
    label: "TITLE",
    key: "title",
  },
  {
    label: "AUTHOR",
    key: "author",
  },
  {
    label: "IBSN",
    key: "ibsn",
  },
]);
function renderIcon() {
  return h(NIcon, null, {
    default: () => h(IosMenu),
  });
}
function handleSelect(key) {
  mode.value = key;
  if (key === "title") {
    placeholder.value = "please input title";
  } else if (key === "author") {
    placeholder.value = "please input author";
  } else if (key === "ibsn") {
    placeholder.value = "please input ibsn";
  } else if ((key = "all")) {
    placeholder.value = "please input";
  }
}

const result = {
  url: "",
  items: ref([]),
  totalItems: 0,
  cntIndex: 0,
};
const activeFlag = ref(false);

let controller;
async function search() {
  console.log(searchKey.value);
  activeFlag.value = true;
  if (controller) controller.abort();
  controller = new AbortController();
  result.items.value = [];
  result.cntIndex = 0;
  result.url =
    "https://www.googleapis.com/books/v1/volumes?q=" + searchKey.value;
  try {
    const { data } = await axios.get(result.url, {
      signal: controller.signal,
      params: {
        startIndex: result.cntIndex,
      },
    });
    console.log(`共${data.totalItems}条`);
    result.totalItems = data.totalItems;
    for (let i = 0; i < data.items.length; ++i) {
      result.items.value.push(
        (await axios.get(data.items[i].selfLink, { signal: controller.signal }))
          .data
      );
    }
    result.cntIndex += data.items.length;

    console.log(result.cntIndex);
  } catch (error) {
    console.log(error);
  }
}

async function load() {
  const { data } = await axios.get(result.url, {
    signal: controller.signal,
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
      <n-dropdown trigger="click" :options="options" @select="handleSelect">
        <n-button color="#757575" :render-icon="renderIcon" />
      </n-dropdown>
      <n-input :placeholder="placeholder" v-model:value="input" />
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
      :buy-link="i.saleInfo.buyLink"
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
